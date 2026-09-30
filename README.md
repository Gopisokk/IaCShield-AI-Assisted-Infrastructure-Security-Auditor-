# IaCShield

> AI-powered Terraform security scanner with closed-loop remediation verification.

<p align="center">
  <img src="https://img.shields.io/badge/python-3.10+-3776ab?style=flat-square&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/React_19-61dafb?style=flat-square&logo=react&logoColor=black" alt="React">
  <img src="https://img.shields.io/badge/Checkov-ff6384?style=flat-square" alt="Checkov">
  <img src="https://img.shields.io/badge/Docker-2496ed?style=flat-square&logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/license-MIT-green?style=flat-square" alt="MIT License">
</p>

---

## What It Does

IaCShield scans Terraform configurations for security misconfigurations using **Checkov** static analysis, then uses a generative AI model to explain vulnerabilities and generate least-privilege patches. The key differentiator: **every AI-generated fix is re-scanned in an isolated sandbox before being shown to the user** — no blind trust in LLM output.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Terraform   │────▶│   Checkov    │────▶│   LLM Fix    │────▶│  Re-Scan &   │
│    Input     │     │  AST Scan    │     │  Generation  │     │   Verify     │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                        Findings &           Patched HCL          Verified
                        Score (35/100)       + Explanation         Score (95/100)
```

### Why Re-Scan?

LLMs can hallucinate invalid HCL, introduce new misconfigurations, or silently drop security controls. The verification pass catches all of that before any code reaches the developer.

---

## Features

- **Closed-loop verification** — remediated code is re-scanned to catch regressions and hallucinations
- **Security scoring** — weighted 0–100 score (Critical: −25, High: −15, Medium: −10, Low: −5)
- **Multi-provider LLM support** — Google Gemini, Anthropic Claude, OpenAI, local Ollama, or offline mock mode
- **Interactive dashboard** — React 19 + Tailwind CSS with split-diff viewer, severity chips, one-click examples
- **Pre-built test suites** — vulnerable S3, IAM, and security group configs for immediate testing
- **Docker Compose** — single-command deployment with healthchecks

---

## Quick Start

### Docker (recommended)

```bash
git clone https://github.com/Gopisokk/IaCShield-AI-Assisted-Infrastructure-Security-Auditor-.git
cd IaCShield-AI-Assisted-Infrastructure-Security-Auditor-
docker compose up --build
```

Dashboard: `http://localhost:5173` · API docs: `http://localhost:8000/docs`

### Local Setup

**Backend:**
```bash
cd backend
python -m venv venv && source venv/bin/activate  # Windows: .\venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

**Frontend:**
```bash
cd frontend
npm install && npm run dev
```

---

## LLM Configuration

Set the provider in your `.env` file:

```env
# Google Gemini (default)
LLM_PROVIDER=gemini
LLM_API_KEY=your_key
LLM_MODEL=gemini-2.5-flash

# Anthropic Claude
# LLM_PROVIDER=anthropic
# LLM_API_KEY=your_key
# LLM_MODEL=claude-3-5-sonnet-20241022

# Local Ollama (no API key needed)
# LLM_PROVIDER=ollama
# LLM_BASE_URL=http://localhost:11434
# LLM_MODEL=llama3

# Offline mock mode (no API key needed)
# LLM_PROVIDER=mock
```

---

## API Endpoints

| Endpoint | Method | Description |
|---|:---:|---|
| `/api/scan` | POST | Scan Terraform code, return findings + security score |
| `/api/remediate` | POST | Generate AI fix, verify with re-scan, return diff |
| `/api/examples` | GET | List bundled vulnerable Terraform configurations |
| `/api/health` | GET | Readiness probe for Checkov and Python runtime |

---

## Test Suites

The `examples/` directory includes realistic misconfigurations:

| Suite | Checks Triggered |
|---|---|
| `vulnerable-s3` | CKV_AWS_18 (public ACL), CKV_AWS_19 (no encryption), CKV_AWS_21 (no versioning), CKV_AWS_144 (no replication) |
| `vulnerable-iam` | CKV_AWS_62 (wildcard actions), CKV_AWS_1 (admin privileges) |
| `vulnerable-security-group` | CKV_AWS_24 (SSH from 0.0.0.0/0), CKV_AWS_25 (RDP from 0.0.0.0/0) |

---

## Project Structure

```
├── backend/
│   ├── app/
│   │   ├── api/              # FastAPI routes
│   │   ├── models/           # Pydantic schemas
│   │   ├── services/         # Checkov, LLM, remediation, scoring
│   │   ├── utils/            # Sandbox file handling
│   │   └── main.py           # Entrypoint & CORS middleware
│   ├── tests/
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/       # FindingsList, AIAnalysisPanel, TerraformDiff
│   │   ├── services/         # API client
│   │   └── App.jsx           # Main dashboard
│   ├── Dockerfile
│   └── package.json
├── examples/                 # Vulnerable Terraform test configs
├── docker-compose.yml
└── LICENSE
```

---

## License

MIT — see [LICENSE](LICENSE) for details.
