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
