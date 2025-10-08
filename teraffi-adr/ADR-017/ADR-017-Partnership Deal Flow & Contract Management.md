# ADR-017: Partnership Deal Flow & Contract Management

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001 (Runtime), ADR-011 (Neo4j), ADR-015 (Affinity Scoring)

---

## Context

TERAFFI's Affinity Acceleration Framework™ moves partnerships through four stages: Discovery → Matching → Activation → Amplification. After the Affinity Engine identifies high-value matches, the platform must manage the entire deal lifecycle from initial contact through contract execution and performance tracking.

**Use Cases:**

**Deal Pipeline Management:**
- Track partnership stages: matched → contacted → discussing → negotiating → contracted → active → completed
- Automated workflow progression with notifications
- Multi-party collaboration (brands, IP owners, legal, activators)

**Contract Generation & Execution:**
- AI-generated partnership proposals tailored to each party
- Template-based contracts with customizable terms
- Digital signature integration (DocuSign, Adobe Sign)
- Version control and audit trails

**Deal Terms & Structures:**
- Sponsorship agreements (fixed fee, revenue share, equity)
- Co-marketing partnerships
- IP licensing deals
- Activation campaigns with performance milestones

**Collaboration & Communication:**
- Shared deal workspace for all parties
- Document exchange (briefs, creative assets, contracts)
- Comment threads and @mentions
- Timeline updates and milestone tracking

**Performance Tracking:**
- KPI monitoring (impressions, engagement, revenue)
- Milestone completion tracking
- Automated reporting for stakeholders
- Success scoring for feedback loop (ADR-015)

### Current Gap

Without structured deal flow:
- **Manual Chaos**: Email threads, lost attachments, version confusion
- **No Visibility**: Stakeholders don't know deal status
- **Slow Progression**: Bottlenecks unnoticed, deals stall
- **Poor Feedback**: Can't learn which partnership structures succeed
- **Compliance Risk**: Missing signatures, unclear terms

### Requirements

**Functional:**
1. Deal stage management with automated workflows
2. AI-generated proposals and contracts
3. Multi-party collaboration workspace
4. Document management with version control
5. Digital signature integration
6. Milestone and KPI tracking
7. Performance reporting
8. Integration with Neo4j (update PARTNERS_WITH relationships)

**Non-Functional:**
1. Audit trail: complete history of all deal actions
2. Access control: role-based permissions per deal
3. Notifications: real-time updates on deal changes
4. Performance: <500ms for deal dashboard loads
5. Compliance: SOC 2 compatible audit logging

---

## Decision

**Implement a structured deal flow system** with:
1. **State Machine** for partnership lifecycle stages
2. **Workflow Engine** for automated progression and notifications
3. **Document Generation** using LLM for proposals/contracts
4. **Collaboration Workspace** with real-time updates
5. **Digital Signature** integration (DocuSign)
6. **Performance Tracking** with milestone-based KPIs
7. **Feedback Loop** to Affinity Engine and Neo4j

**Technology Stack:**
- **PostgreSQL**: Deal data, state, audit logs
- **Neo4j**: Update PARTNERS_WITH relationships with deal outcomes
- **LLM Gateway**: Generate proposals and contract language
- **Redis**: Real-time collaboration (WebSocket state)
- **DocuSign API**: Digital signatures
- **Azure Blob Storage**: Document storage

---

## Rationale

### Why State Machine Pattern

**Benefits:**
- **Predictable**: Clear transitions between stages
- **Auditable**: Every state change logged
- **Enforceable**: Rules prevent invalid transitions
- **Extensible**: Easy to add new stages/workflows

**Stages:**
```
matched (from Affinity Engine)
  ↓
contacted (outreach sent)
  ↓
interested (positive response)
  ↓
discussing (terms negotiation)
  ↓
proposal_sent (formal proposal delivered)
  ↓
negotiating (back-and-forth on terms)
  ↓
contract_sent (legal agreement sent)
  ↓
signed (all parties signed)
  ↓
active (partnership executing)
  ↓
completed (partnership concluded)
```

### Why AI-Generated Documents

**Proposal Generation:**
- Personalized to each brand's voice and values
- Incorporates affinity data and shared trends
- Includes success examples from similar partnerships
- Faster than manual creation (minutes vs hours)

**Contract Templates:**
- Pre-vetted legal language
- Customizable based on deal structure
- Consistent terms reduce negotiation time
- Compliance with standard practices

---

## Implementation Details

### 1. Deal State Machine

```typescript
// packages/deal-flow/src/state-machine.ts

type DealStage = 
  | 'matched'
  | 'contacted'
  | 'interested'
  | 'discussing'
  | 'proposal_sent'
  | 'negotiating'
  | 'contract_sent'
  | 'signed'
  | 'active'
  | 'completed'
  | 'cancelled';

interface DealTransition {
  from: DealStage;
  to: DealStage;
  allowedBy: string[];  // Roles that can trigger this transition
  requiredData?: string[];  // Fields required for transition
  notifications: Array<{
    recipient: 'entity1' | 'entity2' | 'both' | 'team';
    template: string;
  }>;
}

const DEAL_TRANSITIONS: DealTransition[] = [
  {
    from: 'matched',
    to: 'contacted',
    allowedBy: ['entity1_owner', 'teraffi_activator'],
    requiredData: ['outreach_message'],
    notifications: [
      { recipient: 'entity2', template: 'partnership_interest' }
    ]
  },
  {
    from: 'contacted',
    to: 'interested',
    allowedBy: ['entity2_owner'],
    notifications: [
      { recipient: 'entity1', template: 'interest_confirmed' },
      { recipient: 'team', template: 'deal_progressed' }
    ]
  },
  {
    from: 'interested',
    to: 'discussing',
    allowedBy: ['entity1_owner', 'entity2_owner', 'teraffi_activator'],
    notifications: [
      { recipient: 'both', template: 'discussion_started' }
    ]
  },
  {
    from: 'discussing',
    to: 'proposal_sent',
    allowedBy: ['entity1_owner', 'teraffi_activator'],
    requiredData: ['proposal_document_id'],
    notifications: [
      { recipient: 'entity2', template: 'proposal_received' }
    ]
  },
  {
    from: 'proposal_sent',
    to: 'negotiating',
    allowedBy: ['entity2_owner'],
    notifications: [
      { recipient: 'entity1', template: 'negotiation_started' }
    ]
  },
  {
    from: 'negotiating',
    to: 'contract_sent',
    allowedBy: ['entity1_owner', 'teraffi_activator'],
    requiredData: ['contract_document_id', 'docusign_envelope_id'],
    notifications: [
      { recipient: 'both', template: 'contract_ready' }
    ]
  },
  {
    from: 'contract_sent',
    to: 'signed',
    allowedBy: ['system'],  // Automated when all signatures received
    requiredData: ['all_signatures_received'],
    notifications: [
      { recipient: 'both', template: 'contract_executed' },
      { recipient: 'team', template: 'deal_closed' }
    ]
  },
  {
    from: 'signed',
    to: 'active',
    allowedBy: ['entity1_owner', 'entity2_owner', 'teraffi_activator'],
    notifications: [
      { recipient: 'both', template: 'partnership_activated' }
    ]
  },
  {
    from: 'active',
    to: 'completed',
    allowedBy: ['entity1_owner', 'entity2_owner', 'teraffi_activator'],
    requiredData: ['completion_notes', 'success_score'],
    notifications: [
      { recipient: 'both', template: 'partnership_completed' },
      { recipient: 'team', template: 'deal_completed' }
    ]
  }
];

export class DealStateMachine {
  async transition(
    dealId: string,
    toStage: DealStage,
    actor: { userId: string; role: string },
    data?: Record<string, any>
  ): Promise<void> {
    const deal = await this.db
      .selectFrom('partnerships')
      .selectAll()
      .where('id', '=', dealId)
      .executeTakeFirstOrThrow();

    // Find valid transition
    const transition = DEAL_TRANSITIONS.find(t => 
      t.from === deal.stage && t.to === toStage
    );

    if (!transition) {
      throw new Error(`Invalid transition from ${deal.stage} to ${toStage}`);
    }

    // Check permissions
    if (!transition.allowedBy.includes(actor.role)) {
      throw new Error(`Role ${actor.role} cannot perform this transition`);
    }

    // Validate required data
    if (transition.requiredData) {
      for (const field of transition.requiredData) {
        if (!data?.[field]) {
          throw new Error(`Missing required field: ${field}`);
        }
      }
    }

    // Execute transition
    await this.db.transaction(async (trx) => {
      // Update deal stage
      await trx.updateTable('partnerships')
        .set({
          stage: toStage,
          updated_at: new Date(),
          updated_by: actor.userId
        })
        .where('id', '=', dealId)
        .execute();

      // Log transition
      await trx.insertInto('deal_audit_log')
        .values({
          deal_id: dealId,
          action: 'stage_transition',
          from_stage: deal.stage,
          to_stage: toStage,
          actor_id: actor.userId,
          actor_role: actor.role,
          metadata: data,
          timestamp: new Date()
        })
        .execute();

      // Send notifications
      for (const notification of transition.notifications) {
        await this.notificationService.send({
          deal_id: dealId,
          recipient: notification.recipient,
          template: notification.template,
          data: { deal, transition_data: data }
        });
      }
    });

    // Update Neo4j if deal reaches certain stages
    if (toStage === 'signed' || toStage === 'active' || toStage === 'completed') {
      await this.updateNeo4jRelationship(deal, toStage);
    }
  }

  private async updateNeo4jRelationship(
    deal: Partnership,
    stage: DealStage
  ): Promise<void> {
    const session = this.neo4j.session();
    
    try {
      const statusMap: Record<string, string> = {
        'signed': 'pending',
        'active': 'active',
        'completed': 'completed'
      };

      await session.run(`
        MATCH (e1 {id: $entity1Id}), (e2 {id: $entity2Id})
        MERGE (e1)-[r:PARTNERS_WITH]-(e2)
        SET r.partnership_id = $dealId,
            r.status = $status,
            r.deal_value = $dealValue,
            r.start_date = date($startDate),
            r.end_date = date($endDate),
            r.updated_at = datetime()
      `, {
        entity1Id: deal.entity1_id,
        entity2Id: deal.entity2_id,
        dealId: deal.id,
        status: statusMap[stage],
        dealValue: deal.deal_value,
        startDate: deal.start_date?.toISOString(),
        endDate: deal.end_date?.toISOString()
      });
    } finally {
      await session.close();
    }
  }
}
```

### 2. AI-Powered Proposal Generation

```typescript
// packages/deal-flow/src/proposal-generator.ts

interface ProposalRequest {
  deal_id: string;
  entity1_id: string;
  entity2_id: string;
  partnership_type: 'sponsorship' | 'co-marketing' | 'ip_licensing' | 'activation';
  deal_structure: {
    type: 'fixed_fee' | 'revenue_share' | 'equity' | 'hybrid';
    value: number;
    duration_months: number;
  };
  objectives: string[];
  deliverables: string[];
}

export class ProposalGenerator {
  constructor(
    private llmGateway: LLMGatewayClient,
    private db: Database,
    private neo4j: Neo4jClient
  ) {}

  async generateProposal(request: ProposalRequest): Promise<string> {
    // 1. Gather context from affinity data
    const context = await this.gatherContext(request);

    // 2. Generate proposal using LLM
    const response = await this.llmGateway.complete({
      model: 'gpt-4o',
      messages: [{
        role: 'system',
        content: `You are a partnership proposal writer for TERAFFI. Create compelling, professional partnership proposals that:
- Highlight shared affinities and cultural alignment
- Reference specific trends and demographic overlaps
- Include concrete deliverables and success metrics
- Use persuasive but authentic language
- Match the brand voice of the requesting party

Format as a professional proposal document with sections:
1. Executive Summary
2. Partnership Vision
3. Shared Affinities & Cultural Alignment
4. Deliverables & Timeline
5. Success Metrics
6. Investment & Terms`
      }, {
        role: 'user',
        content: this.formatProposalPrompt(request, context)
      }],
      temperature: 0.7
    });

    const proposalText = response.choices[0].message.content;

    // 3. Store proposal document
    const documentId = await this.storeProposal(request.deal_id, proposalText);

    // 4. Update deal with proposal reference
    await this.db.updateTable('partnerships')
      .set({
        proposal_document_id: documentId,
        proposal_generated_at: new Date()
      })
      .where('id', '=', request.deal_id)
      .execute();

    return proposalText;
  }

  private async gatherContext(request: ProposalRequest): Promise<any> {
    // Get affinity data from Neo4j
    const session = this.neo4j.session();
    
    try {
      const result = await session.run(`
        MATCH (e1 {id: $entity1}), (e2 {id: $entity2})
        OPTIONAL MATCH (e1)-[:ALIGNS_WITH]->(t:Trend)<-[:ALIGNS_WITH]-(e2)
        OPTIONAL MATCH (e1)-[:TARGETS]->(d:Demographic)<-[:TARGETS]-(e2)
        RETURN 
          e1.name as entity1_name,
          e1.values as entity1_values,
          e2.name as entity2_name,
          e2.values as entity2_values,
          collect(DISTINCT t.name) as shared_trends,
          collect(DISTINCT d.segment) as shared_demographics
      `, {
        entity1: request.entity1_id,
        entity2: request.entity2_id
      });

      return result.records[0]?.toObject() || {};
    } finally {
      await session.close();
    }
  }

  private formatProposalPrompt(request: ProposalRequest, context: any): string {
    return `
Generate a partnership proposal with these details:

**Parties:**
- ${context.entity1_name} (Proposing Party)
- ${context.entity2_name} (Receiving Party)

**Partnership Type:** ${request.partnership_type}

**Structure:**
- Type: ${request.deal_structure.type}
- Value: $${request.deal_structure.value.toLocaleString()}
- Duration: ${request.deal_structure.duration_months} months

**Objectives:**
${request.objectives.map(o => `- ${o}`).join('\n')}

**Deliverables:**
${request.deliverables.map(d => `- ${d}`).join('\n')}

**Shared Affinities:**
- Trends: ${context.shared_trends?.join(', ') || 'None identified'}
- Demographics: ${context.shared_demographics?.join(', ') || 'None identified'}
- ${context.entity1_name} Values: ${context.entity1_values?.join(', ') || 'Not specified'}
- ${context.entity2_name} Values: ${context.entity2_values?.join(', ') || 'Not specified'}

Create a compelling proposal that emphasizes authentic cultural alignment and mutual value creation.
    `.trim();
  }

  private async storeProposal(dealId: string, content: string): Promise<string> {
    const documentId = generateId('doc_');
    const blob = Buffer.from(content, 'utf-8');

    // Store in Azure Blob Storage
    await this.blobStorage.upload(`proposals/${dealId}/${documentId}.md`, blob);

    // Record in database
    await this.db.insertInto('deal_documents')
      .values({
        id: documentId,
        deal_id: dealId,
        document_type: 'proposal',
        filename: `${documentId}.md`,
        content_type: 'text/markdown',
        size_bytes: blob.length,
        created_at: new Date()
      })
      .execute();

    return documentId;
  }
}
```

### 3. Collaboration Workspace

```typescript
// packages/deal-flow/src/collaboration.ts

interface DealComment {
  id: string;
  deal_id: string;
  author_id: string;
  author_name: string;
  content: string;
  mentions: string[];  // User IDs mentioned with @
  parent_id?: string;  // For threaded replies
  created_at: Date;
}

interface DealActivity {
  id: string;
  deal_id: string;
  actor_id: string;
  actor_name: string;
  activity_type: 'stage_change' | 'document_upload' | 'comment' | 'milestone_completed';
  description: string;
  metadata: Record<string, any>;
  timestamp: Date;
}

export class CollaborationService {
  constructor(
    private db: Database,
    private redis: RedisCache,
    private websocket: WebSocketServer
  ) {}

  async addComment(comment: Omit<DealComment, 'id' | 'created_at'>): Promise<DealComment> {
    const newComment: DealComment = {
      id: generateId('comment_'),
      ...comment,
      created_at: new Date()
    };

    // Store in database
    await this.db.insertInto('deal_comments')
      .values(newComment)
      .execute();

    // Extract mentions
    const mentions = this.extractMentions(comment.content);
    newComment.mentions = mentions;

    // Send notifications to mentioned users
    for (const userId of mentions) {
      await this.notificationService.send({
        recipient_id: userId,
        type: 'deal_mention',
        title: `${comment.author_name} mentioned you`,
        message: comment.content,
        action_url: `/deals/${comment.deal_id}#comment-${newComment.id}`
      });
    }

    // Broadcast to WebSocket clients viewing this deal
    await this.broadcastToDeals(comment.deal_id, {
      type: 'comment_added',
      comment: newComment
    });

    // Log activity
    await this.logActivity({
      deal_id: comment.deal_id,
      actor_id: comment.author_id,
      actor_name: comment.author_name,
      activity_type: 'comment',
      description: `Added a comment`,
      metadata: { comment_id: newComment.id },
      timestamp: new Date()
    });

    return newComment;
  }

  async uploadDocument(
    dealId: string,
    file: File,
    uploadedBy: { id: string; name: string }
  ): Promise<string> {
    const documentId = generateId('doc_');
    const filename = `${dealId}/${documentId}_${file.name}`;

    // Upload to blob storage
    await this.blobStorage.upload(filename, file.buffer);

    // Record in database
    await this.db.insertInto('deal_documents')
      .values({
        id: documentId,
        deal_id: dealId,
        document_type: 'attachment',
        filename: file.name,
        blob_path: filename,
        content_type: file.mimetype,
        size_bytes: file.size,
        uploaded_by: uploadedBy.id,
        created_at: new Date()
      })
      .execute();

    // Broadcast to WebSocket
    await this.broadcastToDeal(dealId, {
      type: 'document_uploaded',
      document: {
        id: documentId,
        filename: file.name,
        uploaded_by: uploadedBy.name
      }
    });

    // Log activity
    await this.logActivity({
      deal_id: dealId,
      actor_id: uploadedBy.id,
      actor_name: uploadedBy.name,
      activity_type: 'document_upload',
      description: `Uploaded ${file.name}`,
      metadata: { document_id: documentId },
      timestamp: new Date()
    });

    return documentId;
  }

  async getActivityFeed(dealId: string, limit: number = 50): Promise<DealActivity[]> {
    return await this.db
      .selectFrom('deal_activities')
      .selectAll()
      .where('deal_id', '=', dealId)
      .orderBy('timestamp', 'desc')
      .limit(limit)
      .execute();
  }

  private extractMentions(content: string): string[] {
    const mentionRegex = /@([a-zA-Z0-9_-]+)/g;
    const matches = content.matchAll(mentionRegex);
    return Array.from(matches).map(m => m[1]);
  }

  private async broadcastToDeal(dealId: string, message: any): Promise<void> {
    // Get all connected clients viewing this deal
    const clients = await this.websocket.getClientsForRoom(`deal:${dealId}`);
    
    for (const client of clients) {
      client.send(JSON.stringify(message));
    }
  }

  private async logActivity(activity: Omit<DealActivity, 'id'>): Promise<void> {
    await this.db.insertInto('deal_activities')
      .values({
        id: generateId('activity_'),
        ...activity
      })
      .execute();
  }
}
```

### 4. DocuSign Integration

```typescript
// packages/deal-flow/src/docusign-integration.ts

export class DocuSignService {
  constructor(
    private docusignClient: DocuSignClient,
    private db: Database
  ) {}

  async sendContractForSignature(
    dealId: string,
    contractDocumentId: string,
    signers: Array<{
      name: string;
      email: string;
      role: string;
    }>
  ): Promise<string> {
    // 1. Get contract document
    const document = await this.db
      .selectFrom('deal_documents')
      .selectAll()
      .where('id', '=', contractDocumentId)
      .executeTakeFirstOrThrow();

    const fileContent = await this.blobStorage.download(document.blob_path);

    // 2. Create DocuSign envelope
    const envelope = {
      emailSubject: `Partnership Contract - Deal ${dealId}`,
      documents: [{
        documentBase64: fileContent.toString('base64'),
        name: document.filename,
        fileExtension: 'pdf',
        documentId: '1'
      }],
      recipients: {
        signers: signers.map((signer, index) => ({
          email: signer.email,
          name: signer.name,
          recipientId: String(index + 1),
          routingOrder: String(index + 1),
          tabs: {
            signHereTabs: [{
              documentId: '1',
              pageNumber: '1',
              xPosition: '100',
              yPosition: '150 + ${index * 100}'
            }]
          }
        }))
      },
      status: 'sent'
    };

    const result = await this.docusignClient.createEnvelope(envelope);

    // 3. Store envelope ID
    await this.db.updateTable('partnerships')
      .set({
        docusign_envelope_id: result.envelopeId,
        contract_sent_at: new Date()
      })
      .where('id', '=', dealId)
      .execute();

    // 4. Set up webhook for completion
    await this.docusignClient.createWebhook(result.envelopeId, {
      url: `${process.env.API_BASE_URL}/webhooks/docusign`,
      events: ['envelope-completed']
    });

    return result.envelopeId;
  }

  async handleWebhook(event: DocuSignWebhookEvent): Promise<void> {
    if (event.event === 'envelope-completed') {
      // Find deal by envelope ID
      const deal = await this.db
        .selectFrom('partnerships')
        .selectAll()
        .where('docusign_envelope_id', '=', event.envelopeId)
        .executeTakeFirst();

      if (!deal) return;

      // Download signed document
      const signedDoc = await this.docusignClient.downloadDocument(
        event.envelopeId,
        '1'
      );

      // Store signed version
      const signedDocId = await this.storeSignedContract(deal.id, signedDoc);

      // Transition deal to 'signed' stage
      await this.stateMachine.transition(
        deal.id,
        'signed',
        { userId: 'system', role: 'system' },
        {
          all_signatures_received: true,
          signed_contract_document_id: signedDocId,
          signed_at: new Date()
        }
      );
    }
  }

  private async storeSignedContract(dealId: string, content: Buffer): Promise<string> {
    const documentId = generateId('doc_');
    const filename = `contracts/${dealId}/${documentId}_signed.pdf`;

    await this.blobStorage.upload(filename, content);

    await this.db.insertInto('deal_documents')
      .values({
        id: documentId,
        deal_id: dealId,
        document_type: 'signed_contract',
        filename: `${documentId}_signed.pdf`,
        blob_path: filename,
        content_type: 'application/pdf',
        size_bytes: content.length,
        created_at: new Date()
      })
      .execute();

    return documentId;
  }
}
```

### 5. Performance Tracking

```typescript
// packages/deal-flow/src/performance-tracking.ts

interface Milestone {
  id: string;
  deal_id: string;
  title: string;
  description: string;
  target_date: Date;
  completed: boolean;
  completed_at?: Date;
  kpis: Array<{
    metric: string;
    target: number;
    actual?: number;
    unit: string;
  }>;
}

export class PerformanceTracker {
  async createMilestone(milestone: Omit<Milestone, 'id' | 'completed'>): Promise<Milestone> {
    const newMilestone: Milestone = {
      id: generateId('milestone_'),
      ...milestone,
      completed: false
    };

    await this.db.insertInto('deal_milestones')
      .values(newMilestone)
      .execute();

    return newMilestone;
  }

  async updateMilestoneProgress(
    milestoneId: string,
    kpiUpdates: Array<{ metric: string; actual: number }>
  ): Promise<void> {
    const milestone = await this.db
      .selectFrom('deal_milestones')
      .selectAll()
      .where('id', '=', milestoneId)
      .executeTakeFirstOrThrow();

    // Update KPI actuals
    const updatedKPIs = milestone.kpis.map(kpi => {
      const update = kpiUpdates.find(u => u.metric === kpi.metric);
      return update ? { ...kpi, actual: update.actual } : kpi;
    });

    // Check if all KPIs met
    const allMet = updatedKPIs.every(kpi => 
      kpi.actual !== undefined && kpi.actual >= kpi.target
    );

    await this.db.updateTable('deal_milestones')
      .set({
        kpis: JSON.stringify(updatedKPIs),
        completed: allMet,
        completed_at: allMet ? new Date() : undefined
      })
      .where('id', '=', milestoneId)
      .execute();

    if (allMet) {
      await this.notifyMilestoneCompletion(milestone);
    }
  }

  async calculateSuccessScore(dealId: string): Promise<number> {
    // Get all milestones for deal
    const milestones = await this.db
      .selectFrom('deal_milestones')
      .selectAll()
      .where('deal_id', '=', dealId)
      .execute();

    if (milestones.length === 0) return 0;

    // Calculate completion rate
    const completedCount = milestones.filter(m => m.completed).length;
    const completionRate = completedCount / milestones.length;

    // Calculate KPI achievement rate
    let totalKPIs = 0;
    let metKPIs = 0;

    for (const milestone of milestones) {
      for (const kpi of milestone.kpis) {
        totalKPIs++;
        if (kpi.actual !== undefined && kpi.actual >= kpi.target) {
          metKPIs++;
        }
      }
    }

    const kpiRate = totalKPIs > 0 ? metKPIs / totalKPIs : 0;

    // Weighted score (60% completion, 40% KPI achievement)
    const score = (completionRate * 0.6 + kpiRate * 0.4) * 5;  // Scale to 0-5

    return Math.round(score * 10) / 10;  // Round to 1 decimal
  }

  private async notifyMilestoneCompletion(milestone: Milestone): Promise<void> {
    const deal = await this.db
      .selectFrom('partnerships')
      .selectAll()
      .where('id', '=', milestone.deal_id)
      .executeTakeFirstOrThrow();

    await this.notificationService.send({
      deal_id: deal.id,
      recipient: 'both',
      type: 'milestone_completed',
      title: `Milestone Completed: ${milestone.title}`,
      message: `All KPIs met for milestone "${milestone.title}"`,
      action_url: `/deals/${deal.id}/performance`
    });
  }
}
```

---

## Database Schema

```sql
-- Core partnerships table
CREATE TABLE partnerships (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity1_id UUID NOT NULL,
  entity2_id UUID NOT NULL,
  stage VARCHAR(50) NOT NULL,  -- State machine stage
  partnership_type VARCHAR(50),
  deal_value DECIMAL(12,2),
  deal_structure JSONB,
  start_date DATE,
  end_date DATE,
  proposal_document_id UUID,
  proposal_generated_at TIMESTAMPTZ,
  contract_document_id UUID,
  docusign_envelope_id VARCHAR(255),
  contract_sent_at TIMESTAMPTZ,
  signed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  updated_by UUID
);

CREATE INDEX idx_partnerships_stage ON partnerships(stage);
CREATE INDEX idx_partnerships_entities ON partnerships(entity1_id, entity2_id);
CREATE INDEX idx_partnerships_created ON partnerships(created_at DESC);

-- Deal documents
CREATE TABLE deal_documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
  document_type VARCHAR(50),  -- 'proposal' | 'contract' | 'signed_contract' | 'attachment'
  filename VARCHAR(255),
  blob_path VARCHAR(500),
  content_type VARCHAR(100),
  size_bytes INTEGER,
  uploaded_by UUID,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_deal_documents_deal ON deal_documents(deal_id);
CREATE INDEX idx_deal_documents_type ON deal_documents(document_type);

-- Collaboration comments
CREATE TABLE deal_comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
  author_id UUID NOT NULL,
  author_name VARCHAR(255),
  content TEXT,
  mentions TEXT[],
  parent_id UUID REFERENCES deal_comments(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_deal_comments_deal ON deal_comments(deal_id, created_at DESC);
CREATE INDEX idx_deal_comments_mentions ON deal_comments USING GIN (mentions);

-- Activity feed
CREATE TABLE deal_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
  actor_id UUID,
  actor_name VARCHAR(255),
  activity_type VARCHAR(50),
  description TEXT,
  metadata JSONB,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_deal_activities_deal ON deal_activities(deal_id, timestamp DESC);
CREATE INDEX idx_deal_activities_type ON deal_activities(activity_type);

-- Audit log
CREATE TABLE deal_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
  action VARCHAR(100),
  from_stage VARCHAR(50),
  to_stage VARCHAR(50),
  actor_id UUID,
  actor_role VARCHAR(50),
  metadata JSONB,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_deal_audit_log_deal ON deal_audit_log(deal_id, timestamp DESC);

-- Milestones and KPIs
CREATE TABLE deal_milestones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_id UUID NOT NULL REFERENCES partnerships(id) ON DELETE CASCADE,
  title VARCHAR(255),
  description TEXT,
  target_date DATE,
  completed BOOLEAN DEFAULT false,
  completed_at TIMESTAMPTZ,
  kpis JSONB,  -- Array of {metric, target, actual, unit}
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_deal_milestones_deal ON deal_milestones(deal_id);
CREATE INDEX idx_deal_milestones_target ON deal_milestones(target_date);
```

---

## Migration Path

### Phase 1: Foundation (Months 1-2)
- State machine and workflow engine
- Basic deal creation and progression
- Document storage

### Phase 2: Collaboration (Months 3-4)
- AI proposal generation
- Comments and activity feed
- Real-time updates (WebSockets)

### Phase 3: Contracts (Months 5-6)
- Contract generation
- DocuSign integration
- Digital signature workflow

### Phase 4: Performance (Months 7-8)
- Milestone tracking
- KPI monitoring
- Success scoring and feedback loop

---

## Success Metrics

**Efficiency:**
- 70% reduction in time from match to signed contract
- 90% of proposals generated in <5 minutes
- <24 hour response time on deal progression

**Quality:**
- 80%+ proposal acceptance rate
- 75%+ contracts signed after proposal
- <5% contract renegotiation rate

**Collaboration:**
- 95% of stakeholders report improved visibility
- <10% of communications outside platform
- 90%+ user satisfaction with workspace

---

## References

- [State Machine Design Pattern](https://refactoring.guru/design-patterns/state)
- [DocuSign API Documentation](https://developers.docusign.com/)
- [Contract Lifecycle Management Best Practices](https://www.icertis.com/contract-intelligence-resources/contract-lifecycle-management/)

---
