---
name: runhuman-testing
version: 2.2.0
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

# 3. Set up a project
#    List existing projects:
runhuman projects list
#    Or create a new one:
runhuman projects create "My App" --organization <orgId> --default-url https://staging.myapp.com --set-default
#    Switch to an existing project:
runhuman projects switch <projectId>

# 4. Create a test
runhuman create https://staging.myapp.com \
  -d "Test the signup flow: click Sign Up, fill the form, verify confirmation page"
```

**Deciding between new vs existing project:** Ask the user whether they want to create a new project or use an existing one. Use `runhuman projects list` to show what exists, then either `projects create` with `--set-default` or `projects switch` to set the active project. `projects switch` sets the default globally (in `~/.config/runhuman/config.json`) by default; pass `--global false` to set it locally for the current directory only.

**Setting defaults:** Use `runhuman config set <key> <value>` to set defaults like `project`, `color`, or `apiUrl`. Use `runhuman config list` to see the full configuration hierarchy.

### Truncated IDs

All commands accept truncated ID prefixes, similar to git short hashes. The CLI resolves the best match automatically:

```bash
runhuman status 712e           # resolves to full job ID
runhuman projects switch proj  # resolves to matching project
```

If multiple IDs match, the CLI asks for more characters. Destructive operations (`delete`, `transfer`) require full IDs.

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

## Creating Tests

The `create` command is the primary way to submit tests. At minimum, provide a URL and either a description or a template.

### With a Description

```bash
runhuman create https://staging.myapp.com \
  -d "Test the signup flow: click Sign Up, fill the form, verify confirmation page"
```

### With an Output Schema

Use `--schema` or `--schema-inline` to get structured JSON results back:

```bash
# Schema from a file
runhuman create https://staging.myapp.com \
  -d "Test the search feature" \
  --schema ./search-schema.json \
  --sync --json

# Inline schema
runhuman create https://staging.myapp.com \
  -d "Test signup and login" \
  --schema-inline '{"signupWorks":{"type":"boolean"},"loginWorks":{"type":"boolean"},"issues":{"type":"array","items":{"type":"string"}}}' \
  --sync --json
```

Example schema file (`search-schema.json`):
```json
{
  "searchWorks": {
    "type": "boolean",
    "description": "Does the search return results?"
  },
  "resultCount": {
    "type": "number",
    "description": "Number of results shown"
  },
  "issues": {
    "type": "array",
    "description": "List of any bugs or issues found"
  }
}
```

When a job completes with a schema, `runhuman results <jobId> --json` returns the extracted data in `result.data`:

```json
{
  "jobId": "job_abc123",
  "status": "completed",
  "result": {
    "passed": true,
    "explanation": "All tests passed successfully",
    "data": {
      "searchWorks": true,
      "resultCount": 10,
      "issues": []
    }
  },
  "costUsd": 0.18,
  "testDurationSeconds": 120
}
```

Use `--schema-only` to get just the extracted schema data: `runhuman results <jobId> --schema-only`.

### With a Template

Templates are reusable test configurations. Use `--template` to reference one by name or `--template-file` to point to a local `.md` file:

```bash
# By name (resolved from repo .runhuman/templates/ → project templates → built-ins)
runhuman create https://staging.myapp.com --template "Find Bugs"

# By local file path
runhuman create --template-file .runhuman/templates/smoke-test.md
```

When using a template, URL and description are optional — they can come from the template itself.

### Device Class

Specify `--device-class desktop` or `--device-class mobile` to control what device the tester uses:

```bash
runhuman create https://staging.myapp.com -d "Test mobile layout" --device-class mobile
```

### Async Workflow (Create, Wait, Get Results)

If you don't use `--sync`, create the job first, then wait for it separately:

```bash
# 1. Create the job (returns immediately)
JOB_ID=$(runhuman create https://staging.myapp.com -d "Test search" --json | jq -r '.jobId')

# 2. Wait for the tester to complete (blocks with live status updates)
runhuman wait "$JOB_ID"

# 3. Get the results
runhuman results "$JOB_ID" --json
```

`wait` accepts `--timeout <seconds>` (default: 600). Use `--json` on any command when you need structured output for scripting or automation.

## Templates

Templates define what to test, how long to test, and what results to collect. There are three sources, checked in this order:

| Source | Location | How to create |
|--------|----------|---------------|
| **Repo templates** | `.runhuman/templates/*.md` in the user's repo | Commit markdown files |
| **Project templates** | Stored in Runhuman database | `templates create` or dashboard |
| **Built-in templates** | Bundled with Runhuman | Available automatically |

### Built-in Templates

Three built-in templates cover common QA scenarios:

| Template | What it does |
|----------|-------------|
| **Find Bugs** | Test the app and report bugs with severity, steps to reproduce, expected behavior |
| **Assess UX** | Evaluate usability, visual design, mobile-friendliness, and rate 1-10 |
| **Give Product Feedback** | Provide honest feedback: first impressions, strengths, weaknesses, suggestions |

### Repo Templates

Repo templates are `.md` files in `.runhuman/templates/` with YAML frontmatter:

```markdown
---
name: Smoke Test
duration: 5
device_class: desktop
url: https://staging.example.com
---

Verify the app loads and core features work:

1. Load the homepage and confirm it renders without errors
2. Navigate to the login page and verify the form appears
3. Check that the main navigation links work

## Results

Page loads without errors: [ ]
Login form is accessible: [ ]
Navigation works: [ ]
Issues found: ___
```

Frontmatter fields (all optional): `name`, `duration` (minutes, 1-60), `device_class` (`desktop`/`mobile`), `url`, `max_extension_minutes`, `extension_count`, `github_repos` (YAML list of `owner/repo` strings).

### Creating Project Templates via CLI

```bash
runhuman templates create "Search Test" \
  --project proj_abc123 \
  -d "Search for 'recursion' and confirm the joke appears" \
  --schema ./search-test-schema.json
```

Options: `--duration <minutes>`, `--device-class <class>`, `--schema <path>`.

List templates: `runhuman templates list --project <projectId>`

## Command Groups

Run `runhuman <command> --help` for full usage details on any command.

| Group | Commands | Purpose |
|-------|----------|---------|
| **Jobs** | `create`, `status`, `wait`, `results`, `list`, `delete`, `watch` | Create and manage QA test jobs |
| **Auth** | `login`, `logout`, `whoami` | Authentication |
| **Projects** | `projects list`, `create`, `show`, `switch`, `update`, `delete` | Manage projects |
| **Organizations** | `orgs list`, `show`, `balance`, `projects`, `switch` | Manage organizations |
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

> **When users ask about CI/CD, GitHub Actions, or automating QA in their pipeline:** Fetch the comprehensive agent guide at `https://runhuman.com/for_agents_github_actions.md` and follow its instructions to walk the user through setup. The guide covers PR-triggered testing, scheduled exploratory QA, on-demand testing, label management, and platform-specific examples (Vercel, Netlify, Fly.io, Railway).

> **Determining the latest action version:** Before writing any workflow file, check the latest release of the GitHub Action by running `gh release list --repo volter-ai/runhuman-action --limit 1` or checking `https://github.com/volter-ai/runhuman-action/releases`. Use the latest version tag rather than hardcoding an outdated version.

**Using the GitHub Action (recommended for CI/CD):**

```yaml
# .github/workflows/qa.yml
- name: Run QA tests
  uses: volter-ai/runhuman-action@v1  # Check latest version first
  with:
    url: ${{ env.PREVIEW_URL }}
    pr-numbers: '[${{ github.event.pull_request.number }}]'
    api-key: ${{ secrets.RUNHUMAN_API_KEY }}
    on-success-add-labels: '["qa:passed"]'
    on-failure-add-labels: '["qa:failed"]'
    fail-on-failure: true
```

**Using the CLI in a workflow:**

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
- **GitHub Actions Setup Guide (for agents):** https://runhuman.com/for_agents_github_actions.md - Fetch this when users ask about GitHub Actions, CI/CD pipelines, or automating QA.
