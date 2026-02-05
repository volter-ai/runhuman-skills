# Common Runhuman Workflows

Real-world examples and best practices for integrating Runhuman into your development workflow.

---

## Pre-Deployment Testing

Test critical user flows before deploying to production.

```bash
#!/bin/bash
# test-before-deploy.sh

echo "🧪 Testing staging site before deployment..."

# Test signup flow
npx runhuman create https://staging.myapp.com/signup \
  -d "Test user signup: click Sign Up, fill email/password, submit, verify confirmation message appears" \
  --sync

if [ $? -ne 0 ]; then
  echo "❌ Signup test failed - aborting deployment"
  exit 1
fi

# Test checkout flow
npx runhuman create https://staging.myapp.com \
  -d "Test checkout: add item to cart, proceed to checkout, fill shipping info, verify order summary" \
  --sync

if [ $? -ne 0 ]; then
  echo "❌ Checkout test failed - aborting deployment"
  exit 1
fi

echo "✅ All tests passed - proceeding with deployment"
./deploy.sh
```

**Exit codes:**
- `0` = All tests passed, safe to deploy
- `1` = Test failed, deployment aborted

---

## GitHub Actions CI/CD

Integrate human QA into your pull request workflow.

### Option 1: Test PR Preview Deployments

```yaml
name: Human QA Test

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy PR preview
        id: deploy
        run: |
          # Your preview deployment logic
          echo "preview_url=https://pr-${{ github.event.pull_request.number }}.preview.app" >> $GITHUB_OUTPUT

      - name: Run human QA test
        env:
          RUNHUMAN_API_KEY: ${{ secrets.RUNHUMAN_API_KEY }}
        run: |
          npx runhuman create ${{ steps.deploy.outputs.preview_url }} \
            --description "Test the changes in PR #${{ github.event.pull_request.number }}: ${{ github.event.pull_request.title }}" \
            --project ${{ vars.RUNHUMAN_PROJECT_ID }} \
            --sync \
            --wait 600
```

### Option 2: Test Staging on PR Merge

```yaml
name: Staging QA Test

on:
  push:
    branches: [main, develop]

jobs:
  test-staging:
    runs-on: ubuntu-latest
    steps:
      - name: Run QA tests on staging
        env:
          RUNHUMAN_API_KEY: ${{ secrets.RUNHUMAN_API_KEY }}
        run: |
          # Test critical flows
          npx runhuman create https://staging.myapp.com \
            -d "Test user signup and login flows. Report any errors or UX issues." \
            --sync

          npx runhuman create https://staging.myapp.com \
            -d "Test main product features. Check for broken functionality." \
            --sync
```

### Option 3: Scheduled Nightly Tests

```yaml
name: Nightly QA Suite

on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM daily

jobs:
  comprehensive-testing:
    runs-on: ubuntu-latest
    steps:
      - name: Full regression test
        env:
          RUNHUMAN_API_KEY: ${{ secrets.RUNHUMAN_API_KEY }}
        run: |
          # Homepage
          npx runhuman create https://myapp.com \
            -d "Test homepage: navigation, hero section, footer links, responsive layout" &

          # Signup
          npx runhuman create https://myapp.com/signup \
            -d "Test signup: form validation, error messages, success flow" &

          # Dashboard
          npx runhuman create https://myapp.com/dashboard \
            -d "Test dashboard: all widgets load, data displays correctly, actions work" &

          # Mobile
          npx runhuman create https://myapp.com \
            -d "Test on mobile: responsive layout, touch interactions, navigation" \
            --screen-size mobile &

          wait  # Wait for all background jobs to finish
```

---

## Mobile Responsiveness Testing

Test your application on different device sizes.

### Test Mobile Layout

```bash
npx runhuman create https://myapp.com \
  --description "Test on mobile device:
  - Does the navigation menu work?
  - Is text readable without zooming?
  - Are buttons easy to tap?
  - Does content fit without horizontal scrolling?
  - Check for any layout issues or overlapping elements

  Report any problems you find." \
  --screen-size mobile
```

### Test Across All Device Sizes

```bash
#!/bin/bash
# test-responsive.sh

URL="https://myapp.com"
DESCRIPTION="Test responsive layout: check navigation, content readability, and interactive elements"

# Test each screen size
for SIZE in desktop laptop tablet mobile; do
  echo "Testing on $SIZE..."
  npx runhuman create "$URL" \
    -d "$DESCRIPTION on $SIZE screen" \
    --screen-size "$SIZE" &
done

wait
echo "✅ All device size tests submitted"
```

---

## Batch Testing Multiple Flows

Test multiple user flows in parallel to save time.

```bash
#!/bin/bash
# test-suite.sh

PROJECT_ID="my-project-id"
BASE_URL="https://staging.myapp.com"

echo "🧪 Running Runhuman Test Suite"

# Submit all tests in parallel (faster)
npx runhuman create "$BASE_URL" \
  -d "Test homepage: hero section, navigation menu, footer links" \
  --project "$PROJECT_ID" &

npx runhuman create "$BASE_URL/signup" \
  -d "Test signup flow: form validation, error handling, success redirect" \
  --project "$PROJECT_ID" &

npx runhuman create "$BASE_URL/login" \
  -d "Test login: valid credentials work, invalid show errors, forgot password link works" \
  --project "$PROJECT_ID" &

npx runhuman create "$BASE_URL/dashboard" \
  -d "Test dashboard: all sections load, data displays correctly, navigation works" \
  --project "$PROJECT_ID" &

npx runhuman create "$BASE_URL" \
  -d "Test on mobile: responsive layout, touch interactions" \
  --screen-size mobile \
  --project "$PROJECT_ID" &

# Wait for all tests to be submitted
wait

echo "✅ All tests submitted. Check dashboard for results:"
echo "https://runhuman.com/dashboard"
```

**Cost optimization:** Submit in parallel (5 concurrent tests) rather than sequentially with `--sync`.

---

## Using Templates for Repeated Tests

Create reusable templates for common test scenarios.

### Step 1: Create Template

```bash
# Create a template for signup testing
npx runhuman templates create "Signup Flow Test" \
  --description "Test user signup: fill form, check validation, verify success" \
  --duration 5 \
  --project my-project-id
```

### Step 2: Use Template

```bash
# Use the template for different environments
npx runhuman create https://staging.myapp.com/signup \
  --template "Signup Flow Test"

npx runhuman create https://preview.myapp.com/signup \
  --template "Signup Flow Test"
```

**Benefits:**
- Consistent test descriptions
- Reuse configuration (duration, screen size, schema)
- Faster test creation
- Easier tracking and reporting

---

## Structured Output with JSON Schema

Get structured, parseable results from tests.

### Step 1: Define Schema

Create `test-schema.json`:

```json
{
  "type": "object",
  "properties": {
    "bugs_found": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "severity": { "type": "string", "enum": ["critical", "high", "medium", "low"] },
          "description": { "type": "string" },
          "steps_to_reproduce": { "type": "string" }
        }
      }
    },
    "ux_issues": {
      "type": "array",
      "items": { "type": "string" }
    },
    "overall_quality": {
      "type": "string",
      "enum": ["excellent", "good", "fair", "poor"]
    }
  }
}
```

### Step 2: Create Test with Schema

```bash
npx runhuman create https://myapp.com \
  -d "Test the checkout flow and report bugs and UX issues" \
  --schema ./test-schema.json \
  --sync

# Get structured output
npx runhuman results <job-id> --schema-only --json > results.json
```

### Step 3: Parse Results

```bash
#!/bin/bash
# Parse structured results and fail if critical bugs found

RESULTS=$(npx runhuman results <job-id> --schema-only --json)

# Check for critical bugs
CRITICAL_BUGS=$(echo "$RESULTS" | jq '.bugs_found[] | select(.severity == "critical") | length')

if [ "$CRITICAL_BUGS" -gt 0 ]; then
  echo "❌ Critical bugs found - deployment blocked"
  echo "$RESULTS" | jq '.bugs_found[] | select(.severity == "critical")'
  exit 1
fi

echo "✅ No critical bugs found"
```

---

## GitHub Issue Integration

Automatically create test jobs for GitHub issues.

### Test a Specific Issue

```bash
# Test the bug reported in issue #42
npx runhuman github test 42 \
  --repo myorg/myapp \
  --url https://myapp.com \
  --sync

# Results are linked to the issue
```

### Bulk Test Open Bugs

```bash
# Test all open issues labeled as "bug"
npx runhuman github bulk-test \
  --repo myorg/myapp \
  --url https://staging.myapp.com \
  --labels bug \
  --limit 10
```

### Workflow: Test Before Closing Issues

```yaml
# .github/workflows/test-before-close.yml
name: Test Before Closing Issue

on:
  issue_comment:
    types: [created]

jobs:
  test-fix:
    if: contains(github.event.comment.body, '/test-fix')
    runs-on: ubuntu-latest
    steps:
      - name: Run human test for issue
        env:
          RUNHUMAN_API_KEY: ${{ secrets.RUNHUMAN_API_KEY }}
        run: |
          npx runhuman github test ${{ github.event.issue.number }} \
            --repo ${{ github.repository }} \
            --url https://staging.myapp.com \
            --sync
```

**Usage:** Comment `/test-fix` on an issue to trigger a human test.

---

## File Watching for Development

Automatically test changes during development.

### Watch and Test on Changes

```bash
# Watch source files and test on changes
npx runhuman watch "src/**/*.{js,jsx,ts,tsx}" \
  --url http://localhost:3000 \
  --description "Test the latest changes on dev server" \
  --debounce 3000
```

### Configuration File

Create `.runhumanrc` for persistent watch config:

```json
{
  "project": "my-project-id",
  "defaultUrl": "http://localhost:3000",
  "watch": {
    "patterns": ["src/**/*.{js,jsx,ts,tsx}"],
    "ignore": ["node_modules/**", "dist/**", "*.test.js"],
    "debounce": 3000,
    "description": "Test changes to {file}"
  }
}
```

Then simply run:

```bash
npx runhuman watch
```

**Stop watching:**

```bash
npx runhuman watch --stop
```

---

## Cost Optimization Strategies

Runhuman tests cost ~$1-3 per test. Optimize your spending:

### 1. Batch Related Tests

**❌ Expensive (3 tests = ~$6):**
```bash
npx runhuman create url -d "Test signup"
npx runhuman create url -d "Test login"
npx runhuman create url -d "Test password reset"
```

**✅ Cheaper (1 test = ~$2):**
```bash
npx runhuman create url -d "Test authentication flows: signup, login, and password reset"
```

### 2. Use `--sync` Judiciously

**❌ Slow (sequential, 15 minutes total):**
```bash
npx runhuman create url1 -d "..." --sync
npx runhuman create url2 -d "..." --sync
npx runhuman create url3 -d "..." --sync
```

**✅ Fast (parallel, 5 minutes total):**
```bash
npx runhuman create url1 -d "..." &
npx runhuman create url2 -d "..." &
npx runhuman create url3 -d "..." &
wait
```

Only use `--sync` when you need blocking behavior (CI/CD).

### 3. Test Staging, Not Production

Catch bugs early before they reach production.

### 4. Prioritize Critical Flows

**80/20 Rule:** Focus on flows that handle:
- Money (payments, subscriptions)
- User data (signup, profile)
- Core features (main product functionality)

Don't test every page on every deploy. Test critical paths.

### 5. Use Appropriate Test Frequency

- **Per commit:** Only critical flows (signup, checkout)
- **Per PR:** Moderate coverage (flows affected by PR)
- **Nightly:** Comprehensive (full regression)
- **Pre-release:** Exhaustive (everything)

---

## Writing Effective Test Descriptions

Good descriptions get better results from testers.

### ✅ Good Descriptions

**Specific with steps:**
```
Test the checkout process:
1. Add a product to cart from homepage
2. Click "Checkout"
3. Fill in shipping address (123 Main St, City, State 12345)
4. Select "Credit Card" payment
5. Review order summary
6. Click "Place Order"

Report: any errors, confusing UI, or missing information
```

**Clear objective:**
```
Test mobile navigation:
- Tap the hamburger menu
- Verify all menu items are visible
- Try navigating to each section
- Check if menu closes properly

Report: any layout issues, hard-to-tap buttons, or broken links
```

**Exploratory with guidance:**
```
Explore the dashboard as a new user. Try to:
- Create a project
- Add a team member
- View analytics
- Customize settings

Report anything confusing, broken, or surprising. Pay attention to first-time user experience.
```

### ❌ Bad Descriptions

**Too vague:**
```
"Test the website"  ❌
```
Better: "Test the homepage: click all navigation links, check hero section loads, verify footer has correct copyright year"

**Too technical:**
```
"Verify JWT token expires after 15 minutes of inactivity and redirects to /auth/login with 401 status"  ❌
```
Better: "Log in, wait 15 minutes without clicking anything, then try to use the app. Check if you're automatically logged out."

**Missing context:**
```
"Click the button"  ❌
```
Better: "Click the 'Get Started' button in the center of the homepage"

**No success criteria:**
```
"Test checkout"  ❌
```
Better: "Test checkout: add item, proceed through steps, verify order confirmation appears with order number"

---

## Best Practices Summary

### When to Use `--sync` Flag

**✅ Use `--sync` when:**
- Running in CI/CD pipelines
- Need to block deployment on test results
- Testing critical pre-release flows
- Script needs to wait for results

**❌ Don't use `--sync` when:**
- Running multiple tests in parallel
- Testing non-critical features
- Doing exploratory testing
- Want faster turnaround (submit and check later)

### Frequency Recommendations

| Trigger | Scope | Example |
|---------|-------|---------|
| **Per Commit** | Critical only | Signup, checkout, payment |
| **Per PR** | Affected flows | Features changed in PR |
| **Nightly** | Comprehensive | Full regression suite |
| **Pre-Release** | Exhaustive | Everything + edge cases |

### Screen Size Selection

| Size | Use When |
|------|----------|
| `desktop` | Default, most users |
| `laptop` | Smaller screens (1366x768) |
| `tablet` | iPad, tablet testing |
| `mobile` | Phone, touch interactions |

### Configuration Best Practices

1. **Use `.runhumanrc` for project settings** (committed to repo)
2. **Use environment variables for secrets** (API keys in CI/CD)
3. **Use global config for personal defaults** (~/.config/runhuman/)
4. **Use templates for repeated tests** (consistency)

### Security Best Practices

1. **Never commit API keys** to git
2. **Use environment variables in CI/CD:**
   ```yaml
   env:
     RUNHUMAN_API_KEY: ${{ secrets.RUNHUMAN_API_KEY }}
   ```
3. **Rotate API keys regularly**
4. **Use separate keys for dev/staging/production**

---

## Troubleshooting Common Issues

For detailed troubleshooting, see [troubleshooting.md](troubleshooting.md).

Quick fixes:

- **"Authentication failed"** → Run `npx runhuman login`
- **"Project ID required"** → Run `npx runhuman projects switch <id>`
- **"Invalid URL"** → Ensure URL includes `https://` and is publicly accessible
- **Test timeout** → Increase duration: `--duration 10`

---

## Additional Resources

- **Dashboard:** https://runhuman.com/dashboard
- **API Documentation:** https://runhuman.com/docs/api
- **CLI Source Code:** https://github.com/volter-ai/runhuman/tree/main/packages/cli
- **GitHub Action:** https://github.com/marketplace/actions/runhuman-qa-test
- **Examples Repository:** https://github.com/volter-ai/runhuman-examples
