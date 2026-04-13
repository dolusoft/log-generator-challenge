# Challenge Template Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the public-facing template repository for the log-generator-challenge so candidates can fork it, read complete specs, and start implementing. Excludes the receiver service itself (will be authored separately by the owner).

**Architecture:** Pure documentation + mock data + a docker-compose skeleton. No application code. Files are organized so a candidate cloning the template gets: a README that orients them, four detailed spec docs, ready-to-use Fortigate sample templates, and a compose stub that already wires the (future) receiver image.

**Tech Stack:** Markdown, JSON, YAML. Validated with `python -m json.tool` and `docker compose config`.

---

## File Structure

```
log-generator-challenge/
├── .gitignore                         # Task 1
├── README.md                          # Task 2  — top-level brief
├── docs/
│   ├── requirements.md                # Task 3  — feature spec
│   ├── api-contracts.md               # Task 4  — generator + receiver API
│   ├── syslog-format.md               # Task 5  — RFC 5424 + Fortigate format
│   └── evaluation.md                  # Task 6  — rubric + measurement
├── mock-data/
│   ├── fortigate-samples.json         # Task 7  — 20 templates
│   └── usernames.json                 # Task 8  — random user pool
├── receiver/
│   └── README.md                      # Task 9  — image usage placeholder
└── docker-compose.example.yml         # Task 10 — skeleton compose
```

Each file has one clear responsibility:
- `README.md` orients candidates and links to detailed docs.
- Each `docs/*.md` covers exactly one topic area.
- Mock data is data only — no executable content.
- The compose example is a skeleton with the receiver wired in and a placeholder for the candidate's services.

---

## Task 1: .gitignore + repo housekeeping

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Create `.gitignore`**

```gitignore
# OS
.DS_Store
Thumbs.db
desktop.ini

# Editors
.vscode/
.idea/
*.swp
*.swo

# Node (in case anyone scaffolds something here)
node_modules/
dist/
.output/
.nuxt/
.svelte-kit/

# Env
.env
.env.local
.env.*.local

# Logs
*.log
npm-debug.log*

# Docker
.docker/
```

- [ ] **Step 2: Commit**

```bash
git add .gitignore
git commit -m "chore: add .gitignore"
```

---

## Task 2: README.md (top-level challenge brief)

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write `README.md`**

````markdown
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
````

- [ ] **Step 2: Verify Markdown renders cleanly**

Open `README.md` in your editor's preview, or run:
```bash
npx -y markdownlint-cli2 README.md || true
```
Expected: no broken links to local files (the four `docs/*.md` and `receiver/README.md` will exist after later tasks; that's fine).

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add top-level README with challenge brief"
```

---

## Task 3: docs/requirements.md (feature spec)

**Files:**
- Create: `docs/requirements.md`

- [ ] **Step 1: Write `docs/requirements.md`**

````markdown
# Requirements

## Story

The operations team is rolling out a new central log collector. Before the cutover, they need to verify the collector can sustain expected throughput. They want a small load-generation tool that mimics a chatty Fortigate firewall: realistic syslog packets, configurable rate, and a UI to drive tests and watch results.

You are building that tool.

## User Flow

1. The operator opens the dashboard at `http://localhost:5173`.
2. They enter a **target rate** (packets per second) and an optional **duration** (seconds).
3. They click **Start**. The frontend resets the receiver's counters, then asks the generator to begin a run.
4. The generator produces RFC 5424 syslog messages with Fortigate-style payloads and ships them over UDP to the receiver at port `514`.
5. The dashboard shows live: target vs actual rate, total sent / received / drop, CPU%, memory MB, elapsed/remaining time, and a preview of the most recent log lines.
6. When the duration expires (or the operator clicks **Stop**), the generator halts and the dashboard shows a final summary.

## Functional Requirements

### F1 — Configuration & Control

- **F1.1** UI accepts a target rate input (integer, 1–100,000).
- **F1.2** UI accepts an optional duration input (seconds). Empty = run until Stop.
- **F1.3** Start button is enabled only when no run is active.
- **F1.4** Stop button is enabled only when a run is active.
- **F1.5** While a run is active, rate and duration inputs are disabled.
- **F1.6** A status badge displays one of: `IDLE`, `RUNNING`, `STOPPED`.
- **F1.7** Invalid rate (outside 1–100,000) shows an inline validation error and blocks Start.

### F2 — Generator Behavior

- **F2.1** On `POST /run`, the generator must begin emitting UDP packets at the requested rate.
- **F2.2** Rate accuracy must be within **±2%** of the target at 100, 1k, 10k, and 50k packets/sec.
- **F2.3** A multi-threaded / multi-worker / multi-process strategy is required for the higher rates (50k/sec is not achievable single-threaded on most runtimes).
- **F2.4** Each packet must be a different randomized payload (see F4).
- **F2.5** When duration is provided, the run auto-stops at expiry.
- **F2.6** `POST /stop` halts an active run immediately.
- **F2.7** A second `POST /run` while a run is active must return `409` (the operator must Stop first).

### F3 — Receiver Integration

- **F3.1** All UDP packets target host `receiver` port `514` on the docker-compose internal network.
- **F3.2** Before each Start, the frontend calls `POST receiver:4000/reset` to zero counters.
- **F3.3** The receiver image is provided — see [receiver/README.md](../receiver/README.md). The candidate must not modify or replace it.

### F4 — Log Content

- **F4.1** Every packet wraps a Fortigate-style key=value payload in an RFC 5424 envelope. Format details in [syslog-format.md](syslog-format.md).
- **F4.2** Templates come from `mock-data/fortigate-samples.json`. The candidate may add fields but must not alter the existing structure.
- **F4.3** The following fields must be randomized per packet: `srcip`, `srcport`, `dstip`, `dstport`, `sessionid`, `duration`, `sentbyte`, `rcvdbyte`, `user`, `time`, `eventtime`.
- **F4.4** Final message length must fall in **300–800 characters**.

### F5 — Live Metrics (Frontend)

The dashboard MUST display, updated at least once per second:

- **F5.1** Target rate vs actual rate (numbers + deviation %).
- **F5.2** Total sent (from generator), total received (from receiver), drop count, drop %.
- **F5.3** CPU % and memory MB of the generator container.
- **F5.4** Elapsed time and remaining time (remaining hidden if no duration).
- **F5.5** A throughput chart (line) of per-second sent and received over the last ≥60 seconds.
- **F5.6** A preview of the last 10–20 emitted syslog lines (monospace, scrollable). Lines must visibly differ.

### F6 — Real-Time Transport

- **F6.1** The generator pushes self-metrics to the frontend over a real-time channel of the candidate's choosing (SSE, WebSocket, or polling).
- **F6.2** The choice and reasoning must be documented in the candidate's README.
- **F6.3** Minimum update frequency: 1 Hz.

### F7 — Containerization

- **F7.1** A single `docker compose up` from the repo root must bring up frontend, generator, and receiver.
- **F7.2** The generator service MUST declare resource limits of 2 CPU / 1 GB under `deploy.resources.limits`.
- **F7.3** UDP traffic between generator and receiver flows on the internal docker network. The receiver's UDP port does not need to be published to the host.
- **F7.4** Receiver's HTTP port `4000` must be reachable from the frontend (and optionally from the host for debugging).

### F8 — Documentation

- **F8.1** Candidate's README must justify the five architectural decisions (backend stack, concurrency model, rate limiter, real-time channel, packet generation strategy).
- **F8.2** README must list which shadcn-svelte components were used and where.
- **F8.3** An `ai/` folder must contain the complete unedited AI conversation history if AI tools were used.

## Non-Functional Requirements

- **N1 — Performance:** Frontend must remain responsive at 50k/sec. Chart implementations that re-render the full canvas every tick will fail this.
- **N2 — Boot time:** `docker compose up` should reach a ready state within **60 seconds** on a modern laptop.
- **N3 — Type safety:** TypeScript mandatory in frontend. Strict mode encouraged. Backend type safety is a bonus.

## Out of Scope

- TLS / authentication on any service (this is a local-only tool).
- Multi-user or multi-tenant features.
- Persistent storage of run history (localStorage is acceptable; backend persistence is a bonus, not required).
- Log parsing, search, or SIEM-style features — you are the producer, not the consumer.
````

- [ ] **Step 2: Commit**

```bash
git add docs/requirements.md
git commit -m "docs: add detailed requirements spec"
```

---

## Task 4: docs/api-contracts.md

**Files:**
- Create: `docs/api-contracts.md`

- [ ] **Step 1: Write `docs/api-contracts.md`**

````markdown
# API Contracts

This document defines the HTTP and real-time interfaces between the three services. The candidate implements the **generator** API. The **receiver** API is provided.

## Generator (candidate implements)

Base URL inside the docker network: `http://generator:3000`
Base URL from host (recommended): `http://localhost:3000`

### `POST /run` — start a new run

**Request body:**
```json
{
  "rate": 10000,
  "duration": 30
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `rate` | integer | yes | Target packets per second, 1–100000 |
| `duration` | integer | no | Seconds; if omitted, run until Stop |

**Response 200:**
```json
{
  "runId": "8f2c9a1e-...",
  "startedAt": "2026-04-13T10:00:00.000Z"
}
```

**Response 409 — when a run is already active:**
```json
{
  "error": "run_in_progress",
  "runId": "8f2c9a1e-..."
}
```

**Response 400 — invalid input:**
```json
{
  "error": "invalid_rate",
  "message": "rate must be between 1 and 100000"
}
```

### `POST /stop` — stop the active run

**Response 200:**
```json
{
  "runId": "8f2c9a1e-...",
  "stoppedAt": "2026-04-13T10:00:30.000Z",
  "sent": 298745
}
```

**Response 404 — when no run is active:**
```json
{
  "error": "no_active_run"
}
```

### `GET /status` — current state (polling fallback)

**Response 200:**
```json
{
  "running": true,
  "runId": "8f2c9a1e-...",
  "targetRate": 10000,
  "actualRate": 9982,
  "sent": 152400,
  "elapsedMs": 15240,
  "remainingMs": 14760,
  "cpu": 78.4,
  "memoryMb": 312
}
```

When `running: false`, `runId`, `targetRate`, `remainingMs` may be omitted.

### Real-time channel (transport: candidate's choice)

The generator MUST push metrics to the frontend at minimum 1 Hz over one of: SSE, WebSocket, or HTTP polling. The transport choice is the candidate's; the **payload schema** is fixed.

**Event payload schema:**
```json
{
  "ts": "2026-04-13T10:00:01.000Z",
  "sent": 9982,
  "actualRate": 9982,
  "targetRate": 10000,
  "cpu": 78.4,
  "memoryMb": 312,
  "samples": [
    "<189>1 2026-04-13T10:00:01.000Z fgt-01 - - - - date=... srcip=10.0.1.5 ..."
  ]
}
```

| Field | Type | Notes |
|---|---|---|
| `ts` | ISO 8601 UTC | Timestamp of this snapshot |
| `sent` | integer | Cumulative packets sent in current run |
| `actualRate` | integer | Packets actually sent in the last second |
| `targetRate` | integer | The rate operator requested |
| `cpu` | float | Container CPU usage percent |
| `memoryMb` | float | Container memory usage in MB |
| `samples` | string[] | Optional, last 1–5 emitted lines for UI preview |

Suggested transport endpoints (candidate may rename):
- SSE: `GET /stream`
- WebSocket: `GET /ws`
- Polling: `GET /status` every 1s (or a dedicated `/metrics` endpoint)

## Receiver (provided — do not modify)

Base URL inside the docker network: `http://receiver:4000`

### `GET /stats`

**Response 200:**
```json
{
  "total": 298745,
  "perSecond": 9978,
  "lastSecondTotal": 9978,
  "startedAt": "2026-04-13T10:00:00.000Z",
  "uptimeS": 30,
  "resetAt": "2026-04-13T09:59:55.000Z"
}
```

The frontend SHOULD poll this every 1 second while a run is active.

### `POST /reset`

Zeros the counters. Call before each `POST /run`.

**Response 200:**
```json
{
  "ok": true,
  "resetAt": "2026-04-13T10:00:00.000Z"
}
```

### `GET /sample?n=10`

Returns the most recent N raw UDP packet bodies as strings. For debugging.

**Response 200:**
```json
{
  "samples": [
    "<189>1 2026-04-13T10:00:01.000Z fgt-01 ...",
    "<189>1 2026-04-13T10:00:01.001Z fgt-04 ..."
  ]
}
```

### UDP — port 514

The receiver listens on UDP `:514` on the internal docker network. Every packet increments counters. There is no parsing or validation — the receiver counts every datagram.

## Frontend's expected call sequence (for one run)

1. `POST receiver:4000/reset`
2. `POST generator:3000/run { rate, duration? }`
3. Subscribe to generator real-time channel; poll `receiver:4000/stats` every 1s.
4. On Stop button: `POST generator:3000/stop`. (When duration auto-expires the generator stops itself; the frontend should still close subscriptions.)
````

- [ ] **Step 2: Commit**

```bash
git add docs/api-contracts.md
git commit -m "docs: add API contracts for generator and receiver"
```

---

## Task 5: docs/syslog-format.md

**Files:**
- Create: `docs/syslog-format.md`

- [ ] **Step 1: Write `docs/syslog-format.md`**

````markdown
# Syslog & Fortigate Format

Every UDP packet you send must be a valid RFC 5424 syslog message wrapping a Fortigate-style key=value payload.

## RFC 5424 envelope

```
<PRI>VERSION TIMESTAMP HOSTNAME APP-NAME PROCID MSGID STRUCTURED-DATA MSG
```

| Field | Required value | Notes |
|---|---|---|
| `<PRI>` | `<189>` | Facility = local7 (23) × 8 + Severity = notice (5) = 189. May be fixed. |
| `VERSION` | `1` | RFC 5424 version |
| `TIMESTAMP` | ISO 8601 UTC, ms precision | E.g. `2026-04-13T10:00:01.234Z` |
| `HOSTNAME` | `fgt-XX` | XX random 01–20, simulating 20 firewalls |
| `APP-NAME` | `-` or string | NILVALUE acceptable |
| `PROCID` | `-` or string | NILVALUE acceptable |
| `MSGID` | `-` or string | NILVALUE acceptable |
| `STRUCTURED-DATA` | `-` | NILVALUE recommended for simplicity |
| `MSG` | Fortigate payload | See below |

**Example envelope:**
```
<189>1 2026-04-13T10:00:01.234Z fgt-07 - - - - <fortigate payload here>
```

## Fortigate payload

A Fortigate firewall emits log lines as space-separated key=value pairs, with values quoted when they contain spaces. The payload is one continuous line.

### Required keys (must appear in every payload)

| Key | Type | Example | Notes |
|---|---|---|---|
| `date` | YYYY-MM-DD | `2026-04-13` | UTC |
| `time` | HH:MM:SS | `10:00:01` | UTC, randomize per packet |
| `devname` | quoted string | `"FGT-01"` | One of the 20 hosts |
| `devid` | quoted string | `"FG100ETK20000001"` | Stable per devname |
| `logid` | quoted string | `"0000000013"` | Tied to subtype |
| `type` | quoted string | `"traffic"` | See subtype matrix |
| `subtype` | quoted string | `"forward"` | See subtype matrix |
| `level` | quoted string | `"notice"` | One of: information, notice, warning, error |
| `vd` | quoted string | `"root"` | Virtual domain |
| `eventtime` | integer | `1773043201234567890` | Nanoseconds since epoch, randomize |

### Conditional keys (per type)

For **`type=traffic`**, the payload must additionally include:
- `srcip`, `srcport`, `srcintf`, `dstip`, `dstport`, `dstintf`
- `sessionid`, `proto`, `action`, `policyid`, `service`
- `srccountry`, `dstcountry`
- `duration`, `sentbyte`, `rcvdbyte`
- `user` (often empty `"N/A"` but should be a real username from the pool)

For **`type=event`**, the payload must additionally include:
- `logdesc`, `action`, `msg`

For **`type=utm`**, the payload must additionally include:
- `subtype` is one of `webfilter`, `ips`, `virus`, `app-ctrl`
- `srcip`, `dstip`, `action`, `severity`, `attack`/`hostname`/`filename` depending on subtype

### Randomized fields (must vary per packet)

The candidate must randomize the following fields for every emitted packet:

- `srcip` (private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
- `dstip` (mix of public and private)
- `srcport` (1024–65535)
- `dstport` (common services: 80, 443, 22, 53, 3389; mix in random)
- `sessionid` (8-digit integer)
- `duration` (1–7200 seconds)
- `sentbyte` (0–10,000,000)
- `rcvdbyte` (0–10,000,000)
- `user` (pick from `mock-data/usernames.json`)
- `time` (current UTC HH:MM:SS)
- `eventtime` (current UTC nanoseconds)

### Length constraint

Final assembled message (envelope + payload) must be **300–800 characters**. Truncate or pad keys as needed to stay in this range. Most realistic Fortigate lines fall naturally in this window.

### Realistic example (full)

```
<189>1 2026-04-13T10:00:01.234Z fgt-07 - - - - date=2026-04-13 time=10:00:01 devname="FGT-07" devid="FG100ETK20000007" logid="0000000013" type="traffic" subtype="forward" level="notice" vd="root" eventtime=1773043201234567890 srcip=10.0.1.45 srcport=51234 srcintf="port1" dstip=8.8.8.8 dstport=443 dstintf="port2" sessionid=12345678 proto=6 action="accept" policyid=1 service="HTTPS" srccountry="Turkey" dstcountry="United States" duration=120 sentbyte=4523 rcvdbyte=89234 user="ahmet.yilmaz"
```

## Source data

Pre-built templates: [`mock-data/fortigate-samples.json`](../mock-data/fortigate-samples.json)
Username pool: [`mock-data/usernames.json`](../mock-data/usernames.json)

The candidate is free to add new templates following the same shape. Templates must not be edited to remove required keys.
````

- [ ] **Step 2: Commit**

```bash
git add docs/syslog-format.md
git commit -m "docs: add syslog and Fortigate format spec"
```

---

## Task 6: docs/evaluation.md

**Files:**
- Create: `docs/evaluation.md`

- [ ] **Step 1: Write `docs/evaluation.md`**

````markdown
# Evaluation

## Measurement Procedure

We evaluate by running your project locally, exactly as a user would:

1. `git clone <your repo>`
2. `cd <your repo> && docker compose up -d`
3. Open `http://localhost:5173` in a browser.
4. For each of the four target rates (100, 1000, 10000, 50000):
   - Set rate = N, duration = 30 seconds.
   - Click **Start**.
   - Wait for the run to auto-complete.
   - Record: target rate, generator's reported `sent`, receiver's reported `total`, container CPU% and memory MB at peak.
5. We also exercise: Stop mid-run, change rate, restart; invalid input; receiver unreachable scenario.

## Scoring Rubric

| Criterion | Weight | What earns maximum |
|---|---|---|
| **DevOps** | 25% | • `docker compose up` works in one command and reaches ready state within 60s<br>• Backend container has correct `deploy.resources.limits: cpus: '2', memory: 1G`<br>• Receiver image integrated cleanly via the provided snippet<br>• Network isolation correct: UDP traffic stays internal, only HTTP ports published as needed |
| **Backend** | 30% | • Rate accuracy within ±2% at 100 / 1k / 10k / 50k packets/sec<br>• Drop < 5% on the docker bridge network at all four rates<br>• Multi-thread / worker / cluster strategy clearly visible in code<br>• RFC 5424 envelope correct; Fortigate payload includes all required keys for chosen subtype<br>• Randomization is real — no two consecutive packets identical, fields vary as specified |
| **Frontend** | 25% | • Built with SvelteKit + shadcn-svelte; shadcn components used in appropriate slots<br>• Real-time metric stream works smoothly; chart updates without freezing UI even at 50k/sec<br>• All required metrics visible: target vs actual rate, sent/received/drop, CPU/memory, elapsed/remaining, last samples<br>• Validation handled (rate range, disabled inputs during run)<br>• Error states handled (generator/receiver unreachable shows a clear Alert) |
| **Architecture & Code Quality** | 20% | • README justifies the five required architectural decisions in clear language<br>• Code is modular, naming is consistent, no dead code<br>• Git history shows incremental progress with meaningful commit messages<br>• TypeScript used in frontend (strict mode preferred); backend type safety is a bonus<br>• `ai/` folder is complete and unedited |

## Automatic Disqualification

Any of the following results in disqualification regardless of other scores:

- `docker compose up` fails or hangs.
- Frontend is not SvelteKit + shadcn-svelte.
- Cannot sustain 1,000 packets/sec (the lowest threshold) within ±5% — meaning the generator never genuinely worked.
- `ai/` folder is missing, or content is selectively curated rather than the complete history.
- The provided receiver image was modified or replaced.

## Bonus Areas (raise the score above baseline)

These are not required, but candidates who demonstrate them earn additional credit:

- **Run history persistence** — SQLite or file-based history of past runs, viewable in UI.
- **Multi-receiver support** — load-balanced UDP send across multiple receiver targets.
- **Latency / histogram view** — distribution of inter-packet timing or end-to-end latency.
- **Polish** — dark mode, animated transitions, keyboard shortcuts, empty states.
- **Observability** — Prometheus `/metrics` endpoint on the backend, structured logs.
- **Tests** — E2E for frontend, integration for backend, load test scripts.

## What we will *not* deduct for

- AI tool usage (provided the `ai/` folder is complete).
- Backend language choice — Go, Rust, Node, Bun, Python all acceptable provided the success criteria are met.
- Real-time transport choice — SSE, WebSocket, or polling all acceptable provided the schema and 1Hz minimum are met.
- Aesthetic preferences in chart library, color palette, or layout — provided clarity is preserved.
````

- [ ] **Step 2: Commit**

```bash
git add docs/evaluation.md
git commit -m "docs: add evaluation rubric and measurement procedure"
```

---

## Task 7: mock-data/fortigate-samples.json

**Files:**
- Create: `mock-data/fortigate-samples.json`

- [ ] **Step 1: Write `mock-data/fortigate-samples.json`**

The file is a JSON array of 20 template objects. Each template specifies the static keys; randomized keys are emitted by the candidate's code per packet. The `template` string uses `{placeholder}` syntax for fields the generator will substitute.

```json
[
  {
    "id": "traffic-forward-accept-https",
    "type": "traffic",
    "subtype": "forward",
    "logid": "0000000013",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0000000013\" type=\"traffic\" subtype=\"forward\" level=\"notice\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srcintf=\"port1\" dstip={dstip} dstport=443 dstintf=\"port2\" sessionid={sessionid} proto=6 action=\"accept\" policyid=1 service=\"HTTPS\" srccountry=\"Turkey\" dstcountry=\"United States\" duration={duration} sentbyte={sentbyte} rcvdbyte={rcvdbyte} user=\"{user}\""
  },
  {
    "id": "traffic-forward-accept-http",
    "type": "traffic",
    "subtype": "forward",
    "logid": "0000000013",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0000000013\" type=\"traffic\" subtype=\"forward\" level=\"notice\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srcintf=\"port1\" dstip={dstip} dstport=80 dstintf=\"port2\" sessionid={sessionid} proto=6 action=\"accept\" policyid=2 service=\"HTTP\" srccountry=\"Turkey\" dstcountry=\"Germany\" duration={duration} sentbyte={sentbyte} rcvdbyte={rcvdbyte} user=\"{user}\""
  },
  {
    "id": "traffic-forward-deny",
    "type": "traffic",
    "subtype": "forward",
    "logid": "0000000014",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0000000014\" type=\"traffic\" subtype=\"forward\" level=\"warning\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srcintf=\"port1\" dstip={dstip} dstport={dstport} dstintf=\"port2\" sessionid={sessionid} proto=6 action=\"deny\" policyid=99 service=\"tcp/{dstport}\" srccountry=\"Turkey\" dstcountry=\"Russia\" duration=0 sentbyte=0 rcvdbyte=0 user=\"{user}\" msg=\"policy violation\""
  },
  {
    "id": "traffic-forward-dns",
    "type": "traffic",
    "subtype": "forward",
    "logid": "0000000013",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0000000013\" type=\"traffic\" subtype=\"forward\" level=\"notice\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srcintf=\"internal\" dstip={dstip} dstport=53 dstintf=\"wan1\" sessionid={sessionid} proto=17 action=\"accept\" policyid=3 service=\"DNS\" srccountry=\"Turkey\" dstcountry=\"Reserved\" duration={duration} sentbyte={sentbyte} rcvdbyte={rcvdbyte} user=\"{user}\""
  },
  {
    "id": "traffic-forward-ssh",
    "type": "traffic",
    "subtype": "forward",
    "logid": "0000000013",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0000000013\" type=\"traffic\" subtype=\"forward\" level=\"notice\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srcintf=\"port1\" dstip={dstip} dstport=22 dstintf=\"port2\" sessionid={sessionid} proto=6 action=\"accept\" policyid=4 service=\"SSH\" srccountry=\"Turkey\" dstcountry=\"Netherlands\" duration={duration} sentbyte={sentbyte} rcvdbyte={rcvdbyte} user=\"{user}\""
  },
  {
    "id": "traffic-forward-rdp-deny",
    "type": "traffic",
    "subtype": "forward",
    "logid": "0000000014",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0000000014\" type=\"traffic\" subtype=\"forward\" level=\"warning\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srcintf=\"wan1\" dstip={dstip} dstport=3389 dstintf=\"internal\" sessionid={sessionid} proto=6 action=\"deny\" policyid=0 service=\"RDP\" srccountry=\"China\" dstcountry=\"Turkey\" duration=0 sentbyte=0 rcvdbyte=0 user=\"{user}\" msg=\"implicit deny\""
  },
  {
    "id": "traffic-local-accept",
    "type": "traffic",
    "subtype": "local",
    "logid": "0000000015",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0000000015\" type=\"traffic\" subtype=\"local\" level=\"notice\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srcintf=\"mgmt\" dstip={dstip} dstport=443 dstintf=\"local\" sessionid={sessionid} proto=6 action=\"accept\" policyid=0 service=\"HTTPS\" srccountry=\"Turkey\" dstcountry=\"Turkey\" duration={duration} sentbyte={sentbyte} rcvdbyte={rcvdbyte} user=\"{user}\""
  },
  {
    "id": "utm-webfilter-block",
    "type": "utm",
    "subtype": "webfilter",
    "logid": "0317013312",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0317013312\" type=\"utm\" subtype=\"webfilter\" eventtype=\"ftgd_blk\" level=\"warning\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} dstip={dstip} dstport=443 sessionid={sessionid} proto=6 service=\"HTTPS\" hostname=\"malicious{sessionid}.example.com\" profile=\"default\" action=\"blocked\" reqtype=\"direct\" url=\"/\" sentbyte={sentbyte} rcvdbyte={rcvdbyte} catdesc=\"Malicious Websites\" user=\"{user}\""
  },
  {
    "id": "utm-webfilter-allow",
    "type": "utm",
    "subtype": "webfilter",
    "logid": "0317013313",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0317013313\" type=\"utm\" subtype=\"webfilter\" eventtype=\"ftgd_allow\" level=\"notice\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} dstip={dstip} dstport=443 sessionid={sessionid} proto=6 service=\"HTTPS\" hostname=\"www.example.com\" profile=\"default\" action=\"passthrough\" reqtype=\"direct\" url=\"/\" sentbyte={sentbyte} rcvdbyte={rcvdbyte} catdesc=\"Information Technology\" user=\"{user}\""
  },
  {
    "id": "utm-ips-attack",
    "type": "utm",
    "subtype": "ips",
    "logid": "0419016384",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0419016384\" type=\"utm\" subtype=\"ips\" eventtype=\"signature\" level=\"alert\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} srccountry=\"Russia\" dstip={dstip} dstport={dstport} sessionid={sessionid} action=\"dropped\" proto=6 service=\"HTTPS\" attack=\"SSH.Brute.Force\" srcintf=\"wan1\" dstintf=\"internal\" attackid=29844 profile=\"default\" ref=\"http://www.fortinet.com/ids/VID29844\" incidentserialno={sessionid} severity=\"critical\" user=\"{user}\""
  },
  {
    "id": "utm-virus-blocked",
    "type": "utm",
    "subtype": "virus",
    "logid": "0211008192",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0211008192\" type=\"utm\" subtype=\"virus\" eventtype=\"infected\" level=\"warning\" vd=\"root\" eventtime={eventtime} srcip={srcip} srcport={srcport} dstip={dstip} dstport=80 sessionid={sessionid} action=\"blocked\" service=\"HTTP\" profile=\"default\" filename=\"sample{sessionid}.exe\" virus=\"W32/Generic.AC.{sessionid}!tr\" dtype=\"Virus\" filehash=\"abc123def456\" filehashsrc=\"sandbox\" url=\"http://malware.example.com/{sessionid}\" user=\"{user}\""
  },
  {
    "id": "utm-app-ctrl",
    "type": "utm",
    "subtype": "app-ctrl",
    "logid": "1059028704",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"1059028704\" type=\"utm\" subtype=\"app-ctrl\" eventtype=\"signature\" level=\"information\" vd=\"root\" eventtime={eventtime} appid=15832 srcip={srcip} dstip={dstip} srcport={srcport} dstport={dstport} srcintf=\"port1\" dstintf=\"port2\" proto=6 service=\"HTTPS\" sessionid={sessionid} applist=\"default\" action=\"pass\" appcat=\"Social.Media\" app=\"Facebook\" hostname=\"www.facebook.com\" url=\"/\" sentbyte={sentbyte} rcvdbyte={rcvdbyte} user=\"{user}\""
  },
  {
    "id": "event-system-login",
    "type": "event",
    "subtype": "system",
    "logid": "0100032001",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0100032001\" type=\"event\" subtype=\"system\" level=\"information\" vd=\"root\" eventtime={eventtime} logdesc=\"Admin login successful\" sn={sessionid} user=\"{user}\" ui=\"https({srcip})\" method=\"https\" srcip={srcip} dstip={dstip} action=\"login\" status=\"success\" reason=\"none\" msg=\"Administrator {user} logged in successfully from https({srcip})\""
  },
  {
    "id": "event-system-login-fail",
    "type": "event",
    "subtype": "system",
    "logid": "0100032002",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0100032002\" type=\"event\" subtype=\"system\" level=\"warning\" vd=\"root\" eventtime={eventtime} logdesc=\"Admin login failed\" sn={sessionid} user=\"{user}\" ui=\"https({srcip})\" method=\"https\" srcip={srcip} dstip={dstip} action=\"login\" status=\"failed\" reason=\"name_invalid\" msg=\"Administrator {user} login failed from https({srcip}) because of invalid user name\""
  },
  {
    "id": "event-system-config-change",
    "type": "event",
    "subtype": "system",
    "logid": "0100044546",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0100044546\" type=\"event\" subtype=\"system\" level=\"information\" vd=\"root\" eventtime={eventtime} logdesc=\"Object attribute configured\" user=\"{user}\" ui=\"https({srcip})\" action=\"Edit\" cfgtid=12345{sessionid} cfgpath=\"firewall.policy\" cfgobj=\"{sessionid}\" cfgattr=\"action[deny->accept]\" msg=\"Edit firewall.policy {sessionid}\""
  },
  {
    "id": "event-vpn-tunnel-up",
    "type": "event",
    "subtype": "vpn",
    "logid": "0101037127",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0101037127\" type=\"event\" subtype=\"vpn\" level=\"notice\" vd=\"root\" eventtime={eventtime} logdesc=\"IPsec tunnel statistics\" msg=\"IPsec tunnel statistics\" action=\"tunnel-stats\" remip={dstip} locip={srcip} remport=4500 locport=4500 outintf=\"wan1\" cookies=\"abc{sessionid}/def{sessionid}\" user=\"{user}\" group=\"N/A\" useralt=\"N/A\" xauthuser=\"N/A\" xauthgroup=\"N/A\" assignip=N/A vpntunnel=\"branch-{sessionid}\" tunnelip=N/A tunnelid={sessionid} sentbyte={sentbyte} rcvdbyte={rcvdbyte} duration={duration}"
  },
  {
    "id": "event-vpn-ssl-login",
    "type": "event",
    "subtype": "vpn",
    "logid": "0101039426",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0101039426\" type=\"event\" subtype=\"vpn\" level=\"information\" vd=\"root\" eventtime={eventtime} logdesc=\"SSL VPN tunnel up\" action=\"tunnel-up\" tunneltype=\"ssl-tunnel\" tunnelid={sessionid} remip={srcip} tunnelip={dstip} user=\"{user}\" group=\"sslvpn-users\" dst_host=\"N/A\" reason=\"login successfully\" msg=\"SSL tunnel established\""
  },
  {
    "id": "event-user-auth",
    "type": "event",
    "subtype": "user",
    "logid": "0102043035",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0102043035\" type=\"event\" subtype=\"user\" level=\"notice\" vd=\"root\" eventtime={eventtime} logdesc=\"Authentication success\" srcip={srcip} dstip={dstip} policyid=1 interface=\"port1\" user=\"{user}\" group=\"users\" authproto=\"HTTP({srcip})\" action=\"authentication\" status=\"success\" reason=\"N/A\" msg=\"User {user} succeeded in authentication\""
  },
  {
    "id": "event-wireless-station-add",
    "type": "event",
    "subtype": "wireless",
    "logid": "0104032300",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0104032300\" type=\"event\" subtype=\"wireless\" level=\"information\" vd=\"root\" eventtime={eventtime} logdesc=\"Wireless station added\" sn=\"AP{sessionid}\" ap=\"AP-{sessionid}\" vap=\"corp-wifi\" ssid=\"CORP-WIFI\" radioid=1 stamac=\"aa:bb:cc:{sessionid}\" channel=36 user=\"{user}\" group=\"wifi-users\" msg=\"New wireless station joined\""
  },
  {
    "id": "event-system-ha",
    "type": "event",
    "subtype": "ha",
    "logid": "0105037889",
    "template": "date={date} time={time} devname=\"{devname}\" devid=\"{devid}\" logid=\"0105037889\" type=\"event\" subtype=\"ha\" level=\"notice\" vd=\"root\" eventtime={eventtime} logdesc=\"HA member synced\" ha_group=0 ha_role=\"slave\" sn=\"{devid}\" user=\"{user}\" msg=\"HA member synchronized successfully with master\""
  }
]
```

- [ ] **Step 2: Validate JSON**

```bash
python -m json.tool mock-data/fortigate-samples.json > /dev/null
```
Expected: no output (file is valid JSON).

- [ ] **Step 3: Confirm 20 templates**

```bash
python -c "import json; print(len(json.load(open('mock-data/fortigate-samples.json'))))"
```
Expected: `20`

- [ ] **Step 4: Commit**

```bash
git add mock-data/fortigate-samples.json
git commit -m "data: add 20 Fortigate template samples"
```

---

## Task 8: mock-data/usernames.json

**Files:**
- Create: `mock-data/usernames.json`

- [ ] **Step 1: Write `mock-data/usernames.json`**

```json
[
  "ahmet.yilmaz",
  "mehmet.demir",
  "ayse.kaya",
  "fatma.celik",
  "mustafa.sahin",
  "ali.koc",
  "zeynep.ozturk",
  "emine.aydin",
  "huseyin.arslan",
  "hatice.dogan",
  "ibrahim.kilic",
  "elif.aslan",
  "murat.cetin",
  "ozlem.simsek",
  "burak.tas",
  "selin.polat",
  "kerem.duman",
  "merve.bozkurt",
  "onur.guler",
  "deniz.korkmaz",
  "cem.acar",
  "ece.ergin",
  "baris.tekin",
  "gizem.uzun",
  "tolga.aksoy",
  "buse.guneri",
  "kaan.altun",
  "irem.bulut",
  "umut.cinar",
  "nazli.sevim",
  "admin",
  "guest",
  "service.account",
  "backup.user",
  "monitor.svc"
]
```

- [ ] **Step 2: Validate JSON**

```bash
python -m json.tool mock-data/usernames.json > /dev/null
```
Expected: no output.

- [ ] **Step 3: Commit**

```bash
git add mock-data/usernames.json
git commit -m "data: add username pool for randomization"
```

---

## Task 9: receiver/README.md

**Files:**
- Create: `receiver/README.md`

This is the placeholder that tells candidates how to use the (separately-built) receiver image.

- [ ] **Step 1: Write `receiver/README.md`**

````markdown
# Receiver Service

The receiver is a small UDP listener + HTTP stats server that we provide as a prebuilt Docker image. **Do not modify or replace it** — every candidate is evaluated against the same receiver, and the image is the reference implementation we use for scoring.

## Image

```
ghcr.io/dolusoft/log-challenge-receiver:latest
```

## Behavior

- Listens on UDP port `514` for syslog packets and increments counters.
- Does **not** parse or validate packet contents — every datagram counts.
- Exposes an HTTP server on port `4000` for stats and reset.
- All state is in-memory; counters are zeroed on container restart or via `POST /reset`.

## How to wire it into your `docker-compose.yml`

```yaml
services:
  receiver:
    image: ghcr.io/dolusoft/log-challenge-receiver:latest
    container_name: receiver
    ports:
      - "4000:4000"      # stats HTTP, also reachable from the host
    networks:
      - challenge

networks:
  challenge:
    driver: bridge
```

The receiver's UDP port `514` does not need to be published to the host — your generator reaches it on the internal docker network.

## HTTP API quick reference

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/stats` | totals and per-second window |
| `POST` | `/reset` | zero counters |
| `GET` | `/sample?n=10` | last N raw packet bodies (debug) |

Full request/response schemas: see [`docs/api-contracts.md`](../docs/api-contracts.md).

## Health check

After `docker compose up`, verify the receiver is reachable:

```bash
curl http://localhost:4000/stats
```

Expected: a JSON response with `total: 0` and `perSecond: 0` immediately after boot.
````

- [ ] **Step 2: Commit**

```bash
git add receiver/README.md
git commit -m "docs: add receiver service usage guide"
```

---

## Task 10: docker-compose.example.yml

**Files:**
- Create: `docker-compose.example.yml`

This is a skeleton the candidate copies to `docker-compose.yml` and fills in. It already wires the receiver and shows the required limits on the generator slot.

- [ ] **Step 1: Write `docker-compose.example.yml`**

```yaml
# Copy this file to docker-compose.yml and fill in the generator and frontend
# build contexts with your own implementation. The receiver service is
# preconfigured and must NOT be modified.
#
# After completion, `docker compose up` from the repo root must bring up all
# three services and reach a ready state within 60 seconds.

services:
  receiver:
    image: ghcr.io/dolusoft/log-challenge-receiver:latest
    container_name: receiver
    ports:
      - "4000:4000"
    networks:
      - challenge
    restart: unless-stopped

  generator:
    # TODO: replace with your build context, e.g.:
    #   build:
    #     context: ./generator
    #     dockerfile: Dockerfile
    image: your-org/log-generator:dev
    container_name: generator
    ports:
      - "3000:3000"
    networks:
      - challenge
    depends_on:
      - receiver
    environment:
      - RECEIVER_HOST=receiver
      - RECEIVER_UDP_PORT=514
    deploy:
      resources:
        limits:
          cpus: "2"
          memory: 1G
    restart: unless-stopped

  frontend:
    # TODO: replace with your build context, e.g.:
    #   build:
    #     context: ./frontend
    #     dockerfile: Dockerfile
    image: your-org/log-frontend:dev
    container_name: frontend
    ports:
      - "5173:5173"
    networks:
      - challenge
    depends_on:
      - generator
      - receiver
    environment:
      - GENERATOR_URL=http://generator:3000
      - RECEIVER_URL=http://receiver:4000
    restart: unless-stopped

networks:
  challenge:
    driver: bridge
```

- [ ] **Step 2: Validate YAML syntax (without trying to actually build)**

```bash
docker compose -f docker-compose.example.yml config > /dev/null
```
Expected: prints the resolved config to stdout (we discard it). If syntax is wrong, an error appears.

If `docker` is not available locally, fall back to:
```bash
python -c "import yaml; yaml.safe_load(open('docker-compose.example.yml'))"
```
Expected: no output.

- [ ] **Step 3: Commit**

```bash
git add docker-compose.example.yml
git commit -m "feat: add docker-compose skeleton with wired receiver"
```

---

## Task 11: Final cross-link sanity check + tag

**Files:**
- Modify: none (this task only verifies)

- [ ] **Step 1: Verify all internal doc links resolve**

The README and docs cross-link to each other. Confirm every `(./...)` / `(../...)` link points to an existing file:

```bash
# List every relative link in the project's markdown
grep -rEn '\]\((\.\.?/[^)]+)\)' README.md docs/ receiver/
```

Then visually confirm each path exists. Expected files referenced:
- `docs/requirements.md`
- `docs/api-contracts.md`
- `docs/syslog-format.md`
- `docs/evaluation.md`
- `receiver/README.md`
- `mock-data/fortigate-samples.json`
- `mock-data/usernames.json`

- [ ] **Step 2: Verify directory tree matches plan's File Structure**

```bash
ls -R . | head -50
```
Expected to see: `.gitignore`, `README.md`, `docker-compose.example.yml`, `docs/`, `mock-data/`, `receiver/`.

- [ ] **Step 3: Tag the template-ready state**

```bash
git tag -a v0.1.0-template -m "Template repo ready for candidates (receiver image to be published separately)"
```

- [ ] **Step 4: Print next steps for the owner**

The repo is now ready as a public template. Remaining manual steps for the owner (Zahid):

1. Author the receiver service (out of scope of this plan).
2. Build and publish the receiver image to `ghcr.io/dolusoft/log-challenge-receiver:latest`.
3. Push this repo to GitHub under `dolusoft/log-generator-challenge`.
4. In the GitHub repo settings, enable **"Template repository"**.
5. Smoke-test by clicking "Use this template" yourself, completing the challenge skeleton with stub services, and running `docker compose up` end-to-end.

---

## Self-Review Notes

After writing the plan, the following self-review was performed:

- **Spec coverage:** Every section of `2026-04-13-log-generator-challenge-design.md` maps to at least one task. Section 1 (Overview & Success Criteria) → README + requirements.md + evaluation.md. Section 2 (Architecture) → requirements.md + docker-compose.example.yml. Section 3 (API Contracts) → api-contracts.md. Section 4 (Frontend Requirements) → requirements.md F5/F6. Section 5 (Submission) → README + evaluation.md. The receiver image build (Section 6) is intentionally excluded per the user's instruction to author it separately.

- **Placeholder scan:** No "TBD" / "TODO" / "implement later" in any task body. The two `# TODO:` markers inside `docker-compose.example.yml` are intentional — they are guides for the candidate, not for the engineer executing this plan.

- **Type/name consistency:** `runId` is consistent across `/run`, `/stop`, `/status`. `targetRate` / `actualRate` are used in both `/status` and the real-time event payload. The generator container name `generator`, receiver container name `receiver`, frontend port `5173`, generator port `3000`, receiver HTTP port `4000`, receiver UDP port `514` are consistent across compose, docs, and API contracts.
