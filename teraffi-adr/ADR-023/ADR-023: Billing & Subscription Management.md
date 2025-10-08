# ADR-023: Billing & Subscription Management

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-019 (Multi-Tenancy), ADR-020 (Authentication)

---

## Context

TERAFFI operates as a SaaS platform with tiered subscriptions. The system must handle subscription management, usage-based billing, payment processing, and revenue recognition.

**Subscription Tiers:**

**Free Tier:**
- 5 partnerships per month
- Basic matching
- Limited analytics
- 3 team members
- Email support

**Professional Tier ($299/month):**
- 50 partnerships per month
- Advanced analytics
- API access
- 10 team members
- Priority support
- Custom branding

**Enterprise Tier (Custom pricing):**
- Unlimited partnerships
- White-label
- SSO
- Dedicated success manager
- Custom integrations
- SLA guarantees

**Usage-Based Add-ons:**
- Additional partnerships: $10/partnership
- Extra team members: $25/member/month
- Premium support: $500/month

### Current Gap

Without billing infrastructure:
- **No Revenue**: Can't charge customers
- **Manual Processes**: Spreadsheets and invoices
- **No Usage Tracking**: Can't enforce quotas or bill overages
- **Poor UX**: No self-service subscription management
- **Compliance Issues**: No proper revenue recognition

### Requirements

**Functional:**
1. Subscription plan management
2. Credit card processing (PCI compliant)
3. Usage tracking and metering
4. Invoice generation
5. Payment retry logic
6. Proration on plan changes
7. Trial period management
8. Dunning (failed payment recovery)
9. Tax calculation (US sales tax, VAT)
10. Revenue recognition

**Non-Functional:**
1. PCI DSS compliance (Level 1)
2. 99.99% payment processing uptime
3. <2 second checkout experience
4. Support 10k+ active subscriptions
5. Handle $10M+ ARR
6. SOC 2 compliant financial controls

---

## Decision

**Implement billing system using Stripe** with:
1. **Stripe Billing** for subscription management
2. **Stripe Checkout** for payment collection
3. **Stripe Webhooks** for event processing
4. **Usage metering** for consumption tracking
5. **Stripe Tax** for automatic tax calculation
6. **Customer Portal** for self-service management
7. **Revenue recognition** tracking

**Architecture:**
```
User Action (Subscribe/Upgrade/Cancel)
    ↓
[Billing Service]
    ↓
Stripe API
    ↓
Stripe Event (webhook)
    ↓
[Webhook Handler]
    ↓
Update Database (subscriptions, usage, invoices)
    ↓
[Notification Service] → Email receipt/notifications
```

---

## Rationale

### Why Stripe

**Stripe Benefits:**
- **PCI Compliance**: Stripe handles card data, we never touch it
- **Global Support**: 135+ currencies, local payment methods
- **Developer Experience**: Excellent API, webhooks, SDKs
- **Subscription Management**: Built-in plans, trials, prorations
- **Tax Handling**: Stripe Tax handles complex tax rules
- **Dunning**: Automatic retry logic for failed payments
- **Customer Portal**: Pre-built self-service UI

**Alternatives Considered:**

**Chargebee:**
- **Pros**: More flexible billing models
- **Cons**: More expensive, smaller ecosystem
- **Decision**: Stripe's simplicity wins

**Build Custom:**
- **Pros**: Full control
- **Cons**: PCI compliance nightmare, months of development
- **Decision**: Not worth the risk and effort

### Why Webhooks

**Event-Driven Architecture:**
- Stripe sends webhooks for all subscription events
- Ensures data consistency between Stripe and our DB
- Enables async processing (invoices, notifications)
- Provides audit trail

---

## Implementation Details

### 1. Database Schema

```sql
-- Subscriptions table
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id) UNIQUE,
  stripe_customer_id VARCHAR(255) UNIQUE NOT NULL,
  stripe_subscription_id VARCHAR(255) UNIQUE,
  
  -- Plan details
  plan_id VARCHAR(50) NOT NULL,  -- 'free' | 'professional' | 'enterprise'
  status VARCHAR(50) NOT NULL,  -- 'active' | 'past_due' | 'canceled' | 'trialing'
  
  -- Pricing
  amount_cents INTEGER NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  billing_interval VARCHAR(20),  -- 'month' | 'year'
  
  -- Dates
  trial_ends_at TIMESTAMPTZ,
  current_period_start TIMESTAMPTZ NOT NULL,
  current_period_end TIMESTAMPTZ NOT NULL,
  canceled_at TIMESTAMPTZ,
  ended_at TIMESTAMPTZ,
  
  -- Payment method
  payment_method_type VARCHAR(50),
  card_last4 VARCHAR(4),
  card_brand VARCHAR(20),
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_subscriptions_tenant ON subscriptions(tenant_id);
CREATE INDEX idx_subscriptions_stripe_customer ON subscriptions(stripe_customer_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status);

-- Usage tracking (for metered billing)
CREATE TABLE usage_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subscription_id UUID REFERENCES subscriptions(id),
  tenant_id UUID REFERENCES tenants(id),
  
  -- Usage details
  metric VARCHAR(100) NOT NULL,  -- 'partnerships' | 'api_requests' | 'team_members'
  quantity INTEGER NOT NULL,
  
  -- Stripe metering
  stripe_usage_record_id VARCHAR(255),
  reported_to_stripe BOOLEAN DEFAULT false,
  
  -- Time period
  period_start TIMESTAMPTZ NOT NULL,
  period_end TIMESTAMPTZ NOT NULL,
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_usage_subscription ON usage_records(subscription_id, period_start);
CREATE INDEX idx_usage_tenant ON usage_records(tenant_id, metric);
CREATE INDEX idx_usage_unreported ON usage_records(reported_to_stripe) WHERE reported_to_stripe = false;

-- Invoices
CREATE TABLE invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subscription_id UUID REFERENCES subscriptions(id),
  tenant_id UUID REFERENCES tenants(id),
  
  stripe_invoice_id VARCHAR(255) UNIQUE NOT NULL,
  stripe_invoice_number VARCHAR(100),
  
  -- Amounts
  subtotal_cents INTEGER NOT NULL,
  tax_cents INTEGER NOT NULL,
  total_cents INTEGER NOT NULL,
  amount_paid_cents INTEGER DEFAULT 0,
  amount_due_cents INTEGER NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  
  -- Status
  status VARCHAR(50) NOT NULL,  -- 'draft' | 'open' | 'paid' | 'void' | 'uncollectible'
  
  -- Payment
  paid BOOLEAN DEFAULT false,
  paid_at TIMESTAMPTZ,
  payment_intent_id VARCHAR(255),
  
  -- Dates
  period_start TIMESTAMPTZ NOT NULL,
  period_end TIMESTAMPTZ NOT NULL,
  due_date DATE,
  
  -- PDF
  invoice_pdf_url VARCHAR(1000),
  hosted_invoice_url VARCHAR(1000),
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_invoices_subscription ON invoices(subscription_id);
CREATE INDEX idx_invoices_tenant ON invoices(tenant_id);
CREATE INDEX idx_invoices_status ON invoices(status);
CREATE INDEX idx_invoices_stripe ON invoices(stripe_invoice_id);

-- Payment attempts (for dunning)
CREATE TABLE payment_attempts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  invoice_id UUID REFERENCES invoices(id),
  subscription_id UUID REFERENCES subscriptions(id),
  tenant_id UUID REFERENCES tenants(id),
  
  attempt_number INTEGER NOT NULL,
  status VARCHAR(50) NOT NULL,  -- 'pending' | 'succeeded' | 'failed'
  
  -- Stripe details
  payment_intent_id VARCHAR(255),
  charge_id VARCHAR(255),
  
  -- Failure details
  failure_code VARCHAR(100),
  failure_message TEXT,
  
  -- Next retry
  next_retry_at TIMESTAMPTZ,
  
  attempted_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_payment_attempts_invoice ON payment_attempts(invoice_id);
CREATE INDEX idx_payment_attempts_next_retry ON payment_attempts(next_retry_at) WHERE status = 'failed';

-- Billing events audit log
CREATE TABLE billing_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  subscription_id UUID,
  
  event_type VARCHAR(100) NOT NULL,
  stripe_event_id VARCHAR(255) UNIQUE,
  
  -- Event data
  payload JSONB NOT NULL,
  
  -- Processing
  processed BOOLEAN DEFAULT false,
  processed_at TIMESTAMPTZ,
  error_message TEXT,
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_billing_events_tenant ON billing_events(tenant_id);
CREATE INDEX idx_billing_events_type ON billing_events(event_type);
CREATE INDEX idx_billing_events_stripe ON billing_events(stripe_event_id);
CREATE INDEX idx_billing_events_unprocessed ON billing_events(processed) WHERE NOT processed;
```

### 2. Billing Service

```typescript
// packages/billing/src/billing-service.ts

import Stripe from 'stripe';

interface SubscriptionRequest {
  tenant_id: string;
  plan_id: 'free' | 'professional' | 'enterprise';
  payment_method_id?: string;
  trial_days?: number;
  billing_interval?: 'month' | 'year';
}

export class BillingService {
  private stripe: Stripe;

  constructor(
    private db: Database,
    private notificationService: NotificationService
  ) {
    this.stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
      apiVersion: '2023-10-16'
    });
  }

  async createSubscription(request: SubscriptionRequest): Promise<Subscription> {
    const tenant = await this.getTenant(request.tenant_id);
    const plan = this.getPlanConfig(request.plan_id);

    // 1. Create or retrieve Stripe customer
    let customerId = await this.getStripeCustomerId(tenant.id);
    
    if (!customerId) {
      const customer = await this.stripe.customers.create({
        email: tenant.billing_email || tenant.admin_email,
        name: tenant.name,
        metadata: {
          tenant_id: tenant.id,
          environment: process.env.NODE_ENV
        }
      });
      
      customerId = customer.id;
      
      // Store in database
      await this.db
        .insertInto('subscriptions')
        .values({
          tenant_id: tenant.id,
          stripe_customer_id: customerId,
          plan_id: 'free',
          status: 'active',
          amount_cents: 0,
          currency: 'USD',
          current_period_start: new Date(),
          current_period_end: addMonths(new Date(), 1)
        })
        .execute();
    }

    // 2. Attach payment method if provided
    if (request.payment_method_id) {
      await this.stripe.paymentMethods.attach(request.payment_method_id, {
        customer: customerId
      });

      await this.stripe.customers.update(customerId, {
        invoice_settings: {
          default_payment_method: request.payment_method_id
        }
      });
    }

    // 3. Create Stripe subscription
    const stripeSubscription = await this.stripe.subscriptions.create({
      customer: customerId,
      items: [{
        price: plan.stripe_price_id
      }],
      trial_period_days: request.trial_days,
      payment_behavior: 'default_incomplete',
      expand: ['latest_invoice.payment_intent'],
      metadata: {
        tenant_id: tenant.id,
        plan_id: request.plan_id
      }
    });

    // 4. Update database
    await this.db
      .updateTable('subscriptions')
      .set({
        stripe_subscription_id: stripeSubscription.id,
        plan_id: request.plan_id,
        status: stripeSubscription.status,
        amount_cents: plan.price_cents,
        billing_interval: request.billing_interval || 'month',
        current_period_start: new Date(stripeSubscription.current_period_start * 1000),
        current_period_end: new Date(stripeSubscription.current_period_end * 1000),
        trial_ends_at: stripeSubscription.trial_end 
          ? new Date(stripeSubscription.trial_end * 1000) 
          : null,
        updated_at: new Date()
      })
      .where('stripe_customer_id', '=', customerId)
      .execute();

    // 5. Send notification
    await this.notificationService.send({
      user_id: tenant.admin_user_id,
      type: 'subscription_created',
      priority: 'high',
      title: 'Subscription Active',
      message: `Your ${request.plan_id} plan is now active!`
    });

    return await this.getSubscription(tenant.id);
  }

  async upgradeSubscription(
    tenantId: string,
    newPlanId: string
  ): Promise<Subscription> {
    const subscription = await this.getSubscription(tenantId);
    const newPlan = this.getPlanConfig(newPlanId);

    // Upgrade in Stripe (with proration)
    await this.stripe.subscriptions.update(subscription.stripe_subscription_id, {
      items: [{
        id: subscription.stripe_subscription_item_id,
        price: newPlan.stripe_price_id
      }],
      proration_behavior: 'create_prorations',
      metadata: {
        plan_id: newPlanId
      }
    });

    // Database update handled by webhook
    return await this.getSubscription(tenantId);
  }

  async cancelSubscription(
    tenantId: string,
    immediate: boolean = false
  ): Promise<void> {
    const subscription = await this.getSubscription(tenantId);

    if (immediate) {
      // Cancel immediately
      await this.stripe.subscriptions.cancel(subscription.stripe_subscription_id);
    } else {
      // Cancel at period end
      await this.stripe.subscriptions.update(subscription.stripe_subscription_id, {
        cancel_at_period_end: true
      });
    }

    await this.notificationService.send({
      user_id: await this.getTenantAdminId(tenantId),
      type: 'subscription_canceled',
      priority: 'high',
      title: 'Subscription Canceled',
      message: immediate 
        ? 'Your subscription has been canceled immediately.'
        : 'Your subscription will end at the current period end.'
    });
  }

  async trackUsage(
    tenantId: string,
    metric: string,
    quantity: number
  ): Promise<void> {
    const periodStart = startOfMonth(new Date());
    const periodEnd = endOfMonth(new Date());

    // Record in database
    await this.db
      .insertInto('usage_records')
      .values({
        subscription_id: await this.getSubscriptionId(tenantId),
        tenant_id: tenantId,
        metric,
        quantity,
        period_start: periodStart,
        period_end: periodEnd,
        reported_to_stripe: false
      })
      .execute();

    // Report to Stripe if metered billing enabled
    if (this.isMeteredMetric(metric)) {
      await this.reportUsageToStripe(tenantId, metric, quantity);
    }

    // Check quota limits
    await this.checkQuotaLimits(tenantId);
  }

  private async reportUsageToStripe(
    tenantId: string,
    metric: string,
    quantity: number
  ): Promise<void> {
    const subscription = await this.getSubscription(tenantId);
    const subscriptionItem = await this.getMeteredSubscriptionItem(
      subscription.stripe_subscription_id,
      metric
    );

    if (subscriptionItem) {
      const usageRecord = await this.stripe.subscriptionItems.createUsageRecord(
        subscriptionItem.id,
        {
          quantity,
          timestamp: Math.floor(Date.now() / 1000),
          action: 'increment'
        }
      );

      // Update database
      await this.db
        .updateTable('usage_records')
        .set({
          stripe_usage_record_id: usageRecord.id,
          reported_to_stripe: true
        })
        .where('tenant_id', '=', tenantId)
        .where('metric', '=', metric)
        .where('reported_to_stripe', '=', false)
        .execute();
    }
  }

  private getPlanConfig(planId: string): {
    stripe_price_id: string;
    price_cents: number;
    features: string[];
  } {
    const plans = {
      free: {
        stripe_price_id: process.env.STRIPE_PRICE_FREE!,
        price_cents: 0,
        features: ['basic_matching', 'limited_analytics']
      },
      professional: {
        stripe_price_id: process.env.STRIPE_PRICE_PROFESSIONAL!,
        price_cents: 29900,
        features: ['advanced_analytics', 'api_access', 'priority_support']
      },
      enterprise: {
        stripe_price_id: process.env.STRIPE_PRICE_ENTERPRISE!,
        price_cents: 0,  // Custom pricing
        features: ['white_label', 'sso', 'unlimited', 'dedicated_support']
      }
    };

    return plans[planId] || plans.free;
  }
}
```

### 3. Webhook Handler

```typescript
// packages/billing/src/webhook-handler.ts

export class StripeWebhookHandler {
  constructor(
    private stripe: Stripe,
    private db: Database,
    private notificationService: NotificationService
  ) {}

  async handleWebhook(
    payload: string,
    signature: string
  ): Promise<void> {
    // Verify webhook signature
    const event = this.stripe.webhooks.constructEvent(
      payload,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );

    // Log event
    await this.logEvent(event);

    // Route to appropriate handler
    switch (event.type) {
      case 'customer.subscription.created':
      case 'customer.subscription.updated':
        await this.handleSubscriptionUpdate(event);
        break;

      case 'customer.subscription.deleted':
        await this.handleSubscriptionDeleted(event);
        break;

      case 'invoice.paid':
        await this.handleInvoicePaid(event);
        break;

      case 'invoice.payment_failed':
        await this.handleInvoicePaymentFailed(event);
        break;

      case 'payment_intent.succeeded':
        await this.handlePaymentSucceeded(event);
        break;

      case 'payment_intent.payment_failed':
        await this.handlePaymentFailed(event);
        break;

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    // Mark as processed
    await this.markEventProcessed(event.id);
  }

  private async handleSubscriptionUpdate(event: Stripe.Event): Promise<void> {
    const subscription = event.data.object as Stripe.Subscription;
    const tenantId = subscription.metadata.tenant_id;

    await this.db
      .updateTable('subscriptions')
      .set({
        status: subscription.status,
        current_period_start: new Date(subscription.current_period_start * 1000),
        current_period_end: new Date(subscription.current_period_end * 1000),
        canceled_at: subscription.canceled_at 
          ? new Date(subscription.canceled_at * 1000) 
          : null,
        updated_at: new Date()
      })
      .where('stripe_subscription_id', '=', subscription.id)
      .execute();

    // Update tenant features based on subscription status
    if (subscription.status === 'active') {
      await this.enableTenantFeatures(tenantId, subscription.metadata.plan_id);
    } else if (subscription.status === 'past_due' || subscription.status === 'canceled') {
      await this.restrictTenantFeatures(tenantId);
    }
  }

  private async handleInvoicePaid(event: Stripe.Event): Promise<void> {
    const invoice = event.data.object as Stripe.Invoice;

    // Update invoice in database
    await this.db
      .updateTable('invoices')
      .set({
        status: 'paid',
        paid: true,
        paid_at: new Date(),
        amount_paid_cents: invoice.amount_paid,
        updated_at: new Date()
      })
      .where('stripe_invoice_id', '=', invoice.id)
      .execute();

    // Send receipt
    const tenantId = await this.getTenantIdFromInvoice(invoice.id);
    await this.notificationService.send({
      user_id: await this.getTenantAdminId(tenantId),
      type: 'invoice_paid',
      priority: 'medium',
      title: 'Payment Received',
      message: `Payment of $${(invoice.amount_paid / 100).toFixed(2)} received. Thank you!`,
      action_url: invoice.hosted_invoice_url
    });
  }

  private async handleInvoicePaymentFailed(event: Stripe.Event): Promise<void> {
    const invoice = event.data.object as Stripe.Invoice;
    const tenantId = await this.getTenantIdFromInvoice(invoice.id);

    // Log payment attempt
    await this.db
      .insertInto('payment_attempts')
      .values({
        invoice_id: await this.getInvoiceId(invoice.id),
        subscription_id: await this.getSubscriptionId(tenantId),
        tenant_id: tenantId,
        attempt_number: invoice.attempt_count,
        status: 'failed',
        payment_intent_id: invoice.payment_intent as string,
        failure_code: invoice.charge?.failure_code,
        failure_message: invoice.charge?.failure_message,
        next_retry_at: invoice.next_payment_attempt 
          ? new Date(invoice.next_payment_attempt * 1000) 
          : null
      })
      .execute();

    // Notify user
    await this.notificationService.send({
      user_id: await this.getTenantAdminId(tenantId),
      type: 'payment_failed',
      priority: 'critical',
      title: 'Payment Failed',
      message: `Your payment of $${(invoice.amount_due / 100).toFixed(2)} failed. Please update your payment method.`,
      action_url: `/billing/payment-method`
    });

    // If multiple failures, suspend account
    if (invoice.attempt_count >= 3) {
      await this.suspendTenant(tenantId);
    }
  }

  private async logEvent(event: Stripe.Event): Promise<void> {
    await this.db
      .insertInto('billing_events')
      .values({
        tenant_id: this.extractTenantId(event),
        event_type: event.type,
        stripe_event_id: event.id,
        payload: JSON.stringify(event.data.object),
        processed: false
      })
      .execute();
  }
}
```

### 4. Customer Portal Integration

```typescript
// packages/api/src/routes/billing.ts

export const billingRoutes: FastifyPluginAsync = async (fastify) => {
  // Create checkout session
  fastify.post('/checkout', {
    preHandler: [requireAuth]
  }, async (request, reply) => {
    const { plan_id, billing_interval } = request.body;
    const tenantId = request.tenantContext.tenant_id;

    const session = await fastify.stripe.checkout.sessions.create({
      customer: await fastify.billing.getStripeCustomerId(tenantId),
      payment_method_types: ['card'],
      line_items: [{
        price: fastify.billing.getPlanConfig(plan_id).stripe_price_id,
        quantity: 1
      }],
      mode: 'subscription',
      success_url: `${process.env.APP_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${process.env.APP_URL}/billing`,
      metadata: {
        tenant_id: tenantId,
        plan_id
      }
    });

    return { checkout_url: session.url };
  });

  // Get billing portal link
  fastify.post('/portal', {
    preHandler: [requireAuth]
  }, async (request, reply) => {
    const tenantId = request.tenantContext.tenant_id;

    const session = await fastify.stripe.billingPortal.sessions.create({
      customer: await fastify.billing.getStripeCustomerId(tenantId),
      return_url: `${process.env.APP_URL}/billing`
    });

    return { portal_url: session.url };
  });

  // Get current subscription
  fastify.get('/subscription', {
    preHandler: [requireAuth]
  }, async (request, reply) => {
    const tenantId = request.tenantContext.tenant_id;
    return await fastify.billing.getSubscription(tenantId);
  });

  // Get usage for current period
  fastify.get('/usage', {
    preHandler: [requireAuth]
  }, async (request, reply) => {
    const tenantId = request.tenantContext.tenant_id;
    return await fastify.billing.getCurrentUsage(tenantId);
  });

  // Get invoices
  fastify.get('/invoices', {
    preHandler: [requireAuth]
  }, async (request, reply) => {
    const tenantId = request.tenantContext.tenant_id;
    return await fastify.db
      .selectFrom('invoices')
      .selectAll()
      .where('tenant_id', '=', tenantId)
      .orderBy('created_at', 'desc')
      .limit(12)
      .execute();
  });
};
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- Database schema
- Stripe integration
- Basic subscription management
- Webhook handling

### Phase 2: Self-Service (Months 3-4)
- Checkout flow
- Customer portal
- Usage tracking
- Invoice management

### Phase 3: Intelligence (Months 5-6)
- Dunning automation
- Usage-based billing
- Analytics dashboard
- Revenue recognition

### Phase 4: Enterprise (Months 7-8)
- Custom pricing
- Contracts and quotes
- Multi-currency support
- Advanced tax handling

---

## Success Metrics

**Financial:**
- Monthly Recurring Revenue (MRR) growth
- Churn rate <5%
- Failed payment recovery >60%
- Average Revenue Per User (ARPU)

**Operational:**
- 99.99% payment processing uptime
- <2% failed payment rate
- <1% billing disputes
- <5% refund rate

**User Experience:**
- <2 minutes to checkout
- >90% successful first-time payments
- <1% billing support tickets

---

## Task Dependencies

001 (Schema) → 002 (Stripe SDK) → 003 (Plans) → 004 (Core Billing) → 005 (Usage Tracking)
                                                → 006 (Webhooks) → 007 (Invoices)
                                                                 → 008 (Payment Methods)
                                                                 → 011 (Dunning)

004 → 009 (Checkout)
006 → 010 (Portal)

004 → 014 (Plan Changes)
004 → 015 (Trials)
004 → 016 (Tax)

007 → 017 (Revenue Recognition)
001 → 018 (Analytics)

006 → 019 (Monitoring)

All → 012 (REST API) → 013 (UI Dashboard)

All → 020 (Test Suite)
All → 021 (Documentation)

## References

- [Stripe API Documentation](https://stripe.com/docs/api)
- [PCI DSS Compliance](https://www.pcisecuritystandards.org/)
- [SaaS Metrics Guide](https://www.forentrepreneurs.com/saas-metrics-2/)
- [Revenue Recognition (ASC 606)](https://www.fasb.org/revenue)

---
