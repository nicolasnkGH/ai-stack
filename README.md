# Private AI Stack

A self-hosted, private AI system similar to ChatGPT with image generation, web search, speech-to-text, and text-to-speech, running on Proxmox with Docker, GPU acceleration, and secure external access via Cloudflare Tunnel.

## ✨ Features

- 💬 Multiple chat models (Llama 3.1, Qwen 7B, Gemma 27B)
- 🖼️ Image generation with ComfyUI
- 👁️ Vision models (image + text using Qwen Vision)
- 🌐 Web search via SearXNG
- 🔊 Text-to-Speech (OpenedAI-Speech, OpenAI-compatible API)
- 🎙️ Speech-to-Text (Whisper / faster-whisper)
- 🔐 Secure access via Cloudflare Tunnel + MFA
- 📦 All data stored on NAS via NFS mount
- ⚡ GPU-accelerated inference

---

## 🧱 Architecture

```
Proxmox
└── AI VM (Ubuntu)
    ├── GPU Passthrough
    ├── Ollama (LLMs)
    └── NFS Mount → NAS (/mnt/ai)
    ├── Docker
    │   ├── Open WebUI
    │   ├── ComfyUI (Images)
    │   ├── OpenedAI-Speech (TTS)
    │   ├── Whisper (STT)
    │   ├── SearXNG (Web Search)
    │   └── Redis
```

---

## 📁 Storage Layout

The VM is running on a separate disk for the operating system only. All data persistence of Docker containers is configured to use an NFS mount that maps local OS mounting points inside the Docker containers at the mounting point /mnt/ai-stack.

```
/mnt/ai/
├── openwebui/
│   └── cache/audio/speech
├── comfyui/
    ├── models/
├── tts/
│   ├── voices/
│   └── config/
├── redis/
├── searxng/
```

---

## 🧩 Services

| Service           | Purpose                          |
|-------------------|----------------------------------|
| Open WebUI        | Main UI and user system          |
| Ollama            | Local LLM runtime                |
| ComfyUI           | Image generation                 |
| OpenedAI-Speech   | Text-to-Speech (OpenAI API)      |
| Whisper            | Speech-to-Text                  |
| SearXNG           | Web search engine                |
| Redis             | Cache/session backend            |
| Cloudflare Tunnel | Secure external access           |

---

# 🤖 Models

### Chat Models
#### Llama 3.1
* Description:
    - A fast and versatile chat model designed for general conversations, covering a wide range of topics from everyday discussions to more in-depth subjects.
* Use Cases:
  - General conversation
  - Everyday discussions
  - Informal knowledge sharing

#### Qwen 2.5 14B
* Description:
    - A technical chat model with enhanced vision capabilities, designed for handling complex technical discussions and tasks that involve images and multimodal data.
    - Features a 14.7 billion parameter transformer-based architecture [13].
    - Developed by Alibaba Cloud's Qwen Team, featuring a large context window of up to 128,000 tokens [13].
* Use Cases:
    - Detailed technical analysis
    - Multilingual applications
    - Long-form content generation
    - Image understanding (use Qwen Vision)
    - Code generation

#### Gemma 27B
* Description:
    - A strong reasoning chat model, designed for complex problem-solving, abstract thinking, and engaging in debates and discussions that require deep cognitive processing.
* Use Cases:
    - Complex problem-solving
    - Abstract thinking
    - Debate and discussion

---

### Vision Models
#### Qwen 7B
* Description:
    - A technical chat model with vision capabilities, designed for handling complex technical discussions and tasks that involve images.
* Use Cases:
    - Technical discussions
    - Image understanding (use Qwen Vision)
    - Code generation

---

### Image Generation
#### ComfyUI backend – User interface-focused
* Description:
    - A user interface-focused chat model designed for generating interactive UI components, suitable for UI design and prototyping, human-computer interaction, and user experience research.
    - Features **Juggernaut XL v9 with Lora** models, which enhance its capabilities in handling complex UI tasks.
* Use Cases:
    - UI design and prototyping
    - Human-computer interaction
    - User experience research

---

# 🔊 Text-to-Speech
* Description:
    - A text-to-speech system designed to convert written text into spoken words in both Portuguese and English.
    - Features models trained on datasets specific to Brazilian Portuguese and English, ensuring accurate pronunciation and natural speech synthesis.
* Use Cases:
    - Converting written text into audible speech for multilingual applications
    - Enhancing accessibility by providing voice outputs in both languages

---

# 🌍 Web Search
* Description:
    - Enabled capability for the models to perform web search for information retrieval.
    - Powered by SearXNG
    - Enable per-message using the integrations icon
    - Slower than offline responses due to live queries
        - Users can enable and disable this option as needed

---

# ⚙️ Performance Notes
Typical VM allocation:
  - 6–8 vCPU
  - 32-64 GB RAM
  - GPU passthrough (for this project, I am using a NVIDIA RTX 3090 TI)
  - Image generation is GPU-bound
  - Chat mostly GPU-bound, low CPU usage

---

Developed and maintained by Nicolas Teixeira.

