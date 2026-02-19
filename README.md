# Runhuman Agent Skills

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-v1.0.0-blue)](https://agentskills.io)

Agent Skills for Runhuman - procedural knowledge for AI agents working with human-powered QA testing via the Runhuman CLI.

## What are Agent Skills?

[Agent Skills](https://agentskills.io/) are folders of instructions, scripts, and resources that AI agents can discover and use to do things more accurately and efficiently.

## Installation

```bash
npx skills add volter-ai/runhuman-skills
```

## Supported AI Tools

Works with any Agent Skills-compatible tool: Claude Code, Cursor, Codex, Cline, and others.

## What's Included

`SKILL.md` teaches AI agents how to:

- **Quick start** - Install, authenticate, initialize a project, and create a test
- **Core workflow** - Create jobs, wait for results, check status
- **Command reference** - Overview of all 9 command groups with pointers to `--help`
- **Test descriptions** - Write effective descriptions that get useful results
- **CI/CD** - Integrate Runhuman into automated pipelines

## Usage with Claude Code

```bash
# 1. Install skills
npx skills add volter-ai/runhuman-skills

# 2. Start Claude Code
claude

# 3. Prompt
"Test my checkout flow at https://staging.myapp.com using Runhuman"
```

## Links

- **Website:** https://runhuman.com
- **CLI Docs:** `runhuman --help`
- **GitHub Action:** https://github.com/marketplace/actions/runhuman-qa-test
- **Agent Skills Standard:** https://agentskills.io

## License

MIT
