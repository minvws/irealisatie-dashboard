# iRealisatie Dashboard

An automated, self-provisioning Grafana monitoring platform designed to
aggregate and visualize software development lifecycle (SDLC) metrics, security
posture, code quality, and project statuses for the **minvws** (Ministerie van
Volksgezondheid, Welzijn en Sport) organization repositories.

It aggregates data directly from the **GitHub API** (using both the official
GitHub plugin and the Infinity API plugin), **SonarCloud**, **DefectDojo**,
**Zammad** (via Elasticsearch), and local mock endpoints.

## Architecture and Component Overview

The environment is containerized via Docker Compose and consists of two main
services:

1. **Grafana (`localhost:3000`)**:
   - Pre-configured with automatic provisioning of datasources and dashboards.
   - Installs the `grafana-github-datasource` and
     `yesoreyeram-infinity-datasource` plugins via a compose entrypoint (which
     reinstalls them if missing, e.g. after a volume purge).
2. **JSON Server (`localhost:3001`)**:
   - A lightweight mock API server built from `./json-server/` serving data
     defined in `db.json` for testing local data streams.

### Folder Structure

```text
├── compose.yml              # Docker Compose definition
├── .env.example             # Template for local environment variables
├── json-server/             # Mock API configuration
│   ├── Dockerfile
│   └── data/db.json         # Mock database endpoints
└── grafana/
    └── provisioning/        # Automatic provisioning for Grafana
        ├── datasources/     # Datasources (GitHub, Infinity, DefectDojo, Zammad/Elasticsearch, Mock)
        └── dashboards/      # Preloaded dashboard layouts (.json files)
            └── draft/       # Experimental / reference dashboards
```

---

## Getting Started

### Prerequisites

- Docker and Docker Compose
- A GitHub Personal Access Token (PAT) (read-only access to organization
  repositories, issues, and pull requests) OR a configured GitHub App.

### 1. Configuration

Copy the template environment file:

```bash
cp .env.example .env
```

Open `.env` in your text editor and specify your GitHub authentication details:

- **Option A (PAT)**: Provide your token in `GITHUB_TOKEN`.
- **Option B (GitHub App)**: Provide `GITHUB_APP_ID`, `GITHUB_APP_INSTALL_ID`,
  and `GITHUB_APP_PRIVATE_KEY`.

### 2. Launch the Platform

Start all services in background mode:

```bash
docker compose up -d
```

Docker will pull the images, construct the custom `json-server` container, and
install the required Grafana plugins on first startup.

### 3. Accessing the Services

- **Grafana**: Open [http://localhost:3000](http://localhost:3000) (Default
  login: `admin` / `admin`).
- **JSON Server (Mock API)**: Open
  [http://localhost:3001](http://localhost:3001) to explore mock database
  endpoints.

## Provisioned Datasources

The dashboard comes pre-provisioned with the following datasources:

- **GitHub Datasource**: Queries repository metrics, Pull Requests, Issues, and
  Code Scanning Alerts directly (`GitHub` PAT and `GitHub App` variants).
- **Infinity Datasource**: Used for querying arbitrary JSON REST endpoints:
  the GitHub REST API (`GitHub API`), `SonarCloud API`, `DefectDojo API`, and
  the `Local JSON Server`.
- **Zammad Tickets (Elasticsearch)**: Queries the Zammad ticket index, pointed
  at a separate Elasticsearch cluster via `ZAMMAD_ES_HOST` / `ZAMMAD_ES_PORT`.
- **Mock (TestData)**: The default datasource, used for prototyping.

## Available Dashboards

Once you log into Grafana, you will find the following pre-configured dashboards
in the **General** folder:

- **Organization Overview**: A high-level view showing active vs. archived
  projects, public vs. private repository ratio, overall project topics, and
  average pull request resolution time across the organization.
- **Projects Overview**: Highlights specific repository metrics grouped by
  project topics (e.g., `icore`).

Additional experimental and reference dashboards (GitHub Dependabot, GitHub
vulnerabilities, SonarQube, Zammad tickets, etc.) are provisioned under the
**draft** folder.

## Development and Resetting

If you need to completely purge local Grafana storage (e.g., to force-reload
newly added provisioned dashboards or datasources), run the following commands:

```bash
# Tear down running containers
docker compose down

# Remove the persistent Grafana volume to clear settings
docker volume rm irealisatie-dashboard_grafana-data

# Spin up cleanly
docker compose up -d
```
