# Models Guide

This document describes the models available in the Private AI Stack and how to pick the right one quickly.

> [!TIP]
> If you only read one section: **Start with the Quick Picks** and the **Decision Cheatsheet**.

---

## ⚡ Quick Picks

| Goal | Best Pick | VRAM Usage | Notes |
| --- | --- | --- | --- |
| **Fast Everyday Chat** | **Gemma 3 (12B)** | ~8GB | Your primary driver. Now handles images and logic in a single session. |
| **Deep Engineering** | **Qwen 2.5 Coder** | ~8-12GB | Optimized for the code/logic tasks you do. |
| **Complex Reasoning** | **Gemma 3 (27B)** | ~18-22GB | Peak intelligence for multi-step engineering logic and detailed image analysis. |
| **Research with Documents (RAG)** | **bge-m3** | **Minimal** | Powers the background search for your uploaded technical PDFs. |

---

### 📥 Model Sources & Documentation

Instead of manual descriptions, refer to the official libraries for full parameters and benchmarks:

*   **Ollama Model Library** | [Browse Models](https://ollama.com/library)

    *   Use this to pull the specific versions of **Gemma 3** (12B/27B) and **Qwen 2.5 Coder** used in this stack.

    *   **Command**: `docker exec -it ollama ollama pull gemma3:12b`.

*   **Civitai (Stable Diffusion)** | [Browse SDXL Checkpoints](https://civitai.com/)

    *   Source for **Juggernaut XL** and custom LoRAs used by the ComfyUI backend.

*   **Hugging Face** | [Technical Specs](https://huggingface.co/)

    *   Reference for deep technical architecture on the **bge-m3** embedding model and vision encoders.

---

## 🌐 Web Search Behavior (SearXNG)

**When enabled**, Open WebUI can pull live search results into the model context.

**Tradeoffs**
- ✅ More up-to-date answers
- ❌ Slightly slower responses

**Operator tip**
- Use web search for: versions, breaking changes, niche facts, “what’s new”
- Disable for: offline reasoning, coding, private docs

---

## 🎨 Image Generation (ComfyUI)

Image generation is handled by a node-based **ComfyUI** backend utilizing **Stable Diffusion XL (SDXL)** models.

### 🛠️ Hardware & Storage Logic

*   **Performance Tier (Local SSD)**: All checkpoints (e.g., **Juggernaut XL v9**) and LoRAs are stored in `/opt/ai/comfyui/models` to ensure rapid injection into VRAM.

*   **Capacity Tier (NFS)**: Final image outputs and workflow JSONs are saved to `/mnt/ai/comfyui/output` for long-term persistence.

*   **VRAM Contention**: Running high-resolution SDXL workflows concurrently with **Gemma 3 (27B)** can exceed the **24GB limit** of the RTX 3090 Ti. It is recommended to unload LLMs before heavy creative sessions.

### 📋 Operator Workflow

1.  **Model Loading**: Checkpoints are pulled from `/opt/ai/comfyui/models/checkpoints/`.

2.  **Enhancements**: LoRAs for style or character consistency are loaded from `/opt/ai/comfyui/models/loras/`.

3.  **Resolution Scaling**: Higher resolutions significantly increase VRAM usage and I/O wait.

4.  **Failure Recovery**: If a generation fails (OOM), lower the resolution or reduce the batch size in the ComfyUI nodes.

---

## 🧩 Known Behaviors & Operational Fixes

### 🌐 Multi-Language Handling

Some models (especially **Gemma 3**) may default to English for technical DevOps or Linux queries due to training bias.

*   **Fix**: Append a System Prompt instruction: _"Respond only in Portuguese"_ for general chat.

*   **Audio Fix**: If TTS output is garbled, verify the voice mapping in your `/opt/ai/tts/config/voice_to_speaker.yaml`.

### 🧠 Hallucinations & Fact-Checking

*   **Technical Docs**: Force RAG for uploaded PDFs to ensure the model uses your specific documentation as the "source of truth".

*   **Live Data**: Toggle **SearXNG** for queries regarding version numbers or breaking changes (e.g., "What's the latest Proxmox 8.3 bug?").

*   **Verification**: Always ask the model to _"provide direct quotes from the uploaded document"_ to confirm accuracy.
---

## 📥 Model Sources & Management

🖥️ **How to find it in Open WebUI**

1.  **Access the Interface**: Log in to your instance at `https://YOUR.DOMAIN.HERE`.

2.  **Open Settings**: Click on your **Profile Picture/Name** in the bottom-left corner and select **Settings**.

3.  **Navigate to Models**: Click the **Models** tab in the sidebar of the settings window.

4.  **Pull a Model**:

    *   Find the input field labeled **"Pull a model from Ollama.com"**.

    *   Type the name and tag of the model you want (e.g., `gemma3:12b` or `qwen2.5-coder`).

    *   Click the **Download icon** (downward arrow) next to the input box.

5.  **Monitor Progress**: A progress bar will appear at the top of the screen while the model is pulled directly into your **Local SSD (**`/opt/ai`**)** storage.

🤖 **Ollama (Chat, Vision & RAG)**

Managed within the `ollama` container. Explore models at the [Ollama Library](https://ollama.com/library).

*   **GUI (Recommended)**: Navigate to **Settings > Models** in the WebUI and enter the model tag (e.g., `gemma3:12b`).

*   **CLI**: Pull models directly into the local SSD:

    ```bash
    docker exec -it ollama ollama pull gemma3:12b
    ```

*   **Embedding Model**: `bge-m3` is pre-configured for background RAG indexing.

### 🎨 ComfyUI (Image Generation)

Checkpoints and LoRAs are sourced from [Civitai](https://civitai.com/) or [Hugging Face](https://huggingface.co/).

*   **Performance Path**: Models **must** be placed on the SSD to avoid GPU starvation:

    ```
    /opt/ai/comfyui/models/
    ```

*   **Hierarchy**: Place base models in `checkpoints/` and style refinements in `loras/`.

---
_Maintained by Nicolas Teixeira._
