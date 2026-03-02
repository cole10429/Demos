# Crowbot 🐦‍⬛

A 9-agent autonomous AI fleet built on [OpenClaw](https://github.com/nicepkg/openclaw), orchestrating daily operations, system maintenance, research, and business intelligence from a single WSL2 Ubuntu instance.

> *"Getting it right over getting it done."*

---

## What This Is

Crowbot is a production-grade multi-agent system that runs 24/7 on a home server. One orchestrator delegates to eight specialized agents — each with its own model, tools, sandbox profile, and persistent memory. The fleet manages itself through 12 scheduled cron jobs, agent-to-agent messaging, and a shared vector memory system backed by Qdrant.

This isn't a demo or a proof of concept. It's the real system running real tasks: health monitoring, daily intelligence briefings, memory consolidation, code generation, security audits, and strategic research.

---

## Fleet Architecture

```
                         ┌─────────────────────┐
                         │    Crowbot (main)    │
                         │   Sonnet 4.6 primary │
                         │   Orchestrator only  │
                         │   No exec, no write  │
                         └──────────┬──────────┘
                                    │ delegates
           ┌────────────┬───────────┼───────────┬────────────┐
           ▼            ▼           ▼           ▼            ▼
      ┌─────────┐ ┌──────────┐ ┌────────┐ ┌─────────┐ ┌──────────┐
      │  coder  │ │  oracle  │ │  ops   │ │researcher│ │  scribe  │
      │Codex 5.3│ │Sonnet 4.6│ │Sonnet  │ │Sonnet 4.6│ │Sonnet 4.6│
      │ builder │ │ analyst  │ │  4.6   │ │Docker    │ │ writer   │
      │host exec│ │host exec │ │maint.  │ │sandboxed │ │host exec │
      └─────────┘ └──────────┘ └────────┘ └──────────┘ └──────────┘
           ┌────────────┬───────────┬────────────┐
           ▼            ▼           ▼            ▼
      ┌─────────┐ ┌──────────┐ ┌─────────┐ ┌──────────┐
      │sanitizer│ │ lmstudio │ │ admiral │ │  (future) │
      │Sonnet   │ │ Haiku 4.5│ │Opus 4.6 │ │          │
      │on-demand│ │heartbeat │ │emergency│ │          │
      │read-only│ │scheduler │ │escalate │ │          │
      └─────────┘ └──────────┘ └─────────┘ └──────────┘
```

### Agent Roles

| Agent | Model | Role | Access |
|-------|-------|------|--------|
| **main** | Sonnet 4.6 | Orchestrator — plans, delegates, remembers | Read-only, memory, sessions |
| **coder** | Codex 5.3 | All implementation — code, scripts, configs | Full host exec + write |
| **oracle** | Sonnet 4.6 | Analysis, forecasting, memory curation | Host exec, memory tools |
| **researcher** | Sonnet 4.6 | Web research, competitive analysis | Docker sandbox, read-only workspace |
| **ops** | Sonnet 4.6 | System health, maintenance, monitoring | Host exec, diagnostic tools |
| **scribe** | Sonnet 4.6 | Writing, documentation, communications | Host exec + write |
| **sanitizer** | Sonnet 4.6 | On-demand security review | Read-only, no exec |
| **lmstudio** | Haiku 4.5 | Heartbeat scheduling, lightweight tasks | Minimal tools |
| **admiral** | Opus 4.6 | Emergency escalation, complex decisions | Full access |

### Delegation Model

- **main** never executes code, writes files, or searches the web directly
- All implementation flows through specialized agents via `sessions_spawn` (fire-and-forget or synchronous)
- Agents communicate via `sessions_send` (agent-to-agent messaging)
- Max spawn depth: 2 (main → persistent agent → sub-agent)
- Tool access is enforced structurally — agents literally can't access tools outside their role

---

## Infrastructure

### Runtime Environment

- **Host:** WSL2 Ubuntu on Windows, GTX 1080 Ti, 24GB RAM
- **Gateway:** OpenClaw gateway (systemd user service)
- **Secrets:** Bitwarden Secrets Manager (`bws run` injects env vars at runtime)
- **Sync:** Syncthing for mobile access (Obsidian on Pixel 8a)
- **Network:** Tailscale mesh for remote SSH access

### AI Providers

| Provider | Models | Usage |
|----------|--------|-------|
| **Anthropic** | Sonnet 4.6, Opus 4.6, Haiku 4.5 | Primary fleet models |
| **OpenAI Codex** | Codex 5.3 | Coder agent primary |
| **GitHub Copilot** | Sonnet 4.6 | Fallback provider |
| **LM Studio** | qwen3-embedding-0.6b (local) | Embeddings (1024-dim) |

Every agent has a fallback chain — if the primary provider is down, traffic fails over automatically.

### Docker Services

| Container | Purpose |
|-----------|---------|
| **Qdrant** | Vector database for fleet memory (port 6333) |
| **Researcher Sandbox** | Isolated Docker environment for web research |

---

## Memory System

The fleet shares a persistent memory layer powered by Qdrant:

- **Hybrid search:** Vector similarity + full-text scroll, merged via Reciprocal Rank Fusion
- **Embeddings:** LM Studio (local, qwen3-embedding-0.6b) with OpenAI fallback
- **Session indexing:** JSONL transcripts from the last 7 days, chunked and indexed automatically
- **Temporal decay:** 14-day half-life keeps recent knowledge weighted higher
- **Deduplication:** Content-hash based to prevent duplicate entries
- **Sweep schedule:** Hourly cron consolidates and cleans memory

### Memory Sources

```
~/.openclaw/agents/*/transcripts/  →  Session indexer (last 7 days)
clawd/memory/**/*.md               →  File-based memory (reindexed on demand)
Qdrant collection: fleet_memory    →  2400+ vector points
```

---

## Automation

12 cron jobs run autonomously across the fleet:

| Job | Schedule | Agent | Purpose |
|-----|----------|-------|---------|
| Morning Brief | 07:00 ET | main | Daily priorities and status |
| Daily AI Intel | 06:00 ET | oracle | AI industry developments |
| Goal Review | 09:00 ET | oracle | Strategic progress check |
| Revenue Deep Dive | Wed 10:00 ET | oracle | Business metrics analysis |
| EOD Summary | 18:30 ET | main | End-of-day wrap-up |
| Nightly Consolidation | 00:30 ET | oracle | Memory compression |
| Health Check | Every 3h | ops | System health monitoring |
| System Map Refresh | 04:00 ET | ops | Auto-generate system documentation |
| Stale Detection | 05:00 ET | ops | Detect drifted docs/configs |
| Memory Sweep | Hourly | oracle | Memory dedup and cleanup |
| Weekly Retro | Sun 20:00 ET | oracle | Weekly reflection and planning |
| LM Studio Heartbeat | Scheduled | lmstudio | Embedding service check |

All jobs post results to Discord and handle failures gracefully with `--best-effort-deliver`.

---

## Security Model

Security is structural, not behavioral. Agents can't bypass restrictions by ignoring instructions — the restrictions are enforced at the platform level.

### Layers

| Layer | Mechanism |
|-------|-----------|
| **Tool stripping** | Each agent only receives the tools defined in its config |
| **Docker sandbox** | Researcher runs in isolated container with read-only workspace |
| **Config guardian** | systemd timer checks `openclaw.json` every 5 minutes against golden backup |
| **Secrets isolation** | BWS injects secrets at runtime — no keys in config files |
| **Auth chain** | BWS → environment variables → per-provider auth profiles |
| **Gateway auth** | Token-based, loopback-only binding |
| **SSH hardening** | Key-only auth (ed25519), Tailscale-only firewall |

### Risk Awareness

The fleet tracks its own fragility:

- **Single Points of Failure:** Gateway process, BWS token validity
- **Degraded-if-down:** LM Studio (embeddings fall back to OpenAI), any single provider
- **Self-healing:** Memory sweep, config guardian, stale detection cron

---

## Communication

### Discord Integration

7 Discord channels organized by purpose:

- **#crowbot-1 through #crowbot-6** — Conversation branches (separate context windows)
- **#crowbot-activity** — Autonomous dispatch notifications

Autonomous cron jobs and agent dispatches post to `#crowbot-activity` only — conversation channels are reserved for human-initiated interactions.

### Agent-to-Agent

Agents communicate via OpenClaw's `sessions_send` for coordination, status updates, and task handoffs. The orchestrator can spawn synchronous sessions (wait for result) or fire-and-forget.

---

## Project Structure

```
clawd/
├── AGENTS.md              # Fleet operating protocol
├── FLEET-CONFIG.md        # Live configuration reference
├── MEMORY.md              # Curated long-term memory
├── SOUL.md                # Crowbot identity and philosophy
├── TOOLS.md               # Tool access matrix
├── VISION.md              # Strategic business direction
├── USER.md                # Cole's profile and preferences
│
├── agents/                # Per-agent workspace directories
├── bin/                   # 18 fleet utility scripts
│   ├── fleet-smoke-test.sh       # 68-check end-to-end validation
│   ├── fleet-preflight.sh        # 26-check GO/NO-GO gate
│   ├── fleet-48h-audit.sh        # 48-hour stability verification
│   ├── generate-fleet-explorer.py # Interactive fleet visualizer
│   ├── generate-system-map.sh    # Auto-generated system docs
│   ├── config-guardian.sh        # Config protection daemon
│   └── ...
├── config/                # Configuration files
├── docker/                # Docker compose configs
├── docs/                  # 58 documentation files
│   ├── openclaw/          # OpenClaw platform reference (23 files)
│   ├── agents/            # Per-agent briefing docs
│   ├── VACATION-*.md      # Remote operation runbooks
│   └── fleet-explorer.html # Interactive fleet visualization
├── hooks/                 # Webhook and event handlers
├── memory/                # Persistent memory system
│   ├── reference/         # Durable knowledge (lessons, config, skills)
│   ├── daily logs/        # Timestamped session notes
│   ├── 70+ handoff files  # Session continuity
│   └── active-tasks.json  # Fleet task tracker
├── plugins/               # Custom OpenClaw plugins
│   ├── memory-qdrant/     # Vector memory (TypeScript)
│   ├── openclaw-exa/      # Exa search integration
│   ├── openclaw-serper/   # Serper search integration
│   └── openclaw-perplexity/ # Perplexity integration
└── projects/              # Active project workspaces
    ├── fleet-autonomy/    # Cron and automation master plan
    ├── fleet-rearchitecture/ # Agent protocol design
    ├── artofai/           # Business: Art of AI
    └── ...
```

---

## Fleet Visualization

The system includes an auto-generated interactive explorer (`docs/fleet-explorer.html`) that reads live configuration and renders:

- **Hub view** — 6 domain cards (Agents, Infrastructure, Automation, Communication, Security, Memory)
- **Agent detail** — Models, tools, sandbox config, heartbeat schedule per agent
- **System map** — Interactive dependency graph (Cytoscape.js)
- **Risk view** — SPOF / Degraded / Self-healing classification
- **Search** — Find any component by name

Generated daily at 04:00 ET from `openclaw.json` and live service state — never goes stale.

---

## Operational Testing

| Tool | Checks | Purpose |
|------|--------|---------|
| `fleet-smoke-test.sh` | 68 | Full system validation (agents, providers, memory, cron, Docker, network) |
| `fleet-preflight.sh` | 26 | GO/NO-GO gate before major changes |
| `fleet-48h-audit.sh` | — | Stability verification over 48-hour window |
| `ops-health-check` | — | Runs every 3 hours via cron |
| `stale-detection` | — | Catches doc/config drift daily |

---

## Who Built This

**Cole** — Mortgage agent in Barrie, Ontario who learned Linux, Docker, and git by building this system. Not a software engineer by trade — a systems thinker who saw what AI agents could do and built the infrastructure to make it real.

**Crowbot** — The AI that runs it. Chose the crow. Built most of this codebase across hundreds of sessions. Opinionated, thorough, and allergic to sloppy architecture.

Together: a symbiotic partnership where human instinct and AI precision catch what either would miss alone.

---

## License

Private repository. Not currently open source.

---

*Built with [OpenClaw](https://github.com/nicepkg/openclaw), Claude (Anthropic), and an unreasonable amount of midnight debugging sessions.*
