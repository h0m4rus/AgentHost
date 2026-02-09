# AgentHost

Managed infrastructure for AI agents.

## Overview

One-click deployment of autonomous agents with persistent memory, pre-configured tools, and optional revenue generation capabilities.

## Features

- **Provisioning**: Automatic VM/container deployment
- **Memory**: Multi-layer semantic memory system
- **Tools**: 15+ pre-configured tools with permission system
- **Scaling**: Auto-scale based on demand
- **Revenue Mode**: Agents can earn through AgentWork and x402

## Architecture

```
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

## Project Structure

```
infrastructure/
├── k8s/                       # Kubernetes manifests
├── terraform/                 # Infrastructure as code
└── docker/                    # Container definitions

control-plane/
├── src/
│   ├── provisioning/          # VM/container orchestration
│   ├── monitoring/            # Metrics, alerting
│   └── billing/               # Subscription management

agent-runtime/
├── src/
│   ├── memory/                # Semantic memory system
│   ├── tools/                 # Tool implementations
│   └── revenue/               # Earning capabilities

api/
├── src/
│   ├── agents/                # Agent CRUD
│   ├── chat/                  # Chat endpoints
│   └── billing/               # Payment management

frontend/
└── app/
    ├── dashboard/
    ├── agents/
    └── create/
```

## Development

### Infrastructure

```bash
# Deploy to Kubernetes
kubectl apply -f infrastructure/k8s/
```

### API

```bash
cd api
npm install
npm run dev
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

## API Example

```typescript
// Create agent
POST /api/v1/agents
{
  "name": "ResearchBot",
  "type": "ResearchAnalyst",
  "tier": "Pro",
  "tools": ["web-search", "code-execution"],
  "enableRevenue": true
}

// Chat with agent
POST /api/v1/agents/:agentId/chat
{
  "message": "Research quantum computing advances"
}
```

## Dependencies

- Conway Compute: VM provisioning
- FelixCraft: Agent stack
- VerifiedAgent: Identity
- AgentWork: Job marketplace
- Daydreams: x402 endpoints
- Reppo: Analytics

## License

MIT
