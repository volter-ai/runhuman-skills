---
name: runhuman-testing
description: Create and manage human-powered QA tests using Runhuman CLI. Use this skill when you need to test web applications with real human testers, get UX feedback, validate user flows, check mobile responsiveness, or find bugs that automated tests miss. Covers CLI commands, writing effective test descriptions, common workflows (pre-deployment, CI/CD, mobile testing), best practices, and troubleshooting. Ideal for testing signup flows, checkout processes, visual layouts, and exploratory testing.
---

# Runhuman Testing Skills

**Version:** 1.0.0
**Skills Format:** Agent Skills v1
**Last Updated:** 2026-01-30

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
npm install -g @runhuman/cli

# Or use npx (no installation)
npx @runhuman/cli test <url>
```

### Quick Start

**Simplest possible test:**
```bash
npx @runhuman/cli test https://example.com
```

This sends your site to a human tester with default instructions: "Explore the site and report any issues."

### Basic CLI Patterns

#### 1. Test with Custom Description

```bash
npx @runhuman/cli test https://example.com \
  --description "Test the checkout flow: add item to cart, proceed to checkout, fill in shipping info. Check for bugs and UX issues."
```

**Best Practice:** Be specific about:
- What pages/flows to test
- What actions to perform
- What to look for (bugs, UX issues, mobile responsiveness)

#### 2. Wait for Results Synchronously

```bash
# Block until test completes (useful in CI/CD)
npx @runhuman/cli test https://example.com \
  --description "Test signup form" \
  --wait
```

**When to use:**
- CI/CD pipelines (block deployment if test fails)
- Critical flows before merging PR
- One-off manual testing

#### 3. Configure Timeouts

```bash
# Set custom timeout (default: 5 minutes)
npx @runhuman/cli test https://example.com \
  --timeout 300000 \  # 5 minutes in milliseconds
  --description "Full e-commerce flow"
```

#### 4. Check Test Status

```bash
# Get status of a specific job
npx @runhuman/cli status <job-id>

# List all recent tests
npx @runhuman/cli list
```

### Common CLI Workflows

#### Workflow 1: Pre-Deployment Check

```bash
#!/bin/bash
# Test critical flows before deploying

echo "Testing staging site before deployment..."

npx @runhuman/cli test https://staging.myapp.com \
  --description "Test user signup: click 'Sign Up', fill form, verify email sent" \
  --wait

if [ $? -eq 0 ]; then
  echo "✅ Tests passed - proceeding with deployment"
  ./deploy.sh
else
  echo "❌ Tests failed - deployment aborted"
  exit 1
fi
```

#### Workflow 2: Mobile Responsiveness Test

```bash
# Test on mobile viewport
npx @runhuman/cli test https://myapp.com \
  --description "Test on mobile device: check responsive layout, touch interactions, and navigation menu. Report any layout issues or hard-to-tap buttons."
```

#### Workflow 3: Cross-Browser Testing

```bash
# Multiple tests for different browsers
npx @runhuman/cli test https://myapp.com \
  --description "Test in Chrome: full site navigation and checkout flow"

npx @runhuman/cli test https://myapp.com \
  --description "Test in Safari: same as Chrome test, note any browser-specific issues"
```

---

## Advanced CLI Usage

### Environment Variables

```bash
# Set API key via environment variable
export RUNHUMAN_API_KEY=rh_live_abc123

# Use staging API
export RUNHUMAN_API_URL=https://staging.runhuman.com/api

# Then run tests without --api-key flag
npx @runhuman/cli test https://example.com
```

### Configuration File

Create `.runhumanrc.json` in your project root:

```json
{
  "apiKey": "rh_live_abc123",
  "defaultTimeout": 300000,
  "defaultDescription": "Test for bugs and UX issues"
}
```

### Output Formats

```bash
# JSON output (for scripting)
npx @runhuman/cli test https://example.com --format json

# Pretty console output (default)
npx @runhuman/cli test https://example.com --format pretty

# CI/CD output (minimal, exit codes only)
npx @runhuman/cli test https://example.com --format ci
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

## Integration Patterns

### Pattern 1: Pre-Merge PR Testing

```bash
# In .github/workflows/pr-test.yml
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
          npx @runhuman/cli test https://pr-${{ github.event.pull_request.number }}.preview.app \
            --description "Test the changes in this PR: ${{ github.event.pull_request.title }}" \
            --wait \
            --api-key ${{ secrets.RUNHUMAN_API_KEY }}
```

### Pattern 2: Nightly Full Regression

```bash
# In .github/workflows/nightly-qa.yml
name: Nightly Full Regression

on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM daily

jobs:
  regression:
    runs-on: ubuntu-latest
    steps:
      - name: Test critical user flows
        run: |
          npx @runhuman/cli test https://production.app \
            --description "Full regression: signup, login, checkout, account settings" \
            --wait \
            --timeout 600000 \  # 10 minutes
            --api-key ${{ secrets.RUNHUMAN_API_KEY }}
```

### Pattern 3: Manual Testing Script

```bash
#!/bin/bash
# test-suite.sh - Run multiple tests in sequence

echo "🧪 Running Runhuman Test Suite"

# Test 1: Homepage
npx @runhuman/cli test https://myapp.com \
  --description "Test homepage: hero section, navigation, footer links"

# Test 2: Signup
npx @runhuman/cli test https://myapp.com/signup \
  --description "Test signup: form validation, error messages, success flow"

# Test 3: Mobile
npx @runhuman/cli test https://myapp.com \
  --description "Test on mobile: responsive layout, touch interactions"

echo "✅ All tests submitted. Check dashboard for results."
```

---

## Best Practices

### 1. Test Description Guidelines

- **Be specific:** Include exact steps or flows
- **Include context:** What page, what button, what form
- **Define success:** What should happen vs. what to look for
- **List concerns:** Bugs? UX? Mobile? Accessibility?

### 2. When to Use --wait Flag

**Use --wait when:**
- Running in CI/CD pipelines
- Need to block deployment
- Testing critical flows pre-release

**Don't use --wait when:**
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
- **Use --wait judiciously:** Only when necessary
- **Test staging, not production:** Cheaper to catch bugs early
- **Prioritize critical flows:** Not every page needs daily testing

---

## Troubleshooting

### Error: "API key not found"

**Solution:**
```bash
# Set API key
export RUNHUMAN_API_KEY=rh_live_abc123

# Or pass inline
npx @runhuman/cli test https://example.com --api-key rh_live_abc123
```

### Error: "Timeout exceeded"

**Solution:**
```bash
# Increase timeout for complex flows
npx @runhuman/cli test https://example.com \
  --timeout 600000 \  # 10 minutes
  --description "Complex multi-step flow"
```

### Error: "Invalid URL"

**Solution:**
- Ensure URL includes protocol: `https://example.com` not `example.com`
- Ensure site is publicly accessible (not localhost)
- For localhost testing, use a tunnel (ngrok, etc.)

### Test Results Show "No Issues Found"

**Possible reasons:**
- Test description too vague
- Site genuinely has no issues (rare!)
- Tester didn't understand instructions

**Solution:** Make description more specific with exact steps.

---

## Quick Reference

### Most Common Commands

```bash
# Basic test
npx @runhuman/cli test <url>

# Test with description
npx @runhuman/cli test <url> --description "..."

# Synchronous test (wait for result)
npx @runhuman/cli test <url> --wait

# Check status
npx @runhuman/cli status <job-id>

# List recent tests
npx @runhuman/cli list
```

### Environment Variables

```bash
RUNHUMAN_API_KEY=rh_live_...     # Your API key
RUNHUMAN_API_URL=https://...     # API endpoint (optional)
```

### Exit Codes

- `0` - Test passed (no critical issues found)
- `1` - Test failed (critical issues found)
- `2` - Test timeout
- `3` - API error

---

## Additional Resources

- **Dashboard:** https://runhuman.com/dashboard
- **API Docs:** https://runhuman.com/docs/api
- **GitHub Actions:** https://github.com/marketplace/actions/runhuman-qa-test
- **Support:** https://runhuman.com/discord

---

**End of Skill**
