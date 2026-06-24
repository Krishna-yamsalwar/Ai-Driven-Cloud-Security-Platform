# Ram Antivirus Cloud Security AI Platform - Project Report

## 1. Project Title

Ram Antivirus Cloud Security AI Platform

## 2. Abstract

Ram Antivirus Cloud Security AI Platform is a full-stack security monitoring and response system for cloud environments. The project combines Wazuh-style security logs, machine learning based anomaly detection, Cloud Security Posture Management (CSPM), Cloud Infrastructure Entitlement Management (CIEM), compliance mapping, risk scoring, and automated remediation into one web dashboard.

The system can read real Wazuh alert logs when available and falls back to realistic simulated cloud security events when real logs are not present. It uses three specialist PyOD models to score events, derives cloud misconfiguration and identity risks from those events, calculates a unified risk score, maps findings to compliance frameworks, and provides attack simulation workflows to verify detection and remediation.

The frontend is built with React and Vite, while the backend is built with FastAPI and Python. SQLite is used for persistence, and optional integrations include SMTP email alerts and LocalStack for AWS-style cloud resource simulation.

## 3. Problem Statement

Cloud environments generate large volumes of security events from identity systems, workloads, storage, networking, and monitoring tools. Security teams need a practical way to identify suspicious activity, correlate it with cloud posture and identity risk, calculate business impact, and trigger remediation quickly.

Traditional dashboards often show raw alerts without enough context. This project addresses that gap by combining anomaly detection, posture analysis, identity risk analysis, compliance mapping, and automated playbooks in a single operational platform.

## 4. Objectives

The major objectives of the project are:

- Build a cloud security monitoring dashboard with real-time or simulated event ingestion.
- Detect anomalous activity using machine learning models.
- Identify CSPM misconfigurations from cloud security events.
- Identify CIEM risks such as over-privileged users, risky roles, and privilege escalation paths.
- Calculate a unified risk score using ML, CSPM, and CIEM signals.
- Provide automated remediation playbooks for common attack categories.
- Map findings to ISO 27001, SOC 2, and GDPR controls.
- Generate compliance and security reports.
- Provide attack simulation workflows to test whether the system detects known attack scenarios.
- Implement role-based access control for admin, analyst, and viewer users.

## 5. Technology Stack

### Backend

- Python
- FastAPI
- Uvicorn
- PyOD
- scikit-learn
- pandas
- NumPy
- PyJWT
- boto3
- SQLite
- WebSockets

### Frontend

- React
- Vite
- Axios
- Recharts
- lucide-react icons
- CSS

### Data and Simulation

- Wazuh-style alert logs
- SQLite database
- NSL-KDD dataset
- Synthetic CICIDS-2017 style dataset
- Synthetic UNSW-NB15 style dataset
- LocalStack optional AWS simulation

## 6. Project Structure

```text
Industry(Ram)/
├── backend/
│   ├── main.py
│   ├── train_model.py
│   ├── requirements.txt
│   ├── security.db
│   ├── data/
│   │   ├── alerts.json
│   │   ├── KDDTrain+.txt
│   │   └── KDDTest+.txt
│   ├── engines/
│   │   ├── alert_engine.py
│   │   ├── auth_engine.py
│   │   ├── ciem_engine.py
│   │   ├── compliance_engine.py
│   │   ├── cspm_engine.py
│   │   ├── db.py
│   │   ├── localstack_engine.py
│   │   ├── ml_engine.py
│   │   ├── remediation_engine.py
│   │   └── risk_engine.py
│   ├── models/
│   │   ├── iforest.pkl
│   │   ├── knn.pkl
│   │   ├── hbos.pkl
│   │   ├── scaler_iforest.pkl
│   │   ├── scaler_knn.pkl
│   │   ├── scaler_hbos.pkl
│   │   └── training_meta.json
│   └── simulator/
│       ├── attack_simulator.py
│       ├── log_generator.py
│       ├── wazuh_reader.py
│       └── wazuh_simulator.py
└── frontend/
    ├── package.json
    ├── vite.config.js
    └── src/
        ├── App.jsx
        ├── index.css
        ├── main.jsx
        └── components/
            ├── AttackSimulationTab.jsx
            ├── LoginPage.jsx
            ├── MLComparisonTab.jsx
            ├── RiskScoreTab.jsx
            ├── SharedComponents.jsx
            └── UserManagementTab.jsx
```

## 7. System Architecture

The system follows a three-layer architecture:

### 7.1 Data Layer

The data layer consists of Wazuh-style security events, simulated cloud security logs, trained ML model files, and SQLite persistence. Events can come from a real Wazuh alerts file or from the simulator.

### 7.2 Backend Layer

The backend exposes REST APIs and WebSocket endpoints through FastAPI. It handles authentication, event analysis, ML scoring, CSPM scanning, CIEM scanning, risk scoring, remediation, compliance reporting, export features, and attack simulations.

### 7.3 Frontend Layer

The frontend is a React dashboard that communicates with the backend through Axios. It provides tabs for overview, risk score, ML comparison, remediation, compliance, attack simulation, and user management.

## 8. End-to-End Workflow

1. Events are read from Wazuh logs or generated by the simulator.
2. The ML engine extracts event features and scores each event.
3. CSPM and CIEM engines derive posture and identity findings from event data.
4. The risk engine calculates ML, CSPM, CIEM, and unified risk scores.
5. Compliance mappings convert findings into framework control status.
6. Remediation playbooks can be executed manually or automatically.
7. Results are displayed in the React dashboard.
8. Historical records are stored in SQLite.
9. Reports and CSV exports can be generated from backend endpoints.

## 9. Backend Modules

### 9.1 FastAPI Server

File: `backend/main.py`

This is the central API server. It defines endpoints for authentication, event simulation, CSPM, CIEM, dashboard summary, ML comparison, remediation, compliance, risk score, attack simulation, history, export, status, test alerts, and WebSocket event streaming.

Main responsibilities:

- Start the FastAPI application.
- Configure CORS for local frontend access.
- Handle authentication routes.
- Provide dashboard APIs.
- Run ML, CSPM, CIEM, compliance, remediation, and risk workflows.
- Provide WebSocket updates every 10 seconds.
- Return structured JSON responses.

### 9.2 ML Engine

File: `backend/engines/ml_engine.py`

The ML engine loads three trained PyOD models from `backend/models/`:

- Isolation Forest for NSL-KDD style attacks.
- KNN for CICIDS-2017 style attacks.
- HBOS for UNSW-NB15 style attacks.

Features used:

- `level`
- `bytes_transferred`
- `failed_attempts`

Scoring strategy:

- Each model produces an anomaly score.
- The final score is the maximum score across available models.
- The best scoring model is stored as `detected_by`.
- Severity is derived from the anomaly score.

Severity mapping:

- Score above 0.90: Critical
- Score above 0.75: High
- Score above 0.50: Medium
- Otherwise: Low

The engine also sanitizes non-JSON-safe values such as NaN and Infinity before API responses.

### 9.3 Model Training

File: `backend/train_model.py`

The training script prepares datasets, trains specialist models, evaluates them, and saves model files and scalers. It uses NSL-KDD data and synthetic datasets shaped like CICIDS-2017 and UNSW-NB15.

Saved outputs:

- `iforest.pkl`
- `knn.pkl`
- `hbos.pkl`
- `scaler_iforest.pkl`
- `scaler_knn.pkl`
- `scaler_hbos.pkl`
- `training_meta.json`

### 9.4 Wazuh Reader

File: `backend/simulator/wazuh_reader.py`

This module reads Wazuh alert JSON records and normalizes them into the internal event format used by the rest of the platform. If real Wazuh logs are unavailable, the backend falls back to simulation.

### 9.5 Wazuh Simulator

File: `backend/simulator/wazuh_simulator.py`

This simulator generates normal and attack events. Attack categories include brute force, privilege escalation, data exfiltration, lateral movement, credential stuffing, port scan, ransomware precursor, and crypto mining.

### 9.6 Live Log Generator

File: `backend/simulator/log_generator.py`

The live log generator appends Wazuh-style alerts to `backend/data/alerts.json` at a configurable interval. This simulates continuous activity in a monitored environment.

### 9.7 Attack Simulator

File: `backend/simulator/attack_simulator.py`

The attack simulator generates targeted attack scenarios. It supports intensity levels:

- low
- medium
- high
- extreme

Attack scenarios:

- Brute force
- Privilege escalation
- Data exfiltration
- Abnormal access

Each run now generates unique run IDs, timestamps, random event volumes, random users, random IP addresses, sample generated events, and intensity-dependent behavior. This makes the attack simulation dynamic instead of static.

### 9.8 CSPM Engine

File: `backend/engines/cspm_engine.py`

The CSPM engine derives cloud posture findings from security events. It maps rule IDs and attack patterns to cloud misconfiguration categories such as public S3 access, open ports, missing MFA, unencrypted storage, and disabled key rotation.

### 9.9 CIEM Engine

File: `backend/engines/ciem_engine.py`

The CIEM engine identifies identity and entitlement risks from event data. It detects risky users, service accounts, roles, privilege escalation chains, and over-permissioned identities.

### 9.10 Risk Engine

File: `backend/engines/risk_engine.py`

The risk engine calculates component scores and a unified score.

Formula:

```text
Unified Risk = 0.35 * ML Risk + 0.35 * CSPM Risk + 0.30 * CIEM Risk
```

Risk levels:

- 76 to 100: critical
- 51 to 75: high
- 26 to 50: medium
- 0 to 25: low

The engine also generates recommendations and threshold actions such as auto-remediation, blocking suspicious IPs, disabling risky users, fixing configurations, SOC escalation, and incident report generation.

### 9.11 Remediation Engine

File: `backend/engines/remediation_engine.py`

The remediation engine contains automated playbooks for major attack and risk types.

Playbooks:

- Brute Force Response
- Privilege Escalation Response
- Data Exfiltration Response
- Lateral Movement Response
- Misconfiguration Auto-Fix
- Crypto Mining Response
- Credential Stuffing Response
- Ransomware Precursor Response

It also supports configuration fixes for CSPM findings and access revocation for risky CIEM entities.

### 9.12 Compliance Engine

File: `backend/engines/compliance_engine.py`

The compliance engine maps CSPM and CIEM findings to compliance frameworks:

- ISO/IEC 27001:2022
- SOC 2 Type II
- GDPR

It produces:

- Framework compliance scores.
- Passing and failing control counts.
- Critical gaps.
- JSON compliance reports.
- CSV compliance reports.

### 9.13 Authentication Engine

File: `backend/engines/auth_engine.py`

The authentication engine uses JWT and SQLite-backed users.

Default users:

| Username | Password | Role |
|---|---|---|
| admin | admin123 | admin |
| analyst1 | analyst123 | analyst |
| viewer1 | viewer123 | viewer |

Roles:

- Admin: full access including user management and remediation.
- Analyst: view, ML, compliance, risk, simulation, and limited remediation access.
- Viewer: read-only dashboard access.

### 9.14 Database Layer

File: `backend/engines/db.py`

SQLite is used for persistence. Tables include:

- users
- risk_history
- cspm_history
- ciem_history
- alert_log

The database stores users, scan history, risk trends, compliance scores, and alert logs.

### 9.15 Alert Engine

File: `backend/engines/alert_engine.py`

The alert engine sends SMTP email alerts when configured. If SMTP variables are not configured, alerts are printed/logged and still recorded locally.

Environment variables:

- `ALERT_EMAIL_FROM`
- `ALERT_EMAIL_PASS`
- `ALERT_EMAIL_TO`

### 9.16 LocalStack Engine

File: `backend/engines/localstack_engine.py`

The LocalStack engine optionally scans simulated AWS resources using boto3. If LocalStack is running, CSPM and CIEM can use local AWS-like resources instead of only event-derived simulation.

## 10. Frontend Modules

### 10.1 Main Dashboard

File: `frontend/src/App.jsx`

The main dashboard handles login state, tab navigation, data fetching, dashboard summary, remediation workflow, compliance dashboard, and toast notifications.

Main tabs:

- Overview
- Risk Score
- ML Comparison
- Remediation
- Compliance
- Attack Sim
- Users

Tabs are filtered based on user permissions.

### 10.2 Login Page

File: `frontend/src/components/LoginPage.jsx`

The login page supports:

- Login
- Registration
- Password visibility toggle
- Demo login buttons for admin, analyst, and viewer

### 10.3 Attack Simulation Tab

File: `frontend/src/components/AttackSimulationTab.jsx`

The attack simulation tab connects directly to the Python backend. It supports:

- Scenario loading from `/api/simulation/scenarios`
- Individual attack simulation
- Full attack simulation
- Intensity selector
- Run ID and timestamp display
- Detection verification
- Sample generated event preview

This tab is backend-driven and does not use hardcoded fake simulation results.

### 10.4 Risk Score Tab

File: `frontend/src/components/RiskScoreTab.jsx`

Displays:

- Unified risk score
- Component score breakdown
- Risk trend
- Recommendations
- Threshold actions

### 10.5 ML Comparison Tab

File: `frontend/src/components/MLComparisonTab.jsx`

Displays:

- Model comparison
- Accuracy, precision, recall, and F1 score
- Best model indicator
- Bar chart and radar chart visualization

### 10.6 User Management Tab

File: `frontend/src/components/UserManagementTab.jsx`

Admin-focused UI for listing users and activating or deactivating accounts.

### 10.7 Shared Components

File: `frontend/src/components/SharedComponents.jsx`

Contains reusable UI components:

- Badge
- ScoreRing
- Toast

### 10.8 UI Theme

File: `frontend/src/index.css`

The current UI theme is a minimalist, light, professional dashboard style. It uses neutral surfaces, restrained colors, lucide SVG icons, clean cards, and chart-friendly styling. Emoji-based UI markers were removed and replaced with SVG icons.

## 11. API Endpoints

### Authentication

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/auth/register` | Register user |
| POST | `/api/auth/login` | Login and return JWT |
| GET | `/api/auth/me` | Get current user |
| GET | `/api/auth/roles` | List role definitions |
| GET | `/api/auth/users` | List users |
| POST | `/api/auth/users/{username}/deactivate` | Deactivate user |
| POST | `/api/auth/users/{username}/reactivate` | Reactivate user |

### Security Data

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/simulate/events` | Get analyzed events |
| GET | `/api/cspm/scan` | Run CSPM scan |
| GET | `/api/ciem/scan` | Run CIEM scan |
| GET | `/api/dashboard/summary` | Get dashboard summary |
| GET | `/api/simulate/attack` | Trigger targeted attack event |

### Machine Learning

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/ml/compare` | Compare ML models |
| GET | `/api/ml/model-info` | Get model metadata |

### Remediation

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/remediation/playbooks` | List playbooks |
| POST | `/api/remediation/execute/{attack_type}` | Execute playbook |
| POST | `/api/remediation/auto` | Auto-remediate anomalies |
| POST | `/api/remediation/config-fix/{vulnerability_id}` | Apply config fix |
| POST | `/api/remediation/revoke-access` | Revoke risky entity access |
| GET | `/api/remediation/full-pipeline` | Run full remediation pipeline |

### Compliance

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/compliance/frameworks` | List frameworks |
| GET | `/api/compliance/dashboard` | Compliance dashboard |
| GET | `/api/compliance/map` | Finding-to-control mapping |
| GET | `/api/compliance/report` | Generate report |
| GET | `/api/compliance/report/executive` | Executive summary |

### Risk and Simulation

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/risk/score` | Unified risk score |
| GET | `/api/simulation/scenarios` | List attack scenarios |
| POST | `/api/simulation/run/{scenario}` | Run scenario |
| POST | `/api/simulation/full` | Run full simulation |

### History, Export, and Status

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/history/cspm` | CSPM history |
| GET | `/api/history/ciem` | CIEM history |
| GET | `/api/history/alerts` | Alert history |
| GET | `/api/export/events` | Export event CSV |
| GET | `/api/export/cspm` | Export CSPM CSV |
| GET | `/api/export/ciem` | Export CIEM CSV |
| GET | `/api/status` | System status |
| POST | `/api/alerts/test` | Send test alert |
| WS | `/ws/events` | Live event stream |

## 12. Database Design

SQLite is used because the project is designed as a local prototype and demonstration platform.

Tables:

- `users`: stores user accounts, roles, password hashes, and account status.
- `risk_history`: stores risk scores and component scores over time.
- `cspm_history`: stores CSPM scan findings.
- `ciem_history`: stores CIEM scan findings.
- `alert_log`: stores alert events, compliance score records, and email status.

## 13. Security Design

The project includes several security features:

- JWT authentication.
- Role-based permissions.
- Password hashing with SHA-256 and salt.
- Admin-only user management routes.
- Permission checks for sensitive backend operations.
- CORS configured for local frontend origins.
- Email alert support through SMTP.
- Risk-based remediation triggers.

Security note: For production, the JWT secret and password hashing should be strengthened. A production system should use environment-managed secrets, bcrypt or Argon2 password hashing, HTTPS, stronger audit logging, and stricter authorization enforcement on all sensitive routes.

## 14. Machine Learning Design

The ML pipeline is designed around event-level anomaly scoring.

Input event features:

- Rule severity level.
- Bytes transferred.
- Failed login attempts.

Model strategy:

- Train specialist models on separate attack-style datasets.
- Load models at backend startup.
- Score each event using all available models.
- Use the maximum anomaly probability as the final score.
- Assign model attribution using the highest scoring model.

Advantages:

- Simple and explainable feature set.
- Multiple model comparison available.
- Lightweight enough for local execution.
- Works with both real and simulated events.

Limitations:

- Feature set is intentionally small.
- Synthetic data is used for some model domains.
- Model version compatibility can affect loaded pickle files.
- KNN scoring may fail on some Windows setups due to access or environment issues.

## 15. CSPM and CIEM Design

The CSPM and CIEM engines are event-derived. They do not simply pick random findings. They map observed logs, rule IDs, and attack types into cloud posture and identity risk findings.

CSPM examples:

- Public object storage exposure.
- Open administrative ports.
- Missing root MFA.
- Unencrypted database storage.
- Disabled KMS rotation.

CIEM examples:

- Over-privileged service accounts.
- Dormant high-privilege users.
- Privilege escalation paths.
- Wildcard IAM permissions.

## 16. Attack Simulation Design

The attack simulation module verifies detection by generating known attack patterns and sending them through the same ML and risk pipeline.

Supported scenarios:

- Brute force: repeated login failures followed by successful login.
- Privilege escalation: policy creation, policy attachment, role assumption, and optional extra actions.
- Data exfiltration: high-volume object transfer to external IPs.
- Abnormal access: unusual location login followed by suspicious cloud API calls.

Supported intensity:

- Low
- Medium
- High
- Extreme

Each run includes:

- Unique simulation ID.
- Timestamp.
- Randomized event counts.
- Randomized users and IPs.
- Detection rate.
- Sample generated events.

## 17. Compliance Design

The compliance engine maps findings to:

- ISO/IEC 27001:2022
- SOC 2 Type II
- GDPR

Compliance score is calculated from passing and failing controls. Failed controls are derived from CSPM and CIEM findings. Baseline controls are included to represent controls that are considered implemented in the demonstration environment.

Outputs:

- Overall compliance score.
- Per-framework status.
- Passing and failing control counts.
- Critical control gaps.
- JSON report.
- CSV report.

## 18. Remediation Design

The remediation engine simulates cloud response playbooks. Each playbook contains ordered automated steps and returns execution status.

Example brute force steps:

- Block source IP.
- Lock account.
- Force MFA.
- Alert SOC.
- Generate audit log.

Example data exfiltration steps:

- Block egress destination.
- Revoke storage access.
- Enable enhanced logging.
- Isolate source instance.
- Create incident ticket.
- Preserve evidence.

## 19. Frontend Design

The frontend is a role-aware security operations dashboard.

Current UI principles:

- Minimalist light dashboard theme.
- SVG icons through lucide-react.
- Responsive grid layout.
- Charts through Recharts.
- Compact panels for operational scanning.
- No emoji markers in UI.
- User feedback through toast notifications.

Frontend pages:

- Login and registration.
- Overview dashboard.
- Unified risk score.
- ML model comparison.
- Remediation playbooks.
- Compliance dashboard and report preview.
- Attack simulation.
- User management.

## 20. Setup and Execution

### Backend Setup

```bash
cd backend
pip install -r requirements.txt
python train_model.py
python main.py
```

Recommended Python version: Python 3.10 to 3.12 for best compatibility with pinned dependency versions.

### Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

Frontend URL:

```text
http://localhost:5173
```

Backend URL:

```text
http://localhost:8000
```

API documentation:

```text
http://localhost:8000/docs
```

## 21. Testing and Verification

The following workflows can be used to verify the project:

- Login using default accounts.
- Open the dashboard and confirm summary data loads.
- Run ML comparison and verify model metrics.
- Run risk score calculation and verify component scores.
- Run attack simulation with different intensity values.
- Confirm simulation run IDs and event counts change between runs.
- Execute remediation playbooks.
- Generate compliance dashboard and reports.
- Export event, CSPM, and CIEM CSV files.
- Check `/api/status` for Wazuh, LocalStack, and database status.

## 22. Current Implementation Status

Implemented:

- FastAPI backend.
- React frontend.
- JWT authentication.
- Role-based UI filtering.
- SQLite persistence.
- ML anomaly detection.
- CSPM and CIEM engines.
- Unified risk scoring.
- Compliance mapping and reporting.
- Remediation playbooks.
- Attack simulation with intensity.
- CSV exports.
- WebSocket event streaming.
- Minimalist UI theme with SVG icons.

Partially simulated:

- Cloud resource remediation.
- SOC alerting when SMTP is not configured.
- LocalStack-based AWS scans unless LocalStack is running.
- Some ML datasets are synthetic approximations.

## 23. Limitations

- The system is a prototype and demonstration platform, not a production SOC product.
- Some cloud actions are simulated instead of applied to real AWS.
- Default credentials are included for demo purposes and must be changed in production.
- JWT secret has a default value and should be moved to secure environment management.
- Password hashing should use bcrypt or Argon2 in production.
- ML model pickle files can have version compatibility issues across scikit-learn versions.
- The feature set for anomaly detection is limited to three numeric features.
- Some compliance baseline controls are assumed for demonstration.
- Full production deployment would require HTTPS, audit logs, secure secret storage, containerization, monitoring, and a production database.

## 24. Future Enhancements

Recommended future work:

- Add Docker Compose for backend, frontend, database, and LocalStack.
- Replace SQLite with PostgreSQL for multi-user deployment.
- Add stronger password hashing with bcrypt or Argon2.
- Add refresh tokens and token revocation.
- Add endpoint-level RBAC enforcement for all sensitive APIs.
- Add real AWS remediation actions behind an approval workflow.
- Add audit trail UI for remediation and user actions.
- Add richer ML features such as geo-location, user baseline, service frequency, and time-of-day.
- Add model retraining UI and model version management.
- Add SIEM integrations beyond Wazuh.
- Add PDF report export.
- Add unit and integration tests.
- Add CI/CD pipeline.

## 25. Conclusion

Ram Antivirus Cloud Security AI Platform demonstrates a complete cloud security monitoring workflow from event ingestion to ML detection, posture analysis, identity risk analysis, risk scoring, remediation, and compliance reporting. The project is suitable as an academic or industry prototype because it shows how several security disciplines can be connected into one operational dashboard.

The main strength of the project is integration. Instead of treating alerts, posture findings, identity risks, compliance, and remediation as separate modules, the platform links them into one workflow. This makes it useful for demonstrating how AI-assisted cloud security operations can reduce alert overload, improve visibility, and accelerate response.

