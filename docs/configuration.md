## Service Configurations and Optimizations

This document details the precise service configurations and storage optimizations for the Private AI Stack, reflecting the current **Hybrid Storage** model and **Containerized Ollama** architecture.

### 1. Service Environment Variables

Configurations are now managed via a centralized `.env` file to support portability and security.

### Open WebUI (`ai-openwebui`)

*   **Ollama Connection:** Directs Open WebUI to the containerized Ollama service within the Docker network.

    ```bash
    OLLAMA_BASE_URL=http://ollama:11434
    ```

*   **Performance Caching:** Optimizes UI responsiveness and reduces redundant inference.

    ```bash
    ENABLE_BASE_MODELS_CACHE=True
    ENABLE_QUERIES_CACHE=True
    ```

*   **Web Search (SearXNG):** Enables real-time RAG-based retrieval.

    ```bash
    ENABLE_WEB_SEARCH=true
    SEARXNG_QUERY_URL=http://searxng:8080/search?q=<query>&format=json
    ```

*   **Public URL:** Must match your Cloudflare Tunnel domain to ensure audio/TTS playback.

    ```bash
    WEBUI_URL=https://YOUR.DOMAIN.HERE
    ```

### Google Drive Integration (Optional)

*   **Opt-in Setup:** Requires a Google Cloud Project and API keys.

    ```bash
    # ENABLE_GOOGLE_DRIVE_INTEGRATION=True
    # GOOGLE_DRIVE_CLIENT_ID=${GOOGLE_DRIVE_CLIENT_ID}
    # GOOGLE_DRIVE_API_KEY=${GOOGLE_DRIVE_API_KEY}
    ```
---


## 2. Hybrid Persistence & Volume Mapping

The stack utilizes a tiered storage strategy: **Local SSD** for I/O performance and **NFS/ZFS** for high-capacity asset storage.

| **Service** | **Host Path (Local SSD)** | **Host Path (NFS)** | **Container Path** | **Purpose** |
| --- | --- | --- | --- | --- |
| **Ollama** | `/opt/ai/ollama` | — | `/root/.ollama` | **Hot**: Performance-critical model weights. |
| **Open WebUI** | `/opt/ai/openwebui/webui.db` | — | `/app/backend/data/webui.db` | **Hot**: Fast UI database response. |
| **Open WebUI** | — | `/mnt/ai/openwebui` | `/app/backend/data` | **Cold**: Large user uploads & RAG documents. |
| **ComfyUI** | `/opt/ai/comfyui/models` | — | `/root/ComfyUI/models` | **Hot**: Rapid model loading into VRAM. |
| **ComfyUI** | — | `/mnt/ai/comfyui/output` | `/root/ComfyUI/output` | **Cold**: Generated high-resolution image assets. |
| **Redis** | `/opt/ai/redis` | — | `/data` |  |

---

## 3. GPU Hardware Integration

The **RTX 3090 Ti (24GB)** is passed through Proxmox to the Ubuntu VM and shared across containers.

*   **Ollama & ComfyUI:** Both utilize `deploy.resources.reservations` for direct GPU access.

*   **VRAM Management:** Main models use a `keep_alive` of **5m to 10m** to ensure VRAM is released for heavy image generation workflows.

*   **Driver Requirements:** The host requires NVIDIA 580.95+ and the NVIDIA Container Toolkit.


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
