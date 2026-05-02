# Tracer

**The reliability and proof layer for autonomous AI agents.**

Tracer captures every LLM decision, tool call, onchain transaction, and KeeperHub execution into a structured, inspectable, shareable trace. Teams can debug executions, share verifiable evidence, and understand exactly what their agent did — and why.

Built for ETHGlobal and the **KeeperHub Prize**.

---

## The Problem

Autonomous agents can execute powerful actions — LLM reasoning, smart contract calls, token transfers, multi-step workflows — but there's still a fundamental trust gap:

- Logs are scattered across runtime outputs, RPC providers, and LLM APIs
- No single place exists to see what happened across an entire execution
- Debugging relies on hope and console.log
- Sharing evidence with operators, partners, or auditors is manual and error-prone

Teams can't answer: *"What exactly happened, why, and can I prove it?"*

---

## What Tracer Does

Tracer wraps your existing agent runtime with a lightweight SDK and turns every execution into a structured, verifiable trace.

```
Agent Runtime → SDK Capture → KeeperHub Execution → Tracer Trace → Verifiable Evidence
```

Each trace contains:
- Full event timeline (LLM calls, tool calls, EVM transactions, KeeperHub executions)
- Payload-level inspection for every event
- KeeperHub execution IDs, retries, and reliability status
- On-chain anchor hash for proof
- Shareable public link with copy-ready token

---

## Architecture

```
┌──────────────────────────────────────────────┐
│             Agent Runtime                     │
│   (OpenAI / Anthropic / Vercel AI / viem)     │
└────────────────┬─────────────────────────────┘
                 │  @tracerlabs/sdk wraps all calls
                 ▼
┌──────────────────────────────────────────────┐
│          Tracer Ingest API                    │
│    Captures events → Postgres via Prisma      │
└────────────────┬─────────────────────────────┘
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
  Anchor Worker       Enrichment Worker
  (EVM anchor hash)   (AI analysis, gas enrichment)
       │
       ▼
┌──────────────────────────────────────────────┐
│          KeeperHub Execution Layer            │
│    Reliable onchain execution + status sync   │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│          Tracer Dashboard                     │
│    Console · Traces · Inspector · Evidence    │
└──────────────────────────────────────────────┘
```

---

## Monorepo Structure

```
apps/
  dashboard/        Next.js 15 frontend — console, register, trace detail
  server/           Next.js API server — tRPC routers, Privy auth
  ingest/           Ingest API — receives SDK events
  anchor-worker/    Anchors trace hashes to EVM on a schedule
  enrichment-worker/ AI analysis + gas cost enrichment

packages/
  sdk/              @tracerlabs/sdk — wraps OpenAI, Anthropic, Vercel AI, LangChain, viem
  db/               Prisma schema + client
  shared/           Shared types (Trace, TraceEvent, EvmTxPayload, etc.)
```

---

## SDK Integration

### Install

```bash
npm install @tracerlabs/sdk
```

### Environment variables

```env
TRACER_API_KEY=your_api_key
TRACER_AGENT_ID=your_agent_id
TRACER_VERIFY_TOKEN=your_verify_token
TRACER_CHAIN_ID=84532
```

### OpenAI

```typescript
import OpenAI from "openai"
import { Tracer } from "@tracerlabs/sdk"

const tracer = new Tracer({
  apiKey: process.env.TRACER_API_KEY!,
  agentId: process.env.TRACER_AGENT_ID!,
  chainId: parseInt(process.env.TRACER_CHAIN_ID!, 10),
  verifyToken: process.env.TRACER_VERIFY_TOKEN,
})

const openai = tracer.wrapOpenAI(new OpenAI({ apiKey: process.env.OPENAI_API_KEY! }))
```

### Anthropic

```typescript
import Anthropic from "@anthropic-ai/sdk"

const anthropic = tracer.wrapAnthropic(
  new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! })
)
```

### Vercel AI SDK

```typescript
import { openai } from "@ai-sdk/openai"

const model = tracer.wrapLanguageModel(openai("gpt-4.1"))
```

### LangChain

```typescript
import { ChatOpenAI } from "@langchain/openai"

const llm = new ChatOpenAI({
  model: "gpt-4.1",
  callbacks: [tracer.langchainHandler()],
})
```

### viem (onchain actions)

```typescript
const walletClient = tracer.wrapWalletClient(createWalletClient({ ... }))
const publicClient = tracer.wrapPublicClient(createPublicClient({ ... }))
```

---

## KeeperHub Integration

Tracer treats **KeeperHub as the execution layer** for reliable onchain automation.

When an agent triggers a KeeperHub workflow or direct contract call, Tracer:

1. Captures the execution ID, status, retry count, and settlement evidence inside the event timeline
2. Surfaces KeeperHub reliability status inline in the trace detail view
3. Lets operators re-trigger executions from the trace detail page with full context
4. Auto-refreshes execution status for 45 seconds after trigger

This means every KeeperHub execution becomes **traceable, inspectable, and shareable** — exactly matching KeeperHub's vision of reliable, transparent onchain agent execution.

---

## Dashboard Features

### Agent Console
- Overview of all registered agents
- KeeperHub reliability scorecard per agent
- Quick navigation to traces, settings, and registration

### Register Agent
- One-step agent creation with Privy authentication
- Instant credential generation (API key, verify token, chain ID)
- Copy-ready environment variables and SDK snippets
- Multi-framework integration tabs: OpenAI, Anthropic, Vercel AI, LangChain

### Trace Detail View
- Horizontal metadata grid: status, duration, gas, cost, anchor, share
- Horizontal event timeline scroller with per-card inspector
- KeeperHub execution status inline
- Share trace as a public verifiable link
- On-chain anchor hash with block explorer link
- AI analysis with rerun capability

### Agent Settings
- Update display name, chain, environment, retention
- Rotate API key (invalidates previous immediately)
- Delete agent and all associated traces

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15, React 19, Tailwind CSS |
| Backend | Next.js API routes, tRPC |
| Auth | Privy.io (JWT-based, wallet-aware) |
| Database | PostgreSQL + Prisma |
| Monorepo | pnpm workspaces + Turborepo |
| SDK | TypeScript, wraps OpenAI / Anthropic / Vercel AI / LangChain / viem |
| Execution | KeeperHub (reliable onchain automation layer) |
| Proof | EVM anchor hash, on-chain Merkle proof |
| AI | OpenAI GPT-4.1 for trace analysis |

---

## Local Development

### Prerequisites

- Node.js ≥ 22
- pnpm ≥ 9.15
- PostgreSQL database

### Setup

```bash
# Clone and install
git clone https://github.com/Madhav-Gupta-28/Tracer.git
cd Tracer
pnpm install

# Configure environment
cp apps/dashboard/.env.example apps/dashboard/.env.local
cp apps/server/.env.example apps/server/.env.local
# Fill in: DATABASE_URL, NEXT_PUBLIC_PRIVY_APP_ID, PRIVY_APP_ID, PRIVY_APP_SECRET
# Optional KeeperHub: KEEPERHUB_API_KEY

# Generate Prisma client
pnpm -C packages/db prisma generate

# Run migrations
pnpm -C packages/db prisma migrate dev

# Start frontend + backend
pnpm -C apps/dashboard dev       # http://localhost:3000
PORT=3001 pnpm -C apps/server dev  # http://localhost:3001
```

---

## Why This Wins

### For KeeperHub Prize

Tracer doesn't compete with KeeperHub — it extends it.

KeeperHub handles reliable onchain execution. Tracer wraps that execution in a full proof and observability layer: every KeeperHub execution becomes traceable, inspectable, and verifiable without any extra developer effort. This strengthens the KeeperHub ecosystem by making every execution auditable and shareable.

Key integration points:
- KeeperHub execution IDs sync to the trace event timeline automatically
- Reliability status and retry counts surface inline in the inspector
- Operators can re-trigger KeeperHub runs with full context from the trace detail page
- KeeperHub + Tracer together = autonomous execution that's both reliable and provable

### For ETHGlobal

Autonomous agents operating onchain need more than execution reliability — they need provable execution history. Tracer solves the "trust gap" that currently blocks enterprise and institutional adoption of AI agents in DeFi, DAO tooling, and onchain automation.

The SDK is open-source, framework-agnostic, and production-ready. Tracer is designed to be the proof layer that any AI agent developer reaches for when reliability and transparency matter.

---

## License

MIT

---

Built with ❤️ at ETHGlobal. Powered by KeeperHub.
