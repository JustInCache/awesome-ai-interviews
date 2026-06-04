# Changelog

[← Back to README](README.md)

All notable additions and improvements to this repository are documented here.

---

## [2026-06] — Major Expansion

### Added
- **`topics/13-reasoning-models.md`** — Complete guide to reasoning models: test-time compute, o1/o3/DeepSeek-R1/Claude 3.7, RLVR training, GRPO, process reward models, production engineering, benchmarks (30+ questions)
- **`companies/openai.md`** — OpenAI interview style guide: system design, safety, scaling, RLHF, behavioral questions (12 questions)
- **`companies/anthropic.md`** — Anthropic interview style guide: Constitutional AI, mechanistic interpretability, alignment, scalable oversight (10 questions)
- **`companies/google-deepmind.md`** — Google DeepMind interview style guide: math depth, Chinchilla, TPUs, multimodal AI, AlphaFold (10 questions)
- **`companies/meta-ai.md`** — Meta AI interview style guide: LLaMA, open-source AI, efficiency, PyTorch, distributed training (10 questions)
- **`cheatsheets/llm-formulas.md`** — LLM formulas and numbers cheatsheet: attention math, memory calculations, scaling laws, LoRA parameter counts, sampling parameters, evaluation metrics
- **`cheatsheets/rag-checklist.md`** — End-to-end RAG pipeline checklist: ingestion, retrieval, generation, evaluation, common failure modes, tuning parameters
- **`cheatsheets/system-design-patterns.md`** — 8 canonical AI system design patterns with architecture diagrams: RAG, agent loop, multi-agent, inference stack, eval pipeline, fine-tuning, streaming, model router

### Improved
- **`README.md`**: Added Quick Start section, What's New section, Companies table, Cheatsheets section, star history chart, updated navigation with new files
- **`topics/05-fine-tuning.md`**: Expanded with data synthesis, DPO implementation, merge/serving of adapters, fine-tuning pitfalls
- **`topics/06-vector-databases.md`**: Expanded with HNSW internals, product quantization, ANN benchmark comparisons, production tuning guide
- **`topics/11-multimodal-ai.md`**: Expanded with audio models (Whisper), video understanding, cross-modal retrieval, production considerations
- **`by-experience/junior.md`**: Added 15 practice questions with answers targeted at junior level
- **`by-experience/mid-level.md`**: Added 15 practice questions with answers targeted at mid-level
- **`by-experience/senior-staff.md`**: Added 15 practice questions with answers targeted at senior/staff level
- **`CONTRIBUTING.md`**: Full rewrite with question template, content standards, PR guidelines, and contribution ideas

### Fixed
- Replaced placeholder `your-username` with `JustInCache` in all badge URLs
- Updated question counts in README topic table to reflect actual content

---

## [2026-05] — Initial Structure

### Added
- `README.md` with Top 50 AI interview questions with full answers
- `topics/` directory with 12 topic deep-dive pages
- `roles/` directory with 5 role-specific interview guides (AI Engineer, ML Engineer, AI Architect, AI Researcher, Data Scientist)
- `by-experience/` directory with 3 experience-level study paths (Junior, Mid-Level, Senior/Staff)
- `resources.md` with curated papers, tools, courses
- `CONTRIBUTING.md` initial version

---

## Roadmap

Planned additions for upcoming months:

- [ ] `companies/microsoft.md` — Azure AI, Copilot, GitHub Copilot, Bing AI
- [ ] `companies/amazon-aws.md` — Bedrock, SageMaker, Alexa, Rekognition
- [ ] `companies/nvidia.md` — CUDA, TensorRT, Triton, NeMo
- [ ] `cheatsheets/fine-tuning.md` — LoRA hyperparameters, data requirements, adapter serving
- [ ] Mock interview scripts — complete 45-minute interview simulations for each role
- [ ] Coding exercises — "implement from scratch" exercises for key concepts
- [ ] Video explanations — for complex topics (attention, transformers, RLHF)

Want to contribute to any of these? See [CONTRIBUTING.md](CONTRIBUTING.md).
