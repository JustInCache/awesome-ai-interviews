# Awesome AI Interviews: Repository Organization Plan

## Goal

Build the repository into the most useful open interview-preparation hub for AI, ML, LLM, GenAI, RAG, agents, AI systems, and AI-adjacent roles across experience levels.

The repo should optimize for:

- Fast discovery from GitHub search and Google
- Clear paths by role, topic, and seniority
- High-quality canonical answers instead of repeated lists
- Contribution-friendly structure
- Proper attribution for imported content

## Current Content Audit

| Source directory | Current shape | Strength | Issue to fix |
|---|---:|---|---|
| `LLM-Interview-Questions-and-Answers-Hub/` | 39 grouped QA files, 100+ short Q&A | Strong LLM fundamentals and inference/fine-tuning basics | Questions are grouped in `QA_1-3.md` style, hard to browse by topic |
| `llms-interview-questions/` | One very large README with 63 long-form questions | Deep concept explanations and code examples | Single-file format is difficult to maintain and verify |
| `ai-engineering-interview-questions/` | One curated README organized by topic | Excellent topic taxonomy for AI engineering | Many entries are only question/link references, not canonical local answers |
| `interview_questions/` | 5 roles x 8 scenario questions | Best role-based, production-style interview answers | Needs expansion across seniority and cross-linking to concepts |

## Recommended Repository Structure

```text
awesome-ai-interviews/
├── README.md
├── LICENSE
├── NOTICE.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── docs/
│   ├── REPO_ORGANIZATION_PLAN.md
│   ├── content-style-guide.md
│   ├── question-template.md
│   └── attribution.md
├── questions/
│   ├── by-topic/
│   │   ├── llm-fundamentals/
│   │   ├── prompt-engineering/
│   │   ├── rag/
│   │   ├── agents/
│   │   ├── fine-tuning/
│   │   ├── vector-databases-embeddings/
│   │   ├── evaluation-observability/
│   │   ├── llmops-production/
│   │   ├── ai-system-design/
│   │   ├── safety-security/
│   │   ├── multimodal-ai/
│   │   ├── ml-fundamentals/
│   │   ├── mlops/
│   │   ├── data-science-analytics/
│   │   └── coding-practical/
│   ├── by-role/
│   │   ├── ai-engineer/
│   │   ├── llm-engineer/
│   │   ├── genai-engineer/
│   │   ├── ai-agent-engineer/
│   │   ├── ml-engineer/
│   │   ├── mlops-engineer/
│   │   ├── data-scientist/
│   │   ├── ai-researcher/
│   │   ├── ai-architect/
│   │   └── ai-product-manager/
│   └── by-level/
│       ├── beginner/
│       ├── intermediate/
│       ├── senior/
│       └── staff-principal/
├── guides/
│   ├── roadmaps/
│   ├── study-plans/
│   ├── cheatsheets/
│   ├── system-design/
│   └── mock-interviews/
├── practice/
│   ├── coding/
│   ├── case-studies/
│   ├── take-home-assignments/
│   └── answer-rubrics/
├── assets/
└── archive/
    └── imported-sources/
```

## Content Model

Every canonical question should live once, under `questions/by-topic/...`.

Role and level pages should be curated indexes that link to canonical questions rather than duplicating full answers. This keeps the repo maintainable while still giving candidates multiple navigation paths.

Recommended question filename:

```text
questions/by-topic/llm-fundamentals/001-transformer-positional-embeddings.md
```

Recommended question page format:

```markdown
# Why do transformers need positional embeddings?

**Topic:** LLM Fundamentals
**Roles:** AI Engineer, ML Engineer, LLM Engineer
**Level:** Beginner
**Question type:** Conceptual
**Last reviewed:** YYYY-MM-DD

## Short Answer

2-4 sentences for quick revision.

## Deep Answer

The full explanation with trade-offs, examples, diagrams, or code where useful.

## Interview Signals

What a strong candidate should mention.

## Common Pitfalls

Mistakes, outdated claims, and shallow answers.

## Follow-up Questions

Related questions an interviewer may ask next.

## Related Questions

Links to related local pages.
```

## Homepage Strategy

The root `README.md` should not be a giant dump. It should be a conversion page for GitHub users.

Recommended sections:

1. One-line promise: "The complete open-source AI interview preparation guide by role, topic, and seniority."
2. Badges: license, contributions welcome, questions count, topics count.
3. Quick navigation: Role, topic, level, system design, mock interviews.
4. "Start here" paths:
   - AI Engineer
   - LLM/GenAI Engineer
   - ML Engineer
   - Data Scientist
   - AI Researcher
   - AI Architect
5. Featured collections:
   - Top 100 LLM questions
   - RAG interview guide
   - AI agents interview guide
   - Production AI system design
6. Contribution call-to-action.
7. Attribution and license note.

## Migration Plan

### Phase 1: Foundation

- Create root `README.md`, `LICENSE`, `NOTICE.md`, `CONTRIBUTING.md`, and issue templates.
- Move imported source directories into `archive/imported-sources/` until content is normalized.
- Add attribution for imported Apache-2.0 sources.
- Create the topic and role folder skeleton.

### Phase 2: Normalize Existing Content

- Split `LLM-Interview-Questions-and-Answers-Hub/Interview_QA/QA_*.md` into individual canonical question files.
- Split `llms-interview-questions/README.md` into individual long-form concept pages.
- Convert `ai-engineering-interview-questions/README.md` into topic indexes and fill local canonical answers over time.
- Move `interview_questions/*` role scenarios into `questions/by-role/...` or convert them into role-specific guides linking to canonical topics.

### Phase 3: Build the Moat

- Add answer rubrics: what junior, mid-level, senior, and staff-level answers sound like.
- Add system design questions for RAG, agents, evaluation, memory, observability, cost, latency, and security.
- Add "production failure" scenarios: hallucination, prompt injection, slow retrieval, stale documents, poor evaluation, runaway costs.
- Add coding/practical exercises with expected solutions.
- Add mock interview packs by role and experience level.

### Phase 4: GitHub Growth

- Add a strong visual banner and social preview image.
- Add `good first issue` tasks for missing answers, examples, diagrams, and reviews.
- Add a question contribution template.
- Add weekly update cadence: "10 new questions every week."
- Add GitHub Topics: `ai-interview`, `llm`, `rag`, `genai`, `machine-learning`, `mlops`, `ai-agents`, `interview-preparation`, `system-design`.

## Recommended Navigation Files

Create these index files early:

- `questions/by-topic/README.md`
- `questions/by-role/README.md`
- `questions/by-level/README.md`
- `guides/roadmaps/README.md`
- `guides/system-design/README.md`
- `practice/README.md`

## Quality Bar

To become star-worthy, every answer should be:

- Interview-ready: concise enough to say out loud
- Deep enough for follow-up questions
- Current: avoid outdated model/version claims unless dated
- Practical: include trade-offs, failure modes, and production considerations
- Cross-linked: related role, topic, and level paths
- Attributed: source and license preserved when derived from imported content

## Priority Topic Taxonomy

Use this order for the first public launch:

1. LLM fundamentals
2. Prompt engineering
3. RAG
4. AI agents
5. Fine-tuning and adaptation
6. Embeddings and vector databases
7. Evaluation and observability
8. LLMOps and production AI
9. AI system design
10. AI safety and security
11. Multimodal AI
12. ML engineering
13. Data science and experimentation
14. AI research
15. Coding and practical implementation

## Licensing And Attribution Notes

The imported `LLM-Interview-Questions-and-Answers-Hub` and `ai-engineering-interview-questions` directories include Apache-2.0 licenses. Preserve their license text and original attribution when incorporating or modifying that material.

Before public launch:

- Add a root `NOTICE.md`
- Add `docs/attribution.md`
- Keep imported raw sources in `archive/imported-sources/`
- Mark normalized/rewritten pages as adapted where appropriate

## Suggested First Milestone

Launch v0.1 with:

- Root README
- Topic index
- Role index
- 100 normalized LLM fundamentals questions
- 40 role-based scenario questions
- 5 study paths
- Contribution guide
- Attribution docs

That is enough to feel substantial on day one while leaving clear room for community contribution.
