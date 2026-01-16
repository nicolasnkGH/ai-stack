# Configuration Guide

This document details the precise service configurations and storage optimizations for the Private AI Stack as defined in the current `docker-compose.yml`.

## 1. Service Environment Variables

The following variables are explicitly configured to manage service interaction and external access via your Cloudflare Tunnel.

### Open WebUI (`ai-openwebui`)

Directs Open WebUI to the native Ollama service running on the VM host (outside Docker).
```bash
- OLLAMA_BASE_URL=http://host.docker.internal:11434
```

Enables web search and RAG-based retrieval using the internal SearXNG container.
```bash
- ENABLE_WEB_SEARCH=true
- WEB_SEARCH_ENGINE=searxng
- SEARXNG_QUERY_URL=http://searxng:8080/search?q=<query>&format=json
- ENABLE_RAG_WEB_SEARCH=true
- WEB_SEARCH_RESULT_LIMIT=5
- WEB_SEARCH_TIMEOUT=12
```

Critical for TTS and browser audio playback.
Because access is through Cloudflare Tunnel over HTTPS, this must match the public HTTPS URL or browsers will block audio due to mixed-content rules.
```bash
- WEBUI_URL=https://ai.YOUR.DOMAIN
```

Allows containers to reach services running on the VM host (Ollama).
```bash
extra_hosts:
  - "host.docker.internal:host-gateway"
```

### SearXNG (`ai-searxng`)

Defines the public-facing base URL for SearXNG.
```bash
- BASE_URL=http://localhost:8088/
```
Internally, Open WebUI talks to it using the Docker service name:

Access paths:
- External (LAN): http://VM-IP:8088
- Internal (Docker network): http://searxng:8080

### ComfyUI (`ai-comfyui`)

Allows access from other containers and from the network.
```bash
- CLI_ARGS=--listen 0.0.0.0 --port 8188
```
Passes the RTX 3090 Ti directly to ComfyUI.
```
gpus: all
```

---

> [!IMPORTANT]
>
> **Security Note:** The current configuration stores all variables directly in the docker-compose.yml. If you transition to using sensitive data (e.g., specific API keys), it is a best practice to move those values to a `.env` file to prevent accidental exposure when pushing to GitHub.

---


## 2. ZFS Persistence & Volume Mapping

Persistence is managed through bind mounts to an NFS share mounted at /mnt/ai, backed by a ZFS dataset on the NAS.


| **Service** | **Host Path (NFS)** | **Container Path** | **Purpose** |
| --- | --- | --- | --- |
| **Redis** | `/mnt/ai/redis` | `/data` | Session and cache persistence. |
| **SearXNG** | `/mnt/ai/searxng` | `/etc/searxng` | Configuration and search settings. |
| **TTS** | `/mnt/ai/tts/voices` | `/app/voices` | Stored voice profiles. |
| **TTS** | `/mnt/ai/tts/config` | `/app/config` | TTS mapping config (voice_to_speaker.yaml). |
| **Open WebUI** | `/mnt/ai/openwebui` | `/app/backend/data` | Database and user data. |
| **ComfyUI** | `/mnt/ai/comfyui/root` | `/root` | Models, outputs, and workflows. |

### Storage Tuning

To support these high-volume mounts, your ZFS dataset on the NAS should be optimized for the large file patterns typical of AI models:
*   **Recordsize (1M):** Minimizes metadata overhead for multi-gigabyte LLM and Safetensors files.
    
*   **atime (off):** Eliminates unnecessary write traffic when models are read into VRAM.

---

## 3. GPU Hardware Integration

* **ComfyUI:** Uses `gpus: all` to access the RTX 3090 Ti from inside Docker for image generation.

* **Ollama:** Runs natively on the VM and uses the RTX 3090 Ti directly via the NVIDIA driver and CUDA.
  Docker is not involved in Ollama’s GPU access.

* **Open WebUI:** Does not use the GPU directly. It only connects to Ollama over the network using:

```bash
extra_hosts:
  - "host.docker.internal:host-gateway"
```

Note: `gpus: all` requires NVIDIA drivers + NVIDIA Container Toolkit installed.


---

## 4. Advanced Service Configuration

Native LLMs are offline, text-only models. The following configurations add vision, search, and voice capabilities.

### Web Search (SearXNG)

To enable the Open WebUI engine to retrieve live data, SearXNG must be configured to output JSON and allow local requests.

File:
```bash
/mnt/ai/searxng/settings.yml
```

Must include under the `search:` section:
```bash
search:
  formats:
    - html
    - json
```

And either:
```bash
limiter: false
```
Or allow your Docker subnet; otherwise, Open WebUI may get 403 errors.

### Text-to-Speech (OpenedAI-Speech)

The TTS service is configured to mimic the OpenAI API, allowing seamless integration with the Open WebUI frontend.
    

Important requirements:

* `WEBUI_URL` must be HTTPS (Cloudflare Tunnel)
* Otherwise, browsers may block audio playback due to mixed-content rules.

Voice storage:
```bash
/mnt/ai/tts/voices
```

Voice configuration:
```bash
/mnt/ai/tts/config/voice_to_speaker.yaml
```

Voices are selected by name and mapped to:

* ONNX models
* XTTS voices
* Auto-language support (if configured)


### ComfyUI Workflow Integration

ComfyUI is mapped at the `/root` level to ensure all custom nodes, models, and output images are persisted on the ZFS share.
    
Persistent mapping:
```bash
/mnt/ai/comfyui/root → /root
```

Includes:

- Models
- Custom nodes
- Workflows
- Output images

GPU support:
```bash
gpus: all
```

---

## 5. RAG Engine Tuning (Open WebUI)

For academic research and technical document analysis, the following RAG settings are configured within the UI:

- Embedding model: `bge-m3`
- High recall for long academic and technical documents
- Recommended:

  - Top K: 3–5
  - Score threshold: ≥ 0.7

This minimizes hallucinations in technical and academic workflows.

* * *

## 📚 References

### 🧩 Containers & GPU
- [NVIDIA Container Toolkit – Install Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)  
- [Docker Compose GPU Support](https://docs.docker.com/compose/gpu-support/)

### 🧠 LLM & UI
- [Open WebUI Documentation](https://docs.openwebui.com/)  
- [Ollama Documentation](https://ollama.com/docs)  
- [Ollama GPU Support](https://docs.ollama.com/gpu)

### 🔍 Search & RAG
- [SearXNG Documentation](https://docs.searxng.org/)  
- [Open WebUI RAG Overview](https://docs.openwebui.com/tutorials/tips/rag-tutorial#what-is-rag)

### 🔊 Voice & Audio
- [OpenedAI Speech (GitHub)](https://github.com/matatonic/openedai-speech)  
- [OpenAI Audio API](https://platform.openai.com/docs/api-reference/audio)

### 💾 Storage
- [ZFS Performance & Tuning](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Workload%20Tuning.html)  
- [NFS Performance (Oracle)](https://docs.oracle.com/cd/E19620-01/805-4448/6j47cnj0i/index.html)

### 🌐 Networking & Access
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)  
- [Cloudflare Zero Trust & MFA](https://developers.cloudflare.com/cloudflare-one/identity/)

### 🖥️ Virtualization
- [Proxmox PCI Passthrough](https://pve.proxmox.com/wiki/PCI_Passthrough)  
- [Proxmox NFS Storage](https://pve.proxmox.com/wiki/Storage:_NFS)
