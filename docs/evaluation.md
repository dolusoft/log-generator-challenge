# Evaluation

## Measurement Procedure

We evaluate by running your project locally, exactly as a user would. The provided receiver
is wired to watch your **`generator`** container (Docker socket + `SYSLOG_SINK_TARGET_CONTAINER`,
already set in the compose snippet), so it measures efficiency receiver-side.

1. `git clone <your repo>`
2. `cd <your repo> && docker compose up -d`
3. Open `http://localhost:5173` in a browser; confirm `curl http://localhost:4000/stats` responds.

### A. Efficiency run (the headline)

4. Set a high target rate (we sweep, e.g. **200k / 500k**) and **Start** the generator.
5. Once it ramps, take a timed measurement on the receiver — one call does it all:
   - `POST http://localhost:4000/measure?secs=30` → returns the `ResultDto` directly, **or**
   - the manual equivalent: `POST /rec` → wait ~30 s → `POST /stop` → `GET /result`.
6. From the `ResultDto` we record: **`sustainedEps`**, **`sustainedEff` (EPS/core — the ranking
   metric)**, generator **cores used**, **`minThresholdMet`**, and drop.

### B. Accuracy run (correctness)

7. For each paced rate (**1k / 10k / 100k**): set rate, duration 30 s, **Start**, let it complete.
   Record target vs the receiver's per-second total and drop %; accuracy must hold within ±2%.

### C. Robustness

8. We also exercise: Stop mid-run, change rate, restart; invalid input; receiver-unreachable.

> The receiver counts received EPS itself and reads the generator's CPU from Docker stats — the
> generator never reports its own CPU, so the **EPS/core** number cannot be inflated by the sender.

## Scoring Rubric

| Criterion | Weight | What earns maximum |
|---|---|---|
| **DevOps** | 20% | • `docker compose up` works in one command and reaches ready state within 60s<br>• Backend container has correct `deploy.resources.limits: cpus: '2', memory: 1G`<br>• Receiver image integrated cleanly via the provided snippet (incl. the generator-watch wiring)<br>• Network isolation correct: UDP traffic stays internal, only HTTP ports published as needed |
| **Backend** | 25% | • Rate accuracy within ±2% at the paced rates (1k / 10k / 100k)<br>• Drop < 5% on the docker bridge network<br>• Multi-thread / worker / cluster strategy clearly visible in code<br>• RFC 5424 envelope correct; Fortigate payload includes all required keys for chosen subtype<br>• Randomization is real — no two consecutive packets identical, fields vary as specified |
| **Efficiency** | 20% | • Sustains the **≥100k EPS floor** for ≥10 s with **< 1% drop** (`minThresholdMet: true`)<br>• High **EPS/core** (`sustainedEff`) — approaches the reference band of ~150–200k EPS/core<br>• Worker count matched to the rate (too few can't reach it; right count spreads it over fewer cores)<br>• Memory stays modest — this workload is CPU-bound, not memory-bound |
| **Frontend** | 20% | • Built with SvelteKit + shadcn-svelte; shadcn components used in appropriate slots<br>• Real-time metric stream works smoothly; chart updates without freezing UI even past 100k/sec<br>• All required metrics visible: target vs actual rate, sent/received/drop, CPU/memory, elapsed/remaining, last samples<br>• Validation handled (rate range, disabled inputs during run)<br>• Error states handled (generator/receiver unreachable shows a clear Alert) |
| **Architecture & Code Quality** | 15% | • README justifies the five required architectural decisions in clear language<br>• Code is modular, naming is consistent, no dead code<br>• Git history shows incremental progress with meaningful commit messages<br>• TypeScript used in frontend (strict mode preferred); backend type safety is a bonus<br>• `ai/` folder is complete and unedited |

## Automatic Disqualification

Any of the following results in disqualification regardless of other scores:

- `docker compose up` fails or hangs.
- Frontend is not SvelteKit + shadcn-svelte.
- **Cannot sustain the 100,000 EPS floor** for ≥10 s within the 2 CPU / 1 GB limit (`minThresholdMet: false`) — the generator never genuinely reached the bar.
- `ai/` folder is missing, or content is selectively curated rather than the complete history.
- The provided receiver image was modified, replaced, or pinned to a different digest than the one in the compose snippet.

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
