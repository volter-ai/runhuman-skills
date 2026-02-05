---
name: runhuman-testing
description: Create and manage human-powered QA tests using Runhuman CLI. Use this skill when you need to test web applications with real human testers, get UX feedback, validate user flows, check mobile responsiveness, or find bugs that automated tests miss. Covers CLI commands, writing effective test descriptions, common workflows (pre-deployment, CI/CD, mobile testing), best practices, and troubleshooting. Ideal for testing signup flows, checkout processes, visual layouts, and exploratory testing.
---

# Runhuman Testing Skills

**Version:** 2.0.0
**Skills Format:** Agent Skills v1
**Last Updated:** 2026-02-05

## Overview

This skill helps AI agents create and manage human-powered QA tests using Runhuman. Runhuman connects AI coding tools to on-demand professional human testers who provide real user feedback.

## Prerequisites

- Node.js 18+ or Bun 1.0.3+
- Runhuman API key (get one at https://runhuman.com)
- Basic understanding of QA testing concepts

## Core Concepts

### What is Runhuman?

Runhuman is a human QA testing API. Instead of writing automated tests, you describe what you want tested in natural language, and real human testers perform the test and provide structured feedback.

**Key Benefits:**
- Catch issues automated tests miss (UX problems, visual bugs, mobile responsiveness)
- Get results in 1-2 minutes average
- No test maintenance overhead
- Real human perspective on user experience

### When to Use Runhuman

✅ **Good Use Cases:**
- User flows (signup, checkout, onboarding)
- Visual testing (layout, responsive design, cross-browser)
- Mobile testing (touch interactions, scrolling, gestures)
- UX feedback (confusing UI, accessibility issues)
- Exploratory testing (finding edge cases)

❌ **Not Ideal For:**
- Unit tests (use Jest/Vitest)
- Performance testing (use Lighthouse)
- Security testing (use dedicated security tools)
- Tests that need to run 100+ times per day (cost prohibitive)

---

## CLI Usage (Primary Method)

### Installation

```bash
# Install CLI globally
npm install -g runhuman

# Or use npx (no installation)
npx runhuman create <url> --description "..."
```

### Authentication

**First-time setup:**
```bash
# Login with OAuth (opens browser)
npx runhuman login

# Or login with API key directly
npx runhuman login --token qa_live_xxxxxxxxxxxxx

# Verify authentication
npx runhuman whoami
```

**For CI/CD (environment variable):**
```bash
export RUNHUMAN_API_KEY=qa_live_xxxxxxxxxxxxx
npx runhuman create https://example.com --description "..."
```

**API Key Format:** `qa_live_` followed by 40 hex characters

---

## Essential Commands

### 1. Create a Test Job

**Basic usage:**
```bash
npx runhuman create https://example.com \
  --description "Test the checkout flow: add item to cart, proceed to checkout, fill in shipping info. Check for bugs and UX issues."
```

**Required:**
- URL (as argument or via `--project` with default URL)
- Description (via `-d` flag or `--template`)
- Project ID (via `--project` or set default with `runhuman projects switch`)

**Common options:**
```bash
npx runhuman create https://example.com \
  -d "Test signup form validation" \
  --project my-project-id \
  --duration 5 \
  --screen-size mobile \
  --sync \
  --json
```

**Available options:**
- `-d, --description <text>` - Test instructions (required)
- `--project <id>` - Project ID (required, or set default)
- `--duration <minutes>` - Target duration in minutes (1-60)
- `--screen-size <preset>` - Screen size: `desktop|laptop|tablet|mobile`
- `--template <name>` - Use a template
- `--github-repo <owner/repo>` - Link to GitHub repository
- `--sync` - Wait for result before exiting (blocks until complete)
- `--wait <seconds>` - Max wait time when using `--sync` (default: 300)
- `--json` - Output as JSON
- `--quiet` - Minimal output (only job ID)

**Exit codes:**
- 0 = Success
- 1 = General error
- 2 = Authentication failed
- 4 = Validation error (missing required fields)

---

### 2. Check Job Status

```bash
npx runhuman status <job-id>
```

**Output includes:**
- Current status (pending, claimed, in_progress, completed, etc.)
- Tester name (if claimed)
- URL being tested
- Duration and cost
- Dashboard URL

---

### 3. Wait for Completion

```bash
npx runhuman wait <job-id> --timeout 600
```

**What it does:**
- Polls every 10 seconds
- Shows spinner with elapsed time
- Exits when job reaches terminal state (completed, error, etc.)
- Displays results summary on completion

**Exit codes:**
- 0 = Completed successfully
- 5 = Timeout exceeded

---

### 4. View Results

```bash
npx runhuman results <job-id>
```

**Options:**
- `--schema-only` - Show only extracted structured data
- `--raw` - Show raw tester response
- `--json` - Output as JSON

**Output sections:**
1. Job information (ID, status, URL, timestamps, cost)
2. Structured results (if schema provided)
3. Tester feedback (natural language findings)

---

### 5. List Jobs

```bash
# List all jobs
npx runhuman list

# Filter by status
npx runhuman list completed
npx runhuman list pending

# Filter by project
npx runhuman list --project my-project-id --limit 10
```

**Status filters:** `all|pending|claimed|in_progress|completed|failed|timeout`

**Options:**
- `--project <id>` - Filter by project
- `--limit <number>` - Number of results (default: 20)
- `--offset <number>` - Pagination offset
- `--format <type>` - Output format: `table|json|compact`

---

## Project Management

Runhuman uses **projects** to organize tests. Projects can have default URLs, GitHub repo links, and shared configuration.

### Set Default Project

```bash
# List projects
npx runhuman projects list

# Set default project (saves to global config)
npx runhuman projects switch <project-id>

# Create new project
npx runhuman projects create "My App" \
  --default-url https://myapp.com \
  --github-repo owner/repo
```

**Why projects matter:**
- Required for creating jobs
- Store default URL and test configuration
- Link to GitHub repositories
- Organize tests by application/environment

---

## Configuration

### Config Hierarchy (highest to lowest priority)

1. **CLI flags** - `--api-key`, `--project`, etc.
2. **Environment variables** - `RUNHUMAN_API_KEY`, `RUNHUMAN_PROJECT`
3. **Project config** - `.runhumanrc` in current directory
4. **Global config** - `~/.config/runhuman/config.json`
5. **Defaults**

### Configuration File (`.runhumanrc`)

Create in your project root:

```json
{
  "apiKey": "qa_live_xxxxxxxxxxxxx",
  "apiUrl": "https://runhuman.com",
  "project": "my-project-id",
  "defaultUrl": "https://myapp.com",
  "defaultDuration": 5,
  "defaultScreenSize": "desktop",
  "githubRepo": "owner/repo"
}
```

**Supported fields:**
- `apiKey` - API key for authentication
- `apiUrl` - API server URL (default: https://runhuman.com)
- `project` - Default project ID
- `defaultUrl` - Default test URL
- `defaultDuration` - Default test duration in minutes
- `defaultScreenSize` - Default screen size preset
- `githubRepo` - Linked GitHub repository

### Environment Variables

```bash
RUNHUMAN_API_KEY=qa_live_xxxxxxxxxxxxx
RUNHUMAN_API_URL=https://runhuman.com
RUNHUMAN_PROJECT=my-project-id
RUNHUMAN_DEFAULT_URL=https://example.com
RUNHUMAN_DEFAULT_DURATION=5
RUNHUMAN_DEFAULT_SCREEN_SIZE=desktop
```

### Config Commands

```bash
# View all config
npx runhuman config list

# Get specific value
npx runhuman config get project

# Set value (project-level)
npx runhuman config set project my-project-id

# Set value (global)
npx runhuman config set project my-project-id --global
```

---

## Common Workflows

### Workflow 1: Pre-Deployment Check

```bash
#!/bin/bash
# Test critical flows before deploying

echo "Testing staging site before deployment..."

npx runhuman create https://staging.myapp.com \
  --description "Test user signup: click 'Sign Up', fill form, verify email sent" \
  --sync

if [ $? -eq 0 ]; then
  echo "✅ Tests passed - proceeding with deployment"
  ./deploy.sh
else
  echo "❌ Tests failed - deployment aborted"
  exit 1
fi
```

### Workflow 2: GitHub Actions CI/CD

```yaml
name: Pre-Merge QA Test

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  human-qa:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy PR preview
        run: ./deploy-preview.sh ${{ github.event.pull_request.number }}

      - name: Run human QA test
        run: |
          npx runhuman create https://pr-${{ github.event.pull_request.number }}.preview.app \
            --description "Test the changes in this PR: ${{ github.event.pull_request.title }}" \
            --sync \
            --project ${{ vars.RUNHUMAN_PROJECT_ID }} \
            --api-key ${{ secrets.RUNHUMAN_API_KEY }}
```

### Workflow 3: Mobile Responsiveness Test

```bash
# Test on mobile viewport
npx runhuman create https://myapp.com \
  --description "Test on mobile device: check responsive layout, touch interactions, and navigation menu. Report any layout issues or hard-to-tap buttons." \
  --screen-size mobile
```

### Workflow 4: Manual Testing Script

```bash
#!/bin/bash
# test-suite.sh - Run multiple tests in sequence

echo "🧪 Running Runhuman Test Suite"

# Test 1: Homepage
npx runhuman create https://myapp.com \
  --description "Test homepage: hero section, navigation, footer links"

# Test 2: Signup
npx runhuman create https://myapp.com/signup \
  --description "Test signup: form validation, error messages, success flow"

# Test 3: Mobile
npx runhuman create https://myapp.com \
  --screen-size mobile \
  --description "Test on mobile: responsive layout, touch interactions"

echo "✅ All tests submitted. Check dashboard for results."
```

---

## Writing Effective Test Descriptions

### ✅ Good Descriptions

**Example 1: Specific Flow**
```
Test the checkout process:
1. Add product to cart
2. Click "Checkout"
3. Fill in shipping address
4. Select payment method
5. Review order
6. Complete purchase

Report: bugs, UX confusions, mobile layout issues
```

**Example 2: Exploratory Testing**
```
Explore the dashboard as a new user. Try to:
- Create a project
- Add team members
- Navigate all menu items

Report: anything confusing, broken links, visual bugs
```

**Example 3: Visual Testing**
```
Check the homepage on mobile:
- Does text fit without wrapping awkwardly?
- Are images properly sized?
- Can you tap all buttons easily?
- Does the navigation menu work?

Report: layout issues, hard-to-read text, broken images
```

### ❌ Bad Descriptions

**Too Vague:**
```
"Test the site" ❌
Better: "Test the signup flow: click sign up, fill form, check confirmation email"
```

**Too Technical:**
```
"Verify JWT token expiration after 15 minutes of inactivity" ❌
Better: "Log in, wait 15 minutes without activity, then try to use the app. Check if you're logged out."
```

**Missing Context:**
```
"Click the button" ❌
Better: "Click the 'Get Started' button in the hero section"
```

---

## Best Practices

### 1. Test Description Guidelines

- **Be specific:** Include exact steps or flows
- **Include context:** What page, what button, what form
- **Define success:** What should happen vs. what to look for
- **List concerns:** Bugs? UX? Mobile? Accessibility?

### 2. When to Use `--sync` Flag

**Use `--sync` when:**
- Running in CI/CD pipelines
- Need to block deployment
- Testing critical flows pre-release

**Don't use `--sync` when:**
- Running many tests in parallel
- Testing non-blocking features
- Doing exploratory testing

### 3. Frequency Recommendations

**Per Commit:** Only critical flows (signup, checkout)
**Per PR:** Moderate (flows changed in PR)
**Nightly:** Comprehensive (full regression suite)
**Pre-Release:** Exhaustive (everything)

### 4. Cost Optimization

Tests cost ~$1-3 per test. Optimize by:
- **Batch related tests:** "Test signup AND login" (1 test) vs. two separate tests
- **Use `--sync` judiciously:** Only when necessary
- **Test staging, not production:** Cheaper to catch bugs early
- **Prioritize critical flows:** Not every page needs daily testing

---

## Troubleshooting

### Error: "Authentication failed"

**Solution:**
```bash
# Login again
npx runhuman login

# Or set API key in environment
export RUNHUMAN_API_KEY=qa_live_xxxxxxxxxxxxx

# Verify
npx runhuman whoami
```

### Error: "Project ID is required"

**Solution:**
```bash
# Set default project
npx runhuman projects switch <project-id>

# Or pass explicitly
npx runhuman create <url> --description "..." --project <project-id>
```

### Error: "Validation error"

**Causes:**
- Missing description (use `-d` flag)
- Missing URL (provide as argument or set `defaultUrl`)
- Missing project ID (set default or use `--project`)

**Solution:**
```bash
# Ensure all required fields are provided
npx runhuman create https://example.com \
  -d "Test description" \
  --project my-project-id
```

### Error: "Invalid URL"

**Solution:**
- Ensure URL includes protocol: `https://example.com` not `example.com`
- Ensure site is publicly accessible (not localhost)
- For localhost testing, use a tunnel (ngrok, etc.)

---

## Quick Reference

### Most Common Commands

```bash
# Authentication
npx runhuman login
npx runhuman whoami

# Projects
npx runhuman projects list
npx runhuman projects switch <project-id>

# Create test (basic)
npx runhuman create <url> -d "Test description"

# Create test (with options)
npx runhuman create <url> \
  -d "Test description" \
  --screen-size mobile \
  --sync

# Check status and results
npx runhuman status <job-id>
npx runhuman wait <job-id>
npx runhuman results <job-id>

# List jobs
npx runhuman list
npx runhuman list completed --limit 10
```

### Exit Codes

- `0` - Success
- `1` - General error
- `2` - Authentication error
- `3` - Resource not found
- `4` - Validation error (missing required fields)
- `5` - Timeout error

---

## Additional Resources

- **Dashboard:** https://runhuman.com/dashboard
- **API Docs:** https://runhuman.com/docs/api
- **GitHub Actions:** https://github.com/marketplace/actions/runhuman-qa-test
- **CLI README:** https://github.com/volter-ai/runhuman/tree/main/packages/cli

---

**End of Skill**
