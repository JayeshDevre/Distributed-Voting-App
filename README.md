# Example Voting App

A simple distributed voting application that runs across multiple Docker containers. Users choose between two options (e.g. "cloud" vs "Kubernetes"); votes are queued in Redis, processed by a worker into PostgreSQL, and the result UI shows live tallies via WebSockets.

## Architecture

![Architecture diagram](architecture.excalidraw.png)

- **[Vote](vote/)** — Python (Flask) web app; lets you vote between two options and pushes votes to Redis
- **[Redis](https://hub.docker.com/_/redis/)** — Collects new votes in a list; consumed by the worker
- **[Worker](worker/)** — .NET 7 service that reads votes from Redis and stores them in Postgres
- **[Postgres](https://hub.docker.com/_/postgres/)** — Persists votes (one row per voter); backed by a Docker volume
- **[Result](result/)** — Node.js (Express + Socket.io) web app that shows voting results in real time

**Data flow:** Vote UI → Redis `votes` list → Worker (LPOP) → PostgreSQL `votes` table → Result app (polls DB, broadcasts via Socket.io).

## Technologies

| Component | Stack |
|-----------|--------|
| Vote | Python 3.11, Flask, Redis client, Gunicorn |
| Result | Node.js, Express, Socket.io, pg, AngularJS (frontend) |
| Worker | .NET 7, Npgsql, StackExchange.Redis, Newtonsoft.Json |
| Data | Redis (Alpine), PostgreSQL 15 (Alpine) |
| DevOps | Docker, Docker Compose, Docker Swarm, Kubernetes, GitHub Actions, Azure Pipelines |

## Getting started

### Docker Compose (local development)

```bash
docker compose up -d
```

- **Vote:** http://localhost:8080  
- **Result:** http://localhost:8081  

Optional: seed the database with sample votes:

```bash
docker compose --profile seed up -d
```

### Docker Swarm

```bash
docker stack deploy -c docker-stack.yml voting
```

### Kubernetes

Apply the manifests in `k8s-specifications/`:

```bash
kubectl apply -f k8s-specifications/
```

## CI/CD

- **GitHub Actions** (`.github/workflows/`) — Path-based builds for `vote`, `result`, and `worker`; builds and pushes images to Docker Hub and/or GitHub Container Registry; multi-platform (amd64, arm64, arm/v7).
- **Azure Pipelines** — `azure-pipelines-vote.yml`, `azure-pipelines-result.yml`, `azure-pipelines-worker.yml` build and push images to Azure Container Registry.

## Notes

The voting application only accepts **one vote per client browser**. It uses a `voter_id` cookie and does not register additional votes if one has already been submitted from that client.

## Screenshots

<img width="1511" alt="ss 1" src="https://github.com/user-attachments/assets/4b2646e9-b469-46c1-8c26-9da2e4185688" />
<img width="1512" alt="ss2" src="https://github.com/user-attachments/assets/a7888936-9ae3-412d-8919-8e57411e5492" />
<img width="1512" alt="ss3" src="https://github.com/user-attachments/assets/336936a6-7397-4c11-8633-3ec08e065b91" />
<img width="1512" alt="ss4" src="https://github.com/user-attachments/assets/aaff2ffd-aa4a-4164-bc42-528fc5ab756a" />
<img width="1512" alt="ss5" src="https://github.com/user-attachments/assets/c71a8fb8-9c14-4d16-8c4d-f6942f4347ff" />
<img width="1512" alt="ss6" src="https://github.com/user-attachments/assets/bbf82207-ae0f-452b-93d3-e98bd178077b" />
<img width="1512" alt="ss7" src="https://github.com/user-attachments/assets/01400eb6-5a49-49ba-bbdd-7c3b768d8cf2" />
