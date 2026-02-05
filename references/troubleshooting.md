# Troubleshooting Guide

Common errors and solutions when using the Runhuman CLI.

---

## Authentication Issues

### Error: "Authentication failed"

**Cause:** No valid API key found or token expired.

**Solution:**

```bash
# Re-authenticate with OAuth
npx runhuman login

# Or use token directly
npx runhuman login --token qa_live_xxxxxxxxxxxxx

# Verify authentication
npx runhuman whoami
```

**Check your configuration:**

```bash
# Check where API key is set
npx runhuman config list

# Verify it's valid
npx runhuman whoami
```

**For CI/CD:**

```bash
# Use environment variable
export RUNHUMAN_API_KEY=qa_live_xxxxxxxxxxxxx
npx runhuman create <url> -d "..."
```

---

### Error: "API key format invalid"

**Cause:** API key doesn't match expected format.

**Expected format:** `qa_live_` followed by 40 hexadecimal characters

**Example:** `qa_live_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p7q8r9s0t`

**Solution:**

```bash
# Get a new API key
npx runhuman login

# Or create a new key
npx runhuman keys create "My Key" --copy
```

---

### Error: "Unauthorized"

**Cause:** API key doesn't have permission for the requested operation.

**Solution:**

1. Verify you're authenticated:
   ```bash
   npx runhuman whoami
   ```

2. Check if your key has access to the project:
   ```bash
   npx runhuman projects list
   ```

3. If using a project-specific key, ensure `--project` matches:
   ```bash
   npx runhuman create <url> -d "..." --project <correct-project-id>
   ```

---

## Project Configuration Issues

### Error: "Project ID is required"

**Cause:** No default project configured and `--project` flag not provided.

**Solution:**

```bash
# Step 1: List available projects
npx runhuman projects list

# Step 2: Set default project
npx runhuman projects switch <project-id>

# Verify
npx runhuman config get project
```

**Alternative:** Pass `--project` flag explicitly:

```bash
npx runhuman create <url> -d "..." --project <project-id>
```

---

### Error: "Project not found"

**Cause:** The specified project ID doesn't exist or you don't have access.

**Solution:**

```bash
# List projects you have access to
npx runhuman projects list

# Use a valid project ID
npx runhuman projects switch <valid-project-id>
```

---

### Config shows "(not set)"

**Cause:** Configuration value not set at any level.

**Check configuration hierarchy:**

```bash
# View all config sources
npx runhuman config list

# Check specific value
npx runhuman config get project
```

**Set configuration:**

```bash
# Project-level (creates/updates .runhumanrc)
npx runhuman config set project <project-id>

# Global level (~/.config/runhuman/config.json)
npx runhuman config set project <project-id> --global
```

---

## Validation Errors

### Error: "URL is required"

**Cause:** No URL provided and no default URL configured.

**Solution:**

```bash
# Option 1: Provide URL as argument
npx runhuman create https://example.com -d "..."

# Option 2: Set default URL
npx runhuman config set defaultUrl https://example.com

# Option 3: Use project with default URL
npx runhuman projects update <project-id> --default-url https://example.com
```

---

### Error: "Description is required"

**Cause:** No test description provided.

**Solution:**

```bash
# Always provide -d flag with description
npx runhuman create https://example.com \
  -d "Test the signup flow: fill form and verify success"
```

**Note:** Description cannot be empty or omitted. There is no default description.

---

### Error: "Invalid URL format"

**Cause:** URL doesn't include protocol or is malformed.

**❌ Wrong:**
```bash
npx runhuman create example.com -d "..."
npx runhuman create www.example.com -d "..."
```

**✅ Correct:**
```bash
npx runhuman create https://example.com -d "..."
npx runhuman create http://example.com -d "..."
```

---

### Error: "URL must be publicly accessible"

**Cause:** Trying to test localhost or internal URL.

**Problem:** Human testers cannot access `localhost`, `127.0.0.1`, or internal network URLs.

**Solutions:**

1. **Use a tunnel service:**
   ```bash
   # Option 1: ngrok
   ngrok http 3000
   # Use the https://xxx.ngrok.io URL

   # Option 2: localtunnel
   npx localtunnel --port 3000
   # Use the provided public URL

   # Option 3: Cloudflare Tunnel
   cloudflared tunnel --url http://localhost:3000
   ```

2. **Deploy to staging environment:**
   ```bash
   # Test on staging instead
   npx runhuman create https://staging.myapp.com -d "..."
   ```

3. **Use preview deployments:**
   ```bash
   # Vercel, Netlify, Railway, etc. provide preview URLs
   npx runhuman create https://preview-abc123.vercel.app -d "..."
   ```

---

### Error: "Duration must be between 1 and 60"

**Cause:** Invalid `--duration` value.

**Solution:**

```bash
# Use a value between 1-60 minutes
npx runhuman create <url> -d "..." --duration 5

# Default is 5 minutes if not specified
```

---

### Error: "Invalid screen size"

**Cause:** Invalid `--screen-size` value.

**Valid options:** `desktop`, `laptop`, `tablet`, `mobile`

**Solution:**

```bash
npx runhuman create <url> -d "..." --screen-size mobile
```

---

## Job Status Issues

### Error: "Job not found"

**Cause:** Invalid job ID or job doesn't exist in your accessible projects.

**Solution:**

```bash
# List recent jobs to find correct ID
npx runhuman list --limit 10

# Use correct job ID format (starts with job_)
npx runhuman status job_abc123
```

---

### Job stuck in "pending" status

**Cause:** High demand or tester availability issues.

**What to do:**

1. **Wait:** Most jobs are claimed within 1-2 minutes
2. **Check dashboard:** https://runhuman.com/dashboard

**Normal timing:**
- Claimed: 30 seconds - 2 minutes
- Completed: 3-7 minutes (depending on test complexity)

---

### Job status is "timeout"

**Cause:** Test took longer than allocated duration.

**Solution:**

```bash
# Increase duration for complex tests
npx runhuman create <url> -d "..." --duration 10

# Or break into smaller tests
npx runhuman create <url> -d "Test part 1: signup" --duration 5
npx runhuman create <url> -d "Test part 2: profile" --duration 5
```

---

### Job status is "failed"

**Cause:** Technical issue prevented test completion (site down, browser error, etc.)

**What to do:**

1. **Check the URL is accessible:**
   ```bash
   curl -I https://your-site.com
   ```

2. **Verify site is up:**
   - Visit the URL in your browser
   - Check for SSL certificate issues
   - Verify no 500 errors

3. **Retry the test:**
   ```bash
   npx runhuman create <url> -d "..."
   ```

---

## Sync Mode Issues

### Error: "Timeout waiting for job completion"

**Cause:** Job took longer than `--wait` timeout.

**Solution:**

```bash
# Increase wait timeout
npx runhuman create <url> -d "..." --sync --wait 600

# Or use wait command separately
JOB_ID=$(npx runhuman create <url> -d "..." --quiet)
npx runhuman wait $JOB_ID --timeout 600
```

---

### `--sync` hangs indefinitely

**Cause:** Job stuck or network issue.

**Solution:**

1. **Cancel with Ctrl+C**

2. **Check job status:**
   ```bash
   npx runhuman list --limit 5
   ```

3. **Wait for completion separately:**
   ```bash
   npx runhuman wait <job-id> --timeout 600
   ```

---

## Template Issues

### Error: "Template not found"

**Cause:** Template doesn't exist or you don't have access.

**Solution:**

```bash
# List available templates
npx runhuman templates list

# Use correct template name or ID
npx runhuman create <url> --template "Template Name"
```

---

### Template not applying correctly

**Cause:** Command-line flags override template values.

**Solution:**

Template values are **defaults**. CLI flags take precedence:

```bash
# Template has --duration 10
# This overrides it to 5
npx runhuman create <url> --template "My Template" --duration 5
```

To use template defaults, omit the flag:

```bash
# Uses template's duration value
npx runhuman create <url> --template "My Template"
```

---

## GitHub Integration Issues

### Error: "Invalid repository format"

**Cause:** Repository not in `owner/repo` format.

**❌ Wrong:**
```bash
npx runhuman github link github.com/owner/repo
npx runhuman github link https://github.com/owner/repo
```

**✅ Correct:**
```bash
npx runhuman github link owner/repo
```

---

### Error: "GitHub repository not linked"

**Cause:** Repository not linked to project.

**Solution:**

```bash
# Link repository to project
npx runhuman github link owner/repo --project <project-id>

# Verify
npx runhuman github repos
```

---

### GitHub issue test not working

**Cause:** Issue doesn't exist or repository not linked.

**Solution:**

```bash
# Ensure repo is linked
npx runhuman github link owner/repo --project <project-id>

# Verify issue exists
npx runhuman github issues owner/repo

# Test with correct issue number
npx runhuman github test 42 --repo owner/repo --url <url>
```

---

## Watch Mode Issues

### Watch not detecting file changes

**Cause:** Incorrect glob patterns or files in ignore list.

**Solution:**

```bash
# Check patterns match your files
npx runhuman watch "src/**/*.{js,jsx}" --url <url>

# Ensure not in ignore patterns
npx runhuman watch "src/**/*" \
  --url <url> \
  --ignore "node_modules/**" "dist/**"
```

**Test pattern matching:**
```bash
# Use a glob testing tool
npx glob "src/**/*.js"
```

---

### Watch creating too many tests

**Cause:** Debounce too low or watching too many files.

**Solution:**

```bash
# Increase debounce delay
npx runhuman watch "src/**/*" --url <url> --debounce 5000

# Narrow file patterns
npx runhuman watch "src/components/**/*.jsx" --url <url>
```

---

### Can't stop watch process

**Cause:** Process not responding.

**Solution:**

```bash
# Try stop command
npx runhuman watch --stop

# If that fails, find and kill process
ps aux | grep "runhuman watch"
kill <pid>
```

---

## Configuration File Issues

### `.runhumanrc` not being read

**Cause:** File in wrong location or invalid JSON.

**Solution:**

1. **Check file location:**
   - Must be in current directory (project root)
   - Named exactly `.runhumanrc` (no extension)

2. **Validate JSON:**
   ```bash
   # Check for syntax errors
   cat .runhumanrc | jq .
   ```

3. **Check file permissions:**
   ```bash
   ls -la .runhumanrc
   chmod 644 .runhumanrc
   ```

---

### Config values not taking effect

**Cause:** Configuration priority hierarchy.

**Remember the order (highest to lowest):**

1. CLI flags (`--project`, `--api-key`)
2. Environment variables (`RUNHUMAN_PROJECT`)
3. Project config (`.runhumanrc`)
4. Global config (`~/.config/runhuman/config.json`)

**Debug:**

```bash
# See which values are active
npx runhuman config list

# Check specific value and its source
npx runhuman config get project
```

---

### Can't write to global config

**Cause:** Permission issue or directory doesn't exist.

**Solution:**

```bash
# Create config directory
mkdir -p ~/.config/runhuman

# Fix permissions
chmod 755 ~/.config/runhuman

# Try again
npx runhuman config set project <id> --global
```

---

## API Key Management Issues

### Can't see full API key

**Cause:** Keys are masked by default for security.

**Solution:**

```bash
# Show full keys
npx runhuman keys list --show-keys

# Copy to clipboard when creating
npx runhuman keys create "My Key" --copy
```

**Important:** Keys are only shown in full when created. Save them securely.

---

### API key not working in CI/CD

**Cause:** Environment variable not set or wrong format.

**Solution:**

```yaml
# GitHub Actions
env:
  RUNHUMAN_API_KEY: ${{ secrets.RUNHUMAN_API_KEY }}

# Verify in CI
- run: echo "API key set" | grep -q "$RUNHUMAN_API_KEY" && echo "Yes" || echo "No"
```

**Test locally:**

```bash
export RUNHUMAN_API_KEY=qa_live_xxxxxxxxxxxxx
npx runhuman whoami
```

---

## Performance Issues

### CLI commands are slow

**Causes and solutions:**

1. **Slow network connection:**
   - Check internet connection
   - Try different network

2. **Large project with many jobs:**
   - Use `--limit` flag:
     ```bash
     npx runhuman list --limit 10
     ```

---

### Tests taking too long to complete

**Cause:** Complex test or tester availability.

**Solutions:**

1. **Break into smaller tests:**
   ```bash
   # Instead of one 15-minute test
   npx runhuman create <url> -d "Test everything" --duration 15

   # Do three 5-minute tests
   npx runhuman create <url> -d "Test signup" --duration 5
   npx runhuman create <url> -d "Test dashboard" --duration 5
   npx runhuman create <url> -d "Test settings" --duration 5
   ```

2. **Simplify test description:**
   - Focus on specific flows
   - Remove unnecessary steps

3. **Check status:**
   ```bash
   npx runhuman status <job-id>
   ```

---

## JSON Output Issues

### Invalid JSON output

**Cause:** Command includes non-JSON output (warnings, logs).

**Solution:**

```bash
# Use --quiet flag to suppress non-JSON output
npx runhuman create <url> -d "..." --json --quiet

# Or pipe through jq to extract JSON
npx runhuman create <url> -d "..." --json 2>/dev/null | jq .
```

---

### Can't parse results

**Cause:** Unexpected JSON structure.

**Solution:**

```bash
# Pretty-print to see structure
npx runhuman results <job-id> --json | jq .

# Extract specific fields
npx runhuman results <job-id> --json | jq '.feedback'

# Check schema-only output
npx runhuman results <job-id> --schema-only --json | jq .
```

---

## Common Exit Codes

Understanding exit codes for scripting:

| Code | Meaning | What to do |
|------|---------|------------|
| 0 | Success | Continue |
| 1 | General error | Check error message |
| 2 | Authentication failed | Run `login` command |
| 3 | Resource not found | Verify ID exists |
| 4 | Validation error | Check required fields |
| 5 | Timeout | Increase `--wait` time |

**Example script:**

```bash
npx runhuman create <url> -d "..." --sync
EXIT_CODE=$?

case $EXIT_CODE in
  0) echo "✅ Test passed" ;;
  2) echo "❌ Auth failed - run: npx runhuman login" ;;
  4) echo "❌ Missing required fields" ;;
  5) echo "⏱️ Timeout - increase --wait" ;;
  *) echo "❌ Error: $EXIT_CODE" ;;
esac

exit $EXIT_CODE
```

---

## Getting Help

### Built-in Help

```bash
# General help
npx runhuman --help

# Command-specific help
npx runhuman create --help
npx runhuman projects --help
npx runhuman github test --help
```

### Check Version

```bash
npx runhuman --version
```

### Check Configuration

```bash
# See all config
npx runhuman config list

# Verify authentication
npx runhuman whoami

# Check project
npx runhuman config get project
```

### Debug Mode

```bash
# Enable verbose logging (if available)
DEBUG=* npx runhuman create <url> -d "..."

# Or check your shell for errors
bash -x ./your-script.sh
```

---

## Still Having Issues?

1. **Check the documentation:**
   - [Command Reference](commands.md)
   - [Workflows](workflows.md)
   - CLI help: `npx runhuman <command> --help`

2. **Visit the dashboard:**
   - https://runhuman.com/dashboard
   - Check job status and history
