# ADR-022: Notification & Communication System

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001 (Runtime), ADR-017 (Deal Flow), ADR-019 (Multi-Tenancy)

---

## Context

TERAFFI users need timely, relevant notifications across multiple channels to stay informed about partnership opportunities, deal progress, trend changes, and platform activities.

**Notification Types:**

**Partnership & Deal Flow:**
- "New high-affinity match found: [Brand X]"
- "Partnership proposal received from [Creator Y]"
- "Deal moved to 'negotiating' stage"
- "Contract ready for signature"
- "Milestone completed: [Milestone Name]"

**Cultural Trends:**
- "Emerging trend alert: [Trend Name] gaining momentum"
- "Trend you're aligned with is accelerating"
- "Declining trend warning: [Trend Name]"

**Social & Collaboration:**
- "@mentioned you in a comment"
- "New comment on your partnership"
- "Document uploaded to [Deal Name]"

**Account & System:**
- "Welcome to TERAFFI"
- "Password changed successfully"
- "API rate limit approaching"
- "Subscription expiring soon"

### Current Gap

Without a unified notification system:
- **Inconsistent Experience**: Each service sends notifications differently
- **No User Control**: Can't customize notification preferences
- **Notification Fatigue**: No batching or intelligent throttling
- **Poor Delivery**: No retry logic, missing notifications
- **No Multi-Channel**: Email only, no in-app, push, or Slack

### Requirements

**Functional:**
1. Multi-channel delivery (email, in-app, push, Slack, webhooks)
2. User preferences per notification type
3. Notification templates with variables
4. Batching and digest mode
5. Priority levels (critical, high, medium, low)
6. Read/unread tracking
7. Notification history
8. Delivery status tracking

**Non-Functional:**
1. Delivery latency <5 seconds (critical notifications)
2. 99.9% delivery success rate
3. Handle 100k+ notifications daily
4. Support 10k+ concurrent users
5. Idempotent delivery (no duplicates)

---

## Decision

**Implement multi-channel notification system** with:
1. **Notification Service** as central coordinator
2. **Queue-based processing** for reliability
3. **Channel adapters** (email, in-app, push, Slack, webhooks)
4. **Template engine** for consistent messaging
5. **User preferences** for granular control
6. **Batching and throttling** to prevent fatigue
7. **Delivery tracking** and retry logic

**Architecture:**
```
Event Source (Deal Flow, Trends, etc.)
    ↓
[Notification Service] → Create notification
    ↓
Queue (BullMQ)
    ↓
[Notification Processor]
    ↓
Check user preferences
    ↓
Apply batching/throttling
    ↓
Parallel Channel Delivery:
├─ Email (SendGrid/AWS SES)
├─ In-App (WebSocket + Database)
├─ Push (Firebase/APNs)
├─ Slack (Slack API)
└─ Webhooks (HTTP POST)
    ↓
Track delivery status
```

---

## Rationale

### Why Queue-Based

**Benefits:**
- **Reliability**: Messages persisted until delivered
- **Retry logic**: Automatic retries on failure
- **Load leveling**: Prevents overwhelming downstream services
- **Scalability**: Process notifications asynchronously

**Alternative (Direct Send):**
- Faster but less reliable
- No retry on failure
- Can't throttle/batch
- **Decision**: Use queues for reliability

### Why Multi-Channel

**User Needs Vary:**
- Executives: Email digests
- Active users: In-app notifications
- Mobile users: Push notifications
- Teams: Slack integration

**Fallback Strategy:**
- Critical: All channels
- High: Email + in-app
- Medium: In-app only
- Low: Digest only

### Template Engine

**Benefits:**
- Consistent branding
- Easy to update messaging
- Localization support
- Variable substitution

---

## Implementation Details

### 1. Database Schema

```sql
-- Notification definitions
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  user_id UUID REFERENCES users(id),
  type VARCHAR(100) NOT NULL,  -- 'deal_stage_change', 'trend_alert', etc.
  priority VARCHAR(20) NOT NULL,  -- 'critical' | 'high' | 'medium' | 'low'
  
  -- Content
  title VARCHAR(500) NOT NULL,
  message TEXT NOT NULL,
  action_url VARCHAR(1000),
  action_label VARCHAR(100),
  
  -- Metadata
  metadata JSONB,
  related_entity_id UUID,
  related_entity_type VARCHAR(50),
  
  -- Status
  read BOOLEAN DEFAULT false,
  read_at TIMESTAMPTZ,
  archived BOOLEAN DEFAULT false,
  
  -- Batching
  batch_key VARCHAR(255),  -- For grouping related notifications
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ
);

CREATE INDEX idx_notifications_user ON notifications(user_id, created_at DESC) WHERE NOT archived;
CREATE INDEX idx_notifications_unread ON notifications(user_id, read) WHERE NOT read AND NOT archived;
CREATE INDEX idx_notifications_batch ON notifications(batch_key, created_at) WHERE batch_key IS NOT NULL;
CREATE INDEX idx_notifications_tenant ON notifications(tenant_id);

-- Notification delivery tracking
CREATE TABLE notification_deliveries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  notification_id UUID REFERENCES notifications(id) ON DELETE CASCADE,
  channel VARCHAR(50) NOT NULL,  -- 'email' | 'in_app' | 'push' | 'slack' | 'webhook'
  status VARCHAR(50) NOT NULL,  -- 'pending' | 'sent' | 'delivered' | 'failed' | 'bounced'
  
  -- Delivery details
  recipient VARCHAR(500),  -- email address, device token, webhook URL, etc.
  provider_message_id VARCHAR(255),
  error_message TEXT,
  
  -- Timestamps
  sent_at TIMESTAMPTZ,
  delivered_at TIMESTAMPTZ,
  failed_at TIMESTAMPTZ,
  retry_count INTEGER DEFAULT 0,
  next_retry_at TIMESTAMPTZ,
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_deliveries_notification ON notification_deliveries(notification_id);
CREATE INDEX idx_deliveries_status ON notification_deliveries(status, next_retry_at);
CREATE INDEX idx_deliveries_channel ON notification_deliveries(channel, status);

-- User notification preferences
CREATE TABLE notification_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  tenant_id UUID REFERENCES tenants(id),
  
  -- Per-type preferences
  notification_type VARCHAR(100) NOT NULL,
  
  -- Channel preferences
  email_enabled BOOLEAN DEFAULT true,
  in_app_enabled BOOLEAN DEFAULT true,
  push_enabled BOOLEAN DEFAULT false,
  slack_enabled BOOLEAN DEFAULT false,
  webhook_enabled BOOLEAN DEFAULT false,
  
  -- Frequency
  frequency VARCHAR(50) DEFAULT 'realtime',  -- 'realtime' | 'hourly' | 'daily' | 'weekly' | 'never'
  quiet_hours_start TIME,
  quiet_hours_end TIME,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(user_id, notification_type)
);

CREATE INDEX idx_preferences_user ON notification_preferences(user_id);

-- Notification templates
CREATE TABLE notification_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),  -- NULL for platform templates
  type VARCHAR(100) NOT NULL,
  channel VARCHAR(50) NOT NULL,
  locale VARCHAR(10) DEFAULT 'en',
  
  -- Template content
  subject VARCHAR(500),  -- For email
  body TEXT NOT NULL,
  html_body TEXT,  -- For email
  
  -- Variables
  required_variables TEXT[],
  
  -- Metadata
  is_system_template BOOLEAN DEFAULT false,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(tenant_id, type, channel, locale)
);

CREATE INDEX idx_templates_type ON notification_templates(type, channel);
```

### 2. Notification Service

```typescript
// packages/notifications/src/notification-service.ts

interface NotificationRequest {
  user_id: string;
  type: string;
  priority: 'critical' | 'high' | 'medium' | 'low';
  title: string;
  message: string;
  action_url?: string;
  action_label?: string;
  metadata?: Record<string, any>;
  related_entity?: {
    id: string;
    type: string;
  };
  batch_key?: string;  // For grouping
  expires_at?: Date;
}

export class NotificationService {
  constructor(
    private db: Database,
    private queue: BullQueue,
    private websocket: WebSocketServer
  ) {}

  async send(request: NotificationRequest): Promise<string> {
    // 1. Create notification record
    const notification = await this.db
      .insertInto('notifications')
      .values({
        tenant_id: this.getTenantId(),
        user_id: request.user_id,
        type: request.type,
        priority: request.priority,
        title: request.title,
        message: request.message,
        action_url: request.action_url,
        action_label: request.action_label,
        metadata: JSON.stringify(request.metadata || {}),
        related_entity_id: request.related_entity?.id,
        related_entity_type: request.related_entity?.type,
        batch_key: request.batch_key,
        expires_at: request.expires_at,
        read: false,
        archived: false
      })
      .returning('*')
      .executeTakeFirstOrThrow();

    // 2. Queue for processing
    await this.queue.add('process-notification', {
      notification_id: notification.id
    }, {
      priority: this.getPriorityWeight(request.priority),
      removeOnComplete: 100,
      removeOnFail: 1000
    });

    return notification.id;
  }

  async sendBatch(notifications: NotificationRequest[]): Promise<string[]> {
    // Batch insert for efficiency
    const results = await this.db
      .insertInto('notifications')
      .values(notifications.map(n => ({
        tenant_id: this.getTenantId(),
        user_id: n.user_id,
        type: n.type,
        priority: n.priority,
        title: n.title,
        message: n.message,
        action_url: n.action_url,
        action_label: n.action_label,
        metadata: JSON.stringify(n.metadata || {}),
        batch_key: n.batch_key,
        expires_at: n.expires_at
      })))
      .returning('id')
      .execute();

    // Queue all
    await Promise.all(
      results.map(r => this.queue.add('process-notification', {
        notification_id: r.id
      }))
    );

    return results.map(r => r.id);
  }

  async markAsRead(notificationId: string, userId: string): Promise<void> {
    await this.db
      .updateTable('notifications')
      .set({
        read: true,
        read_at: new Date()
      })
      .where('id', '=', notificationId)
      .where('user_id', '=', userId)
      .execute();
  }

  async markAllAsRead(userId: string): Promise<void> {
    await this.db
      .updateTable('notifications')
      .set({
        read: true,
        read_at: new Date()
      })
      .where('user_id', '=', userId)
      .where('read', '=', false)
      .execute();
  }

  async getUnreadCount(userId: string): Promise<number> {
    const result = await this.db
      .selectFrom('notifications')
      .select(db => db.fn.count<number>('id').as('count'))
      .where('user_id', '=', userId)
      .where('read', '=', false)
      .where('archived', '=', false)
      .executeTakeFirstOrThrow();

    return result.count;
  }

  async list(userId: string, options?: {
    unread_only?: boolean;
    limit?: number;
    offset?: number;
  }): Promise<Notification[]> {
    let query = this.db
      .selectFrom('notifications')
      .selectAll()
      .where('user_id', '=', userId)
      .where('archived', '=', false);

    if (options?.unread_only) {
      query = query.where('read', '=', false);
    }

    return await query
      .orderBy('created_at', 'desc')
      .limit(options?.limit || 50)
      .offset(options?.offset || 0)
      .execute();
  }

  private getPriorityWeight(priority: string): number {
    const weights = {
      critical: 1,
      high: 2,
      medium: 3,
      low: 4
    };
    return weights[priority] || 3;
  }
}
```

### 3. Notification Processor

```typescript
// packages/notifications/src/processor.ts

export class NotificationProcessor {
  constructor(
    private db: Database,
    private emailService: EmailService,
    private pushService: PushNotificationService,
    private slackService: SlackService,
    private webhookService: WebhookService,
    private websocket: WebSocketServer
  ) {}

  async process(job: Job<{ notification_id: string }>): Promise<void> {
    const notificationId = job.data.notification_id;

    // 1. Load notification
    const notification = await this.db
      .selectFrom('notifications')
      .selectAll()
      .where('id', '=', notificationId)
      .executeTakeFirst();

    if (!notification) {
      throw new Error(`Notification ${notificationId} not found`);
    }

    // 2. Load user preferences
    const preferences = await this.getUserPreferences(
      notification.user_id,
      notification.type
    );

    // 3. Check if should send (frequency, quiet hours)
    if (!this.shouldSend(notification, preferences)) {
      // Queue for later (batch/digest)
      await this.queueForBatch(notification, preferences);
      return;
    }

    // 4. Determine channels
    const channels = this.getEnabledChannels(notification, preferences);

    // 5. Send via all channels in parallel
    const deliveryPromises = channels.map(async (channel) => {
      try {
        await this.sendViaChannel(notification, channel, preferences);
      } catch (error) {
        console.error(`Failed to send via ${channel}:`, error);
        await this.logDeliveryFailure(notification.id, channel, error);
      }
    });

    await Promise.allSettled(deliveryPromises);

    // 6. Always send in-app (WebSocket)
    await this.sendInApp(notification);
  }

  private async getUserPreferences(
    userId: string,
    notificationType: string
  ): Promise<NotificationPreferences> {
    const prefs = await this.db
      .selectFrom('notification_preferences')
      .selectAll()
      .where('user_id', '=', userId)
      .where('notification_type', '=', notificationType)
      .executeTakeFirst();

    // Return defaults if not found
    return prefs || this.getDefaultPreferences(notificationType);
  }

  private shouldSend(
    notification: Notification,
    preferences: NotificationPreferences
  ): boolean {
    // Check frequency
    if (preferences.frequency === 'never') {
      return false;
    }

    if (preferences.frequency !== 'realtime') {
      // Will be batched
      return false;
    }

    // Check quiet hours
    if (preferences.quiet_hours_start && preferences.quiet_hours_end) {
      const now = new Date();
      const currentTime = format(now, 'HH:mm');
      
      if (this.isInQuietHours(currentTime, preferences)) {
        // Queue for after quiet hours unless critical
        return notification.priority === 'critical';
      }
    }

    return true;
  }

  private getEnabledChannels(
    notification: Notification,
    preferences: NotificationPreferences
  ): string[] {
    const channels: string[] = [];

    // Critical notifications go to all channels
    if (notification.priority === 'critical') {
      return ['email', 'in_app', 'push', 'slack', 'webhook'];
    }

    if (preferences.email_enabled) channels.push('email');
    if (preferences.push_enabled) channels.push('push');
    if (preferences.slack_enabled) channels.push('slack');
    if (preferences.webhook_enabled) channels.push('webhook');

    // In-app always included (handled separately)
    return channels;
  }

  private async sendViaChannel(
    notification: Notification,
    channel: string,
    preferences: NotificationPreferences
  ): Promise<void> {
    const delivery = await this.createDeliveryRecord(notification.id, channel);

    try {
      switch (channel) {
        case 'email':
          await this.sendEmail(notification, delivery.id);
          break;
        case 'push':
          await this.sendPush(notification, delivery.id);
          break;
        case 'slack':
          await this.sendSlack(notification, delivery.id);
          break;
        case 'webhook':
          await this.sendWebhook(notification, delivery.id);
          break;
      }

      await this.markDeliverySuccess(delivery.id);
    } catch (error) {
      await this.markDeliveryFailure(delivery.id, error);
      throw error;
    }
  }

  private async sendEmail(
    notification: Notification,
    deliveryId: string
  ): Promise<void> {
    // Get user email
    const user = await this.db
      .selectFrom('users')
      .select('email')
      .where('id', '=', notification.user_id)
      .executeTakeFirstOrThrow();

    // Render template
    const template = await this.getTemplate(notification.type, 'email');
    const rendered = this.renderTemplate(template, notification);

    // Send via email service
    const messageId = await this.emailService.send({
      to: user.email,
      subject: rendered.subject,
      html: rendered.html,
      text: rendered.text
    });

    // Update delivery record
    await this.db
      .updateTable('notification_deliveries')
      .set({
        recipient: user.email,
        provider_message_id: messageId,
        sent_at: new Date()
      })
      .where('id', '=', deliveryId)
      .execute();
  }

  private async sendInApp(notification: Notification): Promise<void> {
    // Send via WebSocket to connected clients
    const clients = await this.websocket.getClientsForUser(notification.user_id);
    
    for (const client of clients) {
      client.send(JSON.stringify({
        type: 'notification',
        data: notification
      }));
    }

    // Also create delivery record
    await this.createDeliveryRecord(notification.id, 'in_app');
    await this.markDeliverySuccess(
      notification.id,
      'in_app'
    );
  }

  private async sendPush(
    notification: Notification,
    deliveryId: string
  ): Promise<void> {
    // Get user's device tokens
    const devices = await this.db
      .selectFrom('user_devices')
      .select(['device_token', 'platform'])
      .where('user_id', '=', notification.user_id)
      .where('push_enabled', '=', true)
      .execute();

    for (const device of devices) {
      await this.pushService.send({
        token: device.device_token,
        platform: device.platform,
        title: notification.title,
        body: notification.message,
        data: {
          notification_id: notification.id,
          action_url: notification.action_url
        }
      });
    }

    await this.db
      .updateTable('notification_deliveries')
      .set({
        recipient: `${devices.length} devices`,
        sent_at: new Date()
      })
      .where('id', '=', deliveryId)
      .execute();
  }

  private renderTemplate(
    template: NotificationTemplate,
    notification: Notification
  ): { subject: string; html: string; text: string } {
    // Simple variable substitution
    const variables = {
      title: notification.title,
      message: notification.message,
      action_url: notification.action_url,
      action_label: notification.action_label,
      ...notification.metadata
    };

    const subject = this.interpolate(template.subject, variables);
    const html = this.interpolate(template.html_body, variables);
    const text = this.interpolate(template.body, variables);

    return { subject, html, text };
  }

  private interpolate(template: string, variables: Record<string, any>): string {
    return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
      return variables[key] !== undefined ? String(variables[key]) : match;
    });
  }
}
```

### 4. Batching & Digest Service

```typescript
// packages/notifications/src/digest-service.ts

export class DigestService {
  async generateHourlyDigest(userId: string): Promise<void> {
    // Get notifications from last hour for this user
    const notifications = await this.db
      .selectFrom('notifications')
      .selectAll()
      .where('user_id', '=', userId)
      .where('created_at', '>', subHours(new Date(), 1))
      .where('batch_key', 'is not', null)
      .orderBy('created_at', 'desc')
      .execute();

    if (notifications.length === 0) {
      return;
    }

    // Group by batch_key
    const grouped = groupBy(notifications, 'batch_key');

    // Create digest notification
    const digestHtml = this.renderDigest(grouped);

    await this.emailService.send({
      to: await this.getUserEmail(userId),
      subject: `TERAFFI Digest - ${notifications.length} updates`,
      html: digestHtml
    });

    // Mark as sent
    await this.db
      .updateTable('notifications')
      .set({ read: true })
      .where('id', 'in', notifications.map(n => n.id))
      .execute();
  }

  private renderDigest(
    grouped: Record<string, Notification[]>
  ): string {
    let html = '<h1>Your TERAFFI Updates</h1>';

    for (const [batchKey, notifications] of Object.entries(grouped)) {
      html += `<h2>${batchKey}</h2><ul>`;
      
      for (const notification of notifications) {
        html += `<li>
          <strong>${notification.title}</strong>
          <p>${notification.message}</p>
          ${notification.action_url ? `<a href="${notification.action_url}">${notification.action_label || 'View'}</a>` : ''}
        </li>`;
      }
      
      html += '</ul>';
    }

    return html;
  }
}
```

### 5. User Preferences API

```typescript
// packages/api/src/routes/notification-preferences.ts

export const notificationPreferencesRoutes: FastifyPluginAsync = async (fastify) => {
  // Get user's preferences
  fastify.get('/preferences', {
    preHandler: [requireAuth]
  }, async (request, reply) => {
    const userId = request.user.user_id;

    const preferences = await fastify.db
      .selectFrom('notification_preferences')
      .selectAll()
      .where('user_id', '=', userId)
      .execute();

    return preferences;
  });

  // Update preferences
  fastify.put('/preferences/:type', {
    preHandler: [requireAuth]
  }, async (request, reply) => {
    const userId = request.user.user_id;
    const { type } = request.params;
    const updates = request.body;

    await fastify.db
      .insertInto('notification_preferences')
      .values({
        user_id: userId,
        tenant_id: request.tenantContext.tenant_id,
        notification_type: type,
        ...updates
      })
      .onConflict((oc) => oc
        .columns(['user_id', 'notification_type'])
        .doUpdateSet(updates)
      )
      .execute();

    return { success: true };
  });
};
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- Database schema
- Core notification service
- Email delivery
- In-app notifications (WebSocket)

### Phase 2: Multi-Channel (Months 3-4)
- Push notifications
- Slack integration
- Webhooks
- User preferences

### Phase 3: Intelligence (Months 5-6)
- Batching and digests
- Quiet hours
- Priority routing
- Delivery tracking

### Phase 4: Advanced (Months 7-8)
- Template customization
- A/B testing
- Analytics dashboard
- Smart throttling

---

## Success Metrics

**Delivery:**
- 99.9% delivery success rate
- <5 seconds latency (critical)
- <30 seconds latency (high)
- <5 minutes latency (medium/low)

**Engagement:**
- >40% notification open rate
- >20% click-through rate
- <5% unsubscribe rate

**Quality:**
- <2% spam reports
- User satisfaction >4.0/5
- Support tickets <1% of notifications sent

---

## Task Dependencies

001 (Schema) → 002 (Core Service) → 003 (Templates) → 004 (Email)
                                                    → 005 (In-App)
                                                    → 006 (Push)
                                                    → 007 (Slack)
                                                    → 008 (Webhook)

002 → 010 (Preferences)

004-008 → 009 (Processor) → 012 (Delivery Tracking)
010 → 011 (Batching/Digest)

002 → 013 (REST API) → 016 (Unsubscribe)
010 → 013

002 → 014 (Triggers Integration)
009 → 015 (Analytics)

002 → 017 (Testing Tools)

All → 018 (Test Suite)
All → 019 (Documentation)

## References

- [SendGrid Best Practices](https://sendgrid.com/resource/email-best-practices/)
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- [Slack API - Incoming Webhooks](https://api.slack.com/messaging/webhooks)
- [Apple Push Notification Service](https://developer.apple.com/documentation/usernotifications)

---

