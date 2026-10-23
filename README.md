# Cybersecurity Analyzer

An AI-powered code security analyzer that combines Semgrep with an OpenAI agent to identify vulnerabilities, explain their impact, and recommend fixes.

![Cybersecurity Analyzer](assets/cyber.png)

## Live deployments

The application was deployed as the same Dockerized service to Azure Container Apps and Google Cloud Run:

- Azure Container Apps: [Open the Azure demo](https://cyber-analyzer.blue-sky-4821.eastus.azurecontainerapps.io)
- Google Cloud Run: [Open the GCP demo](https://cyber-analyzer-a7f3c9d2-uc.a.run.app)

These are the current live demo deployments of the Cybersecurity Analyzer.

## What it does

Paste Python code into the analyzer and receive a structured security report containing:

- Vulnerability title and description
- Vulnerable code snippet
- Recommended remediation
- CVSS score
- Severity classification

The backend combines Semgrep static analysis with an OpenAI agent. The frontend provides a simple browser interface, while the backend exposes the `/api/analyze` API.

## How it works

1. The user submits code through the Next.js frontend.
2. The frontend sends the code to the FastAPI backend.
3. The backend writes the code to a temporary file for Semgrep.
4. Semgrep identifies security findings.
5. The OpenAI agent interprets the findings and creates a structured report.
6. Findings are sorted by CVSS score and returned to the frontend.

```text
Browser
  ↓
Next.js static frontend
  ↓ POST /api/analyze
FastAPI backend
  ├── Semgrep MCP server
  └── OpenAI agent
        ↓
Structured security report
```

## Technology stack

- Python 3.12
- FastAPI and Uvicorn
- OpenAI Agents SDK
- Semgrep MCP server
- Next.js and React
- TypeScript and Tailwind CSS
- Docker
- Terraform
- Azure Container Apps
- Google Cloud Run
- GitHub Actions

## Run locally

### 1. Configure environment variables

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key
SEMGREP_APP_TOKEN=your_semgrep_app_token
ENVIRONMENT=development
```

Never commit `.env` or expose API keys in frontend code.

### 2. Start the backend

```bash
cd backend
uv sync
uv run uvicorn server:app --reload --host 0.0.0.0 --port 8000
```

### 3. Start the frontend

In a second terminal:

```bash
cd frontend
npm ci
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### 4. Run with Docker

```bash
docker build -t cyber-analyzer .
docker run --rm -p 8000:8000 --env-file .env cyber-analyzer
```

Open [http://localhost:8000](http://localhost:8000).

## CI/CD

GitHub Actions runs automatically whenever changes are pushed to `main` or submitted as a pull request.

The active build pipeline:

- Installs and lints the frontend
- Builds the static frontend
- Validates backend Python code
- Builds the Docker image for `linux/amd64`
- Checks Terraform formatting
- Validates both Azure and GCP Terraform configurations

The real cloud deployment jobs are currently commented out in [`deploy.yml`](.github/workflows/deploy.yml). This allows the repository to verify every push without creating cloud resources automatically.

## Cloud deployment

Terraform configurations are available for both platforms:

```text
terraform/azure   Azure Container Apps and Azure Container Registry
terraform/gcp     Google Cloud Run and Artifact Registry
```

The deployment workflow is prepared for Azure and GCP authentication through GitHub OIDC. Before enabling the deployment jobs, configure the required cloud credentials and a remote Terraform state backend.

Required application secrets:

- `OPENAI_API_KEY`
- `SEMGREP_APP_TOKEN`

Azure deployment credentials:

- `AZURE_CLIENT_ID`
- `AZURE_TENANT_ID`
- `AZURE_SUBSCRIPTION_ID`
- `AZURE_RESOURCE_GROUP`

GCP deployment credentials:

- `GCP_WORKLOAD_IDENTITY_PROVIDER`
- `GCP_SERVICE_ACCOUNT`
- `GCP_PROJECT_ID`

## Project structure

```text
backend/                 FastAPI API and security-analysis agent
backend/context.py       Agent instructions and report prompts
backend/mcp_servers.py   Semgrep MCP server configuration
frontend/                Next.js static frontend
terraform/azure/         Azure Container Apps infrastructure
terraform/gcp/           Google Cloud Run infrastructure
.github/workflows/       CI, deployment, and destroy workflows
Dockerfile               Multi-stage frontend/backend container build
```

## Deployment availability

The demo deployments are intended to remain available and will not be automatically destroyed after 3 days. Availability can still depend on cloud service status, credentials, maintenance, and future infrastructure changes.

## Project background

This project is part of a cloud deployment exercise covering Docker, Terraform, Azure Container Apps, Google Cloud Run, GitHub Actions, and AI-assisted security analysis.
