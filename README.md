# AegisSOC: Enterprise AI Data Center Cyber Attack Prediction & Threat Detection Platform

**AegisSOC** is an enterprise-grade, production-ready Security Operations Center (SOC) Command Suite designed for core data centers, cloud infrastructure, and enterprise network perimeters. It provides an automated, continuous 6-stage cybersecurity defense lifecycle:
$$\text{Observe} \longrightarrow \text{Analyze} \longrightarrow \text{Predict} \longrightarrow \text{Alert} \longrightarrow \text{Investigate} \longrightarrow \text{Report}$$

---

## 🛡️ Core Highlights & Features

### 1. Module 1: Data Ingestion & Feature Preprocessing (`Observe`)
- **Multi-Format Ingestion**: Real-time synthetic NetFlow v9 telemetry, live mock Syslog (RFC 5424 / CEF Common Event Format), and batch CSV ingestion.
- **Universal CSV Parser**: Automatically detects and normalizes headers for **NSL-KDD** and **CICIDS2017** benchmark datasets.
- **Normalization Pipeline**: MinMax Scaling ($[0, 1]$ bounds) and Standard Scaling across core feature vectors: `packet_rate` (pps), `bytes_per_sec`, `ports_accessed`, `failed_logins`, `active_connections`, `srv_count`, and `protocol`.

### 2. Module 2: AI/ML Threat Prediction Engine (`Analyze & Predict`)
- **Hybrid Dual-Layer ML Scoring**:
  1. **Unsupervised Anomaly Detector**: Lightweight Isolation Forest partitioning feature distributions to compute path length anomaly scores $a \in [0, 1]$.
  2. **Supervised Attack Classifier**: Random Forest ensemble predicting probability distributions across 5 classes:
     - `Normal`
     - `DDoS Attack` (volumetric SYN flood, high packet rate $>10\text{k pkts/s}$)
     - `Port Scan` (rapid elevation in distinct ports probed across subnets)
     - `Suspicious Login / Brute Force` (repeated authentication failures on SSH/RDP/Kerberos)
     - `Malware Activity / Lateral Movement` (anomalous byte distribution, C2 beaconing)
- **Blended Risk Formula**:
  $$\text{Anomaly Score } (a) = \min\left(1.0, \left(\frac{\text{packetRate}}{15000}\right) \times 0.5 + \left(\frac{\text{portsAccessed}}{100}\right) \times 0.3 + \left(\frac{\text{failedLogins}}{20}\right) \times 0.2\right)$$
  $$\text{Attack Probability } (p_a) = 1.0 - P(\text{Normal})$$
  $$\text{Composite Risk Score} = \text{round}\left(100 \times (0.4 \times a + 0.6 \times p_a)\right)$$
- **Dynamic Severity Thresholds**:
  - `Critical`: $\text{Risk} \ge 85$
  - `High`: $70 \le \text{Risk} < 85$
  - `Medium`: $40 \le \text{Risk} < 70$
  - `Low`: $\text{Risk} < 40$
- **5-Minute Short-Term Predictive Forecasting**: Ordinary Least Squares (OLS) trendline calculating projected risk, slope velocity ($\Delta\text{Risk}/\min$), and trend direction ($\nearrow$ Escalating, $\searrow$ De-escalating, $\rightarrow$ Stable).

### 3. Module 3: Alert Management & Deduplication Queue (`Alert`)
- Automated alert triggers when Composite Risk $\ge 70$.
- **5-Minute Sliding-Window Deduplication**: Repeating alarms of identical signature and source IP are aggregated, tracking recurrence counts (e.g. `x14 in 5m`) and peak risk scores without flooding analyst queues.
- Real-time status transitions: `Open` $\rightarrow$ `Investigating` $\rightarrow$ `Resolved`.
- **1-Click Escalation**: Attach single or multiple alerts directly to an official Incident Case.

### 4. Module 4: Forensic Evidence Locker & SHA-256 Immutability (`Investigate`)
- Official Incident Cases (`CASE-2026-001`) with investigator notes, analyst assignments, and linked telemetry.
- **Web Crypto API SHA-256 Hashing**: Instant 64-character hexadecimal checksum generation in the client browser upon artifact drag-and-drop.
- **Recalculate & Verify Integrity**:
  - `Integrity Verified: Match` (emerald badge)
  - `TAMPER DETECTED: Hash Mismatch` (flashing crimson alarm)
- **Simulate Bit Rot / Adversary Tamper**: Interactive toggle to mutate file contents and verify tamper detection.

### 5. Module 5: Tamper-Evident Hash-Chained Audit Ledger (`Integrity`)
- Every security action is appended to an immutable forward-linked blockchain ledger:
  - Events: `ALERT_CREATED`, `ALERT_STATUS_CHANGED`, `CASE_CREATED`, `EVIDENCE_ADDED`, `EVIDENCE_VERIFIED`, `ATTACK_SIMULATED`, `REPORT_GENERATED`.
  - Block Rule: `SHA-256(index + timestamp + action + details + prevHash)`.
  - Genesis block: `prevHash = "GENESIS"`.
- **Audit Verification Engine**: Walks the chain from Index 0 to tip. If any block is modified or deleted, pinpoints `{ valid: false, brokenAtIndex: i, expectedHash, actualHash }`.
- Interactive **"Tamper Block"** and **"Re-Seal Ledger"** testing tools.

### 6. Module 6: Enterprise SOC Command Dashboard (`Report`)
- **Cyber SOC Aesthetic**: Deep slate `#070b12`, emerald `#10b981`, amber `#f59e0b`, crimson `#ef4444`, cyan `#06b6d4`.
- **Dual-Line Synchronized Telemetry Chart**: Real-time ingress packet rate on left axis with area gradient, predicted risk on right axis with 70 and 85 threshold reference lines.
- **Radial Risk Gauge**: Large animated numeric display, term breakdown ($a$ and $p_a$), and 5-minute forecast.
- **Attack Simulator Toolbar**: 1-click simulation buttons (`Simulate DDoS`, `Simulate Port Scan`, `Simulate Brute Force`, `Simulate Malware`, `Normal Baseline`, `Pause / Resume`).
- **Executive Incident Report Generator**: Print-ready and exportable Markdown/PDF dossier with executive summary, MITRE ATT&CK mapping, chronological timeline, forensic digest, and cryptographic audit proofs.

---

## 🚀 Quickstart Guide

### 1. Start the Platform
```bash
# Start the full-stack server (serves REST API, WebSockets, and React SPA)
npm start
```
The server will bind to:
- **Web Application & REST API**: `http://localhost:5000`
- **Real-time WebSocket Stream**: `ws://localhost:5000/ws`

### 2. Run Automated Verification Tests
```bash
npm test
```
Executes automated test assertions verifying:
1. NSL-KDD / CICIDS2017 CSV batch ingestion and ML classification.
2. Cryptographic audit chain tamper detection and block recovery.
3. Forensic evidence SHA-256 verification and bit-rot detection.

---

## 🗄️ Database Architecture (SQLite / Prisma Schema)

```sql
TrafficMetric (id, timestamp, packetRate, bytesPerSec, activeConns, portsAccessed, failedLogins, predictedRisk, anomalyScore)
Case (id, caseNumber, title, description, status, assignedTo, severity, createdAt, updatedAt)
Alert (id, timestamp, type, severity, status, riskScore, packetRateSnapshot, sourceIp, targetIp, mitreTactic, mitreTechnique, recurrenceCount, caseId)
Evidence (id, caseId, fileName, fileSize, fileMimeType, sha256Hash, storagePath, uploadedAt, verifiedStatus, sampleContent)
CaseNote (id, caseId, author, content, createdAt)
AuditBlock (index, timestamp, action, details, prevHash, hash, actor)
```

---

## 📡 REST API Reference

| Endpoint | Method | Description |
|---|---|---|
| `/api/status` | `GET` | System health, uptime, and SOC statistics |
| `/api/telemetry/history` | `GET` | Sliding-window telemetry metrics & 5-minute forecast |
| `/api/telemetry/simulate` | `POST` | Trigger attack vector (`ddos`, `portscan`, `bruteforce`, `malware`) |
| `/api/alerts` | `GET`, `POST` | List and create deduplicated alerts |
| `/api/alerts/:id` | `PATCH` | Update alert status (`Open`, `Investigating`, `Resolved`) |
| `/api/cases` | `GET`, `POST` | List and create incident cases |
| `/api/cases/:id/notes` | `POST` | Add investigator note to case |
| `/api/cases/:id/evidence` | `POST` | Ingest forensic evidence with SHA-256 hash |
| `/api/evidence/:id/verify` | `POST` | Cryptographic integrity verification |
| `/api/evidence/:id/tamper` | `POST` | Simulate bit rot on evidence file |
| `/api/audit` | `GET` | Retrieve entire hash-chained audit ledger |
| `/api/audit/verify` | `GET` | Run full cryptographic audit chain verification |
| `/api/audit/tamper` | `POST` | Simulate adversary block alteration |
| `/api/audit/restore` | `POST` | Re-seal and restore ledger |
| `/api/ingest/csv` | `POST` | Ingest NSL-KDD or CICIDS2017 batch CSV |
| `/api/report/generate` | `GET` | Generate executive incident report dossier |
