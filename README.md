# AI Pipeline Studio - Infrastructure

This repository contains the Docker orchestration and shared infrastructure configuration for the AI Pipeline Studio platform.

## 🏗️ Architecture Overview

AI Pipeline Studio is a microservices-based platform consisting of:

1. **analytics-interface** - Web-based analytics and conversational interface
2. **rag-system** - RAG (Retrieval-Augmented Generation) API with OpenSearch
3. **ai-etl-framework** - AI-powered ETL pipeline framework
4. **ads-infrastructure** (this repo) - Docker Compose orchestration & configuration

## 📋 Prerequisites

- Docker Desktop installed and running
- Docker Compose v2.0+
- Git
- OpenAI API key (for RAG system)

## 🚀 Quick Start

### Step 1: Clone All Repositories

```bash
# Create project directory
mkdir -p ~/Documents/Development/ads
cd ~/Documents/Development/ads

# Clone all 4 repositories
git clone https://github.com/pankajsharma-source/ads-infrastructure.git
git clone https://github.com/pankajsharma-source/analytics-interface.git
git clone https://github.com/pankajsharma-source/rag-system.git
git clone https://github.com/pankajsharma-source/ai-etl-framework.git
```

Your directory structure should look like:
```
ads/
├── ads-infrastructure/
├── analytics-interface/
├── rag-system/
└── ai-etl-framework/
```

### Step 2: Configure Environment

```bash
cd ads-infrastructure

# Copy environment template
cp .env.example ../.env

# Edit with your credentials
nano ../.env  # or use your preferred editor
```

**Required configuration:**
- `OPENAI_API_KEY` - Get from https://platform.openai.com/api-keys
- `POSTGRES_PASSWORD` - Choose a secure password
- AWS credentials (optional, only if using S3 storage)

### Step 3: Start Services

```bash
# From ads-infrastructure directory
docker-compose up -d

# Verify all services are running
docker-compose ps
```

Expected output:
```
NAME                    STATUS          PORTS
opensearch              Up              9200, 9600
opensearch-dashboards   Up              5601
rag-api                 Up              5000
etl-api                 Up              8000
postgres                Up              5432
```

### Step 4: Access the Platform

- **Analytics Interface**: http://localhost:8080
- **RAG API**: http://localhost:5000
- **ETL API**: http://localhost:8000
- **OpenSearch Dashboards**: http://localhost:5601

## 🐳 Docker Compose Files

- **docker-compose.yml** - Main production configuration
- **docker-compose.dev.yml** - Development overrides (volume mounts, debug mode)
- **docker-compose.aws.yml** - AWS deployment configuration (ECS/Fargate)

### Using Development Mode

```bash
# Start with development overrides
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

Development mode provides:
- Hot reload for code changes
- Debug logging enabled
- Source code mounted as volumes
- Development-friendly settings

## 📊 Services

### OpenSearch
- **Purpose**: Vector store and BM25 search engine
- **Port**: 9200
- **Dashboard**: 5601
- **Data**: Persistent volume `opensearch-data`

### PostgreSQL
- **Purpose**: ETL state management and metadata
- **Port**: 5432
- **Database**: `etl`
- **User**: `etl_user`
- **Data**: Persistent volume `postgres-data`

### RAG API
- **Purpose**: Question answering with retrieval-augmented generation
- **Port**: 5000
- **Dependencies**: OpenSearch, OpenAI API
- **Source**: `../rag-system`

### ETL API
- **Purpose**: AI-powered data pipeline orchestration
- **Port**: 8000
- **Dependencies**: PostgreSQL
- **Source**: `../ai-etl-framework`

### Analytics Interface
- **Purpose**: Web-based UI for insights and conversational analytics
- **Port**: 8080
- **Source**: `../analytics-interface`

## 🛠️ Common Operations

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f rag-api
docker-compose logs -f etl-api
```

### Restart Services

```bash
# All services
docker-compose restart

# Specific service
docker-compose restart rag-api
```

### Stop Services

```bash
# Stop but keep data
docker-compose stop

# Stop and remove containers (keeps volumes)
docker-compose down

# Stop and remove everything including volumes (DESTRUCTIVE)
docker-compose down -v
```

### Rebuild After Code Changes

```bash
# Rebuild specific service
docker-compose build rag-api
docker-compose up -d rag-api

# Rebuild all services
docker-compose build
docker-compose up -d
```

## 🔍 Troubleshooting

### Services Won't Start

```bash
# Check logs for errors
docker-compose logs

# Verify Docker is running
docker ps

# Check port conflicts
lsof -i :5000  # Check if RAG API port is in use
lsof -i :8000  # Check if ETL API port is in use
```

### OpenSearch Health Issues

```bash
# Check OpenSearch status
curl http://localhost:9200/_cluster/health?pretty

# View OpenSearch logs
docker-compose logs opensearch
```

### Database Connection Errors

```bash
# Verify PostgreSQL is running
docker-compose ps postgres

# Test connection
docker-compose exec postgres psql -U etl_user -d etl -c "SELECT 1;"
```

### Cannot Access Analytics Interface

```bash
# Verify nginx/web server is running
docker-compose ps analytics-interface

# Check logs
docker-compose logs analytics-interface
```

## 📚 Additional Documentation

- **Analytics Interface**: See `../analytics-interface/README.md`
- **RAG System**: See `../rag-system/README.md`
- **ETL Framework**: See `../ai-etl-framework/docs/CLAUDE_CONTEXT.md`

## 🤝 Contributing

Each service has its own repository. To contribute:

1. Clone the specific service repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

For infrastructure changes, submit PRs to this repository.

## 📝 Environment Variables Reference

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `ENVIRONMENT` | Environment name | No | `development` |
| `OPENAI_API_KEY` | OpenAI API key | Yes | - |
| `OPENSEARCH_HOST` | OpenSearch hostname | No | `http://opensearch` |
| `OPENSEARCH_PORT` | OpenSearch port | No | `9200` |
| `POSTGRES_PASSWORD` | PostgreSQL password | Yes | - |
| `DATABASE_URL` | Full PostgreSQL connection URL | Yes | - |
| `AWS_REGION` | AWS region for S3 | No | `us-east-1` |
| `DEFAULT_BATCH_SIZE` | ETL batch size | No | `1000` |
| `MAX_WORKERS` | Parallel workers | No | `4` |
| `LOG_LEVEL` | Logging level | No | `INFO` |

## 🔐 Security Notes

- Never commit `.env` file to git
- Rotate API keys regularly
- Use strong database passwords
- Restrict network access in production
- Keep Docker images updated

## 📄 License

See individual service repositories for license information.

## 💬 Support

For issues related to:
- **Infrastructure/Docker**: Open issue in this repository
- **Analytics Interface**: Open issue in analytics-interface repo
- **RAG System**: Open issue in rag-system repo
- **ETL Framework**: Open issue in ai-etl-framework repo
