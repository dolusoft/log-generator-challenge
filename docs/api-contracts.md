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
