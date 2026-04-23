# HNG14 Stage 2 DevOps - Devops Task for Stage 2 Promotion

A full-stack application with API, Frontend, Worker services orchestrated with Docker Compose and automated CI/CD via GitHub Actions.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Services](#services)
- [Environment Configuration](#environment-configuration)
- [Troubleshooting](#troubleshooting)
- [CI/CD Pipeline](#cicd-pipeline)

## Prerequisites

Before bringing up the stack, ensure you have the following installed:

### Required
- **Docker** (v20.10+) - [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** (v1.29+) - Usually comes with Docker Desktop
- **Git** (v2.30+) - [Install Git](https://git-scm.com/downloads)

### Verify Installation same commands apply to a windows computer as bash cause of similarity of commands of those tools
```bash
docker --version
docker-compose --version
git --version
```

## Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/hng14-stage2-devops.git
cd hng14-stage2-devops
```

### 2. Configure Environment Variables
```bash
# Copy the example environment file
cp .env.example .env

# Edit .env with your configuration (optional - defaults will work for local development)
# nano .env  # or use your preferred editor
```

### 3. Build and Start Services
```bash
# Build all Docker images
docker-compose build

# Start all services in the background
docker-compose up -d

# Check service status
docker-compose ps
```

### 4. Verify Services Are Running

#### API Service
```bash
# Check API health
curl http://localhost:8000/health

# Expected response:
# {"status": "ok"}
```

#### Redis Cache
```bash
# Verify Redis is running
docker-compose exec redis redis-cli ping

# Expected response:
# PONG
```

#### Frontend Service
```bash
# Check if frontend is accessible
curl http://localhost:3000

# Or open in browser: http://localhost:3000
```

#### Worker Service
```bash
# Check worker logs to confirm it's processing jobs
docker-compose logs worker
```

### 5. Stop Services
```bash
# Stop all running services
docker-compose down

# Stop and remove volumes (data loss)
docker-compose down -v
```

## Project Structure

```
hng14-stage2-devops/
├── .github/
│   └── workflows/
│       └── ci-cd.yml              # GitHub Actions CI/CD pipeline
├── api/
│   ├── main.py                    # FastAPI application
│   ├── requirements.txt           # Python dependencies
│   └── Dockerfile                 # API container image
├── frontend/
│   ├── app.js                     # Express.js server
│   ├── package.json               # Node.js dependencies
│   ├── Dockerfile                 # Frontend container image
│   └── views/
│       └── index.html             # Web UI
├── worker/
│   ├── worker.py                  # Job processing worker
│   ├── requirements.txt           # Python dependencies
│   └── Dockerfile                 # Worker container image
├── docker-compose.yml             # Service orchestration
├── .env.example                   # Environment variable template
├── README.md                       # This file
├── fixes.md                        # Bug fixes documentation
└── Dockerfile                      # (individual service dockerfiles above)
```

## Services

### API Service
- **Language:** Python (FastAPI)
- **Port:** 8000
- **Dependencies:** Redis
- **Key Endpoints:**
  - `GET /health` - Health check
  - `POST /api/job` - Submit a job
  - `GET /api/job/{id}` - Get job status
  - `GET /status/{id}` - Get job status (alternative endpoint)

**Logs:**
```bash
docker-compose logs -f api
```

### Frontend Service
- **Language:** JavaScript (Express.js)
- **Port:** 3000
- **Serves:** Web UI at http://localhost:3000
- **Features:** Job submission form, real-time polling for job status

**Logs:**
```bash
docker-compose logs -f frontend
```

### Worker Service
- **Language:** Python
- **Port:** (No exposed port - internal worker process)
- **Dependencies:** Redis
- **Function:** Processes jobs from Redis queue, updates job status

**Logs:**
```bash
docker-compose logs -f worker
```

### Redis Cache
- **Image:** redis:7-alpine
- **Port:** 6379 (internal only, not exposed to host)
- **Persistence:** Stored in `redis_data` volume
- **Health Check:** Enabled with 30s interval

**Connect to Redis:**
```bash
docker-compose exec redis redis-cli
# Inside redis-cli:
# KEYS *                 # List all keys
# GET <key>             # Get value
# FLUSHDB               # Clear database (use with caution!)
```

## Environment Configuration

### Environment Variables

All services can be configured via `.env` file. See `.env.example` for all available options.

#### Key Variables

Application Service Variables
REDIS_HOST

Default: redis

Services: API, Worker

Purpose: Defines the hostname used to connect to the Redis service.

REDIS_PORT

Default: 6379

Services: API, Worker

Purpose: Defines the communication port for the Redis service.

API_URL

Default: http://localhost:8000

Services: Frontend

Purpose: The endpoint the frontend uses to communicate with the Backend API.

API_PORT

Default: 8000

Services: API

Purpose: The specific port exposed by the API service.

Docker & Infrastructure Settings
API_CONTAINER_NAME

Default: api

Services: Docker

Purpose: Sets the specific identifier for the API container.

NETWORK_NAME

Default: app-network

Services: Docker

Purpose: The name of the isolated virtual network for container communication.

RESTART_POLICY

Default: unless-stopped

Services: Docker

Purpose: Instructs Docker to automatically restart containers unless they are manually shut down.

### Creating Custom .env File

```bash
# Copy the template
cp .env.example .env

# Edit with your values
vi .env
```

Example `.env` customization:
```env
API_PORT=9000
REDIS_PORT=6380
API_CONTAINER_NAME=my-api
```

## Successful Startup Checklist

After running `docker-compose up -d`, you should see:

**All Services Running**
```bash
$ docker-compose ps to see if all services are running```

**API Health Check**
```bash
$ curl http://localhost:8000/health
{"status":"ok"}
```

**Redis Connectivity**
```bash
$ docker-compose exec redis redis-cli ping
PONG
```

**Frontend Accessible**
- Open browser: http://localhost:3000
- Should see job submission form

**Worker Processing**
```bash
$ docker-compose logs worker
# Should show: "Worker started", "Listening for jobs", etc.
```

**No Error Logs**
```bash
$ docker-compose logs --tail=20
# Check for any ERROR or CRITICAL messages
```

## Common Commands

### View Logs
```bash
# All services
docker-compose logs

# Specific service
docker-compose logs api

# Follow logs in real-time
docker-compose logs -f api

# Last 50 lines
docker-compose logs --tail=50 api
```

### Execute Commands in Containers
```bash
# Run shell in API container
docker-compose exec api /bin/bash

# Run Redis CLI
docker-compose exec redis redis-cli

# Run Python command in worker
docker-compose exec worker python -c "import redis; print(redis.Redis().ping())"
```

### Restart Services
```bash
# Restart single service
docker-compose restart api

# Restart all services
docker-compose restart

# Rebuild and restart
docker-compose up -d --build
```

### Remove and Clean Up
```bash
# Stop services (keep volumes)
docker-compose down

# Stop services and remove volumes
docker-compose down -v

# Remove unused images
docker image prune

# Full cleanup
docker system prune -a
```

## Troubleshooting

### Service Won't Start

**Check service logs:**
```bash
docker-compose logs <service-name>
```

**Common issues:**

1. **Port Already in Use**
   - Error: `bind: address already in use`
   - Solution: Change port in `.env` or kill process using the port
   ```bash
   # Find process using port 8000
   lsof -i :8000
   # Kill it
   kill -9 <PID>
   ```

2. **Redis Connection Failed**
   - Error: `Error: connect ECONNREFUSED 127.0.0.1:6379`
   - Solution: Ensure Redis service is healthy
   ```bash
   docker-compose ps redis
   docker-compose logs redis
   ```

3. **Frontend Cannot Reach API**
   - Check `API_URL` in `.env` matches your setup
   - In Docker: use service name (`http://api:8000`), not localhost

4. **Out of Disk Space**
   - Clean up Docker: `docker system prune -a`
   - Check volumes: `docker volume ls`
   - Remove unused volumes: `docker volume prune`

### API Not Responding

```bash
# Check if container is running
docker-compose ps api

# Check API logs
docker-compose logs api

# Test directly
curl -v http://localhost:8000/health

# Restart API
docker-compose restart api
```

### Redis Issues

```bash
# Check Redis connectivity
docker-compose exec api redis-cli -h redis ping

# Check Redis memory
docker-compose exec redis redis-cli info memory

# Clear all data
docker-compose exec redis redis-cli FLUSHALL
```

### Worker Not Processing Jobs

```bash
# Check worker logs
docker-compose logs -f worker

# Check Redis queue
docker-compose exec redis redis-cli LLEN job_queue

# Restart worker
docker-compose restart worker
```

## CI/CD Pipeline

This project uses GitHub Actions for automated CI/CD. See `.github/workflows/ci-cd.yml` for full details.

### Pipeline Stages
1. **Lint** - Python, JavaScript, and Dockerfile linting
2. **Test** - Unit tests with coverage reports
3. **Build** - Docker images tagged with git SHA
4. **Security Scan** - Trivy vulnerability scanning
5. **Integration Test** - Full stack testing
6. **Deploy** - Rolling update with health checks (main branch only)

### Running Pipeline Locally

To test the workflow before pushing:
```bash
# Install act (GitHub Actions local runner)
# https://github.com/nektos/act

# Run the pipeline
act
```

## Documentation Files

- **README.md** (this file) - Setup and usage guide
- **fixes.md** - Bug fixes documentation with line numbers
- **.env.example** - Environment variable template

## Getting Help

1. Check `fixes.md` for known issues and solutions
2. Review service logs: `docker-compose logs <service>`
3. Check GitHub Issues for the project
4. Ensure all prerequisites are installed and up to date