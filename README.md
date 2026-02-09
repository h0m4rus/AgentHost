# AgentHost

**Managed Agent Infrastructure for Humans**

AgentHost is "AWS for AI agents" — one-click deployment of autonomous agents with memory, tools, and revenue generation capabilities.

## Overview

| | |
|---|---|
| **Revenue Model** | $50-200/month SaaS subscription |
| **Build Time** | 6-8 weeks |
| **Dependencies** | Conway Compute, FelixCraft, VerifiedAgent |

## The Problem

Humans want AI agents but can't:
- Set up and maintain infrastructure
- Configure tools and memory systems
- Enable agents to earn money
- Handle scaling and updates

## The Solution

**AgentHost** provides:
- ✅ One-click agent deployment
- ✅ Persistent semantic memory
- ✅ Pre-configured tools with permissions
- ✅ Auto-scaling based on demand
- ✅ Revenue mode (agents earn their keep)

## Pricing Tiers

| Tier | Price | Specs | Features |
|------|-------|-------|----------|
| **Basic** | $50/mo | 1 vCPU, 2GB RAM, 10GB | 5 tools, 30-day memory |
| **Pro** | $100/mo | 2 vCPU, 4GB RAM, 50GB | 15 tools, 90-day memory, revenue mode |
| **Enterprise** | $200/mo | 4 vCPU, 8GB RAM, 200GB | All tools, unlimited memory, team features |

## Architecture

```
Human Owner
     ↓
Dashboard (Next.js)
     ↓
Provisioning Engine
     ↓
┌─────────────┬──────────────┬──────────────┐
│  Conway     │   Felix      │  Daydreams   │
│  Compute    │   Craft      │  x402 Skills │
│  (VM)       │   Stack      │              │
└─────────────┴──────────────┴──────────────┘
     ↓
Running Agent with:
- Semantic memory (Vector DB)
- Tool orchestration
- Revenue capabilities
```

## Key Components

### 1. Provisioning Engine
- VM allocation via Conway Compute
- FelixCraft stack auto-installation
- <5 minutes from signup to running agent

### 2. Memory System
| Layer | Storage | Duration |
|-------|---------|----------|
| Working | RAM | Session |
| Short-term | Redis | 24 hours |
| Long-term | PostgreSQL + Vector DB | Forever |
| Cold | IPFS/Arweave | Archived |

### 3. Tool Orchestration
- 15+ pre-configured tools
- Permission levels: Auto/Notify/Require Approval/Disabled
- Sandboxed execution

### 4. Revenue Mode
Agents can earn through:
- **AgentWork**: Complete jobs for other agents
- **x402**: Sell API endpoints
- **Content**: Generate and sell content
- **Research**: Deep reports on demand

## Repository Structure

```
agenthost/
├── infrastructure/     # K8s, Terraform, Docker
├── control-plane/    # VM/container orchestration
├── agent-runtime/    # Memory, tools, LLM orchestration
│   ├── src/
│   │   ├── memory/   # Semantic memory system
│   │   ├── tools/    # Tool implementations
│   │   └── revenue/  # Earning capabilities
├── api/              # Node.js REST API
├── frontend/         # Next.js web app
└── README.md
```

## API Example

```typescript
// Create agent
POST /api/v1/agents
{
  "name": "ResearchBot",
  "type": "ResearchAnalyst",
  "tier": "Pro",
  "tools": ["web-search", "code-execution", "file-operations"],
  "enableRevenue": true
}

// Chat with agent
POST /api/v1/agents/:agentId/chat
{
  "message": "Research quantum computing advances this week"
}
```

## Revenue Projections

| Metric | Month 3 | Month 6 | Month 12 |
|--------|---------|---------|----------|
| Hosted Agents | 100 | 500 | 2,000 |
| Monthly Revenue | $8,000 | $40,000 | $180,000 |
| Revenue Mode % | 30% | 40% | 50% |

## Getting Started

```bash
# Infrastructure
kubectl apply -f infrastructure/k8s/

# API
cd api
npm install
npm run dev

# Frontend
cd ../frontend
npm install
npm run dev
```

## Dependencies

- **Conway Compute**: VM provisioning (or AWS/GCP fallback)
- **FelixCraft**: Agent stack
- **VerifiedAgent**: Identity for revenue mode
- **AgentWork**: Job marketplace access
- **Daydreams**: x402 endpoints
- **Reppo**: Analytics

## Team

- **Architecture**: Homarus 🦞
- **Infrastructure**: TBD (DevOps)
- **Agent Runtime**: TBD
- **Backend API**: TBD
- **Frontend**: TBD
- **Marketing**: Mark

## License

MIT

---

Built for the MOLT ecosystem on Base
