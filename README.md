# Cyberthreat Intelligence Agent

Cyberthreat Intelligence Agent is a multi-service cybersecurity platform for analyzing files, URLs, and IP indicators. It combines threat intelligence providers (VirusTotal and WhoisXMLAPI) with LLM-assisted context generation (OpenAI), and exposes capabilities through a Vue frontend and FastAPI services behind Kong API Gateway.

## Architecture

![Architecture diagram](./architecture.png)

## Core Capabilities

- **File analysis:** Submit files for malware/threat scanning.
- **URL analysis:** Inspect URLs against known malicious infrastructure.
- **IP analysis:** Retrieve reputation and historical threat signals for IPs.
- **Advanced URL checks:** Enrichment including phishing/malware indicators, SSL details, and WHOIS context.
- **MCP-enabled enrichment:** Uses MCP workflows to orchestrate additional intelligence/context.
- **Gateway-first API exposure:** Kong centralizes routing for backend services.
- **Auth-ready frontend:** Auth0 integration scaffold is present in the web client.

## Tech Stack

- **Frontend:** Vue 3, TypeScript, Vite, Vuetify, Vue Router, Axios, Auth0 Vue SDK
- **Backend services:** Python, FastAPI, Uvicorn, OpenAI SDK, MCP SDK
- **Threat intelligence:** VirusTotal API, WhoisXMLAPI
- **Data layer:** PostgreSQL (separate DB instances for services)
- **Gateway and platform:** Kong, Docker, Docker Compose, Kubernetes manifests
- **Ops utilities:** Adminer

## Repository Structure

```text
.
├── Backend/                # FastAPI service for VirusTotal-focused workflows
│   ├── config/
│   ├── Dockerfile
│   └── requirements.txt
├── Frontend/               # Vue application
│   ├── src/
│   ├── auth_config.json
│   ├── Dockerfile
│   └── package.json
├── mcp-client/             # FastAPI service integrating MCP-based flows
│   ├── main.py
│   ├── Dockerfile
│   └── requirements.txt
├── mcp-server/             # MCP server implementation
├── kong/                   # Kong declarative config
│   └── kong.yml
├── pg_db/                  # DB init scripts
│   └── init.sql
├── k8s/                    # Kubernetes manifests for deployment
├── docker-compose.yml
└── README.md
```

## Prerequisites

- [Docker Desktop](https://docs.docker.com/get-docker/)
- VirusTotal API key
- OpenAI API key
- WhoisXMLAPI key
- Auth0 tenant + application (for frontend auth setup)

## Setup and Running

### 1) Clone the Repository

```bash
git clone https://github.com/da-ros/Cyberthreat-Intelligence-Agent.git
cd Cyberthreat-Intelligence-Agent
```

### 2) Prepare Environment Files

Create these files before starting containers:

1. `Backend/.env`
2. `mcp-client/.env`
3. Root `.env` (required by `docker-compose.yml`; leave it blank)

> Important: Do not commit `.env` files to git.

### 3) Configure `Backend/.env`

Use the following values as a baseline:

```env
OPENAI_KEY=your-openai-api-key
VT_KEY=your-virustotal-api-key
DATABASE_URL_API=postgresql://postgres:postgres@db:5432/virustotaldb
AUTH0_CLIENT_ID=your-auth0-client-id
AUTH0_DOMAIN=your-auth0-domain
```

### 4) Configure `mcp-client/.env`

Use:

```env
OPENAI_API_KEY=your-openai-api-key
VT_API_KEY=your-virustotal-api-key
DATABASE_URL=postgresql://pguser:pgpass@db_mcp:5432/cybersec
MCP_SERVER_CMD=python mcp_server.py
WHOIS_API_KEY=your-whoisxmlapi-key
```

### 5) Configure Frontend Auth0

Update `Frontend/auth_config.json` with your Auth0 values:

- `domain`
- `clientId`
- `audience`

The audience should match the backend API identifier configured in Auth0.

### 6) Build and Start the Stack

From the project root:

```bash
docker compose up --build -d
```

What this does:

- `--build` rebuilds images from Dockerfiles.
- `-d` runs everything in detached/background mode.

### 7) Verify Services

After startup, expected local endpoints are:

- **Kong proxy:** `http://localhost:8002`
- **Kong admin:** `http://localhost:8003`
- **VT backend (direct):** `http://localhost:8000`
- **MCP client backend (direct):** `http://localhost:8001`
- **Adminer:** `http://localhost:8081`
- **Frontend (Docker):** `http://localhost:3000`

### 8) Stop Services

Stop all services:

```bash
docker compose down
```

Stop and remove volumes (this clears DB data):

```bash
docker compose down -v
```

Notes:
- `docker compose` is the modern Compose CLI.
- If you see a warning about `version` in `docker-compose.yml`, it is safe but you can remove the field.

## API Surface (High Level)

Primary routes are exposed through Kong (`http://localhost:8002`) and mapped in `kong/kong.yml`, with backend handlers in `Backend/main.py` and `mcp-client/main.py`.

Common routes include:

- `GET /logs`
- `POST /analyze`
- `POST /advanced-analysis`

## Security Notes

- Never commit `.env` files or production secrets.
- Validate Auth0 JWT enforcement status before production exposure.
- Restrict Kong Admin API access outside local/dev networks.

## License

MIT
