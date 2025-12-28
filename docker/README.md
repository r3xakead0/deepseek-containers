# Ollama + Open WebUI + DeepSeek with Docker Compose

Local stack to run **Ollama** (model server) + **Open WebUI** (chat-style web interface) using **Docker Compose**.

---

## Quick architecture

- **ollama** → Model API at `http://localhost:11434`
- **open-webui** → Web interface at `http://localhost:3000`
- **model-loader** → One-shot container that downloads the `deepseek-r1:1.5b` model and then exits

```mermaid
flowchart TB
  %% ====== System Boundary ======
  subgraph SYS[System: Local AI Chat Stack]
    direction TB

    %% --- Container Boundary ---
    subgraph STACK[Container Stack: Docker Compose]
      direction LR

      WEBUI[
        Container: Open WebUI<br>
        Purpose: Chat UI and user management<br>
        Port: 8080 - mapped to 3000
      ]

      OLLAMA[
        Container: Ollama<br>
        Purpose: Model runtime and API<br>
        Port: 11434
      ]

      LOADER[
        Container: Model Loader<br>
        Purpose: Download model at startup<br>
        Runs: ollama pull deepseek-r1:1.5b
      ]
    end

    %% --- Data Stores ---
    V_OLLAMA[
      Data Store: ollama-data<br>
      Stores: Models and cache
    ]

    V_WEBUI[
      Data Store: open-webui-data<br>
      Stores: Users, chats, config
    ]
  end

  %% ====== External Actor ======
  USER[Person: User]

  %% ====== Relationships ======
  USER -->|Uses browser<br>http://localhost:3000| WEBUI
  USER -->|Optional API access<br>http://localhost:11434| OLLAMA

  WEBUI -->|HTTP<br>OLLAMA_BASE_URL| OLLAMA
  LOADER -->|HTTP<br>Download model| OLLAMA

  OLLAMA -->|Persists model files| V_OLLAMA
  WEBUI -->|Persists application data| V_WEBUI
```

---

## Requirements

- Docker
- Docker Compose (v2 recommended)
- Free ports:
  - `11434` (Ollama)
  - `3000` (Open WebUI)

---

## 1) Installation

### 1.1 Install Docker on Linux

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo usermod -aG docker $USER
```

Log out and log back in, then verify:
```bash
docker --version
docker compose version
```

### 1.2 Install Docker on MacOS
```bash
brew install --cask docker
```

Start Docker Desktop and verify:
```bash
docker --version
docker compose version
```

### 1.3 Install Docker on Windows
Install Docker Desktop and enable WSL2
http://docs.docker.com/desktop/setup/install/windows-install/

Verify:
```bash
docker --version
docker compose version
```

---


## 2) Configuration

Create a `.env` file in the same directory as `docker-compose.yml`:

```env
WEBUI_SECRET_KEY=change-me-to-a-long-random-secret
ENABLE_SIGNUP=true
```

Generate a secure secret:
```bash
openssl rand -hex 32
```

---

## 3) Execution

From the project directory:

```bash
docker compose -f docker-compose.yml up -d
```

Check status:
```bash
docker ps
docker compose -f docker-compose.yml ps
```

Access:
- Open WebUI → http://localhost:3000
- Ollama API → http://localhost:11434

> On first access, Open WebUI will ask you to create the first user (admin). This is expected behavior.

---

## 4) Testing and validation

### 4.1 Verify Ollama
```bash
curl http://localhost:11434
```

Expected response:
```
Ollama is running
```

---

### 4.2 List models
```bash
curl http://localhost:11434/api/tags
```
You should see deepseek-r1:1.5b listed.

---

### 4.3 Validate Open WebUI
Open in the browser:
```
http://localhost:3000
```

Create a user, log in, and test a chat with the model.

---

### 4.4 Logs
```bash
docker logs ollama-deepseek
docker logs open-webui
docker logs model-loader
```

Or all services:
```bash
docker compose logs -f
```

---

## 5) Stop and restart

### Stop
```bash
docker compose -f docker-compose.yml stop
```

### Start again
```bash
docker compose -f docker-compose.yml start
```

---

## 6) Teardown

### 6.1 Bring down containers (keeps data)
```bash
docker compose -f docker-compose.yml down
```

### 6.2 Full cleanup (⚠️ removes volumes)
```bash
docker compose -f docker-compose.yml down -v
```

This removes:
- Downloaded models (Ollama)
- Users and conversations (Open WebUI)

---

## Security recommendations

- Change `WEBUI_SECRET_KEY` to a long, random secret
- After creating the first user, disable signups:
  ```env
  ENABLE_SIGNUP=false
  ```
  then recreate containers:
  ```bash
  podman compose -f podman-compose.yml up -d --force-recreate
  ```

---

## Expected final state

- Ollama responding on `localhost:11434`
- Open WebUI accessible on `localhost:3000`
- Model available and functional
