# Runhuman Agent Skills

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-v1.0.0-blue)](https://agentskills.io)

Agent Skills for Runhuman - Best practices and procedural knowledge for AI agents working with human-powered QA testing.

## What are Agent Skills?

[Agent Skills](https://agentskills.io/) are folders of instructions, scripts, and resources that AI agents can discover and use to do things more accurately and efficiently. This package contains Runhuman-specific knowledge in the Agent Skills format.

## Installation

```bash
npx skills add volter-ai/runhuman-skills
```

## Supported AI Tools

These skills work with any Agent Skills-compatible tool:
- **Claude Code** (Anthropic)
- **Cursor**
- **Codex**
- **Cline**
- **Any AI agent supporting the Agent Skills format**

## What's Included

This skills package teaches AI agents how to:

### CLI Usage (Primary Focus)
- Run QA tests using `@runhuman/cli`
- Write effective test descriptions
- Configure timeouts and options
- Integrate with CI/CD pipelines
- Optimize costs and test frequency

### Core Concepts
- When to use human QA vs. automated testing
- Best practices for test descriptions
- Common workflow patterns
- Troubleshooting guide

### Integration Patterns
- Pre-merge PR testing
- Nightly regression testing
- Manual test scripts
- GitHub Actions integration

## Usage with Claude Code

1. **Install the skills:**
   ```bash
   npx skills add volter-ai/runhuman-skills
   ```

2. **Start Claude Code in your project:**
   ```bash
   claude
   ```

3. **Prompt Claude to create tests:**
   ```
   "Create a QA test for my checkout flow using Runhuman CLI"
   ```

Claude will now use the skills to generate correct commands and best practices automatically!

## Example Prompts

Once skills are installed, try these prompts with Claude Code:

### Basic Testing
```
"Test my signup flow at https://staging.myapp.com using Runhuman"
```

### CI/CD Integration
```
"Add a Runhuman test to my GitHub Actions workflow for PR testing"
```

### Complex Workflows
```
"Create a test suite script that tests signup, login, and checkout flows"
```

### Mobile Testing
```
"Test mobile responsiveness of my landing page with Runhuman"
```

## Contents

- **`SKILL.md`** - Main skill file with comprehensive CLI usage guide (460+ lines)
- **`README.md`** - This file

## Repository

https://github.com/volter-ai/runhuman-skills

## License

MIT

## Links

- **Runhuman Website:** https://runhuman.com
- **Dashboard:** https://runhuman.com/dashboard
- **API Documentation:** https://runhuman.com/docs/api
- **Agent Skills Standard:** https://agentskills.io
