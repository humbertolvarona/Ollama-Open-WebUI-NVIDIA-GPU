# Documentation for Installation and Execution with Docker (Ollama + Open-WebUI + NVIDIA GPU)

This documentation explains step by step how to deploy the **Ollama** service together with **Open-WebUI**, using Docker and NVIDIA GPU acceleration. Additional useful commands for managing the environment are also included.

---

## 1. Prerequisites

Before starting, make sure you have:

* **Ubuntu 22.04 or higher** (Ubuntu 24.04 also supported).
* **Docker** and **Docker Compose** installed.
* An **NVIDIA GPU compatible with CUDA**.
* Updated NVIDIA drivers on the system.

### Check installed drivers

```bash
ubuntu-drivers list
```

This command shows the recommended and available drivers for your NVIDIA GPU.

---

## 2. NVIDIA environment setup in Docker

### Add NVIDIA Container Toolkit repository

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list  \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#'  \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

### Install the toolkit

```bash
sudo apt update
sudo apt install -y nvidia-container-toolkit
```

### Configure Docker runtime for NVIDIA

```bash
sudo nvidia-ctk runtime configure --runtime=docker
```

Restart Docker to apply the changes:

```bash
sudo systemctl restart docker
```

---

## 3. Verify GPU inside Docker container

### Download base CUDA image

```bash
docker pull nvidia/cuda:12.9.1-base-ubuntu24.04
```

### Test GPU access from container

```bash
docker run --rm --gpus all nvidia/cuda:12.9.1-base-ubuntu24.04 nvidia-smi
```

This command should display the `nvidia-smi` output with the GPU status.

---

## 4. `docker-compose.yml` file definition

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    tty: true
    volumes:
      - ollama:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    depends_on:
      - ollama
    volumes:
      - open-webui:/app/backend/data
    ports:
      - "${OPEN_WEBUI_PORT-3000}:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - WEBUI_SECRET_KEY=YOUR_SECRET_KEY
    extra_hosts:
      - host.docker.internal:host-gateway

volumes:
  ollama: {}
  open-webui: {}
```

### How to obtain the `WEBUI_SECRET_KEY`

The `WEBUI_SECRET_KEY` is used to secure access to the Open-WebUI backend. You can generate a random secure key with the following command:

```bash
openssl rand -hex 32
```

This will generate a 64-character hexadecimal string that can be used as the `WEBUI_SECRET_KEY`.

Alternatively, if you prefer a base64-encoded key:

```bash
openssl rand -base64 32
```

Make sure to copy this value and replace it in your `docker-compose.yml` file.

---

## 5. Basic Docker Compose commands

### Start containers

```bash
docker compose up -d
```

### View service logs

```bash
docker compose logs
```

### Stop containers

```bash
docker compose down
```

---

## 6. GPU monitoring

Install **nvtop** (graphical GPU monitor):

```bash
sudo apt install nvtop -y
```

Run:

```bash
nvtop
```

This opens a real-time GPU usage monitor.

---

## 7. Additional useful commands

### View running containers

```bash
docker ps
```

### View resource usage by container

```bash
docker stats
```

### Enter a running container

```bash
docker exec -it ollama bash
```

### Remove unused images

```bash
docker image prune -a
```

### Remove unused volumes

```bash
docker volume prune
```

---

## 8. Recommended workflow

1. Check NVIDIA drivers with `ubuntu-drivers list`.
2. Configure NVIDIA Container Toolkit.
3. Test GPU with `nvidia-smi` inside a CUDA container.
4. Create the `docker-compose.yml` file shown above.
5. Generate a secure `WEBUI_SECRET_KEY` with `openssl rand -hex 32`.
6. Start the services with `docker compose up -d`.
7. Access the Open-WebUI interface at `http://localhost:3000` (or defined port).
8. Monitor GPU and resources with `nvtop` and `docker stats`.

---

## 9. Common troubleshooting

* **Error: "no devices found"** → Check that `nvidia-ctk runtime configure` was executed and Docker restarted.
* **Container does not detect GPU** → Verify `docker run --rm --gpus all nvidia/cuda:12.9.1-base-ubuntu24.04 nvidia-smi` works.
* **Open-WebUI cannot connect to Ollama** → Ensure `OLLAMA_BASE_URL` points to `http://ollama:11434` and both containers are running.
* **Port conflicts** → Modify `${OPEN_WEBUI_PORT-3000}:8080` to change the access port.

---

## 10. Useful references

* [NVIDIA Container Toolkit Docs](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/)
* [Docker Compose Docs](https://docs.docker.com/compose/)
* [Open-WebUI Repository](https://github.com/open-webui/open-webui)
* [Ollama Repository](https://github.com/jmorganca/ollama)

---

*Created by Humberto L. Varona in Aug/20205*

