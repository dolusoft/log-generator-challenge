# Receiver Service

The receiver is a small UDP listener + HTTP stats server that we provide as a prebuilt Docker image. **Do not modify or replace it** — every candidate is evaluated against the same receiver, and the image is the reference implementation we use for scoring.

The receiver is the [`dolusoft/syslog-sink`](https://github.com/dolusoft/syslog-sink) tool, a general-purpose UDP syslog sink. In the challenge `docker-compose`, the service is named `receiver` (for DNS discovery on the challenge network); the underlying image is `syslog-sink`.

## Image

```
ghcr.io/dolusoft/syslog-sink:latest
```

## Behavior

- Listens on UDP port `514` for syslog packets and increments counters.
- Does **not** parse or validate packet contents — every datagram counts.
- Exposes an HTTP server on port `4000` for stats, reset, and the recording/efficiency API.
- All state is in-memory; counters are zeroed on container restart or via `POST /reset`.
- **Measures efficiency receiver-side:** it reads your **`generator`** container's CPU from
  Docker stats and divides received EPS by the cores used. The generator never reports its own
  CPU, so the efficiency number cannot be inflated by the sender.

## How to wire it into your `docker-compose.yml`

```yaml
services:
  receiver:
    image: ghcr.io/dolusoft/syslog-sink:latest
    container_name: receiver
    ports:
      - "4000:4000"      # stats HTTP, also reachable from the host
    networks:
      - challenge
    # Lets the receiver read the generator container's CPU for the EPS/core
    # efficiency metric. Do NOT remove these — without them efficiency is
    # unmeasurable. The target name must match your generator's container_name.
    environment:
      - SYSLOG_SINK_TARGET_CONTAINER=generator
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro

networks:
  challenge:
    driver: bridge
```

The receiver's UDP port `514` does not need to be published to the host — your generator reaches
it on the internal docker network. Your generator service **must** be named
`container_name: generator` so the receiver can find it.

## HTTP API quick reference

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/stats` | totals and per-second window |
| `POST` | `/reset` | zero counters |
| `GET` | `/sample?n=10` | last N raw packet bodies (debug) |
| `POST` | `/rec` | arm a recording window (resets counters, starts watching the generator) |
| `POST` | `/stop` | freeze the window and compute the efficiency result (`ResultDto`) |
| `GET` | `/result` | last finished `ResultDto`; `409` while armed/recording/idle |
| `POST` | `/measure?secs=30` | **one-shot**: arm → hold `secs` → stop → return the `ResultDto` inline |

Full request/response schemas: see [`docs/api-contracts.md`](../docs/api-contracts.md).

## Measuring efficiency (EPS/core)

The headline challenge metric is **received EPS ÷ generator cores used**. The receiver computes it
from a *recording window* — a span during which it samples received EPS and the generator's CPU
once per second, then reports the highest sustained plateau.

The simplest way to capture a result is one call while your generator runs at its target rate:

```bash
# Start your generator (e.g. 300k/sec) from the dashboard, then:
curl -X POST "http://localhost:4000/measure?secs=30"
```

This arms a window, holds it for 30 s (default; clamp 1–300), stops, and returns the `ResultDto`
directly. The manual equivalent is `POST /rec` → wait → `POST /stop` → `GET /result`. Key fields:

- `sustainedEps` — highest 1-second EPS average held for ≥10 s within ±5%.
- `sustainedEff` — **EPS/core** over that window (the ranking metric).
- `minThresholdMet` — `true` only if the sustained plateau reaches the 100k floor.

A meaningful verdict needs ≥ ~15 s of steady traffic (the sustained segment requires a 10 s
stable plateau). Start the generator **before** calling `/measure` — the window is wall-clock
from arming.

## Security: Docker socket access

The receiver reads the generator's CPU through the Docker socket. **Mounting `/var/run/docker.sock`
into a container grants it effectively host-root** — the `:ro` flag only makes the socket *file*
read-only; the Docker API behind it can still create privileged containers. So this mount is a
real privilege grant, not a read-only convenience.

What protects you in this challenge:

- **Pin the receiver by digest.** Reference it as `ghcr.io/dolusoft/syslog-sink@sha256:<digest>`
  (not a mutable tag) so only the exact verified image can ever hold the socket — a candidate
  must not modify or swap the receiver (doing so is an automatic disqualification). Verify the
  digest out of band before each evaluation.
- Evaluators run **untrusted candidate submissions**. Always `docker compose up` a submission in a
  **disposable VM or throwaway sandbox**, never on a host with anything sensitive — this is the
  right control regardless of the socket mount.

**Least-privilege alternative (recommended while the image is on a mutable tag).** Instead of the
raw socket, front it with a read-only Docker socket proxy (e.g. `tecnativa/docker-socket-proxy`)
that allows only `GET /containers/{id}/stats` and denies every write/exec/create endpoint, then
point the receiver at the proxy. This caps the blast radius to read-only stats even if the receiver
were compromised. Routing the receiver through a TCP proxy needs a small receiver-side option to
target a non-default Docker endpoint (a planned enhancement) — until both that lands and the image
is digest-pinned, the proxy is the safer default.

## Health check

After `docker compose up`, verify the receiver is reachable:

```bash
curl http://localhost:4000/stats
```

Expected: a JSON response with `total: 0` and `perSecond: 0` immediately after boot.
