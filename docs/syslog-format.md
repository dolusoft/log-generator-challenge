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
