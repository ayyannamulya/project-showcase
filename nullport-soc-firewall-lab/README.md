# 🛡 NullPort

### AI-Driven Adaptive Firewall & SOC Behavior Lab

NullPort is an AI-assisted firewall behavior analysis platform that
simulates, detects, and autonomously responds to network threats using
behavioral modeling and risk-based containment strategies.

This project demonstrates applied AI in defensive security engineering
by combining real-time traffic ingestion, anomaly detection, dynamic
firewall automation, and SOC-style alert workflows.

------------------------------------------------------------------------

## 🎯 Project Vision

Traditional firewalls rely heavily on static rule matching.\
NullPort explores a more adaptive security model:

> **Behavior Analysis → Risk Modeling → Automated Containment → SOC
> Feedback Loop**

The system continuously evaluates traffic patterns, detects malicious
intent using behavioral signals, and dynamically adjusts firewall
posture based on calculated risk levels.

------------------------------------------------------------------------

## 🧠 AI Security Engineering Focus

NullPort highlights practical AI-driven security principles:

-   Behavioral anomaly detection\
-   Entropy-based traffic analysis\
-   Risk scoring modeling\
-   Adaptive containment automation\
-   Feedback-driven rule evolution\
-   Attack pattern profiling\
-   Distributed microservice architecture

Rather than relying solely on static signatures, NullPort evaluates
attacker behavior over time and assists SOC teams with contextualized
risk intelligence.

------------------------------------------------------------------------

## 🏗 Architecture Overview

simulator/ → api/ → engine/ → PostgreSQL (Railway)\
(Python) (TypeScript + Express + Drizzle) (Python FastAPI)

### 🔹 simulator/

Python-based attack generator simulating:

-   Port scans\
-   Brute force attempts\
-   High-frequency bursts\
-   Beaconing patterns

### 🔹 api/

Core orchestration layer:

-   Traffic ingestion\
-   Firewall rule evaluation engine\
-   Risk threshold enforcement\
-   Alert lifecycle management\
-   Attacker profiling\
-   Adaptive rule insertion

Built with: - TypeScript\
- Express.js\
- Drizzle ORM\
- PostgreSQL (Railway)

### 🔹 engine/

AI-powered detection microservice:

-   Behavioral feature extraction\
-   Port entropy calculation\
-   Frequency spike detection\
-   Persistence scoring\
-   Risk classification

Extensible for machine learning models (Isolation Forest, Autoencoder,
etc.).

------------------------------------------------------------------------

## 🔥 Core Capabilities

### 1️⃣ Behavioral Detection

NullPort evaluates:

-   Port distribution entropy\
-   Connection frequency over sliding time windows\
-   Multi-port scanning behavior\
-   Repeated access persistence

This enables detection of novel, polymorphic, or evolving attack
strategies.

------------------------------------------------------------------------

### 2️⃣ Risk Scoring Model

risk_score = weighted_frequency + weighted_entropy + persistence_score

  Score Range   Severity
  ------------- ----------
  0--40         Low
  41--70        Medium
  71--85        High
  86--100       Critical

High-risk actors automatically trigger containment logic.

------------------------------------------------------------------------

### 3️⃣ Adaptive Firewall Automation

When risk exceeds a defined threshold:

-   Temporary block rule inserted\
-   TTL-based expiration applied\
-   Attacker profile updated\
-   SOC alert generated

This simulates near real-time AI-assisted firewall decision-making.

------------------------------------------------------------------------

### 4️⃣ SOC Workflow Simulation

NullPort models realistic SOC operations:

-   Alert lifecycle (new → triaged → investigating → contained →
    closed)\
-   Investigation notes\
-   Threat actor history\
-   Containment tracking

AI enhances analyst efficiency without replacing human expertise.

------------------------------------------------------------------------

## 📊 API Documentation

Interactive API documentation is available via Swagger:

🔗 Swagger UI: https://your-deployed-api-url.com/docs

This allows recruiters and reviewers to:

-   Explore available endpoints\
-   Inspect request/response schemas\
-   Test API calls directly\
-   Understand system orchestration flow

------------------------------------------------------------------------

## 📘 Project Documentation

Detailed documentation covering:

-   System architecture\
-   Detection logic design\
-   Risk scoring methodology\
-   Database schema design\
-   Adaptive containment strategy

🔗 Technical Documentation: https://your-project-docs-url.com

------------------------------------------------------------------------

## 🗄 Database Design Overview

Core entities:

-   traffic_logs\
-   detections\
-   firewall_rules\
-   alerts\
-   alert_notes\
-   attackers

Optimized for:

-   Time-window behavioral queries\
-   Risk recalculation\
-   Event-to-actor correlation\
-   Efficient PostgreSQL usage

------------------------------------------------------------------------

## 🛠 Technology Stack

  Layer              Technology
  ------------------ ----------------------
  API                TypeScript + Express
  ORM                Drizzle ORM
  Database           PostgreSQL (Railway)
  Detection Engine   Python + FastAPI
  Simulation         Python CLI
  Infrastructure     Railway

------------------------------------------------------------------------

## 📜 License

MIT License
