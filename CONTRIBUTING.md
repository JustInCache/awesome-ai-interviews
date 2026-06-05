# Contributing to Awesome AI Interviews

[← Back to Home](README.md)

Thank you for helping make this the most comprehensive AI interview repository on GitHub! Every contribution — a new question, a better answer, a fixed typo — helps thousands of people prepare for their AI interviews.

---

## Types of Contributions

### What We're Looking For

✅ **High-value contributions**:

- New interview questions with full, production-quality answers
- Expanded answers with additional depth, code examples, or trade-off analysis
- New topic pages for missing areas
- Company-specific interview guides (based on publicly known interview patterns)
- Resources: papers, tools, courses worth adding
- Factual corrections with citations
- Fixed broken links

❌ **Please avoid**:

- Questions without answers (or with vague "it depends" answers that explain nothing)
- Duplicate questions already covered in the same file
- Vendor marketing content or self-promotion
- Answers that recommend a specific paid product without acknowledging alternatives
- Specific leaked interview questions from confidential processes
- Questions from company coding rounds (those belong on LeetCode)

---

## How to Contribute

### Quick Fix (typo, broken link, small improvement)

1. Click the ✏️ edit button on any page (GitHub)
2. Make your change
3. Submit a pull request with a description of what you fixed

### New Question or Expanded Answer

1. **Fork** the repo and create a branch: `git checkout -b add-llm-training-question`
2. Find the right file in `topics/`, `roles/`, `by-experience/`, or `companies/`
3. Add your question using the template below
4. Submit a PR with a short description

### New File (new topic, new company, new cheatsheet)

1. Fork and create a branch
2. Create the file in the right directory (see File Structure below)
3. Follow the existing file format (include navigation header, table of contents, difficulty tags)
4. Add a link to the file in `README.md` in the appropriate section
5. Submit a PR

---

## Question Template

Copy this template when adding a new question:

````markdown
**Q: [Your question here]** `[B]` / `[I]` / `[A]`

[Opening sentence: what is the core concept being tested?]

[Main answer: 2-5 paragraphs. Include:]
- Core explanation with correct terminology
- Why it matters / when you'd use this
- Trade-offs and limitations (don't skip these — interviewers love trade-offs)

[Use tables for comparisons:]
| Approach A | Approach B |
|-----------|-----------|
| ...       | ...       |

[Use code blocks for implementations:]
```python
# Include runnable pseudocode or real code snippets
# that demonstrate the concept
```

[Optionally include a "Production note" for real-world context]

📖 Related: [link to deeper page](relative-link.md)

---
````

### Difficulty Tags

- `[B]` **Beginner**: questions a junior engineer should know cold
- `[I]` **Intermediate**: questions for mid-level / 2–4 years experience
- `[A]` **Advanced**: questions for senior / staff engineers; may require deep domain expertise

---

## File Structure

```text
awesome-ai-interviews/
├── README.md                    # Top 50 questions + navigation hub
├── CONTRIBUTING.md              # This file
├── CHANGELOG.md                 # What's new
├── resources.md                 # Curated papers, tools, courses
│
├── topics/                      # Deep-dive topic pages
│   ├── 01-llm-fundamentals.md   # LLM architecture, training, inference
│   ├── 02-prompt-engineering.md # Prompting techniques, patterns
│   ├── 03-rag.md                # RAG pipelines, retrieval, evaluation
│   ├── 04-ai-agents.md          # Agents, multi-agent, tools
│   ├── 05-fine-tuning.md        # LoRA, RLHF, DPO, data prep
│   ├── 06-vector-databases.md   # Embeddings, similarity search, ANN
│   ├── 07-ai-system-design.md   # System design patterns
│   ├── 08-llmops-production.md  # LLMOps, monitoring, deployment
│   ├── 09-evaluation-testing.md # Eval metrics, LLM-as-judge
│   ├── 10-ai-safety-ethics.md   # Safety, alignment, ethics, regulation
│   ├── 11-multimodal-ai.md      # Vision-language, audio, multimodal
│   ├── 12-ai-infrastructure.md  # Serving, scalability, hardware
│   └── 13-reasoning-models.md   # Test-time compute, o1/o3/R1, RLVR
│
├── roles/                       # Role-specific scenario questions
│   ├── ai-engineer.md
│   ├── ml-engineer.md
│   ├── ai-architect.md
│   ├── ai-researcher.md
│   └── data-scientist.md
│
├── by-experience/               # Study paths + level-specific questions
│   ├── junior.md
│   ├── mid-level.md
│   └── senior-staff.md
│
├── companies/                   # Company-specific interview styles
│   ├── openai.md
│   ├── google-deepmind.md
│   ├── anthropic.md
│   └── meta-ai.md
│
└── cheatsheets/                 # Quick-reference one-pagers
    ├── llm-formulas.md
    ├── rag-checklist.md
    └── system-design-patterns.md
```

---

## Content Standards

### Answer Quality Bar

Every answer should pass this checklist before being submitted:

- [ ] **Correct**: technically accurate as of 2025–2026
- [ ] **Complete**: covers the concept, why it matters, and key trade-offs
- [ ] **Concise**: no fluff; every sentence adds information
- [ ] **Cited**: technical claims reference papers or real systems where applicable
- [ ] **Code**: includes a code snippet if the concept has a natural implementation
- [ ] **Interview-ready**: reads as a response a candidate would give in an interview, not a textbook

### Style Guidelines

- Use `**bold**` for key terms on first use
- Use `code` formatting for technical terms, model names, library names
- Tables for comparisons (avoid prose lists when a table is clearer)
- Include a `📖 Deep dive:` reference line pointing to the detailed topic page

### Accuracy Standard

- If you're not sure about a fact, either verify it or don't include it
- Prefer concrete specifics over vague generalizations: "~2× speedup" > "significant speedup"
- Include the year for papers and for time-sensitive claims (e.g., "as of 2026")
- If a claim was true 2 years ago but may have changed, note this

---

## Pull Request Guidelines

### PR Title Format

```text
[type]: short description

Types:
  add      - new question, new file, new section
  expand   - expanding an existing answer or section
  fix      - factual correction, broken link, typo
  update   - updating information that has changed
  chore    - formatting, file moves, non-content changes
```

Examples:

- `add: reasoning models topic page`
- `expand: LoRA answer with QLoRA details and code example`
- `fix: broken paper link in resources.md`
- `update: flash attention 3 details in llm-fundamentals`

### PR Description Template

```markdown
## What This PR Adds/Changes

[2-3 sentences describing the change]

## Why

[Why is this addition/change valuable? What gap does it fill?]

## Checklist

- [ ] Content is technically accurate
- [ ] Follows the question template format
- [ ] Difficulty tag is appropriate
- [ ] Added navigation links if creating a new file
- [ ] Updated README.md if adding a new file/section
```

---

## Reviewing Contributions

PRs are reviewed for:

1. **Accuracy** — is this technically correct?
2. **Quality** — does it meet the answer quality bar?
3. **Duplication** — is this already covered?
4. **Format** — does it follow the template?

Expect feedback within 1–2 weeks. Reviews are about improving content, not gatekeeping — if your answer is directionally correct, we'll work with you to refine it.

---

## Ideas for Contributions

Looking for inspiration? Here are areas that could use more depth:

**Topic expansions**:

- `06-vector-databases.md`: more on HNSW internals, product quantization
- `11-multimodal-ai.md`: audio models (Whisper, voice AI), video understanding
- `05-fine-tuning.md`: data synthesis techniques, DPO implementation details
- `09-evaluation-testing.md`: benchmarking LLMs, constructing golden datasets

**New company pages** (high demand):

- `companies/microsoft.md` (Azure AI, Copilot, GitHub)
- `companies/amazon-aws.md` (Bedrock, Alexa AI)
- `companies/nvidia.md` (CUDA, TensorRT, Triton)
- `companies/mistral.md`

**New cheatsheets**:

- `cheatsheets/fine-tuning.md` — LoRA hyperparameters, data requirements, evaluation
- `cheatsheets/agent-patterns.md` — ReAct, Plan-Execute, multi-agent topologies

**Coding questions**:
Any topic page that could benefit from "implement this from scratch" exercises:

- Implement scaled dot-product attention
- Implement BPE tokenization
- Build a simple RAG pipeline in 50 lines
- Implement beam search

---

## Code of Conduct

Be kind. Be constructive. Assume good intent. This community is for helping people get jobs in AI — there's no gatekeeping here.

If you see content that is incorrect, outdated, or of low quality, open an issue rather than just complaining — tell us what's wrong and ideally how to fix it.

---

Thank you for contributing. Every improvement you make helps someone land their AI interview. 🎯
