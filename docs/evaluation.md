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
