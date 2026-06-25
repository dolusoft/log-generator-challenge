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
- Exposes an HTTP server on port `4000` for stats and reset.
- All state is in-memory; counters are zeroed on container restart or via `POST /reset`.

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
