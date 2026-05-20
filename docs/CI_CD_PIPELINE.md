# CI/CD Pipeline

This repository uses GitHub Actions to keep documentation quality high as the interview question bank grows.

## Workflows

| Workflow | File | Purpose |
|---|---|---|
| Documentation CI | `.github/workflows/docs-ci.yml` | Runs Markdown lint, spell check, link check, and repository health checks |
| AI Pull Request Review | `.github/workflows/ai-review.yml` | Uses PR-Agent to review pull requests for content quality and documentation risks |
| Scheduled Repository Heartbeat | `.github/workflows/auto-commit.yml` | Updates `.github/auto-commit/heartbeat.md` every 12 hours |
| Dependabot | `.github/dependabot.yml` | Opens weekly pull requests for GitHub Actions updates |

## Required Secrets

| Secret | Used by | Purpose |
|---|---|---|
| `OPENAI_API_KEY` | AI Pull Request Review | Allows PR-Agent to run AI reviews |

## Branch Protection Recommendation

Before launch, protect the default branch and require:

- Documentation CI
- At least one approving review
- Conversation resolution before merge
- Branch up to date before merge

## Notes

The first CI version excludes raw imported source directories from linting and spell checking. As content is normalized into `questions/`, `guides/`, and `practice/`, it will automatically enter the quality gates.

