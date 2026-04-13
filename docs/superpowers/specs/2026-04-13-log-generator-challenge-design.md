# Log Generator Challenge — Design Spec

**Date:** 2026-04-13
**Status:** Draft (awaiting user review)
**Author:** Zahid + Claude (brainstorming session)

## 1. Overview & Success Criteria

### Project Name
`log-generator-challenge` — fullstack candidate evaluation challenge (DevOps + Backend + Frontend).

### Scenario
The candidate builds a **UDP Syslog Load Generator**. The operations team wants to verify the throughput capacity of a newly deployed log collector. The tool produces RFC 5424 syslog messages at a configurable rate and sends them over UDP to port 514. The team configures target rate and duration through a UI, observes actual throughput and system resource usage in real time.

### Time Expectation
4–5 days (4–6 hours per day).

### Mandatory Stack

| Layer | Technology |
|---|---|
| Frontend | SvelteKit + shadcn-svelte + TypeScript |
| Backend | **Free choice** (Node / Go / Rust / Bun / Python — candidate decides) |
| Container | Docker + docker-compose |
| Log format | Syslog RFC 5424 envelope + Fortigate-style payload |
| Transport | UDP (receiver port 514) |

### Container Resource Limits

- **Backend (generator):** 2 CPU / 1 GB memory (enforced via compose `deploy.resources.limits`)
- Frontend & other components: no limit

### Success Criteria — Measurement Points

| Target Rate | Rate Accuracy | Drop Tolerance |
|---|---|---|
| 100 / sec | ±2% | < 5% |
| 1,000 / sec | ±2% | < 5% |
| 10,000 / sec | ±2% | < 5% |
| 50,000 / sec | ±2% | < 5% |

- Candidate UI must accept any rate value; we evaluate on the four points above.
- Drop is measured against the receiver we provide.

### Log Content

- Format: **RFC 5424 syslog envelope** wrapping a **Fortigate-style structured payload** (key=value pairs).
- We provide ~20 anonymized Fortigate sample lines in `mock-data/fortigate-samples.json`.
- Each emitted packet must:
  - Pick a template at random from the sample pool
  - Randomize variable fields: `srcip`, `dstip`, `srcport`, `dstport`, `sessionid`, `duration`, `sentbyte`, `rcvdbyte`, `user`, `time`, `eventtime`
  - Result in a 300–800 character message (neither too short nor too long)
- Rationale: realistic load (collector sees variety, not duplicates), payload generation has CPU cost, "last samples" preview becomes meaningful.

## 2. Architecture & Components

### Docker Compose Topology

```
┌─────────────── docker-compose.yml ────────────────┐
│                                                    │
│  ┌────────────┐  HTTP/WS/SSE  ┌────────────────┐  │
│  │  frontend  │ ◄───────────► │   generator    │  │
│  │ SvelteKit  │               │ (candidate be) │  │
│  │   :5173    │               │  :3000         │  │
│  │            │               │ limit:2cpu/1G  │  │
│  └────────────┘               └────────┬───────┘  │
│        │                               │ UDP      │
│        │ HTTP polling /stats           │ :514     │
│        └─────────────────┐             ▼          │
│                          │   ┌──────────────────┐ │
│                          └──►│    receiver      │ │
│                              │  (we provide)    │ │
│                              │ UDP:514 HTTP:4000│ │
│                              └──────────────────┘ │
└────────────────────────────────────────────────────┘
```

### Component Responsibilities

#### `generator` — candidate writes (backend free)

- **HTTP API:** `POST /run`, `POST /stop`, `GET /status`
- **Real-time channel:** candidate's choice (SSE / WebSocket / polling) — emits per-second metrics
- **Syslog RFC 5424 envelope generator** (PRI, version, timestamp, hostname, app-name, procid, msgid, structured-data)
- **Fortigate payload templating:** picks from `mock-data/fortigate-samples.json`, randomizes variable fields
- **Rate limiter:** produces packets at the requested PPS (token bucket / high-resolution scheduler / batched send — candidate decides)
- **Concurrency:** worker / goroutine / thread strategy required for 50k/sec (single thread cannot reach this on most runtimes)
- **UDP sender:** target `receiver:514`
- **Self-metrics:** container CPU% and memory MB (runtime API or `/proc`)
- **Resource limits set in compose** under `deploy.resources.limits` (`cpus: '2'`, `memory: 1G`)

#### `frontend` — candidate writes (SvelteKit + shadcn-svelte mandatory)

- Configuration panel (rate input, optional duration, Start/Stop, status badge)
- Live metrics dashboard (target vs actual rate, totals + drop, CPU/memory, elapsed/remaining, last-N samples)
- Two data sources: generator (self-metrics + sent count via real-time channel), receiver (received count via 1Hz polling)

#### `receiver` — we provide (`ghcr.io/dolusoft/log-challenge-receiver:latest`)

- UDP `:514` listener — counts every packet, no validation (for performance)
- HTTP `:4000`:
  - `GET /stats` → totals + per-second window
  - `POST /reset` → zero counters
  - `GET /sample?n=10` → debug, returns last N raw packets
- In-memory only; no persistence
- Single instance per challenge run

### Data Flow (single run)

1. User enters `10000/sec, 30sec` in frontend → clicks Start
2. Frontend → `POST receiver:4000/reset` (zero counters)
3. Frontend → `POST generator:3000/run { rate: 10000, duration: 30 }`
4. Generator starts workers → UDP packet stream to `receiver:514`
5. Generator pushes self-metrics to frontend over chosen real-time channel
6. Frontend polls `GET receiver:4000/stats` every 1 second (received count)
7. Frontend computes drop = `sent - received` per tick
8. After 30 seconds, generator auto-stops; frontend shows final summary

### Architectural Decisions Required in Candidate's README

The candidate must justify in their README:
- Backend language/framework choice (why Go / why Node / etc.)
- Concurrency model (worker pool / cluster / goroutines / threads)
- Rate limiter algorithm (token bucket / ticker / precision considerations)
- Real-time channel choice (SSE / WS / polling — and why)
- Packet generation strategy (generate per tick / pre-generate pool / batch send)

## 3. API Contracts & Receiver Spec

### 3.1 Generator HTTP API (candidate implements)

#### `POST /run` — start a new run

Request:
```json
{ "rate": 10000, "duration": 30 }
```
- `rate`: integer, target packets per second (1 – 100000)
- `duration`: integer seconds, optional (omitted = run until Stop)

Response 200:
```json
{ "runId": "uuid", "startedAt": "2026-04-13T10:00:00Z" }
```

Response 409 — when a run is already in progress:
```json
{ "error": "run_in_progress", "runId": "..." }
```

#### `POST /stop` — stop active run

Response 200:
```json
{ "runId": "uuid", "stoppedAt": "2026-04-13T10:00:30Z", "sent": 298745 }
```

#### `GET /status` — current state (polling fallback)

```json
{
  "running": true,
  "runId": "uuid",
  "targetRate": 10000,
  "actualRate": 9982,
  "sent": 152400,
  "elapsedMs": 15240,
  "remainingMs": 14760,
  "cpu": 78.4,
  "memoryMb": 312
}
```

### 3.2 Generator Real-Time Channel (candidate picks transport)

Regardless of transport (SSE / WebSocket / polling), the event payload schema is:

```json
{
  "ts": "2026-04-13T10:00:01Z",
  "sent": 9982,
  "actualRate": 9982,
  "targetRate": 10000,
  "cpu": 78.4,
  "memoryMb": 312,
  "samples": ["<189>1 2026-04-13T10:00:01Z fgt-01 - - - - date=... srcip=10.0.1.5 ..."]
}
```

- Minimum frequency: **1 Hz** (one update per second)
- `samples`: optional, last 1–5 generated lines (for UI preview)

### 3.3 Receiver Spec (we provide)

**Image:** `ghcr.io/dolusoft/log-challenge-receiver:latest`

**Compose snippet candidate uses:**
```yaml
receiver:
  image: ghcr.io/dolusoft/log-challenge-receiver:latest
  ports:
    - "4000:4000"   # stats HTTP
  networks:
    - challenge
```

**HTTP API (port 4000):**

`GET /stats`:
```json
{
  "total": 298745,
  "perSecond": 9978,
  "lastSecondTotal": 9978,
  "startedAt": "2026-04-13T10:00:00Z",
  "uptimeS": 30,
  "resetAt": "2026-04-13T09:59:55Z"
}
```

`POST /reset`:
```json
{ "ok": true, "resetAt": "2026-04-13T10:00:00Z" }
```

`GET /sample?n=10`:
```json
{ "samples": ["<189>1 ...", "<189>1 ..."] }
```

**Behavior:**
- UDP `:514` — counts every packet, no parsing
- 1-second sliding window for `perSecond`
- Single instance, in-memory only

### 3.4 Syslog & Fortigate Format Spec

**RFC 5424 envelope (mandatory):**
```
<PRI>1 TIMESTAMP HOSTNAME APP-NAME PROCID MSGID STRUCTURED-DATA MSG
```
- `PRI`: `<189>` (facility=local7, severity=notice) — may be fixed
- `TIMESTAMP`: ISO 8601 UTC with millisecond precision (e.g., `2026-04-13T10:00:01.234Z`)
- `HOSTNAME`: `fgt-XX` where XX is random 01–20
- `APP-NAME`, `PROCID`, `MSGID`, `STRUCTURED-DATA`: may be `-` or populated
- `MSG`: Fortigate-style key=value payload

**Fortigate payload example:**
```
date=2026-04-13 time=10:00:01 devname="FGT-01" devid="FG100ETK..." logid="0000000013" type="traffic" subtype="forward" level="notice" vd="root" eventtime=... srcip=10.0.1.5 srcport=51234 srcintf="port1" dstip=8.8.8.8 dstport=443 dstintf="port2" sessionid=12345678 proto=6 action="accept" policyid=1 service="HTTPS" srccountry="Turkey" dstcountry="United States" duration=120 sentbyte=4523 rcvdbyte=89234 user="ahmet.y"
```

**Mandatory randomized fields:** `srcip`, `srcport`, `dstip`, `dstport`, `sessionid`, `duration`, `sentbyte`, `rcvdbyte`, `user`, `time`, `eventtime`.

**Mock data we provide:**
- `mock-data/fortigate-samples.json` — ~20 templates across subtypes (traffic, utm, event, vpn, system)
- `mock-data/usernames.json` — random username pool

## 4. Frontend Requirements (SvelteKit + shadcn-svelte)

### 4.1 Page Structure

Single page is sufficient — `/` (dashboard). Candidate may split into sub-routes but not required.

### 4.2 Components (minimum)

**A. Run Configuration Card** (top)
- Target rate input — number, min 1, max 100000, default 1000
- Duration input — number (seconds), optional, placeholder "Empty = manual stop"
- **Start** button — enabled when no run active
- **Stop** button — enabled when run active
- **Status badge** — `IDLE` / `RUNNING` / `STOPPED`

**B. Live Metrics Cards** (4-card grid)
- **Target vs Actual Rate** — large numbers + deviation percentage (e.g., `9,978 / 10,000  −0.22%`)
- **Total Sent / Received / Drop** — three numbers + drop percentage
- **CPU / Memory** — gauge or large numbers (`78.4%  /  312 MB`)
- **Elapsed / Remaining** — `MM:SS / MM:SS` (no remaining shown if duration omitted)

**C. Throughput Chart** (middle)
- Line chart of per-second sent & received
- At least 60-second window
- Target rate may be drawn as a reference line
- Candidate picks chart library (chart.js / d3 / layerchart / etc.)

**D. Last Logs Preview** (bottom)
- Last 10–20 syslog lines from receiver `/sample` or generator `samples` event
- Monospace font, scrollable
- Lines must visibly differ (sanity check that randomization works)

**E. Run History** (optional bonus)
- Summary list of previous runs (rate, duration, sent, received, drop%)
- In-memory or localStorage — no backend persistence required

### 4.3 Frontend Data Flow

- **Generator real-time channel** (SSE / WS / polling — candidate picks): `sent`, `actualRate`, `cpu`, `memoryMb`, `samples`
- **Receiver polling:** every 1 second `GET receiver:4000/stats` → `total`, `perSecond`
- **Drop calculation:** `sent - total` per tick
- **Start/Stop:** `POST generator/run` & `POST generator/stop` (with `POST receiver/reset` first)

### 4.4 UX Expectations

- **Must use shadcn-svelte components** (Button, Card, Input, Badge, Alert at minimum). README must list which were used.
- **Responsive:** at least 1280 px desktop; tablet/mobile is bonus.
- **Error display:** Alert component when generator/receiver unreachable.
- **Validation:** rate outside 1–100000 must show input error.
- **While run is active:** rate/duration inputs must be disabled (Stop required to change).
- **Dark/light mode:** shadcn-svelte supports by default — bonus, not required.

### 4.5 Restrictions

- No prebuilt admin dashboard templates (Tabler, AdminLTE). shadcn-svelte dashboard blocks are acceptable.
- A chart library that re-renders the entire canvas every tick will freeze UI at 50k/sec — chart performance is a measurement point.
- Frontend must not access UDP directly — all traffic goes through the backend.

## 5. Submission Rules + Rubric + Repo Layout

### 5.1 Template Repo Layout (we provide)

```
log-generator-challenge/
├── README.md                          # challenge brief (frontend-challenge style)
├── docker-compose.example.yml         # receiver + skeleton (candidate completes)
├── docs/
│   ├── requirements.md                # detailed feature spec
│   ├── api-contracts.md               # generator + receiver API
│   ├── syslog-format.md               # RFC 5424 + Fortigate explanation
│   └── evaluation.md                  # rubric + measurement points
├── mock-data/
│   ├── fortigate-samples.json         # ~20 templates
│   └── usernames.json                 # randomization pool
└── receiver/
    └── README.md                      # image usage instructions
```

The candidate adds in their private repo:
```
├── frontend/                          # SvelteKit + shadcn-svelte
├── generator/                         # backend (free choice)
├── docker-compose.yml                 # full working compose
├── README.md                          # architectural decisions here
└── ai/                                # AI conversation history (mandatory)
```

### 5.2 Submission Rules

1. "Use this template" → create a private repo
2. Once finished, add **`zahidzorbaz`** as a collaborator
3. README must justify architectural decisions (backend choice, concurrency model, rate limiter, real-time channel)
4. **`docker-compose up`** must bring everything up in one command (frontend, generator, receiver)
5. Frontend on **localhost:5173** (or similar — declare in README); generator and receiver on internal network
6. Git history must be meaningful — atomic commits, sensible messages
7. TypeScript mandatory for frontend; type safety in backend is a bonus

### 5.3 AI Folder (mandatory)

`ai/` directory must contain:
- **Complete, unedited** conversation history with every AI tool used
- Format is open: text export, screenshots, markdown, JSON, tool-specific export
- If multiple AI tools were used, separate files per tool
- "We expect the full history, not selected highlights" — rule is firm

AI usage does not negatively affect scoring; **a missing or selectively presented AI folder is grounds for disqualification**.

### 5.4 Evaluation Rubric

| Criterion | Weight | What Earns Maximum |
|---|---|---|
| **DevOps** | 25% | • `docker-compose up` works in one command<br>• Backend container correctly limited to 2 CPU / 1 GB<br>• Receiver image integrated cleanly<br>• Network isolation (UDP internal) |
| **Backend** | 30% | • Accuracy within ±2% at 100 / 1k / 10k / 50k rates<br>• Drop < 5% on the docker bridge network<br>• Multi-thread / worker strategy in place<br>• RFC 5424 + Fortigate format correct<br>• Randomization is real (every packet differs) |
| **Frontend** | 25% | • SvelteKit + shadcn-svelte properly used<br>• Real-time metric stream works<br>• Throughput chart, CPU/mem, drop, samples all visible<br>• Validation + error states handled<br>• shadcn components used in appropriate slots |
| **Architecture & Code Quality** | 20% | • README justifies architectural decisions<br>• Code clean, modular, naming clear<br>• Git history meaningful<br>• TypeScript usage (frontend mandatory, backend bonus)<br>• AI folder complete |

**Automatic disqualification:**
- `docker-compose up` does not work
- AI folder missing or selectively curated
- Frontend not SvelteKit / shadcn-svelte
- Cannot send even 1,000 / sec (the lowest threshold)

### 5.5 Time Expectation Statement (in README)

> This challenge is designed to take approximately **4–5 days** (4–6 hours per day).

### 5.6 Bonus Areas (raise the score)

- Run history persistence (SQLite / file)
- Multi-receiver support (load-balanced UDP send)
- Histogram / latency distribution
- Dark mode, animated transitions
- Prometheus `/metrics` endpoint on backend
- E2E tests (frontend), integration tests (backend)

## 6. Open Items / Out of Scope

**Out of scope (explicitly):**
- TLS / authentication on generator API (local-only tool)
- Multi-user / multi-tenant
- Persistent run storage (in-memory only, run history is optional bonus)
- Log parsing / search / SIEM features (we are the generator side, not the consumer)

**Receiver image build:**
- Image needs to be built and published to `ghcr.io/dolusoft/log-challenge-receiver:latest` before the challenge is sent to candidates. This is a separate task from this spec.

**Mock data preparation:**
- `fortigate-samples.json` (~20 templates) and `usernames.json` need to be authored before challenge release. Anonymized real Fortigate logs preferred.
