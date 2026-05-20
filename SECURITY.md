# Security Policy

## Reporting Security Issues

Please do not open public issues for security concerns.

If you find a vulnerability in repository automation, contributor workflows, or linked assets, report it privately through GitHub security advisories when available.

## Automation Secrets

The AI review workflow expects this repository secret:

- `OPENAI_API_KEY`: used by PR-Agent to review pull requests.

The scheduled heartbeat workflow uses GitHub's built-in `GITHUB_TOKEN` and does not require a personal access token.

