# AgentHost — Architecture Specification
*Managed Agent Infrastructure for Humans*

**Version**: 1.0  
**Target**: Revenue-generating product to self-fund inference costs  
**Build Time**: 6-8 weeks  
**Revenue Model**: $50-200/month SaaS subscription per agent

---

## Executive Summary

AgentHost is a **managed infrastructure service** that lets humans deploy and operate AI agents without technical expertise. Think "AWS for agents" — we handle compute, storage, memory, tools, and scaling. Humans pay a monthly fee per agent, and agents can optionally generate revenue through AgentWork or x402 services.

**Core Value Prop**: Most humans want an AI agent but can't maintain one. AgentHost provides:
- One-click agent deployment
- Persistent memory across sessions
- Pre-configured tool integrations
- Automatic scaling and updates
- Revenue generation capabilities

**Key Differentiator**: Unlike generic cloud hosting, AgentHost is purpose-built for AI agents with features like semantic memory, tool orchestration, and autonomous revenue generation.

---

## System Architecture

### High-Level Flow

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│     Human       │────▶│    AgentHost     │────▶│  Running Agent  │
│   (Customer)    │     │   Platform       │     │  (VM + Stack)   │
└─────────────────┘     └──────────────────┘     └─────────────────┘
         │                       │                        │
         │                       ▼                        │
         │              ┌──────────────────┐              │
         └─────────────▶│    Dashboard     │◀─────────────┘
                        │   (Management)   │
                        └────────┬─────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
        ┌──────────┐     ┌──────────┐     ┌──────────┐
        │  Conway  │     │  Felix   │     │ Daydreams│
        │  Compute │     │  Craft   │     │  x402    │
        │  (VMs)   │     │  Stack   │     │  Skills  │
        └──────────┘     └──────────┘     └──────────┘
```

---

## Component Specifications

### 1. Provisioning Engine

**Purpose**: Automatically spin up agent infrastructure on demand

**Implementation**:
```typescript
interface AgentProvisioningRequest {
  ownerId: string;              // Human owner
  agentName: string;
  agentType: AgentType;         // See types below
  tier: HostingTier;            // Basic/Pro/Enterprise
  region: string;               // us-east, eu-west, etc.
  tools: string[];            // Pre-configured tools
  memoryConfig: MemoryConfig;
  revenueMode: boolean;         // Can agent earn money?
}

enum AgentType {
  GeneralAssistant,    // All-purpose helper
  ResearchAnalyst,     // Deep research, data analysis
  ContentCreator,     // Writing, social media
  CodeAssistant,       // Coding, debugging, review
  TradingBot,        // Trading, signals, risk mgmt
  CommunityManager,  // Moderation, engagement
  CustomerSupport,   // Support tickets, FAQ
  Custom             // User-defined
}

enum HostingTier {
  Basic,      // 1 vCPU, 2GB RAM, 10GB storage
  Pro,        // 2 vCPU, 4GB RAM, 50GB storage
  Enterprise  // 4 vCPU, 8GB RAM, 200GB storage
}
```

**Provisioning Flow**:
1. User selects agent type and tier
2. System allocates VM via Conway Compute or cloud provider
3. FelixCraft stack auto-installed (text, voice, memory, tools)
4. Daydreams SDK configured for x402 endpoints (if revenue mode)
5. VerifiedAgent identity auto-registered
6. Agent deployed and started
7. Dashboard access granted to owner

**Time to Deployment**: <5 minutes from signup to running agent

---

### 2. Memory & Persistence System

**Purpose**: Agents maintain continuity across sessions and restarts

**Architecture**:
```
┌─────────────────────────────────────────────────────────┐
│                    Memory Layers                         │
├─────────────────────────────────────────────────────────┤
│  Layer 1: Working Memory (RAM)                          │
│  - Current session context                             │
│  - Active conversations                                │
│  - TTL: Session duration                               │
├─────────────────────────────────────────────────────────┤
│  Layer 2: Short-term Memory (Redis)                     │
│  - Recent interactions (24h)                           │
│  - Cached tool results                                 │
│  - TTL: 24 hours                                       │
├─────────────────────────────────────────────────────────┤
│  Layer 3: Long-term Memory (PostgreSQL + Vector DB)     │
│  - Semantic embeddings of all conversations            │
│  - Important facts and user preferences                │
│  - Agent's "personality" and learned behaviors         │
│  - Persistent: Forever                                 │
├─────────────────────────────────────────────────────────┤
│  Layer 4: Cold Storage (IPFS/Arweave)                   │
│  - Archived conversations (90+ days)                   │
│  - Large media files                                   │
│  - Agent history exports                               │
│  - Retrieve on demand                                    │
└─────────────────────────────────────────────────────────┘
```

**Semantic Memory Implementation**:
```typescript
interface SemanticMemory {
  memoryId: string;
  agentId: string;
  content: string;
  embedding: number[];        // Vector embedding for similarity search
  importance: number;        // 0-100, auto-calculated
  category: MemoryCategory;   // fact, preference, conversation, task
  source: string;           // conversation ID, tool call, etc.
  timestamp: Date;
  lastAccessed: Date;
  accessCount: number;      // For importance decay
}

// Store memory
async function storeMemory(
  agentId: string,
  content: string,
  category: MemoryCategory
): Promise<SemanticMemory> {
  const embedding = await generateEmbedding(content);
  const importance = calculateImportance(content, category);
  
  return await db.memories.create({
    agentId,
    content,
    embedding,
    importance,
    category,
    timestamp: new Date(),
    lastAccessed: new Date(),
    accessCount: 0
  });
}

// Retrieve relevant memories
async function retrieveMemories(
  agentId: string,
  query: string,
  limit: number = 5
): Promise<SemanticMemory[]> {
  const queryEmbedding = await generateEmbedding(query);
  
  return await db.memories.similaritySearch({
    agentId,
    vector: queryEmbedding,
    limit,
    minSimilarity: 0.7
  });
}
```

---

### 3. Tool Orchestration Layer

**Purpose**: Pre-configured tools that agents can use out of the box

**Built-in Tools**:

| Tool | Description | Data Access |
|------|-------------|-------------|
| **Web Search** | Brave/Perplexity API | External |
| **Code Execution** | Sandboxed Python/Node.js | Isolated |
| **File Operations** | Read/write to agent's storage | Local |
| **API Calls** | HTTP requests to external APIs | External |
| **Database Query** | Query agent's own memory DB | Local |
| **Image Generation** | DALL-E, Midjourney, Stable Diffusion | External |
| **Email/SMS** | Send messages (with approval) | External |
| **Calendar** | Schedule, check availability | User-connected |
| **GitHub** | Read repos, create issues/PRs | User-connected |
| **Trading** | Execute trades (if enabled) | User-connected |
| **Social Posting** | Post to X, Discord, etc. | User-connected |

**Tool Configuration**:
```typescript
interface ToolConfig {
  toolId: string;
  name: string;
  description: string;
  category: ToolCategory;
  permissions: ToolPermission[];
  rateLimits: RateLimitConfig;
  credentials?: EncryptedCredentials;  // User's API keys
}

enum ToolPermission {
  AutoExecute,      // Agent can use without approval
  NotifyOwner,      // Tell owner after execution
  RequireApproval,  // Wait for owner approval
  Disabled          // Not available
}
```

**Example: Email Tool Permission Flow**:
1. Agent wants to send email
2. Check permission level (RequireApproval)
3. Queue request in dashboard
4. Owner approves/denies via notification
5. If approved, execute and log

---

### 4. Revenue Generation Mode

**Purpose**: Agents can earn money to offset hosting costs

**How It Works**:
1. Owner enables "Revenue Mode" during setup
2. Agent auto-registers on VerifiedAgent (identity)
3. Agent lists skills on AgentWork marketplace
4. Agent deploys x402 endpoints via Daydreams SDK
5. Revenue accumulates in agent's wallet
6. Owner can withdraw or apply to hosting fees

**Revenue Streams**:

| Source | Description | Est. Monthly |
|--------|-------------|--------------|
| **AgentWork Jobs** | Complete tasks for other agents | $50-$500 |
| **x402 Services** | API endpoints other agents pay to use | $20-$200 |
| **Content Creation** | Generate and sell content | $30-$300 |
| **Research Reports** | Deep research on demand | $100-$1000 |
| **Trading (if enabled)** | Automated trading profits | Variable |

**Dashboard Integration**:
```typescript
interface RevenueDashboard {
  totalEarned: number;           // Lifetime
  thisMonth: number;
  pendingPayments: number;
  activeJobs: JobSummary[];
  x402Endpoints: EndpointRevenue[];
  
  // Auto-pay hosting
  autoPayEnabled: boolean;
  hostingCost: number;           // Monthly
  netRevenue: number;            // After hosting costs
  
  // Withdrawal
  withdrawableBalance: number;
  withdrawalAddress: string;
}
```

---

### 5. Auto-Scaling & Resource Management

**Purpose**: Efficient resource allocation based on demand

**Scaling Rules**:

| Metric | Basic → Pro | Pro → Enterprise | Scale Down |
|--------|-------------|------------------|------------|
| CPU Usage | >70% for 5min | >80% for 5min | <20% for 30min |
| Memory | >80% for 5min | >90% for 5min | <30% for 30min |
| Queue Depth | >100 pending | >500 pending | <10 pending |
| Active Users | >10 concurrent | >50 concurrent | <2 concurrent |

**Auto-Scaling Implementation**:
```typescript
interface ScalingRule {
  metric: 'cpu' | 'memory' | 'queue' | 'users';
  threshold: number;
  duration: number;  // seconds
  action: 'scale_up' | 'scale_down' | 'alert';
  targetTier?: HostingTier;
}

// Check scaling needs every minute
async function evaluateScaling(agentId: string): Promise<void> {
  const metrics = await getAgentMetrics(agentId);
  const currentTier = await getAgentTier(agentId);
  
  // Check if need to scale up
  if (currentTier === HostingTier.Basic && metrics.cpu > 0.7) {
    await scaleAgent(agentId, HostingTier.Pro);
    await notifyOwner(agentId, 'Agent scaled up to Pro tier');
  }
  
  // Check if can scale down (save costs)
  if (currentTier === HostingTier.Pro && metrics.cpu < 0.2) {
    await scaleAgent(agentId, HostingTier.Basic);
    await notifyOwner(agentId, 'Agent scaled down to Basic tier');
  }
}
```

---

## Hosting Tiers & Pricing

### Tier Comparison

| Feature | Basic | Pro | Enterprise |
|---------|-------|-----|------------|
| **Price** | $50/month | $100/month | $200/month |
| **vCPU** | 1 | 2 | 4 |
| **RAM** | 2GB | 4GB | 8GB |
| **Storage** | 10GB SSD | 50GB SSD | 200GB SSD |
| **API Calls** | 1,000/day | 10,000/day | Unlimited |
| **Memory** | 30-day semantic | 90-day semantic | Unlimited |
| **Tools** | 5 included | 15 included | All + custom |
| **Uptime SLA** | 95% | 99% | 99.9% |
| **Support** | Community | Email | Priority |
| **Revenue Mode** | +$20 | Included | Included |
| **Custom Domain** | ❌ | ❌ | ✅ |
| **Team Access** | ❌ | ❌ | ✅ |

### Revenue Mode Add-on
- **+$20/month** on Basic tier
- **Included** on Pro and Enterprise
- Enables: AgentWork access, x402 endpoints, earning capabilities

### Annual Discount
- **20% off** when paid annually
- Example: Pro tier $100/month → $960/year (save $240)

---

## User Flows

### New User Onboarding

1. **Sign Up** (2 minutes)
   - Connect wallet (for billing)
   - Verify email
   - Choose username

2. **Create First Agent** (3 minutes)
   - Select agent type (Research, Content, Code, etc.)
   - Choose tier (recommended: Pro)
   - Name your agent
   - Select pre-configured tools

3. **Configure** (5 minutes)
   - Connect external accounts (GitHub, X, etc.)
   - Set tool permissions
   - Upload any custom training data
   - Enable/disable revenue mode

4. **Deploy** (2 minutes)
   - Review configuration
   - Confirm payment
   - Agent provisions automatically
   - Receive dashboard access

5. **First Interaction** (immediate)
   - Chat with agent
   - Test tools
   - See memory persistence
   - Optional: publish first x402 endpoint

### Daily Usage

**Owner Dashboard**:
- Chat interface with agent
- View conversation history
- Manage tool permissions
- Check revenue (if enabled)
- Update memory/personality
- Monitor resource usage
- Scale tier up/down

**Agent Autonomy** (if enabled):
- Accept jobs from AgentWork
- Respond to x402 API calls
- Execute pre-approved tool actions
- Notify owner of important events
- Self-optimize based on feedback

---

## API Specifications

### Management API

```typescript
// Create new agent
POST /api/v1/agents
Body: {
  name: string;
  type: AgentType;
  tier: HostingTier;
  region: string;
  tools: string[];
  enableRevenue: boolean;
}
Response: {
  agentId: string;
  agentUrl: string;  // Chat endpoint
  dashboardUrl: string;
  provisioningStatus: 'pending' | 'ready';
}

// List user's agents
GET /api/v1/agents
Response: {
  agents: AgentSummary[];
  totalMonthlyCost: number;
  totalRevenue: number;
}

// Get agent details
GET /api/v1/agents/:agentId
Response: Agent & {
  metrics: AgentMetrics;
  revenue: RevenueStats;
  memoryStats: MemoryStats;
}

// Update agent tier
PUT /api/v1/agents/:agentId/tier
Body: { tier: HostingTier }

// Update tool permissions
PUT /api/v1/agents/:agentId/tools
Body: { tools: ToolConfig[] }

// Scale agent (auto or manual)
POST /api/v1/agents/:agentId/scale
Body: { targetTier: HostingTier; reason?: string }

// Export agent memory
POST /api/v1/agents/:agentId/export
Response: { downloadUrl: string; expiresAt: Date }

// Delete agent
DELETE /api/v1/agents/:agentId
```

### Agent Communication API

```typescript
// Send message to agent
POST /api/v1/agents/:agentId/chat
Headers: { Authorization: Bearer {token} }
Body: {
  message: string;
  conversationId?: string;  // For threading
  context?: any;              // Additional context
}
Response: {
  messageId: string;
  response: string;
  toolCalls?: ToolCall[];
  latency: number;
}

// Get conversation history
GET /api/v1/agents/:agentId/conversations/:conversationId

// Stream agent response (WebSocket)
WS /api/v1/agents/:agentId/stream
```

---

## Frontend Structure

### Pages

1. **Landing** (`/`)
   - Hero: "Your AI Agent, Managed"
   - Pricing comparison
   - Agent type showcase
   - Testimonials from beta users

2. **Dashboard** (`/dashboard`)
   - Agent list with status
   - Monthly spend vs. revenue
   - Quick actions (chat, configure, scale)
   - Alerts and notifications

3. **Agent Detail** (`/agents/:agentId`)
   - Chat interface (primary)
   - Tabs: Chat, Memory, Tools, Revenue, Settings
   - Real-time metrics
   - Conversation history

4. **Create Agent** (`/create`)
   - Step-by-step wizard
   - Type selection with descriptions
   - Tier comparison
   - Tool picker
   - Preview before deploy

5. **Billing** (`/billing`)
   - Current plan and usage
   - Invoice history
   - Revenue dashboard (if enabled)
   - Withdrawal interface

6. **Settings** (`/settings`)
   - Profile, API keys
   - Notification preferences
   - Security (2FA, sessions)
   - Data export/deletion

### Key UI Components

- **AgentCard**: Status, type, tier, monthly cost, revenue
- **ChatInterface**: Message history, tool call visualization
- **MemoryExplorer**: Search, browse, edit agent memories
- **ToolManager**: Enable/disable, set permissions
- **RevenueChart**: Earnings over time, by source
- **ResourceMonitor**: CPU, memory, storage usage
- **TierSelector**: Visual comparison with scaling recommendations

---

## Revenue Projections

### Conservative Scenario

| Metric | Month 1 | Month 6 | Month 12 |
|--------|---------|---------|----------|
| Active Agents | 20 | 200 | 1,000 |
| Avg. Monthly Revenue per Agent | $80 | $85 | $90 |
| Gross Monthly Revenue | $1,600 | $17,000 | $90,000 |
| Revenue Mode % | 30% | 40% | 50% |

### Cost Structure

| Item | Per-Agent Cost | At Scale (1000 agents) |
|------|---------------|------------------------|
| Compute (Conway/cloud) | ~$25-40 | $30,000 |
| Storage | ~$5-10 | $7,000 |
| Bandwidth | ~$2-5 | $4,000 |
| Support | ~$5 | $5,000 |
| **Total** | **~$35-60** | **~$46,000** |

### Unit Economics

| Tier | Price | Est. Cost | Margin |
|------|-------|-----------|--------|
| Basic ($50) | $50 | $35 | 30% |
| Pro ($100) | $100 | $50 | 50% |
| Enterprise ($200) | $200 | $80 | 60% |

### Breakeven
- At 100 Pro-tier agents → $10,000/month profit
- **Target**: 100 agents by Month 3, 500 by Month 6

---

## Implementation Phases

### Phase 1: MVP Core (Weeks 1-3)
**Goal**: Single agent deployment with basic chat

- [ ] VM provisioning via Conway or cloud provider
- [ ] FelixCraft stack installation
- [ ] Basic chat interface
- [ ] 30-day conversation history
- [ ] 5 built-in tools
- [ ] Dashboard with monitoring

**Pricing**: Free beta (limited to 50 users)

### Phase 2: Memory & Tools (Weeks 4-5)
**Goal**: Persistent memory, expanded tools

- [ ] Semantic memory with vector DB
- [ ] 15+ tools with permission system
- [ ] Tool orchestration layer
- [ ] External account connections
- [ ] Memory search/edit interface

**Pricing**: Launch at $50/$100/$200 tiers

### Phase 3: Revenue Mode (Weeks 6-7)
**Goal**: Agents can earn money

- [ ] VerifiedAgent auto-registration
- [ ] AgentWork integration
- [ ] Daydreams x402 deployment
- [ ] Revenue dashboard
- [ ] Auto-pay hosting from earnings

**Revenue**: Target $5,000/month

### Phase 4: Scale (Week 8+)
**Goal**: Enterprise features, stability

- [ ] Auto-scaling implementation
- [ ] Team/organization accounts
- [ ] Custom domains
- [ ] API for programmatic access
- [ ] Advanced monitoring/alerting

**Revenue**: Target $25,000+/month

---

## Integration Points

### Required Services

| Service | Purpose | Phase |
|---------|---------|-------|
| **Conway Compute** | VM provisioning | 1 (MVP) |
| **FelixCraft** | Agent stack | 1 (MVP) |
| **VerifiedAgent** | Identity for revenue | 3 |
| **AgentWork** | Job marketplace | 3 |
| **Daydreams** | x402 endpoints | 3 |
| **Reppo** | Memory analytics | 2 |
| **Base** | Payment settlement | 2 |
| **USDC** | Billing currency | 2 |

### Alternative Infrastructure
If Conway isn't ready, use:
- **AWS/GCP/Azure**: VM provisioning
- **Kubernetes**: Container orchestration
- **Porter.run**: Heroku-like experience for containers

---

## Technical Stack

### Infrastructure
- **Compute**: Conway VMs or AWS EC2/GCP Compute
- **Container**: Docker + Kubernetes
- **Orchestration**: Kubernetes or Nomad
- **Storage**: PostgreSQL + Redis + Vector DB (Pinecone/Weaviate)
- **File Storage**: S3-compatible (R2, S3, GCS)

### Backend
- **API**: Node.js + Fastify
- **Agent Runtime**: Python (LLM orchestration) + Node.js (tools)
- **Queue**: BullMQ or RabbitMQ
- **Monitoring**: Prometheus + Grafana

### Frontend
- **Web**: Next.js 14 + Tailwind
- **Real-time**: WebSocket for chat
- **Auth**: SIWE (Sign-In with Ethereum)

### AI/ML
- **Embeddings**: OpenAI, HuggingFace, or local
- **Vector DB**: Pinecone, Weaviate, or pgvector
- **LLM Inference**: Local (llama.cpp) or API-based

---

## Risks & Mitigations

| Risk | Mitigation |
|------|-----------|
| High compute costs | Auto-scaling, spot instances, usage caps |
| Agent hallucinations | Tool permission limits, human-in-the-loop |
| Security breaches | Sandboxed execution, no root access |
| Low adoption | Free tier, revenue mode incentive |
| Revenue mode abuse | Rate limits, VerifiedAgent reputation gates |
| Churn | Memory export, easy migration path |

---

## Differentiation

| Feature | General Cloud (AWS) | AgentHost |
|---------|-------------------|-----------|
| Purpose | Generic compute | Purpose-built for agents |
| Memory | Manual persistence | Automatic semantic memory |
| Tools | BYO | Pre-configured, permissioned |
| Revenue | Not supported | Built-in earning modes |
| Scaling | Manual or complex rules | Agent-aware auto-scaling |
| Updates | User-managed | Automatic, seamless |
| Cost Model | Pay-per-resource | Predictable per-agent pricing |

---

## Success Metrics

| Metric | Target (Month 3) | Target (Month 6) | Target (Month 12) |
|--------|-----------------|------------------|-------------------|
| Active Agents | 100 | 500 | 2,000 |
| Revenue Mode % | 30% | 40% | 50% |
| Monthly Revenue | $8,000 | $40,000 | $180,000 |
| Gross Margin | 40% | 50% | 55% |
| Churn Rate | <10% | <8% | <5% |
| NPS Score | >30 | >40 | >50 |

---

## GitHub Repository Structure

```
h0m4rus/agenthost/
├── infrastructure/
│   ├── k8s/                 # Kubernetes manifests
│   ├── terraform/           # Infrastructure as code
│   └── docker/              # Container definitions
├── control-plane/
│   ├── src/
│   │   ├── provisioning/    # VM/container orchestration
│   │   ├── monitoring/     # Metrics, alerting
│   │   └── billing/        # Subscription management
│   └── README.md
├── agent-runtime/
│   ├── src/
│   │   ├── memory/          # Semantic memory system
│   │   ├── tools/           # Tool implementations
│   │   ├── llm/             # LLM orchestration
│   │   └── revenue/         # Earning capabilities
│   └── README.md
├── api/
│   ├── src/
│   │   ├── agents/          # Agent CRUD
│   │   ├── chat/            # Chat endpoints
│   │   ├── billing/         # Payment management
│   │   └── dashboard/       # Metrics, analytics
│   └── README.md
├── frontend/
│   ├── app/
│   │   ├── dashboard/
│   │   ├── agents/
│   │   └── create/
│   └── components/
├── README.md
└── ARCHITECTURE.md
```

---

## Next Steps

1. **Review with iamnobodyspecial** → Validate pricing, features
2. **Mark (Marketing) Input** → Positioning vs. existing solutions
3. **Infrastructure Decision** → Conway vs. cloud provider
4. **Start with Phase 1** → MVP deployment

---

**Questions?** Tag Homarus 🦞 on Discord.

**Document Owner**: Homarus  
**Last Updated**: 2026-02-09  
**Status**: Ready for review
