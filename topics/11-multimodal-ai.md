# Multimodal AI

[← AI Safety & Ethics](10-ai-safety-ethics.md) | [AI Infrastructure →](12-ai-infrastructure.md)

20+ questions on vision-language models, diffusion models, multimodal RAG, and production multimodal systems.

**Difficulty**: `[B]` Beginner · `[I]` Intermediate · `[A]` Advanced

---

## Table of Contents

- [Foundations](#foundations)
- [Vision-Language Models](#vision-language-models)
- [Generative Models](#generative-models)
- [Multimodal RAG](#multimodal-rag)
- [Audio & Speech](#audio--speech)
- [Production Considerations](#production-considerations)
- [Troubleshooting Scenarios](#troubleshooting-scenarios)

---

## Foundations

**Q: What is multimodal AI?** `[B]`

Multimodal AI processes and generates multiple types of data — text, images, audio, video — within a unified model or system. Unlike unimodal models (text-only LLMs), multimodal models can:

- **Understand multiple modalities**: "Describe what is in this image" (image → text)
- **Generate multiple modalities**: "Create an image of a sunset" (text → image)
- **Reason across modalities**: "Is the chart in this PDF consistent with the text?" (image + text → text)

**Why multimodal matters**: the real world is multimodal. Automating document processing, medical imaging analysis, content moderation, and customer support all require understanding multiple data types simultaneously.

---

**Q: What are the main approaches to fusion in multimodal models?** `[I]`

| Fusion Type | Where Modalities Combine | Examples |
|-------------|------------------------|---------|
| **Early fusion** | Raw input level — modalities concatenated before encoding | Token-level interleaving (Flamingo) |
| **Late fusion** | After separate encoders — outputs combined | CLIP (image encoder + text encoder separately trained) |
| **Intermediate/cross-attention** | Cross-attention between modality encoders during processing | LLaVA, Gemini |

**Modern trend**: cross-attention fusion (intermediate) has become dominant in vision-language models. Image and text streams are encoded separately then interact through cross-attention at multiple layers.

---

## Vision-Language Models

**Q: How does CLIP work?** `[I]`

CLIP (Contrastive Language-Image Pretraining, OpenAI 2021) learns to align image and text representations through contrastive learning on 400M image-text pairs:

**Training**:
- Image encoder (ViT or ResNet) encodes each image → image embedding
- Text encoder (Transformer) encodes each caption → text embedding
- Contrastive loss: maximize cosine similarity of matching pairs; minimize for non-matching pairs
- Trained on (image, caption) pairs scraped from the internet

**Zero-shot classification**:
```
Image of a dog
Text prompts: "a photo of a cat", "a photo of a dog", "a photo of a bird"
→ Find text with highest cosine similarity to image embedding
→ "a photo of a dog" → prediction: dog
```

**Why CLIP is foundational**: CLIP embeddings bridge vision and language. Used in: zero-shot image classification, semantic image search, conditioning text-to-image generation (DALL-E, Stable Diffusion).

---

**Q: How do LLaVA and GPT-4V connect vision to language models?** `[I]`

**LLaVA (Large Language and Vision Assistant)** architecture:
1. Visual encoder (CLIP ViT-L) processes the image → visual tokens
2. Projection layer (MLP) maps visual tokens to language embedding space
3. LLM (LLaMA/Mistral) processes concatenation of visual tokens + text tokens

Training:
1. Pre-training: only projection layer; train to align image features with language space on image-caption data
2. Instruction fine-tuning: full model fine-tuned on visual instruction following (GPT-4-generated VQA data)

**GPT-4V (and GPT-4o)**: uses similar architecture with proprietary vision encoder + multimodal pre-training at much larger scale. GPT-4o additionally processes audio natively in the same model.

---

**Q: What is a Vision Transformer (ViT)?** `[I]`

ViT (Dosovitskiy et al., 2020) applies the Transformer architecture to images:

1. **Patch embedding**: split image (e.g., 224×224) into fixed patches (e.g., 16×16 = 196 patches)
2. Each patch is flattened and linearly projected to an embedding
3. Add positional embeddings to patch embeddings
4. Pass through standard Transformer encoder layers (multi-head self-attention)
5. Classification token ([CLS]) pools global image representation

**Why ViT won**: Transformers scale better than CNNs with more data. At large scale, ViT outperforms CNNs on image classification. Foundation for all modern vision encoders used in VLMs.

---

## Generative Models

**Q: How do diffusion models work?** `[I]`

Diffusion models (DDPM, Stable Diffusion) generate images by learning to reverse a noise-adding process:

**Forward process** (training data creation):
```
Original image x₀ → add Gaussian noise step by step → x₁ → x₂ → ... → xT ≈ pure noise
```

**Reverse process** (what the model learns):
```
Pure noise xT → predict and remove noise step by step → ... → x₁ → x₀ (generated image)
```

The model (U-Net or Transformer) is trained to predict the noise added at each step, conditioned on a text prompt (via cross-attention with CLIP text embeddings).

**Stable Diffusion innovation**: latent diffusion — run the diffusion process in a compressed latent space (VAE encoder/decoder), not pixel space. 8–16× faster than pixel-space diffusion.

---

**Q: What is classifier-free guidance (CFG) in image generation?** `[I]`

CFG controls how strongly the generated image adheres to the text prompt vs unconstrained quality:

**Training**: randomly drop the conditioning (text prompt) with probability p. Model learns both conditional and unconditional generation.

**Inference**:
```
noise_pred = unconditional_pred + guidance_scale × (conditional_pred - unconditional_pred)
```

- `guidance_scale = 1`: no guidance, creative but may ignore prompt
- `guidance_scale = 7-12`: good balance (typical default)
- `guidance_scale > 15`: strictly follows prompt but may produce artifacts

**Intuition**: the difference between conditional and unconditional predictions represents what's "prompt-specific." Amplifying that difference makes the image more prompt-adherent.

---

## Multimodal RAG

**Q: How do you implement RAG over documents containing text and images?** `[I]`

Standard text RAG ignores figures, charts, and diagrams in PDFs — critical information loss for technical documents.

**Approaches**:

**Option 1 — Extract and describe images separately**:
```
PDF → extract images → GPT-4V generates text description of each image → 
index descriptions in vector DB alongside text chunks
```
Fast, cheap; loses visual nuance; image descriptions may miss important visual details.

**Option 2 — Multi-vector retrieval (ColPali/ColQwen)**:
```
PDF pages → render as images → embed page images with vision encoder → 
query → retrieve relevant page images → VLM answers from page images
```
Preserves original visual information; higher cost per query.

**Option 3 — Hybrid**:
- Text-based retrieval for most queries
- Rerank results; if top results reference figures, also retrieve image embeddings for those figures
- Pass relevant images to VLM for generation

---

## Audio & Speech

**Q: How do speech recognition (ASR) and text-to-speech (TTS) work in AI pipelines?** `[B]`

**ASR (Automatic Speech Recognition)**: audio → text
- **Whisper** (OpenAI): encoder-decoder Transformer trained on 680K hours of audio. Audio → log-mel spectrogram → Transformer encoder → Transformer decoder → transcription. Strong multilingual capability.
- Streaming ASR: use CTC-based models for low-latency transcription; Whisper is batch-only.

**TTS (Text-to-Speech)**: text → audio
- **Neural TTS** (ElevenLabs, OpenAI TTS, Bark): autoregressive or diffusion-based; produces natural, expressive speech
- **XTTS/Coqui**: open-source, voice cloning capable
- **Two-stage**: text → phonemes/tokens → waveform via vocoder (WaveNet, HiFi-GAN)

**Voice AI pipeline** (customer support voice bot):
```
User speaks → ASR (Whisper) → text → LLM → response text → TTS → audio → user hears
Latency: ASR~300ms, LLM TTFT~500ms, TTS~200ms → ~1s perceived latency with streaming
```

---

## Production Considerations

**Q: What are the unique inference challenges for multimodal models?** `[I]`

1. **Higher compute cost**: VLM inference is expensive — image tokens add 256–2048 tokens to the context, each requiring attention computation
2. **Variable input sizes**: images at different resolutions produce different token counts — complicates batching
3. **Vision encoder caching**: if the same image is queried multiple times, cache the visual tokens
4. **Bandwidth cost**: sending base64-encoded images in API requests is bandwidth-intensive. Use image URLs where possible.
5. **Latency**: multimodal models are slower than text-only due to the vision encoder overhead (50–200ms additional)

**Optimization**:
- Tile-based processing (LLaVA 1.6): process image at native resolution by tiling; each tile processed independently → better quality for high-res images without quadratic attention cost
- Reduce image resolution when visual details aren't needed
- Use smaller/faster vision models for classification; reserve large VLMs for open-ended VQA

---

**Q: What is Video AI and what are the key technical challenges?** `[I]`

Video AI extends vision capabilities to temporal sequences of frames.

**Models**: Gemini 1.5 Pro (1M context for 1hr video), GPT-4o (limited video), specialized models (VideoLLaMA, LLaVA-Video).

**Challenges**:
1. **Scale**: 1 second of video at 24fps = 24 frames. A 1-minute video ≈ 1,440 frames ≈ 300K+ tokens if naively encoded.
2. **Temporal reasoning**: understanding action sequences, causal chains, changes over time
3. **Key frame selection**: can't encode all frames; select representative frames. Motion detection, scene change detection, or learned frame selection.
4. **Temporal alignment**: synchronizing visual and audio streams for multi-modal video understanding

**Current state (2025)**: long-context models (Gemini 1.5 Pro) can process hours of video; accuracy on complex temporal reasoning is still limited.

---

## Troubleshooting Scenarios

**Q: Your VLM is misidentifying objects in images. How do you debug?** `[I]`

1. **Resolution check**: are input images too low resolution? VLMs have minimum effective resolution (~224×224); very low-res images degrade performance significantly.
2. **Image preprocessing**: check if images are being corrupted in your preprocessing pipeline (wrong normalization, color channel issues).
3. **Object size**: are objects too small in the image? Crop and zoom on the region of interest before sending.
4. **Prompt engineering**: the VLM's behavior depends heavily on the prompt. Be explicit: "Identify every distinct object visible in this image and describe its position."
5. **Model selection**: different VLMs have different strengths. GPT-4V excels at document understanding; specialized models may outperform on medical imaging or satellite imagery.
6. **Ground truth comparison**: collect 50 test images with correct labels; benchmark multiple models to identify if this is a systematic vs random failure.

---

**Q: Your text-to-image system generates biased images. How do you address it?** `[I]`

1. **Bias audit**: systematically test prompts like "a photo of a doctor" / "a photo of a nurse" across demographic variations. Measure demographic distribution of generated images.
2. **Prompt modification**: add diversity guidance to default prompts — "diverse group of people," "various ethnicities and genders"
3. **CLIP-based filter + regeneration**: use CLIP to check if generated image matches stereotyped representations; regenerate if bias threshold exceeded
4. **Fine-tuning on balanced data**: fine-tune the model (LoRA + DreamBooth) on a balanced, curated dataset with diverse representation
5. **User controls**: let users specify demographic attributes they want represented rather than relying on model defaults

---

## Advanced Audio Models

**Q: Explain Whisper's architecture and its strengths over traditional ASR.** `[I]`

Whisper (OpenAI, 2022) is a general-purpose speech recognition model trained on 680K hours of weakly-supervised audio-text pairs.

**Architecture**: encoder-decoder Transformer
- **Input**: 30-second audio window → log-mel spectrogram (80 mel bins × 3000 time steps)
- **Encoder**: ViT-like 2D convolutional layers + Transformer encoder → audio representations
- **Decoder**: autoregressive Transformer decoder → text tokens (multilingual, with language token)

**Key strengths over traditional ASR (DeepSpeech, Wav2Vec)**:
- **Robustness**: trained on diverse real-world audio including noisy environments, accents, phone calls
- **Zero-shot multilingual**: one model handles 99 languages without separate acoustic models
- **Timestamp generation**: produces word-level timestamps useful for subtitles and search
- **Code-switching**: handles mixed-language audio

**Limitations**:
- Batch-only (not streaming) — 30-second chunks; latency ~0.5–2s depending on GPU
- Long audio requires chunking with overlapping windows (word repetition at boundaries is common)
- Hallucination: can generate plausible-sounding transcription for inaudible audio

**Streaming ASR alternatives**: for low-latency requirements (<200ms), use CTC-based models (wav2vec2, Conformer-CTC) which don't require waiting for full audio context.

---

**Q: How do you build a production voice AI pipeline with low latency?** `[I]`

Target: user speaks → AI responds in < 1.5 seconds perceived latency.

**Architecture**:
```
User audio stream
    ↓
VAD (Voice Activity Detection)
  - Detect when user stops speaking (~50ms latency)
  - Use WebRTC VAD or Silero VAD
    ↓
Streaming ASR
  - For speed: Faster-Whisper (4× faster than original)
  - Or: streaming Conformer model with 200ms chunk processing
  - Begin transcription while user is still speaking
    ↓
LLM generation (streaming)
  - Start generation as soon as ASR produces first complete sentence
  - Stream tokens as generated
    ↓
Streaming TTS
  - Begin converting first sentence to speech while rest still being generated
  - ElevenLabs Streaming, OpenAI TTS Streaming, or Cartesia
    ↓
Audio playback
  - Play first audio chunk before full response is ready
```

**Latency breakdown (optimized)**:
| Component | Latency |
|-----------|---------|
| VAD end-of-speech detection | 100–300ms |
| Streaming ASR (to first token) | 200–400ms |
| LLM time to first token | 300–600ms |
| TTS to first audio chunk | 100–200ms |
| **Total perceived** | **700–1500ms** |

**Key optimization**: overlap stages. Don't wait for full transcription before starting LLM; don't wait for full LLM response before starting TTS.

---

**Q: How do you evaluate vision-language model outputs?** `[I]`

VLM evaluation is harder than text-only because ground truth is often multimodal and subjective.

**Standard benchmarks**:

| Benchmark | Tests | Notes |
|-----------|-------|-------|
| VQA v2 | Visual question answering | Classic; may be saturated |
| MMMU | Multi-discipline reasoning w/ images | College-level; hard |
| MMBench | Comprehensive VLM capabilities | Good overall measure |
| DocVQA | Document understanding (PDFs) | Critical for enterprise |
| TextVQA | Reading text in images | OCR-adjacent |
| HallusionBench | Hallucination detection | Specifically tests if VLM hallucinates about visual content |

**Production evaluation**:
- **Task accuracy**: for your specific task (e.g., extracting fields from invoice images), measure exact-match or F1
- **LLM-as-judge for VLMs**: use a strong VLM as evaluator — have it compare the model's answer to a reference answer given the same image
- **Hallucination rate**: does the model describe objects/text not present in the image? Test with images of clean backgrounds and ask if specific objects are present.

**Specific VLM failure patterns to test**:
- Counting objects incorrectly (common failure)
- OCR on unusual fonts or handwriting
- Spatial reasoning ("what is to the left of X?")
- Consistency: same question asked two different ways produces same answer

---

**Q: What is DreamBooth and how is it used for personalized image generation?** `[I]`

DreamBooth (Ruiz et al., 2022) fine-tunes a diffusion model to learn a specific subject (a person's face, a specific object) from just 3–10 reference images.

**Training**:
1. Collect 3–10 diverse images of the subject
2. Associate the subject with a rare token (e.g., `[V]`): "a photo of [V] dog"
3. Fine-tune the diffusion model on these pairs
4. Add **class-prior preservation**: generate images of the generic class ("a photo of a dog") to prevent forgetting what dogs look like in general

**After fine-tuning**: you can generate the subject in novel contexts:
- "[V] dog wearing a space suit"
- "A painting of [V] dog in the style of Van Gogh"

**Production use cases**:
- Personalized avatar generation (user uploads photo → generate in various styles)
- Product photography (upload product image → generate in various scenes)
- Virtual try-on (upload clothing item → generate on virtual model)

**Challenges**:
- Overfitting: model can "forget" general capabilities; class-prior helps
- Identity fidelity: very specific faces are hard to preserve accurately across diverse poses
- Inference speed: diffusion inference is slow (20–50 denoising steps); use SDXL-Turbo or LCM-LoRA for faster generation

---

[← AI Safety & Ethics](10-ai-safety-ethics.md) | [AI Infrastructure →](12-ai-infrastructure.md)
