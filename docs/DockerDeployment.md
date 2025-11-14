# LightRAG Docker Deployment

A lightweight Knowledge Graph Retrieval-Augmented Generation system with multiple LLM backend support.

## 🚀 Preparation

### Clone the repository:

```bash
# Linux/MacOS
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG
```
```powershell
# Windows PowerShell
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG
```

### Configure your environment:

```bash
# Linux/MacOS
cp env.example .env
# Edit .env with your preferred configuration
```
```powershell
# Windows PowerShell
Copy-Item env.example .env
# Edit .env with your preferred configuration
```

LightRAG can be configured using environment variables in the `.env` file:

**Server Configuration**

- `HOST`: Server host (default: 0.0.0.0)
- `PORT`: Server port (default: 9621)

**LLM Configuration**

- `LLM_BINDING`: LLM backend to use (openai/ollama/lollms/azure_openai/aws_bedrock/gemini)
- `LLM_BINDING_HOST`: LLM server host URL or endpoint
- `LLM_MODEL`: Model name to use
- `LLM_BINDING_API_KEY`: API key for LLM service
- `LLM_TIMEOUT`: LLM request timeout (default: 180 seconds)

**Embedding Configuration**

- `EMBEDDING_BINDING`: Embedding backend (openai/ollama/azure_openai/jina/lollms/aws_bedrock/gemini)
- `EMBEDDING_BINDING_HOST`: Embedding server host URL or endpoint
- `EMBEDDING_MODEL`: Embedding model name
- `EMBEDDING_DIM`: Embedding dimensions
- `EMBEDDING_BINDING_API_KEY`: API key for embedding service
- `EMBEDDING_TIMEOUT`: Embedding request timeout (default: 30 seconds)

**RAG Configuration**

- `MAX_ASYNC`: Maximum async operations for LLM (default: 4)
- `MAX_PARALLEL_INSERT`: Number of parallel processing documents (default: 2)
- `EMBEDDING_FUNC_MAX_ASYNC`: Maximum async operations for embedding (default: 8)
- `TOP_K`: Number of entities or relations retrieved from KG (default: 40)
- `CHUNK_TOP_K`: Maximum number of chunks for naive vector search (default: 20)

## 🐳 Docker Deployment

Docker instructions work the same on all platforms with Docker Desktop installed.

### Build Optimization

The Dockerfile uses BuildKit cache mounts to significantly improve build performance:

- **Automatic cache management**: BuildKit is automatically enabled via `# syntax=docker/dockerfile:1` directive
- **Faster rebuilds**: Only downloads changed dependencies when `uv.lock` or `bun.lock` files are modified
- **Efficient package caching**: UV and Bun package downloads are cached across builds
- **No manual configuration needed**: Works out of the box in Docker Compose and GitHub Actions

### Start LightRAG  server:

```bash
docker compose up -d
```

LightRAG Server uses the following paths for data storage:

```
data/
├── rag_storage/    # RAG data persistence
└── inputs/         # Input documents
```

### Updates

To update the Docker container:
```bash
docker compose pull
docker compose down
docker compose up
```

### Offline deployment

The Docker image includes all dependencies for offline operation:

- **Pre-installed packages**: All `api` and `offline` extras are included (storage backends, LLM providers, document processing libraries)
- **Tiktoken cache**: Pre-downloaded BPE encoding models for common LLM models (gpt-4o-mini, gpt-4o, gpt-4, etc.)
- **Environment variables**: Set `TIKTOKEN_CACHE_DIR=/app/data/tiktoken` to use the pre-cached files
- **Data directories**: `/app/data/rag_storage` for RAG data, `/app/data/inputs` for input documents

**Note**: Software packages requiring `transformers`, `torch`, or `cuda` are not preinstalled in the Docker images. Consequently, document extraction tools such as Docling, as well as local LLM models like Hugging Face and LMDeploy, cannot be used. These high-compute-resource-demanding services should be deployed as standalone services.

## 📦 Build Docker Images

### For local development and testing

```bash
# Build and run with Docker Compose (BuildKit automatically enabled)
docker compose up --build

# Or explicitly enable BuildKit if needed
DOCKER_BUILDKIT=1 docker compose up --build
```

**Note**: BuildKit is automatically enabled by the `# syntax=docker/dockerfile:1` directive in the Dockerfile, ensuring optimal caching performance.

### For production release

 **multi-architecture build and push**:

```bash
# Use the provided build script
./docker-build-push.sh
```

**The build script will**:

- Check Docker registry login status
- Create/use buildx builder automatically
- Build for both AMD64 and ARM64 architectures
- Push to GitHub Container Registry (ghcr.io)
- Verify the multi-architecture manifest

**Prerequisites**:

Before building multi-architecture images, ensure you have:

- Docker 20.10+ with Buildx support
- Sufficient disk space (20GB+ recommended for offline image)
- Registry access credentials (if pushing images)
