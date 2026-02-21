<div align="center">

<img src="https://img.shields.io/badge/status-production--ready-22c55e?style=for-the-badge" />
<img src="https://img.shields.io/badge/tests-102%20passing-22c55e?style=for-the-badge" />
<img src="https://img.shields.io/badge/license-MIT-3b82f6?style=for-the-badge" />

<br />
<br />

```
███╗   ██╗██╗   ██╗██╗     ██╗     ██████╗  ██████╗ ██████╗ ████████╗
████╗  ██║██║   ██║██║     ██║     ██╔══██╗██╔═══██╗██╔══██╗╚══██╔══╝
██╔██╗ ██║██║   ██║██║     ██║     ██████╔╝██║   ██║██████╔╝   ██║   
██║╚██╗██║██║   ██║██║     ██║     ██╔═══╝ ██║   ██║██╔══██╗   ██║   
██║ ╚████║╚██████╔╝███████╗███████╗██║     ╚██████╔╝██║  ██║   ██║   
╚═╝  ╚═══╝ ╚═════╝ ╚══════╝╚══════╝╚═╝      ╚═════╝ ╚═╝  ╚═╝   ╚═╝   
```

### Adaptive Firewall Behavior Deception Lab

*A full-stack security operations platform that detects attack patterns using behavioral analysis and machine learning, then applies adaptive containment automatically.*

<br />

</div>

---

## Overview

NullPort is a production-grade SOC (Security Operations Center) automation system built from scratch. It simulates network traffic ingestion, runs behavioral detection and ML anomaly scoring, manages the full alert lifecycle, and autonomously blocks, rate-limits, or tarpits attackers based on risk — all through a typed REST API backed by a real PostgreSQL database.

The project covers the full stack from database schema design through Python ML microservices, with 102 passing tests across both runtimes.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        simulator/                               │
│              Python CLI · 5 named attack profiles               │
│         port_sweep · brute_ssh · distributed_scan               │
│                  slow_burn · exfil_sim                          │
└───────────────────────┬─────────────────────────────────────────┘
                        │  POST /traffic
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                          api/                       :3000        │
│                   TypeScript · Express                           │
│                                                                  │
│  ┌─────────────────┐   ┌──────────────────┐                     │
│  │ Firewall Engine │   │ Behavior Windows  │                     │
│  │ Priority rules  │   │ 5min · 1hr aggreg │                    │
│  │ Tarpit · Expiry │   │ per source IP     │                     │
│  └─────────────────┘   └──────────────────┘                     │
│                                                                  │
│  ┌─────────────────┐   ┌──────────────────┐                     │
│  │ Adaptive        │   │ SOC Alert         │                    │
│  │ Response Engine │   │ Lifecycle         │                     │
│  │ Block·Rate·Trap │   │ Triage → Contain  │                    │
│  └─────────────────┘   └──────────────────┘                     │
└───────────────────────┬─────────────────────────────────────────┘
                        │  POST /analyze
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                        engine/                      :8001        │
│                  Python · FastAPI                                │
│                                                                  │
│   Entropy Score  ·  Frequency Score  ·  Persistence Score       │
│   Isolation Forest (scikit-learn) · Anomaly Detection           │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Railway PostgreSQL                             │
│  firewall_rules · traffic_logs · detections · attackers          │
│  alerts · alert_notes · whitelist                               │
│  Cron: log cleanup · rule expiry · reputation decay             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| API | TypeScript 5, Express 4, Drizzle ORM |
| Database | PostgreSQL on Railway |
| Detection Engine | Python 3.11, FastAPI, Pydantic |
| Machine Learning | scikit-learn Isolation Forest |
| Attack Simulator | Python, Click, HTTPX, Rich |
| API Documentation | Swagger / OpenAPI 3.0 |
| Validation | Zod (API), Pydantic (engine) |
| Logging | Winston (structured JSON) |
| Testing | Jest + Supertest, pytest |
| Infrastructure | Docker, Docker Compose |
| Cron Jobs | node-cron |

---

## Core Features

### Firewall Rule Engine
Priority-based rule matching (lower number = higher priority). Rules specify source IP, destination port, protocol, and action. Unspecified fields act as wildcards. Supports static permanent rules and dynamic rules with expiry timestamps inserted automatically by the adaptive response engine.

| Action | Behaviour |
|---|---|
| `allow` | Traffic passes through |
| `deny` | Packet dropped, no response |
| `rate_limit` | Throttled to N requests per minute |
| `tarpit` | 5-second artificial delay — wastes attacker time |

### Behavioral Detection Engine
Each traffic event is scored across three axes using a pre-aggregated behavior window pulled from the last 5 minutes and 1 hour of activity for the source IP.

```
risk_score = (frequency × 0.40) + (entropy × 0.35) + (persistence × 0.25)
```

- **Entropy score** — Shannon entropy of unique ports targeted. High diversity = port scan.
- **Frequency score** — Request rate in 5-minute and 1-hour windows. High rate = brute force or DoS.
- **Persistence score** — How long the IP has been active and how many total attempts it has made.
- **Honeypot bonus** — Any traffic to honeypot ports (Telnet, VNC, MSSQL, RDP, Metasploit) adds +20 to the score.

### Isolation Forest Anomaly Detection
A scikit-learn Isolation Forest is trained on synthetic normal traffic at engine startup. Each event is scored across 6 features: packet size, destination port, 5-minute count, 1-hour count, unique port count, and honeypot hits. Events that cannot be isolated quickly are flagged as anomalies. When the ML model flags an event that the rule-based scorer rates as low risk, a +15 soft boost is applied.

### Adaptive Response Engine
After detection, the system automatically decides what to do:

| Risk Level | Reputation | Response | Duration |
|---|---|---|---|
| Low | Any | Monitor only | — |
| Medium | ≥ 40 | Rate limit | 30 min |
| Medium | Any | Temp block (honeypot hit) | 60 min |
| High | ≥ 40 | Temporary block | 60 min |
| High | < 40 | Permanent block | Permanent |
| Critical | Any | Permanent block | Permanent |

Analysts can override with manual block, tarpit, or unblock via dedicated endpoints.

### Attacker Reputation System
Every source IP builds a profile starting at 100 (clean). Each alert deducts points based on severity. A background cron job runs hourly and increments reputation by 1 for IPs that have gone quiet — a slow forgiveness mechanism. IPs below 40 reputation are permanently blocked regardless of current risk level.

| Alert Level | Reputation Deduction |
|---|---|
| Critical | −30 |
| High | −15 |
| Medium | −5 |
| Low | 0 |

### SOC Alert Lifecycle
Alerts are created automatically for medium, high, and critical detections. They follow a strict state machine with enforced transitions. Invalid transitions (e.g. closed → new) are rejected with a 400.

```
new → triaged → investigating → contained → closed
 └─────────────────────────────────────────→ false_positive
```

Analysts can add timestamped investigation notes to any alert at any stage.

### Background Cron Jobs

| Job | Schedule | Action |
|---|---|---|
| Traffic cleanup | Every hour at :00 | Deletes `traffic_logs` older than retention window |
| Rule expiry | Every 5 minutes | Removes expired dynamic firewall rules |
| Reputation decay | Every hour at :30 | +1 reputation for inactive attackers |

---

## API Reference

### Health
```
GET  /health                     System health + engine status
GET  /health/model               Isolation Forest model status (engine)
```

### Firewall Rules
```
GET    /firewall/rules           All rules sorted by priority
GET    /firewall/rules/active    Non-expired rules only
POST   /firewall/rules           Create rule
DELETE /firewall/rules/:id       Delete rule
```

### Traffic Ingestion
```
POST   /traffic                  Ingest and evaluate a traffic event
GET    /traffic                  Recent traffic logs (filterable)
GET    /traffic/behavior/:ip     Behavior window for a source IP
```

### Adaptive Response
```
POST   /response/block           Manually block an IP
POST   /response/tarpit          Manually tarpit an IP
DELETE /response/block/:ip       Unblock an IP
GET    /response/status/:ip      Current containment status
```

### Alerts
```
GET    /alerts                   List alerts (filter: status, limit)
GET    /alerts/:id               Single alert with notes
PATCH  /alerts/:id               Update status / assign analyst
POST   /alerts/:id/notes         Add investigation note
GET    /alerts/:id/notes         Read all notes
```

### Attackers
```
GET    /attackers                All profiles sorted by reputation
GET    /attackers/:ip            Full profile with recent alerts
PATCH  /attackers/:ip            Update notes or status
```

### Metrics
```
GET    /metrics                  Live SOC dashboard — all aggregated stats
```

Full interactive documentation available at `/api-docs` (Swagger UI).

---

## Attack Simulator Profiles

The simulator is a Python CLI that sends crafted traffic to the API, each profile targeting a different detection pathway.

```bash
python -m src.main --list-profiles
```

| Profile | Packets | Delay | Detection Target |
|---|---|---|---|
| `port_sweep` | 50+ | 50ms | Entropy (high unique port count) |
| `brute_ssh` | 80+ | 100ms | Frequency (SSH port 22 only) |
| `distributed_scan` | 50+ | 200ms | Multi-IP behavior window isolation |
| `slow_burn` | Any | 3000ms | Persistence (evades basic rate limits) |
| `exfil_sim` | 30+ | 500ms | Large packets + unusual high ports |

---

## Database Schema

Seven tables covering the full operational surface:

| Table | Purpose |
|---|---|
| `firewall_rules` | Priority-ordered rules with optional expiry |
| `traffic_logs` | Every ingested packet (purged after 48h by cron) |
| `detections` | Engine analysis result per traffic event |
| `attackers` | Per-IP profiles with reputation scoring |
| `alerts` | SOC alerts with full lifecycle status |
| `alert_notes` | Timestamped analyst notes per alert |
| `whitelist` | IPs exempt from adaptive response |

All tables use UUID primary keys, `created_at` / `updated_at` timestamps, and are indexed for the access patterns used during traffic ingestion and SOC queries.

---

## Setup

### Requirements
- Node.js 20+
- Python 3.11+
- PostgreSQL (Railway recommended)

### Environment Variables

Copy `api/.env.example` to `api/.env` and fill in:

| Variable | Default | Required |
|---|---|---|
| `DATABASE_URL` | — | ✅ |
| `PORT` | `3000` | |
| `NODE_ENV` | `development` | |
| `ENGINE_URL` | `http://localhost:8001` | |
| `LOG_LEVEL` | `info` | |
| `TRAFFIC_RETENTION_HOURS` | `48` | |

### Database Migrations

```bash
psql $DATABASE_URL < api/src/db/migrations/0001_initial_schema.sql
psql $DATABASE_URL < api/src/db/migrations/0002_whitelist_indexes.sql
```

### Start Services

```bash
# Detection engine (trains Isolation Forest model on startup)
cd engine && uvicorn src.main:app --reload --port 8001

# API (starts cron jobs automatically)
cd api && npm run dev
```

### Docker

```bash
docker compose up
```

---

## Testing

```bash
# TypeScript API — 74 tests
cd api && npm test

# Python engine — 28 tests
cd engine && pytest tests/ -v
```

**Total: 102 tests across both runtimes — all passing.**

Test coverage spans:

- Firewall rule matching logic (priority, wildcard, honeypot flags)
- Traffic ingestion validation (IP format, port bounds, protocol enum)
- Detection pipeline (entropy, frequency, persistence algorithms)
- Isolation Forest (training, score bounds, anomaly vs normal comparison)
- Adaptive response decisions (all risk levels, honeypot escalation, reputation escalation)
- Alert lifecycle (all valid transitions, invalid transition rejection)
- Attacker profile updates and metrics aggregation
- Cron job cleanup logic and whitelist service

---

## License

MIT