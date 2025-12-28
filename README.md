# DeepSeek Containers

Local stacks for running **Ollama** (model server) and **Open WebUI** (chat UI) with the DeepSeek model. Pick Docker Compose or Podman Compose based on your runtime and follow the matching guide in this repo.

## Repo layout
- `docker/` → Docker Compose stack and docs
- `podman/` → Podman Compose stack and docs

## Quick start
### Option A: Docker Compose
```bash
cd docker
cat > .env <<'EOF'
WEBUI_SECRET_KEY=change-me
ENABLE_SIGNUP=true
EOF
docker compose -f docker-compose.yml up -d
```
Open WebUI: http://localhost:3000  
Ollama API: http://localhost:11434

### Option B: Podman Compose
```bash
cd podman
cat > .env <<'EOF'
WEBUI_SECRET_KEY=change-me
ENABLE_SIGNUP=true
EOF
podman compose -f podman-compose.yml up -d
```
Open WebUI: http://localhost:3000  
Ollama API: http://localhost:11434

> The stack pulls the `deepseek-r1:1.5b` model on first start via a one-shot loader container.

## More details
- Configuration, security recommendations, testing steps, and troubleshooting live in `docker/README.md` and `podman/README.md`.
- Tear down the stack with `docker compose -f docker-compose.yml down -v` or `podman compose -f podman-compose.yml down -v` to remove volumes and downloaded models.
