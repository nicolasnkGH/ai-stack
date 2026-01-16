# Usage Guide (Operator’s Manual)

This document explains how to use the Private AI Stack in day-to-day operation for research, development, and creative workflows.
It focuses on how to interact with the system - not how to install it.

---

## 1. Accessing the System

The stack is accessed through a Cloudflare Tunnel with HTTPS and optional MFA.

*  Primary access URL:
    

`https://ai.YOUR.DOMAIN`

*  Authentication is handled by:
    - Open WebUI local accounts
    - Cloudflare Zero Trust (if enabled) and higly suggested for max security.
    
Once logged in, all interaction happens through the Open WebUI interface.

## 2. Model Selection & Usage

Model selection happens in the top bar of Open WebUI.

### Available Model Types

Typical models in this stack:
*   General chat models (LLaMA, Qwen, Gemma, etc.)
    
*   Vision models (Qwen-Vision)
    
*   Embedding models (bge-m3, etc.)

### Switching Models

1.  Click the model selector at the top of the chat
    
2.  Choose the desired model
    
3.  The model applies immediately to new messages

Best practices:
*   Use lighter models for:
    *   Quick questions
    *   Brainstorming
    *   Casual chat
*   Use larger models for:
    *   Long reasoning
    *   Code review
    *   Academic analysis

---

## 3. Research Workflow

### Web Search & RAG

If web search is enabled:

*   Open WebUI automatically queries SearXNG when:
    
    *   Web search is enabled in the chat
        
    *   RAG Web Search is active
        
Behind the scenes:

*   Queries go to: `http://searxng:8080/search?q=<query>&format=json`
    
*   Results are injected into the model context
    
To control behavior:

*   Enable/disable web search from the chat UI
    
*   Adjust result limits and timeouts via environment variables

### Document Upload (RAG)

You can upload:

*   PDFs
        
*   Text files
        
*   Notes
        
*   Research papers
        
Workflow:
    
1.  Upload document in chat
    
2.  System chunks and embeds using `bge-m3`
    
3.  Ask questions referencing the document
    
4.  Model retrieves relevant sections automatically

Recommended settings:

*   Top K: 3–5
    
*   Score threshold: ≥ 0.7
    
This avoids low-quality or hallucinated citations.

---

## 4. Vision & Image Understanding

### Vision Models

Vision requires selecting a vision-capable model (example: Qwen-Vision).

Workflow:
    
1.  Switch model to a vision model
    
2.  Upload an image
    
3.  Ask questions about the image
    
4.  When done, switch back to your normal text model
    
Vision models are heavier and slower. Only use when needed.

---

## 5. Image Generation (ComfyUI Integration)

Image generation is handled by ComfyUI.

### Basic Workflow

1.  Open ComfyUI (port 8188 or via Open WebUI integration)
    
2.  Load or build a workflow
    
3.  Select:
        
    *   Model
        
    *   Resolution
        
    *   Sampler
        
4.  Generate image
    
5.  Outputs are saved under:
    
`/mnt/ai/comfyui/root/output`

---

## Hardware Notes
- ComfyUI uses the RTX 3090 Ti directly
- High-resolution workflows consume significant VRAM
- If generation fails, reduce:
  - Resolution
  - Batch size
  - Steps

---

## 6. Text-to-Speech (TTS)

TTS is provided by OpenedAI-Speech.

### Usage

*   Enable TTS in Open WebUI
    
*   When the model responds, audio playback is available
    
*   Works only if:
    
    *   `WEBUI_URL` is HTTPS
        
    *   Browser is not blocking mixed content
        
### Voices

Voices are stored at:

`/mnt/ai/tts/voices`

Voice mapping is controlled by:

`/mnt/ai/tts/config/voice_to_speaker.yaml`


You can:
*   Add new voices by placing audio samples
    
*   Map voices to models or languages
    
*   Enable automatic language selection if configured

---

## 7. Multi-Language Usage

If multi-language models and TTS are configured:

*   Models can respond in different languages
    
*   TTS can map voices by language
    
*   Some models may mix languages if prompts are unclear

Best practice:
*   Explicitly specify language in your prompt:
    
    `Respond in Portuguese.`

---

## 8. Daily Operator Patterns

### Research Mode

*   Use:
    
    *   Larger models
    
    *   RAG document upload
    
    *   Web search
    
*   Workflow:
    
    1. Upload paper
    
    2. Ask structured questions
    
    3. Refine with follow-ups

### Coding / Engineering

*   Use:
    
    *   Medium or large models
    
    *   No web search unless needed
    
*   Ask for:
    
    *   Code review
    
    *   Architecture feedback
    
    *   Debug help

### Creative Work

*   Use:
    
    *   Image generation (ComfyUI)
        
    *   TTS for narration
        
    *   Vision models for image feedback
---

## 9. Performance Awareness

Key things to watch:

*   **GPU usage spikes during:**

    *   Image generation
    
    *   Large model inference
    
*   **CPU usage mostly during:**
    
    *   Web search
    
    *   Embedding generation
    
*   **Disk I/O heavy when:**
    
    *   Uploading large models
    
    *   Writing ComfyUI outputs

If performance drops:

*   Check GPU VRAM usage
*   Reduce image resolution
*   Switch to smaller models
*   Limit concurrent users

---

## Summary

This stack is designed to act like a private, self-hosted ChatGPT-style system with:
*   Local LLM inference
    
*   Web search and RAG
    
*   Vision
    
*   Image generation
    
*   Text-to-speech

Daily operation happens almost entirely through Open WebUI, with ComfyUI used for advanced image workflows.

---

## 📚 References

### 💬 Open WebUI

- **Open WebUI Documentation**  
  https://docs.openwebui.com/  
  Official guide for chat UI, models, settings, and features.

- **RAG Tutorial**  
  https://docs.openwebui.com/tutorials/tips/rag-tutorial#what-is-rag  
  Practical examples of using RAG for research and document analysis.

---

### 🖼 ComfyUI

- **ComfyUI Documentation**  
  https://docs.comfy.org/get_started/first_generation  
  Main user guide for building and running image generation workflows.

- **ComfyUI GitHub Repository**  
  https://github.com/comfyanonymous/ComfyUI  
  Source code, examples, and community workflows.



