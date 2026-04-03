# Gemma 4 + MLX — Technical Evaluation for Cepessa

**Date:** April 3, 2026  
**Status:** Recommendation for team discussion

---

## TL;DR

**Recommended: Yes.** Cepessa is a greenfield macOS app on Apple Silicon — this is the perfect time to build with local AI as core architecture, not as an afterthought.

---

## 1. Current App Status

The app is a skeleton — `CepessaApp.swift` + `ContentView.swift`, totaling 41 lines. No features, no data processing, no AI. Platform: **macOS 26.2+, SwiftUI, Apple Silicon only**.

This is an advantage — we can build the architecture *around* Gemma 4's capabilities instead of retrofitting.

---

## 2. What Gemma 4 Brings

### Model Variants

| Variant | Total Params | Active Params | Architecture | Modalities |
|---------|-------------|---------------|--------------|------------|
| **E2B** | ~2B | ~2B | Dense + PLE | Text, Image, Video, **Audio** |
| **E4B** | ~4B | ~4B | Dense + PLE | Text, Image, Video, **Audio** |
| **26B-A4B** | 26B | ~3.8B | MoE (128 experts, 8+1 active) | Text, Image, Video |
| **31B** | 31B | 31B | Dense | Text, Image, Video |

### Key Capabilities
- **Vision:** OCR, document parsing, chart/diagram comprehension, UI understanding, object detection with native JSON bounding-box output
- **Audio (E2B/E4B):** Speech recognition, speech translation, up to 30s audio input
- **Text/Reasoning:** Configurable thinking modes, agentic workflows, tool calling, structured JSON output, up to 256K context with TurboQuant
- **License:** Apache 2.0 (commercially permissive)

### MLX Ecosystem
- **mlx-vlm v0.4.3** (April 2, 2026): Day-0 support for full Gemma 4 family (vision + audio + MoE)
- **125+ quantized models** on mlx-community (4-bit to bf16)
- TurboQuant KV cache compression, LoRA/QLoRA fine-tuning on Mac
- FastAPI server with OpenAI-compatible endpoints

---

## 3. Performance on Apple Silicon

### Benchmarks (Mac Studio M4 Max 36GB)

| Metric | Qwen 3.5 27B (4-bit) | Gemma 4 26B-A4B (4-bit) |
|--------|----------------------|--------------------------|
| Generation speed | ~26 tok/s | **~100-112 tok/s** |
| Prompt processing | N/A | ~140-670 tok/s |
| Peak memory | 27-30 GB | **15.4-19.6 GB** |
| Disk size | ~15 GB | ~14 GB |

- **4x faster** generation than comparable models
- **50% less memory** consumption
- MoE architecture pairs perfectly with Apple's unified memory
- E2B/E4B run comfortably on a base M1 MacBook Air

### Memory Requirements

| Model | 4-bit | 8-bit | FP16 |
|-------|-------|-------|------|
| E2B | ~5 GB | — | ~15 GB |
| E4B | ~5 GB | — | ~15 GB |
| 26B-A4B (MoE) | ~18 GB | ~28 GB | ~52 GB |
| 31B (Dense) | ~20 GB | ~34 GB | ~62 GB |

### TurboQuant
- KV cache compression: ~4-5x reduction
- Perplexity within 2-7% of baseline
- Context window expansion: ~109K → ~536K tokens on same hardware
- Full Metal/Flash Attention support on Apple Silicon

---

## 4. Where to Integrate — Concrete Proposals

### A. Smart Document / Image Understanding
- **What:** User drags image/PDF/screenshot → app analyzes, extracts text (OCR), summarizes, classifies
- **Why here:** Vision is Gemma 4's strongest capability. On-device OCR without sending sensitive documents to cloud = clear value proposition
- **User value:** Absolute privacy for sensitive documents (contracts, medical, financial)

### B. Audio Processing Pipeline
- **What:** Transcription, call summarization, speaker identification — all local
- **Why here:** Gemma 4 E2B/E4B support audio natively (no separate Whisper + LLM pipeline)
- **User value:** Single pipeline from audio → text → insights, no cloud latency

### C. Contextual Screen/File Assistant
- **What:** App "sees" what's on screen / in file and provides smart context
- **Why here:** Vision + Text combination enables rich context understanding
- **User value:** Personal assistant that understands what you're doing without sending screenshots to cloud

### D. Local Knowledge Base
- **What:** User feeds documents → app builds local index → smart queries
- **Why here:** 32K-256K context window + MoE = efficient long document processing
- **User value:** "Private RAG" — all data stays on machine

---

## 5. Pros and Cons

### Pros
- **Absolute privacy** — zero data leaves the device
- **Zero API costs** — after initial download, everything is free
- **Excellent performance** — 100+ tok/s on Apple Silicon, real-time feel
- **Native multimodal** — vision + audio + text in one model
- **MoE efficiency** — only 3.8B active params out of 26B = memory and energy savings
- **Full offline** — works without internet
- **Day-0 ecosystem** — mlx-vlm, unsloth, 125+ quantized models, active community
- **Competitive advantage** — most apps still depend on cloud

### Cons / Risks
- **Requires Apple Silicon** — not relevant for Intel Macs (but we target macOS 26.2+, so fine)
- **Requires ~16-20GB RAM** — M1/M2 base (8GB) will struggle. M1 Pro/Max/M2 Pro+ work well
- **Heavy initial download** — 4-bit model is ~14-16GB
- **MLX ecosystem still young** — less documentation than PyTorch/ONNX
- **No easy fine-tuning** — domain-specific adaptation is more complex
- **Maintenance** — need to track mlx-vlm and model updates
- **Quality vs frontier cloud models** — in general reasoning, Gemma 4 < GPT-4o/Claude. In narrow tasks (OCR, classification) — excellent

---

## 6. Recommendation

**Build Cepessa as a "narrow & private local AI" app.**

Not a general chatbot. Not "local ChatGPT." Instead: **focused features that leverage vision + audio + text on-device.**

### Suggested Approach
1. Start with **Document Intelligence** (OCR + summarization + classification) — most mature use case, clearest value
2. Add **Audio Processing** as second feature
3. Build UX around drag & drop + quick actions — not a chat interface
4. Offer model size selection (4-bit / 8-bit) based on user's RAM

### Proposed Architecture
```
Cepessa/
├── Core/
│   ├── MLXEngine.swift          # Wrapper for mlx-vlm
│   ├── ModelManager.swift       # Download + model management
│   └── InferenceQueue.swift     # Background processing
├── Features/
│   ├── DocumentIntel/           # OCR, summarization, classification
│   ├── AudioProcess/            # Transcription, call summarization
│   └── ScreenAssist/            # Visual context understanding
└── UI/
    ├── MainDashboard.swift
    ├── DropZone.swift
    └── ResultsView.swift
```

---

## 7. Discussion Points

1. **Which use case to start with?** Document Intelligence seems most mature — agree?
2. **Minimum RAM requirement?** 16GB (M1 Pro+) or 24GB (M2 Pro+)?
3. **Cloud fallback?** For cases where local model isn't enough, or 100% on-device?
4. **Specific domain?** (medical, legal, financial) to focus on?
5. **Include Gemma 4 31B dense** as "pro mode" for users with more RAM?

---

## Sources

- [Google Blog: Gemma 4](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/)
- [HuggingFace Blog: Welcome Gemma 4](https://huggingface.co/blog/gemma4)
- [WaveSpeed AI: What Is Gemma 4?](https://wavespeed.ai/blog/posts/what-is-google-gemma-4/)
- [mlx-community Gemma 4 Collection](https://huggingface.co/collections/mlx-community/gemma-4)
- [MLX-VLM GitHub](https://github.com/Blaizzy/mlx-vlm)
- [Gemma 4 Local Operator (benchmarks)](https://github.com/EJellerson/gemma4-local-operator)
- [TurboQuant MLX PoC](https://github.com/sharpner/turboquant-mlx)
- [Unsloth: Gemma 4 Local Setup](https://unsloth.ai/docs/models/gemma-4)
