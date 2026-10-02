# Personal Gitea Server

Docker-based deployment of a personal [Gitea](https://about.gitea.com/) server with Gitea Actions.

This repository contains the infrastructure and configuration needed to run the personal Git service locally. The actual Git repositories hosted by Gitea are stored in Docker volumes and are **not** part of this GitHub repository.

## Architecture

```text
┌─────────────────────────────────────────────┐
│                  Docker Host                │
│                                             │
│  ┌─────────────────┐                        │
│  │     Gitea       │                        │
│  │                 │                        │
│  │ Web UI          │                        │
│  │ Git HTTP/SSH    │                        │
│  │ Repository      │                        │
│  │ management      │                        │
│  └────────┬────────┘                        │
│           │                                 │
│           │ Actions                         │
│           ▼                                 │
│  ┌─────────────────┐                        │
│  │  Gitea Runner   │                        │
│  │   act_runner    │                        │
│  └────────┬────────┘                        │
│           │                                 │
│           │ Docker                          │
│           ▼                                 │
│  ┌─────────────────┐                        │
│  │   Job Container  │                       │
│  │                  │                       │
│  │ Build / Test /   │                       │
│  │ Deploy           │                       │
│  └──────────────────┘                       │
│                                             │
│  Persistent Docker Volumes                  │
│  ├── Gitea data                             │
│  └── Gitea configuration                    │
└─────────────────────────────────────────────┘
```

GitHub is used as the **source repository for this infrastructure project**. Gitea is the **self-hosted Git service** used for personal repositories and CI/CD.

## Components

- **Gitea** — self-hosted Git service
- **Gitea Actions** — CI/CD workflow system
- **Gitea Runner** — executes Actions jobs
- **Docker Compose** — manages the services
- **GitHub** — stores this project's source/configuration

## Repository Structure

```text
.
├── compose.yaml
├── .env.example
├── .gitignore
├── README.md
├── gitea/
│   └── ...
├── runner/
│   ├── config.yaml
│   └── data/
└── docs/
    └── ...
```

The exact directory structure may evolve as the setup becomes more complete.

## Requirements

- Docker
- Docker Compose
- A machine capable of running Linux containers
- Sufficient storage for Gitea repositories and Actions artifacts

## Configuration

Create a local environment file from the example:

```bash
cp .env.example .env
```

Configure values such as:

```dotenv
GITEA_ROOT_URL=http://localhost:3000/
GITEA_INSTANCE_URL=http://gitea:3000/
GITEA_RUNNER_REGISTRATION_TOKEN=
```

The `.env` file contains local configuration and secrets and must **not** be committed to GitHub.

## Start the Server

Start the services with:

```bash
docker compose up -d
```

Check the service status:

```bash
docker compose ps
```

View Gitea logs:

```bash
docker compose logs -f gitea
```

View runner logs:

```bash
docker compose logs -f runner
```

After startup, Gitea should be available at:

```text
http://localhost:3000
```

The actual URL depends on the host and reverse-proxy configuration.

## Gitea Actions

Gitea Actions requires a runner to execute workflow jobs.

The runner is configured as a separate Docker container and uses Docker to create the job containers.

A workflow is stored in the Gitea repository under:

```text
.gitea/workflows/
```

For example:

```text
.gitea/
└── workflows/
    └── test.yml
```

A minimal workflow:

```yaml
name: Test

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Test
        run: |
          echo "Running tests..."
```

## Runner Security

The default Docker-based runner may require access to the Docker daemon:

```text
/var/run/docker.sock
```

This provides the runner with significant control over the Docker host.

For this reason:

- Only trusted repositories should be allowed to execute jobs on the runner.
- Action workflows should be treated as code with access to the runner environment.
- Secrets should not be committed to repositories.
- Runner permissions should be kept as narrow as practical.
- The runner should not be exposed directly to the public internet unless required.

For a personal server with trusted repositories, Docker-based execution provides a relatively simple setup. Stronger isolation can be considered later if required.

## Persistent Data

Gitea data is stored using Docker volumes rather than in this GitHub repository.

The repository should contain:

- Docker Compose configuration
- Environment variable examples
- Runner configuration templates
- Scripts
- Documentation

It should **not** contain:

- Git repositories managed by Gitea
- Database data
- Gitea-generated configuration containing secrets
- Runner registration tokens
- Private keys
- Access tokens
- Other credentials

## Backup

The Gitea persistent data should be backed up independently of this repository.

The infrastructure repository itself is reproducible from GitHub, while the Gitea data requires separate backup and recovery procedures.

A future backup setup may include:

```text
GitHub
  │
  └── Infrastructure configuration

Backup storage
  │
  └── Gitea persistent data

Docker host
  │
  ├── Gitea
  └── Gitea Runner
```

## Development Philosophy

This project is intended to be a small personal self-hosting environment rather than a production Git hosting platform.

The initial priorities are:

1. Simple deployment
2. Reproducible configuration
3. Persistent data
4. Automated builds and tests
5. Easy backup and recovery
6. Minimal maintenance overhead

More advanced features such as reverse proxies, HTTPS, external databases, stronger runner isolation, monitoring, and automated backups can be introduced when they provide a practical benefit.

## License

This repository contains infrastructure configuration for personal use. Add a license here if the configuration is intended to be shared or reused.
