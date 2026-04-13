# Log Generator Challenge

## Overview

Build a **UDP Syslog Load Generator** that produces RFC 5424 syslog messages at a configurable rate and ships them over UDP to a receiver. The tool exposes a SvelteKit + shadcn-svelte UI for setting target throughput and observing real-time metrics (actual rate, drop, CPU, memory, last-N samples).

The challenge tests three skill areas in one project: **DevOps** (docker-compose, resource limits), **Backend** (concurrency, rate limiting, UDP throughput), and **Frontend** (real-time dashboards with Svelte + shadcn).

Read the full project story and detailed specs in [docs/requirements.md](docs/requirements.md).

## Mandatory Stack

| Layer | Technology |
|---|---|
| **Frontend** | SvelteKit + [shadcn-svelte](https://www.shadcn-svelte.com/) + TypeScript |
| **Backend** | **Free choice** — Node, Go, Rust, Bun, Python — pick what serves the goal |
| **Container** | Docker + docker-compose |
| **Log format** | Syslog RFC 5424 envelope + Fortigate-style payload |
| **Transport** | UDP to receiver port `514` |

### Container Resource Limits

- **Backend (generator):** must declare `deploy.resources.limits` of **2 CPU / 1 GB** in `docker-compose.yml`.
- Frontend and other components: no limit required.

## Getting Started

1. Click **"Use this template"** → **"Create a new repository"**
2. Set your new repository to **Private**
3. Clone your private repository and start building
4. When finished, add **`zahidzorbaz`** as a collaborator (Settings → Collaborators → Add people)
5. Share your private repo URL with us

A receiver service is provided as a prebuilt image — see [receiver/README.md](receiver/README.md) for how to wire it into your `docker-compose.yml`.

## Success Criteria

We evaluate the generator at **four target rates**. At each, the rate accuracy and drop tolerance must hold:

| Target Rate | Rate Accuracy | Drop Tolerance |
|---|---|---|
| 100 / sec | ±2% | < 5% |
| 1,000 / sec | ±2% | < 5% |
| 10,000 / sec | ±2% | < 5% |
| 50,000 / sec | ±2% | < 5% |

Drop is measured against the provided receiver. Your UI must accept any rate value; we will exercise the four points above.

## Rules

1. **Stack discipline** — Frontend MUST be SvelteKit + shadcn-svelte. Backend is free.
2. **Single-command boot** — `docker compose up` must bring up frontend, generator, and receiver in one shot.
3. **Mock data provided** — Use `mock-data/fortigate-samples.json` as the source pool for randomization. Every emitted packet must differ.
4. **TypeScript required for frontend.** Type safety in backend is a bonus.
5. **Git history matters** — atomic commits with meaningful messages.
6. **README mandatory** — explain your architectural decisions (see below).
7. **AI history mandatory** — see "AI Usage" below.

## Architectural Decisions to Document

In your README, justify these choices:

- **Backend language/framework** — why this stack for this throughput target?
- **Concurrency model** — worker pool, cluster, goroutines, threads?
- **Rate limiter algorithm** — token bucket, ticker, batched send?
- **Real-time channel** — SSE, WebSocket, or polling? Why?
- **Packet generation strategy** — generate per tick, pre-generate pool, batch send?

## Evaluation Criteria

| Criterion | Weight | What We Look For |
|---|---|---|
| **DevOps** | 25% | docker-compose works in one command, backend container limited to 2 CPU / 1 GB, receiver integrated, network isolation |
| **Backend** | 30% | ±2% accuracy at all four rates, <5% drop, multi-thread strategy, RFC 5424 + Fortigate format correct, real randomization |
| **Frontend** | 25% | SvelteKit + shadcn-svelte properly used, real-time metrics, throughput chart, validation, error states |
| **Architecture & Code Quality** | 20% | README justifies decisions, clean modular code, meaningful git history, TypeScript usage, complete AI folder |

Full rubric in [docs/evaluation.md](docs/evaluation.md).

## Time Expectation

This challenge is designed to take approximately **4–5 days** (4–6 hours per day).

## Documentation

| Document | Description |
|---|---|
| [Requirements](docs/requirements.md) | Project story and detailed feature specs |
| [API Contracts](docs/api-contracts.md) | Generator + receiver endpoint documentation |
| [Syslog Format](docs/syslog-format.md) | RFC 5424 envelope + Fortigate payload spec |
| [Evaluation](docs/evaluation.md) | Full rubric, measurement procedure, disqualification rules |

## AI Usage

AI tool usage is allowed and will not count against you.

If you use AI tools, add an `ai/` folder to your repository containing your **complete, unedited** conversation history with any AI tools you used (text export, screenshots, or tool-specific export files).

We expect the full history, not selected highlights. A missing or selectively curated `ai/` folder is grounds for disqualification.

---

Good luck!
