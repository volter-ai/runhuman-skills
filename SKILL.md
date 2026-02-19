---
name: runhuman-testing
description: Create and manage human-powered QA tests using Runhuman CLI. Use this skill when you need to test web applications with real human testers, get UX feedback, validate user flows, check mobile responsiveness, or find bugs that automated tests miss.
---

# Runhuman CLI

Runhuman connects AI coding tools to on-demand professional human testers. You describe what to test in natural language, a real human performs the test, and you get structured feedback back.

## When to Use Runhuman

**Good for:** User flows (signup, checkout), visual/layout testing, mobile responsiveness, UX feedback, exploratory testing, cross-browser checks.

**Not for:** Unit tests, performance benchmarks, security audits, or high-frequency automated test suites.

## Quick Start

```bash
# 1. Install (or use npx runhuman for any command without installing)
npm install -g runhuman

# 2. Authenticate (opens browser for GitHub OAuth)
runhuman login

# 3. Initialize project (creates .runhumanrc in current directory)
runhuman init

# 4. Create a test
runhuman create https://staging.myapp.com \
  -d "Test the signup flow: click Sign Up, fill the form, verify confirmation page"
```

The `init` command walks you through selecting an organization, creating or choosing a project, and setting a default URL. After init, `runhuman create` uses those defaults automatically.

### Organizations and Projects

Runhuman is organized around **organizations** (billing/team boundary) and **projects** (group of tests). Most commands accept `--organization <id>` or `--project <id>` flags.

```bash
# List your organizations and get an org ID
runhuman orgs list

# List projects in an organization
runhuman orgs projects <organizationId>

# Pass IDs to other commands
runhuman projects create "My App" --organization <organizationId>
runhuman create https://myapp.com -d "Test signup" --project <projectId>
```

After `runhuman init`, the default org and project are saved to `.runhumanrc` so you don't need to pass them every time.

### Wait for Results

```bash
# Block until the test completes (useful in CI/CD)
runhuman create https://staging.myapp.com \
  -d "Test checkout flow end-to-end" \
  --sync

# Check status of an existing job
runhuman status <jobId>

# Wait for a previously-created job to finish
runhuman wait <jobId>

# View detailed results
runhuman results <jobId>
```

## Command Groups

The CLI is organized into these command groups. Run `runhuman <command> --help` for full usage details on any command.

| Group | Commands | Purpose |
|-------|----------|---------|
| **Jobs** | `create`, `status`, `wait`, `results`, `list`, `delete`, `watch` | Create and manage QA test jobs |
| **Auth** | `login`, `logout`, `whoami` | Authentication |
| **Projects** | `projects list`, `create`, `show`, `switch`, `update`, `delete` | Manage projects |
| **Organizations** | `orgs list`, `show`, `balance`, `projects` | Manage organizations |
| **Templates** | `templates list`, `create`, `show`, `update`, `delete` | Reusable test configurations |
| **API Keys** | `keys list`, `create`, `show`, `delete` | Manage API keys |
| **GitHub** | `github link`, `repos`, `issues`, `test`, `bulk-test` | GitHub integration |
| **Config** | `config get`, `set`, `list`, `reset` | CLI configuration |

## Writing Effective Test Descriptions

The `-d` / `--description` flag is the most important part of a test. Be specific.

**Good:**
```
Test the checkout process:
1. Add a product to cart
2. Click "Checkout"
3. Fill shipping address
4. Complete purchase
Report: bugs, UX confusions, mobile layout issues
```

**Bad:**
```
Test the site
```

Tips:
- Include exact steps or flows to follow
- Specify what pages and buttons to interact with
- State what to look for (bugs, UX issues, visual problems)
- For mobile testing, mention that explicitly in the description

## CI/CD Integration

```bash
# Environment variable auth (for CI)
export RUNHUMAN_API_KEY=rh_live_...

# Synchronous test that blocks until complete
runhuman create https://staging.myapp.com \
  -d "Test signup flow" \
  --sync \
  --wait 300
```

All commands support `--json` for machine-readable output.

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Authentication error |
| `3` | Not found |
| `4` | Validation error |
| `5` | Timeout |
| `6` | Insufficient balance (add funds at the link in the error output) |

## Learning More

```bash
# Top-level help
runhuman --help

# Help for any command
runhuman create --help
runhuman projects --help
runhuman github test --help
```

## Resources

- **Website:** https://runhuman.com
- **Dashboard:** https://runhuman.com/dashboard
- **GitHub Action:** https://github.com/marketplace/actions/runhuman-qa-test
