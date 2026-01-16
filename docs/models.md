# Models Guide

This document describes the models available in the Private AI Stack and how to pick the right one quickly.

> [!TIP]
> If you only read one section: **Start with the Quick Picks** and the **Decision Cheatsheet**.

---

## ⚡ Quick Picks

| Goal | Best pick | Notes |
|---|---|---|
| Fast everyday chat | **Llama 3.1** | Lowest latency, good general use |
| Technical deep-dive / structured answers | **Qwen** | Great for engineering-style responses |
| Strong reasoning / long-form thinking | **Gemma 27B** | Slower, but higher “thinking” quality |
| Understand an image / screenshot | **Qwen Vision** | Switch back to your chat model after |
| Research with documents (RAG) | **bge-m3 (Embeddings)** | Used behind the scenes for document search |

---

## 🧭 Decision Cheatsheet

- **Need speed?** → Llama 3.1  
- **Need structure + technical detail?** → Qwen  
- **Need careful reasoning?** → Gemma 27B  
- **Need to analyze an image?** → Qwen Vision (then switch back)  
- **Need better citations from a PDF?** → Upload + RAG (embedding model does the indexing)

---

## 💬 Chat Models

### 🟢 Llama 3.1
**Tags:** `Fast` · `General` · `Low latency`

**Best for**
- Quick questions
- Brainstorming
- Summaries and rewrites

**Avoid when**
- You need very strict formatting or deep technical correctness

**Operator notes**
- If responses feel “too shallow,” switch to **Qwen** or **Gemma 27B**

---

### 🔵 Qwen (Chat)
**Tags:** `Technical` · `Structured` · `Good formatting`

**Best for**
- System engineering / DevOps questions
- Step-by-step troubleshooting
- Writing structured docs (Markdown, checklists)

**Avoid when**
- You want the fastest possible response time

**Operator notes**
- Great default when you care about correctness and structure.

---

### 🟣 Gemma 27B
**Tags:** `Reasoning` · `Long-form` · `Heavier`

**Best for**
- Complex analysis
- Tradeoffs and decision-making
- Multi-step reasoning

**Avoid when**
- You want low latency or many concurrent users

**Operator notes**
- If the answer is too long, ask: “Answer in 6 bullets.”  
- If it’s too slow, switch back to **Qwen**.

---

## 👁️ Vision Models

### 🟠 Qwen Vision
**Tags:** `Image understanding` · `Screenshots` · `Heavier`

**Use when**
- You need the model to interpret an image (UI screenshot, error screen, photo, diagram)

**Workflow**
1. Switch model → **Qwen Vision**
2. Upload image
3. Ask your question
4. Switch back to your normal chat model

> [!IMPORTANT]
> Vision models are slower and heavier. Use them only when needed.

---

## 📚 Embedding Models (RAG)

### 🟡 bge-m3
**Tags:** `RAG` · `Document search` · `Embeddings`

This model powers document retrieval:
- Chunking + indexing documents you upload
- Finding relevant sections during Q&A

**Recommended RAG settings**
- Top K: **3–5**
- Score threshold: **≥ 0.7**

> [!NOTE]
> You don’t usually “chat” with an embedding model — it’s used by the system to retrieve context.

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

## 🎨 Image Generation Models (ComfyUI)

Image generation is handled by ComfyUI using Stable Diffusion–style models.

### 🟥 Juggernaut XL v9 + LoRA
**Tags:** `SDXL` · `High quality` · `Creative` · `Heavy`

**Model type**
- Base: **Juggernaut XL v9 (SDXL)**
- Enhancements: **LoRA add-ons**

**Best for**
- Fantasy and RPG-style art
- Portraits and characters
- Stylized illustrations
- High-quality creative images

**Avoid when**
- You need very fast generations
- VRAM is limited or heavily used by chat models

**Operator notes**
- Higher resolution = much higher VRAM usage  
- If generation fails:
  - Lower resolution
  - Lower batch size
  - Reduce steps or sampler complexity
- LoRAs are used to:
  - Enforce style
  - Add specific character traits
  - Improve faces, clothing, or themes

**Typical workflow**
1. Open ComfyUI
2. Load Juggernaut XL v9 workflow
3. Select LoRA(s) if needed
4. Set:
   - Resolution
   - Steps
   - Sampler
5. Generate

**Storage path**
```bash
/mnt/ai/comfyui/root/models
```
LoRAs and models are stored under the ComfyUI root path so they persist across restarts.

---

## 🧩 Known Behaviors & Fixes

### Language mixing (Portuguese + English)
Some models may occasionally mix languages, especially if:
- Prompt language is unclear
- The model’s training bias favors English

**Fix**
- Add a hard instruction:  
  **“Respond only in Portuguese.”**
- If TTS sounds wrong, try a different voice mapping or model.

### Hallucinations / confident mistakes
**Fix**
- Prefer RAG when using uploaded docs
- Use web search for facts that change over time
- Ask for citations / quotes from the uploaded document

---

## 📥 Model Sources & Resources

### 💬 Ollama Models (Chat & Vision)

All chat and vision models used in this stack are managed by Ollama.

- **Ollama Model Library**  
  https://ollama.com/library  

Use it to:
- Browse available LLMs
- Check model sizes and hardware needs
- Pull models with:
```bash
  ollama pull <model-name>
```

Examples:
```bash
ollama pull llama3.1
ollama pull qwen2.5
ollama pull gemma
```
---

## 🎨 ComfyUI Models (Image Generation)

ComfyUI uses Stable Diffusion–style models, checkpoints, and LoRAs.

### Common Model Sources

- **Civitai (Models, LoRAs, Styles)**  
  https://civitai.com  

- **Hugging Face (Checkpoints, VAEs, ControlNets)**  
  https://huggingface.co  

- **ComfyUI Community Workflows**  
  https://github.com/comfyanonymous/ComfyUI  
  https://github.com/comfyanonymous/ComfyUI_examples  

### Where to Place Models

```bash
/mnt/ai/comfyui/root/models/
```

### Typical Subfolders

- `checkpoints/` – Base models (Juggernaut, SDXL, etc.)  
- `loras/` – Style and character LoRAs  
- `vae/` – VAEs  
- `controlnet/` – ControlNet models  

---

## 📚 RAG / Embedding Models

Embedding models are managed by Open WebUI and Ollama.

### References

- https://ollama.com/library  

### Common Embedding Models

- `bge-m3`  
- `nomic-embed-text`  


