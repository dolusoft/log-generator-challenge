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
| `rate` | integer | yes | Target packets per second, 1–1000000 |
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
  "message": "rate must be between 1 and 1000000"
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

### Recording & efficiency endpoints

These power the headline metric (**EPS/core**). The receiver samples received EPS and the
generator's CPU once per second over a *recording window*, then reports the highest sustained
plateau. The generator never self-reports CPU, so the number cannot be inflated.

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/rec` | arm a window (resets counters; starts watching the `generator` container) → `{ state, elapsedMs }` |
| `POST` | `/stop` | freeze the window + compute the result → `ResultDto` |
| `GET` | `/result` | last finished `ResultDto`; `409` while armed/recording/idle |
| `POST` | `/measure?secs=30` | one-shot: arm → hold `secs` (default 30, clamp 1–300) → stop → returns `ResultDto`; `409` if already recording |

**`ResultDto` (200):**
```json
{
  "peakEps": 312044,
  "totalReceived": 8901230,
  "durationS": 30,
  "sustainedEps": 300120.5,
  "sustainedCpuPct": 152.3,
  "sustainedEff": 197060.4,
  "bestEffEps": 300120.5,
  "bestEffCpuPct": 152.3,
  "bestEff": 197060.4,
  "fullrunEps": 296700.0,
  "fullrunCpuPct": 158.1,
  "fullrunEff": 187665.0,
  "genCpuPeak": 165.2,
  "genMemPeak": 2231132,
  "generatorMetricsAvailable": true,
  "minThresholdMet": true
}
```

| Field | Type | Notes |
|---|---|---|
| `sustainedEps` | float \| null | Highest 1 s EPS average held ≥ 10 s within ±5%; `null` if no such plateau |
| `sustainedEff` | float \| null | **EPS/core** over that plateau — the ranking metric |
| `sustainedCpuPct` | float \| null | Generator CPU over the plateau (raw Docker %, `100` = 1 core) |
| `minThresholdMet` | bool | `true` only if `sustainedEps` reaches the 100,000 floor |
| `generatorMetricsAvailable` | bool | `false` if the receiver couldn't read the generator's CPU (check the `docker.sock` mount + `SYSLOG_SINK_TARGET_CONTAINER`) |
| `peakEps` | integer | Highest single-second EPS in the window |
| `genMemPeak` | integer \| null | Peak generator memory (bytes) |

> **cores used = `cpuPct / 100`**, and **`sustainedEff = sustainedEps / (sustainedCpuPct / 100)`**.
> An empty `sustainedEps`/`sustainedEff` means the window had no ≥10 s stable plateau (too short or
> too noisy) — record a longer, steadier run.

### UDP — port 514

The receiver listens on UDP `:514` on the internal docker network. Every packet increments counters. There is no parsing or validation — the receiver counts every datagram.

## Frontend's expected call sequence (for one run)

1. `POST receiver:4000/reset`
2. `POST generator:3000/run { rate, duration? }`
3. Subscribe to generator real-time channel; poll `receiver:4000/stats` every 1s.
4. On Stop button: `POST generator:3000/stop`. (When duration auto-expires the generator stops itself; the frontend should still close subscriptions.)
