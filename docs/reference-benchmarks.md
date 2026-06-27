# Reference Benchmarks

These numbers come from **our own reference generator** measured against the exact
receiver you are given (`dolusoft/syslog-sink`). They exist to give you a concrete
performance bar to aim for — not a hard pass/fail. A strong submission gets close to,
or beats, the efficiency figures below.

> **The comparable metric is EPS per CPU core**, not absolute EPS. Your generator runs
> under the challenge's container resource limits; ours was measured unconstrained.
> Efficiency (events/sec ÷ cores used) is normalized, so it is the number to compare
> yourself against.

## How these were measured

The receiver measures everything **receiver-side** — it counts the UDP it actually
receives and reads the *generator container's* CPU via `docker stats`. The generator
never reports its own CPU, so the efficiency number cannot be inflated by the sender.

- **Sustained EPS** — the highest 1-second average the receiver held for ≥10s within ±5%.
- **EPS/core** — sustained EPS ÷ generator cores used in that window (the efficiency metric).
- **Host:** 12 logical cores, **no CPU limit** on the generator. Payload: Fortigate-style, UDP.

## Results

### Steady, paced rates

| Target | Sustained EPS | **EPS/core** | Cores used | Loss |
|------:|--------------:|-------------:|-----------:|-----:|
| 100k  | 100,043 | **123,941** | 0.81 | ~0% |
| 250k  | 250,081 | **136,689** | 1.83 | ~0% |
| 500k  | 500,033 | **142,703** (4 workers) | 3.50 | ~1.7% |
| 500k  | 498,548 | **197,339** (8 workers) | 2.53 | ~3% |

At a fixed target rate, **matching the worker count to the rate** pushes efficiency up
sharply (~140k → ~197k EPS/core at 500k). Too few workers can't reach the rate at all
(2 workers cap out near 250k). Memory stayed flat at **~2 MB** — this workload is
CPU-bound, not memory-bound.

### Ceilings (unconstrained, "blast" mode)

| Workers | Sustained / Peak EPS | Cores | Loss | Note |
|:------:|---------------------:|------:|-----:|------|
| 4  | 640,270 / 663,862 | 4.42 | **0.08%** | **Lossless ceiling** — 4 workers don't saturate the host |
| 8  | — / 2,006,422 | 8.79 | 7.5% | Receiver approaching its limit |
| 12 | 1,999,514 / 2,093,797 | 9.79 | 42% | Absolute peak; receiver saturates, excess is dropped |

At 12 workers the generator *sent* ~3.3M packets/sec but the receiver only absorbed
~2.0M — so **~2.0M EPS is the receiver's ceiling on this host**, not the generator's
send limit. The practical **lossless** maximum here is **~640k EPS with 4 workers**.

## What "good" looks like for this challenge

- **Floor:** comfortably sustain the minimum throughput the challenge asks for, with low drop.
- **Efficiency:** approach **~150–200k EPS/core**. That is the band a well-tuned generator
  (token-bucket pacing, worker count matched to the rate, no per-packet allocation) reaches.
- **Headroom:** within a small CPU budget you should still have room above the floor — the
  reference sustains well past the minimum on under one core.

If your generator lands in that efficiency band and stays lossless at the required rate,
you are matching the reference.
