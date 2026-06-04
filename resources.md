# Resources

[← Back to Home](../README.md)

A curated collection of papers, tools, courses, and communities for AI interview preparation and ongoing learning.

---

## Foundational Papers

### Must-Read (Every AI Engineer)

| Paper | Year | Why It Matters |
|-------|------|----------------|
| [Attention Is All You Need](https://arxiv.org/abs/1706.03762) | 2017 | Original Transformer — understand self-attention |
| [BERT: Pre-training of Deep Bidirectional Transformers](https://arxiv.org/abs/1810.04805) | 2018 | Bidirectional pre-training, encoder architecture |
| [Language Models are Few-Shot Learners (GPT-3)](https://arxiv.org/abs/2005.14165) | 2020 | In-context learning at scale |
| [Training language models to follow instructions (InstructGPT)](https://arxiv.org/abs/2203.02155) | 2022 | RLHF, how ChatGPT-style alignment works |
| [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685) | 2021 | Efficient fine-tuning — know this cold |
| [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) | 2020 | Original RAG paper |
| [Flash Attention](https://arxiv.org/abs/2205.14135) | 2022 | IO-aware exact attention — efficient long context |
| [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073) | 2022 | Safety alignment via self-critique |
| [Chinchilla: Training Compute-Optimal Large Language Models](https://arxiv.org/abs/2203.15556) | 2022 | Scaling laws: optimal token-to-parameter ratio |

### Fine-Tuning & Alignment

| Paper | Year | Why It Matters |
|-------|------|----------------|
| [QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314) | 2023 | 4-bit fine-tuning on consumer GPUs |
| [Direct Preference Optimization (DPO)](https://arxiv.org/abs/2305.18290) | 2023 | Alignment without reward model |
| [Self-Instruct](https://arxiv.org/abs/2212.10560) | 2022 | Synthetic instruction data generation |
| [RLHF: Learning to Summarize with Human Feedback](https://arxiv.org/abs/2009.01325) | 2020 | Foundation of RLHF |

### Agents & Reasoning

| Paper | Year | Why It Matters |
|-------|------|----------------|
| [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) | 2022 | ReAct agent pattern |
| [Chain-of-Thought Prompting Elicits Reasoning](https://arxiv.org/abs/2201.11903) | 2022 | CoT prompting |
| [Tree of Thoughts: Deliberate Problem Solving with LLMs](https://arxiv.org/abs/2305.10601) | 2023 | ToT reasoning |
| [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366) | 2023 | Self-improving agents |

### Infrastructure

| Paper | Year | Why It Matters |
|-------|------|----------------|
| [Efficient Memory Management for LLM Serving (vLLM / PagedAttention)](https://arxiv.org/abs/2309.06180) | 2023 | How vLLM achieves high throughput |
| [FlashAttention-2: Faster Attention with Better Parallelism](https://arxiv.org/abs/2307.08691) | 2023 | Improved FA with better GPU utilization |
| [ZeRO: Memory Optimizations Toward Training Trillion Parameter Models](https://arxiv.org/abs/1910.02054) | 2019 | DeepSpeed ZeRO memory optimization |

---

## Tools & Frameworks

### LLM Frameworks

| Tool | Use Case | Docs |
|------|----------|------|
| [LangChain](https://python.langchain.com/) | RAG, agents, chains | python.langchain.com |
| [LlamaIndex](https://docs.llamaindex.ai/) | RAG-first, data connectors | docs.llamaindex.ai |
| [LangGraph](https://langchain-ai.github.io/langgraph/) | Stateful multi-agent workflows | langchain-ai.github.io/langgraph |
| [DSPy](https://dspy-docs.vercel.app/) | Programmatic prompt optimization | dspy-docs.vercel.app |
| [Instructor](https://python.useinstructor.com/) | Structured LLM output | python.useinstructor.com |
| [Outlines](https://outlines-dev.github.io/outlines/) | Constrained LLM generation | outlines-dev.github.io |

### LLM Serving

| Tool | Use Case | Notes |
|------|----------|-------|
| [vLLM](https://docs.vllm.ai/) | High-throughput inference, PagedAttention | De facto standard for OSS serving |
| [Text Generation Inference (TGI)](https://huggingface.co/docs/text-generation-inference) | HuggingFace-native serving | Continuous batching, quantization |
| [SGLang](https://github.com/sgl-project/sglang) | Structured generation, RadixAttention | Fast for structured outputs |
| [Ollama](https://ollama.com/) | Local model serving | MacOS/Linux, easy setup |
| [llama.cpp](https://github.com/ggerganov/llama.cpp) | CPU/GPU inference for quantized models | GGUF format |

### Evaluation

| Tool | Use Case | Notes |
|------|----------|-------|
| [RAGAS](https://docs.ragas.io/) | RAG evaluation | Faithfulness, relevancy, precision, recall |
| [DeepEval](https://docs.confident-ai.com/) | LLM unit testing framework | G-Eval, hallucination, toxicity metrics |
| [LangSmith](https://docs.smith.langchain.com/) | LLM tracing + evaluation | From LangChain team |
| [Arize Phoenix](https://phoenix.arize.com/) | Observability + evaluation | Open-source LLM monitoring |
| [PromptFoo](https://www.promptfoo.dev/) | Prompt testing + red teaming | YAML-based test suites |
| [Evals (OpenAI)](https://github.com/openai/evals) | OpenAI eval framework | Community-contributed eval sets |

### Observability / LLMOps

| Tool | Use Case | Notes |
|------|----------|-------|
| [Langfuse](https://langfuse.com/) | Tracing, evaluation, prompt management | Open-source, self-hostable |
| [Helicone](https://www.helicone.ai/) | Proxy-based LLM observability | 1-line integration |
| [LiteLLM](https://litellm.ai/) | Unified LLM API proxy, routing, caching | 100+ provider support |
| [PromptLayer](https://promptlayer.com/) | Prompt versioning + analytics | SaaS |

### Vector Databases

| Tool | Best For | Notes |
|------|----------|-------|
| [Qdrant](https://qdrant.tech/) | Production, filtering, multi-tenant | Rust-based, fast |
| [Weaviate](https://weaviate.io/) | Hybrid search, GraphQL | Schema-based |
| [Chroma](https://www.trychroma.com/) | Development, prototyping | Simple Python API |
| [Pinecone](https://www.pinecone.io/) | Managed, serverless | Easy setup, scalable |
| [pgvector](https://github.com/pgvector/pgvector) | PostgreSQL-native | Good for existing Postgres stacks |
| [Milvus](https://milvus.io/) | Enterprise, on-premise | IVF, HNSW, billion-scale |

### Fine-Tuning

| Tool | Use Case | Notes |
|------|----------|-------|
| [Unsloth](https://github.com/unslothai/unsloth) | Fast LoRA/QLoRA fine-tuning | 2× faster than HuggingFace PEFT |
| [TRL (HuggingFace)](https://huggingface.co/docs/trl) | RLHF, DPO, SFT | Official HF reinforcement learning |
| [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl) | Config-based fine-tuning | YAML configs, many model support |
| [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) | Web UI + code fine-tuning | Supports 100+ models |

---

## Courses & Learning Paths

### Free Courses

| Course | Provider | Topics |
|--------|----------|--------|
| [DeepLearning.AI Short Courses](https://www.deeplearning.ai/short-courses/) | deeplearning.ai | RAG, agents, fine-tuning, LLMOps |
| [fast.ai Practical Deep Learning](https://course.fast.ai/) | fast.ai | Deep learning from scratch |
| [HuggingFace NLP Course](https://huggingface.co/learn/nlp-course) | HuggingFace | Transformers, tokenization, fine-tuning |
| [LangChain Academy](https://academy.langchain.com/) | LangChain | LangGraph, agents, RAG |
| [Stanford CS224N: NLP with Deep Learning](https://web.stanford.edu/class/cs224n/) | Stanford (free lectures) | NLP foundations |
| [Andrej Karpathy: Let's build GPT](https://www.youtube.com/watch?v=kCc8FmEb1nY) | YouTube | Build GPT from scratch |

### Paid Courses Worth It

| Course | Platform | Why Worth It |
|--------|----------|--------------|
| [LLM Engineering: Master AI & LLMs](https://www.udemy.com/course/llm-engineering-master-ai-and-large-language-models/) | Udemy | End-to-end practical LLM engineering |
| [MLOps Specialization](https://www.coursera.org/specializations/machine-learning-engineering-for-production-mlops) | Coursera / deeplearning.ai | Production ML lifecycle |

---

## Books

| Book | Author | Best For |
|------|--------|----------|
| *Designing Machine Learning Systems* | Chip Huyen | Production ML, ML system design |
| *Building LLMs for Production* | Louis-François Bouchard et al. | LLMOps, RAG, evaluation |
| *Natural Language Processing with Transformers* | Lewis Tunstall et al. | Transformers, HuggingFace, fine-tuning |
| *Hands-On Large Language Models* | Jay Alammar & Maarten Grootendorst | Visual explanations, practical |
| *The Deep Learning Book* | Goodfellow, Bengio, Courville | Theoretical foundations |
| *Probabilistic Machine Learning* | Kevin Murphy | Advanced ML theory |

---

## Newsletters & Communities

### Newsletters (Subscribe)

| Newsletter | Focus | Frequency |
|------------|-------|-----------|
| [The Batch (DeepLearning.AI)](https://www.deeplearning.ai/the-batch/) | AI news + research | Weekly |
| [Import AI (Jack Clark)](https://importai.substack.com/) | Research frontier | Weekly |
| [Ahead of AI (Sebastian Raschka)](https://magazine.sebastianraschka.com/) | Deep paper dives | Bi-weekly |
| [The Gradient](https://thegradient.pub/) | Research + industry analysis | Variable |
| [Last Week in AI](https://lastweekin.ai/) | AI news roundup | Weekly |
| [AI Snake Oil](https://www.aisnakeoil.com/) | Critical AI analysis | Variable |

### Communities

| Community | Platform | Best For |
|-----------|----------|----------|
| Hugging Face Forums | [discuss.huggingface.co](https://discuss.huggingface.co/) | Models, datasets, fine-tuning |
| LangChain Discord | Discord | LangChain, agents, RAG |
| r/MachineLearning | Reddit | Research discussion |
| r/LocalLLaMA | Reddit | Open-source models, local inference |
| MLOps Community | Slack | Production ML, LLMOps |
| AI Engineer Discord | Discord | AI Engineering practice |

---

## Datasets for Practice

| Dataset | Use Case |
|---------|----------|
| [MMLU](https://huggingface.co/datasets/cais/mmlu) | LLM benchmarking across 57 subjects |
| [TruthfulQA](https://huggingface.co/datasets/truthful_qa) | Hallucination evaluation |
| [HotpotQA](https://huggingface.co/datasets/hotpot_qa) | Multi-hop reasoning |
| [MS MARCO](https://microsoft.github.io/msmarco/) | Passage retrieval evaluation |
| [Natural Questions](https://huggingface.co/datasets/natural_questions) | Open-domain Q&A |
| [Alpaca](https://huggingface.co/datasets/tatsu-lab/alpaca) | Instruction following (52K examples) |
| [ShareGPT](https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered) | Multi-turn conversation fine-tuning |

---

## Key Benchmarks to Know

| Benchmark | What It Measures |
|-----------|-----------------|
| MMLU | Multitask language understanding (57 subjects) |
| HumanEval | Code generation (Python function completion) |
| MATH | Mathematical reasoning |
| GSM8K | Grade school math word problems |
| HellaSwag | Commonsense NLI |
| TruthfulQA | Truthfulness / hallucination |
| MT-Bench | Multi-turn instruction following |
| LMSYS Chatbot Arena | Human preference (ELO-style) |

---

[← Back to Home](../README.md)
