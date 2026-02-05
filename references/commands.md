# Runhuman CLI Command Reference

> **💡 Tip:** For the most up-to-date options, always use `npx runhuman <command> --help`

This reference documents all commands organized by category. The CLI uses Commander.js with comprehensive help text built-in.

---

## Authentication Commands

### login

Authenticate with Runhuman via OAuth (browser) or API token.

```bash
# OAuth login (opens browser)
npx runhuman login

# Direct token login
npx runhuman login --token qa_live_xxxxxxxxxxxxx

# Print auth URL instead of opening browser
npx runhuman login --no-browser
```

**Options:**
- `--token <key>` - API key for direct authentication (format: `qa_live_` + 40 hex chars)
- `--no-browser` - Print authentication URL instead of opening browser

**Exit codes:** 0 (success), 2 (authentication failed)

**Example:**
```bash
# First-time setup
npx runhuman login
# Browser opens, you authenticate, token saved to ~/.config/runhuman/config.json
```

---

### logout

Clear stored credentials and log out.

```bash
npx runhuman logout
```

**Options:**
- `--json` - Output as JSON

**What it does:**
- Removes saved API key from config
- Clears OAuth tokens
- You'll need to run `login` again to use the CLI

---

### whoami

Display current authenticated user information and account balance.

```bash
npx runhuman whoami
```

**Options:**
- `--json` - Output as JSON

**Output includes:**
- Username and email
- Current project
- Account balance (in dollars)
- Estimated tests remaining

**Example output:**
```
User: john@example.com
Project: My App (adfe4b15...)
Balance: $10.00 (~10 tests)
```

---

### tokens

Manage account balance and view usage history.

#### tokens balance

Check your current account balance and estimated tests.

```bash
npx runhuman tokens balance
```

**Options:**
- `--json` - Output as JSON

**Shows:**
- Current balance (in dollars)
- Estimated number of tests remaining
- Cost per test estimate

#### tokens history

View balance usage history (transaction log).

```bash
npx runhuman tokens history [options]
```

**Options:**
- `--limit <number>` - Number of records to show
- `--offset <number>` - Pagination offset
- `--json` - Output as JSON

**Note:** This command requires API endpoint support. Check with `--help` for current status.

---

## Job Commands

### create

Create a new QA test job with a human tester.

```bash
npx runhuman create [url] [options]
```

**Required (one of):**
- URL as argument, OR
- `--project` with a project that has a default URL

**Required:**
- `-d, --description <text>` - Test instructions for the human tester

**Common options:**
- `--project <id>` - Project ID (or use default from config)
- `--duration <minutes>` - Target duration (1-60 minutes)
- `--screen-size <preset>` - Screen size: `desktop|laptop|tablet|mobile`
- `--sync` - Wait for result before exiting (synchronous/blocking mode)
- `--wait <seconds>` - Max wait time when using `--sync` (default: 300)
- `--json` - Output as JSON (for scripting)
- `--quiet` - Minimal output (only job ID)

**Advanced options:**
- `-t, --template <name>` - Use a template as base configuration
- `--schema <file>` - Path to JSON schema file for structured output
- `--schema-inline <json>` - Inline JSON schema string
- `--metadata <json>` - Metadata for tracking (JSON string)
- `--github-repo <owner/repo>` - GitHub repository for context
- `--create-issues` - Auto-create GitHub issues from test findings
- `--api-key <key>` - Override API key from config/env

**Exit codes:**
- 0 = Success
- 1 = General error
- 2 = Authentication failed
- 4 = Validation error (missing required fields)

**Examples:**

```bash
# Basic test
npx runhuman create https://example.com \
  -d "Test signup flow: fill form, check validation"

# Mobile test with sync mode
npx runhuman create https://example.com \
  -d "Test mobile menu and navigation" \
  --screen-size mobile \
  --sync

# With structured output schema
npx runhuman create https://example.com \
  -d "Test checkout and report any bugs" \
  --schema ./test-schema.json

# CI/CD blocking test
npx runhuman create https://staging.app.com \
  -d "Critical: Test payment flow end-to-end" \
  --sync \
  --wait 600
```

**For all options:** `npx runhuman create --help`

---

### status

Check the current status of a test job.

```bash
npx runhuman status <jobId>
```

**Options:**
- `--json` - Output as JSON

**Shows:**
- Current status (pending, claimed, in_progress, completed, failed, timeout)
- Tester name (if claimed)
- URL being tested
- Duration and cost
- Dashboard URL

**Example:**
```bash
npx runhuman status job_abc123

# Output:
# Status: in_progress
# Tester: Sarah M.
# URL: https://example.com
# Duration: 5 minutes
# Dashboard: https://runhuman.com/dashboard/jobs/job_abc123
```

---

### wait

Wait for a job to complete and display results when done.

```bash
npx runhuman wait <jobId> [options]
```

**Options:**
- `--timeout <seconds>` - Maximum wait time in seconds (default: 600)
- `--json` - Output as JSON

**What it does:**
- Polls job status every 10 seconds
- Shows spinner with elapsed time
- Exits when job reaches terminal state (completed, error, timeout, failed)
- Displays results summary on completion

**Exit codes:**
- 0 = Completed successfully
- 5 = Timeout exceeded

**Example:**
```bash
# Wait up to 10 minutes
npx runhuman wait job_abc123 --timeout 600

# Wait with default timeout (10 minutes)
npx runhuman wait job_abc123
```

---

### results

Display detailed results for a completed test job.

```bash
npx runhuman results <jobId> [options]
```

**Options:**
- `--schema-only` - Show only extracted structured data (if schema provided)
- `--raw` - Show raw tester response text
- `--json` - Output as JSON

**Output sections:**
1. Job information (ID, status, URL, timestamps, cost)
2. Structured results (if schema was provided in create)
3. Tester feedback (natural language findings and observations)

**Example:**
```bash
# View all results
npx runhuman results job_abc123

# View only structured data
npx runhuman results job_abc123 --schema-only

# Get raw JSON for parsing
npx runhuman results job_abc123 --json
```

---

### list

List test jobs with optional filtering.

```bash
npx runhuman list [filter] [options]
```

**Arguments:**
- `[filter]` - Status filter: `all|pending|claimed|in_progress|completed|failed|timeout`

**Options:**
- `--project <id>` - Filter by project ID
- `--limit <number>` - Number of results to show (default: 20)
- `--offset <number>` - Pagination offset
- `--format <type>` - Output format: `table|json|compact` (default: table)

**Examples:**
```bash
# List all jobs
npx runhuman list

# List only completed jobs
npx runhuman list completed

# List jobs for specific project
npx runhuman list --project my-project-id --limit 10

# Get JSON output for scripting
npx runhuman list completed --format json
```

---

### delete

Delete a test job permanently.

```bash
npx runhuman delete <jobId> [options]
```

**Options:**
- `--confirm` - Skip confirmation prompt

**What it does:**
- Permanently deletes the job and all associated data
- Cannot be undone
- Prompts for confirmation unless `--confirm` is passed

**Example:**
```bash
# Delete with confirmation prompt
npx runhuman delete job_abc123

# Delete without prompt (CI/CD)
npx runhuman delete job_abc123 --confirm
```

---

### watch

Watch files and automatically create QA test jobs when changes are detected.

```bash
npx runhuman watch [patterns...] [options]
```

**Arguments:**
- `[patterns...]` - Glob patterns to watch (e.g., `src/**/*.js`)

**Options:**
- `--url <url>` - URL to test (required)
- `--template <name>` - Template to use for auto-created tests
- `--description <text>` - Test description template
- `--debounce <ms>` - Debounce delay in milliseconds (default: 1000)
- `--ignore <patterns...>` - Patterns to ignore (e.g., `node_modules/**`)
- `--stop` - Stop existing watch process
- `--status` - Check if watch is running

**What it does:**
- Monitors files for changes using chokidar
- Automatically creates test jobs when files change
- Debounces rapid changes to avoid spam
- Runs as background process

**Configuration:**
Can also be configured in `.runhumanrc`:
```json
{
  "watch": {
    "url": "https://localhost:3000",
    "patterns": ["src/**/*.{js,jsx,ts,tsx}"],
    "ignore": ["node_modules/**", "dist/**"],
    "debounce": 2000
  }
}
```

**Examples:**
```bash
# Watch source files
npx runhuman watch "src/**/*.js" --url https://localhost:3000

# Watch with template
npx runhuman watch "src/**/*" \
  --url https://staging.app.com \
  --template "quick-check"

# Stop watching
npx runhuman watch --stop

# Check watch status
npx runhuman watch --status
```

---

## Project Commands

Projects organize your tests and store default configuration like URLs and GitHub repos.

### projects list

List all available projects.

```bash
npx runhuman projects list [options]
```

**Aliases:** `ls`

**Options:**
- `--json` - Output as JSON

**Shows:**
- Project ID
- Project name
- Default URL (if set)
- GitHub repository (if linked)
- Whether it's your current default

---

### projects create

Create a new project.

```bash
npx runhuman projects create <name> [options]
```

**Aliases:** `new`

**Arguments:**
- `<name>` - Project name (required)

**Options:**
- `--default-url <url>` - Default URL for tests in this project
- `--github-repo <owner/repo>` - Link GitHub repository
- `--json` - Output as JSON

**Example:**
```bash
npx runhuman projects create "My App" \
  --default-url https://myapp.com \
  --github-repo myorg/myapp
```

---

### projects show

Show detailed information about a specific project.

```bash
npx runhuman projects show <projectId> [options]
```

**Aliases:** `info`

**Options:**
- `--json` - Output as JSON

**Shows:**
- Project details
- Default configuration
- Linked resources
- Usage statistics

---

### projects switch

Set a project as the default for CLI commands.

```bash
npx runhuman projects switch <projectId> [options]
```

**Aliases:** `use`

**Options:**
- `--global` - Set as global default (this is the default behavior)

**What it does:**
- Sets the specified project as your default
- Saved to `~/.config/runhuman/config.json`
- All future commands use this project unless overridden with `--project`

**Example:**
```bash
# List projects to find ID
npx runhuman projects list

# Set as default
npx runhuman projects switch adfe4b15...
```

---

### projects update

Update project settings.

```bash
npx runhuman projects update <projectId> [options]
```

**Options:**
- `--name <name>` - Update project name
- `--default-url <url>` - Update default URL
- `--github-repo <owner/repo>` - Update linked GitHub repo
- `--json` - Output as JSON

**Example:**
```bash
npx runhuman projects update adfe4b15... \
  --name "My App (Production)" \
  --default-url https://myapp.com
```

---

### projects delete

Delete a project permanently.

```bash
npx runhuman projects delete <projectId> [options]
```

**Aliases:** `rm`

**Options:**
- `--force` - Skip confirmation prompt

**Warning:** This permanently deletes the project and all associated data.

---

## Configuration Commands

Manage CLI configuration at global and project levels.

### config list

Display all current configuration values.

```bash
npx runhuman config list [options]
```

**Options:**
- `--json` - Output as JSON

**Shows:**
- API key (masked)
- API URL
- Default project
- Default URL
- Other configuration values

**Shows sources:**
- Global config (`~/.config/runhuman/config.json`)
- Project config (`.runhumanrc`)
- Environment variables

---

### config get

Get a specific configuration value.

```bash
npx runhuman config get <key>
```

**Arguments:**
- `<key>` - Configuration key (e.g., `project`, `apiKey`, `defaultUrl`)

**Returns:**
- The value if set
- `(not set)` if not configured

**Example:**
```bash
npx runhuman config get project
# Output: adfe4b15...

npx runhuman config get apiKey
# Output: qa_live_********... (masked)
```

---

### config set

Set a configuration value.

```bash
npx runhuman config set <key> <value> [options]
```

**Arguments:**
- `<key>` - Configuration key
- `<value>` - Value to set

**Options:**
- `--global` - Set in global config (default: project-level)

**Configuration keys:**
- `project` - Default project ID
- `apiKey` - API key
- `apiUrl` - API server URL
- `defaultUrl` - Default test URL
- `defaultDuration` - Default test duration
- `defaultScreenSize` - Default screen size

**Examples:**
```bash
# Set project-level config (writes to .runhumanrc)
npx runhuman config set defaultUrl https://myapp.com

# Set global config
npx runhuman config set project my-project-id --global

# Set API key
npx runhuman config set apiKey qa_live_xxxxxxxxxxxxx --global
```

---

### config reset

Reset configuration to defaults.

```bash
npx runhuman config reset [options]
```

**Options:**
- `--global` - Reset global config only
- `--project` - Reset project config only (.runhumanrc)
- `--all` - Reset both global and project config
- `--force` - Skip confirmation prompt

**Warning:** This action cannot be undone.

**Example:**
```bash
# Reset project config
npx runhuman config reset --project --force

# Reset everything
npx runhuman config reset --all
```

---

## Template Commands

Templates are reusable test configurations that can be applied to multiple test jobs.

### templates list

List all available templates.

```bash
npx runhuman templates list [options]
```

**Options:**
- `--project <id>` - Filter by project
- `--json` - Output as JSON

**Shows:**
- Template ID
- Template name
- Description
- Associated project

---

### templates create

Create a new test template.

```bash
npx runhuman templates create <name> [options]
```

**Arguments:**
- `<name>` - Template name (required)

**Options:**
- `--description <text>` - Template description
- `--duration <minutes>` - Default duration (1-60)
- `--screen-size <preset>` - Default screen size (desktop|laptop|tablet|mobile)
- `--schema <file>` - JSON schema file for structured output
- `--project <id>` - Associate with specific project
- `--json` - Output as JSON

**Example:**
```bash
npx runhuman templates create "Mobile Signup Test" \
  --description "Test signup flow on mobile device" \
  --duration 5 \
  --screen-size mobile \
  --project my-project-id
```

---

### templates show

Show detailed information about a template.

```bash
npx runhuman templates show <templateId> [options]
```

**Options:**
- `--json` - Output as JSON

**Shows:**
- Template configuration
- Default values
- Associated project
- Usage statistics

---

### templates update

Update an existing template.

```bash
npx runhuman templates update <templateId> [options]
```

**Options:**
- `--name <name>` - Update name
- `--description <text>` - Update description
- `--duration <minutes>` - Update default duration
- `--screen-size <preset>` - Update default screen size
- `--schema <file>` - Update JSON schema
- `--json` - Output as JSON

---

### templates delete

Delete a template permanently.

```bash
npx runhuman templates delete <templateId> [options]
```

**Options:**
- `--force` - Skip confirmation prompt

**Warning:** This action cannot be undone.

---

## API Key Commands

Manage API keys for programmatic access and CI/CD.

### keys list

List all API keys for a project.

```bash
npx runhuman keys list [options]
```

**Options:**
- `--project <id>` - Filter by project
- `--show-keys` - Display actual key values (otherwise masked)
- `--json` - Output as JSON

**Shows:**
- Key ID
- Key name
- Key value (masked unless `--show-keys`)
- Created date
- Last used date

**Example:**
```bash
# List keys (masked)
npx runhuman keys list

# Show full keys
npx runhuman keys list --show-keys
```

---

### keys create

Create a new API key.

```bash
npx runhuman keys create <name> [options]
```

**Arguments:**
- `<name>` - Key name/description (required)

**Options:**
- `--project <id>` - Associate with specific project
- `--copy` - Copy key to clipboard after creation
- `--json` - Output as JSON

**Important:** The full key is only shown once at creation. Save it securely.

**Example:**
```bash
npx runhuman keys create "CI/CD Key" --project my-project-id --copy
```

---

### keys show

Show details about a specific API key.

```bash
npx runhuman keys show <keyId> [options]
```

**Options:**
- `--show-key` - Display the actual key value (masked by default)
- `--json` - Output as JSON

**Note:** This command requires API endpoint support. Use `keys list` as an alternative.

---

### keys delete

Delete an API key permanently.

```bash
npx runhuman keys delete <keyId> [options]
```

**Options:**
- `--force` - Skip confirmation prompt

**Warning:** Any systems using this key will lose access immediately.

---

## GitHub Integration Commands

Link GitHub repositories and integrate with issues/PRs.

### github link

Link a GitHub repository to a project.

```bash
npx runhuman github link <repo> [options]
```

**Aliases:** `gh link`

**Arguments:**
- `<repo>` - Repository in format `owner/repo` (required)

**Options:**
- `--project <id>` - Project to link to (required)
- `--json` - Output as JSON

**What it does:**
- Associates GitHub repo with Runhuman project
- Enables issue creation from test findings
- Provides context to testers

**Example:**
```bash
npx runhuman github link myorg/myapp --project my-project-id
```

---

### github repos

List linked GitHub repositories.

```bash
npx runhuman github repos [options]
```

**Aliases:** `gh repos`

**Options:**
- `--project <id>` - Filter by project
- `--json` - Output as JSON

**Shows:**
- Repository name
- Associated project
- Link status

---

### github issues

List GitHub issues for a repository.

```bash
npx runhuman github issues <repo> [options]
```

**Aliases:** `gh issues`

**Arguments:**
- `<repo>` - Repository in format `owner/repo`

**Options:**
- `--state <state>` - Filter by state: `open|closed|all` (default: open)
- `--labels <labels>` - Comma-separated labels to filter by
- `--json` - Output as JSON

**Example:**
```bash
# List open issues
npx runhuman github issues myorg/myapp

# List bugs
npx runhuman github issues myorg/myapp --labels bug,critical
```

---

### github test

Create a test job for a specific GitHub issue.

```bash
npx runhuman github test <issueNumber> [options]
```

**Aliases:** `gh test`

**Arguments:**
- `<issueNumber>` - GitHub issue number (required)

**Options:**
- `--repo <owner/repo>` - Repository (required)
- `--url <url>` - URL to test (required)
- `--template <id>` - Template to use (optional)
- `--sync` - Wait for completion (optional)
- `--json` - Output as JSON

**What it does:**
- Creates a test job using the issue description as context
- Links the job to the GitHub issue
- Can optionally create issue comments with results

**Example:**
```bash
npx runhuman github test 42 \
  --repo myorg/myapp \
  --url https://myapp.com \
  --sync
```

---

### github bulk-test

Create test jobs for multiple GitHub issues at once.

```bash
npx runhuman github bulk-test [options]
```

**Aliases:** `gh bulk-test`

**Options:**
- `--repo <owner/repo>` - Repository (required)
- `--url <url>` - URL to test (required)
- `--labels <labels>` - Comma-separated labels to filter issues
- `--state <state>` - Issue state: `open|closed|all` (default: open)
- `--template <id>` - Template to use for all tests
- `--limit <number>` - Maximum number of issues to test (default: 10)
- `--json` - Output as JSON

**What it does:**
- Finds issues matching criteria
- Creates test jobs for each issue
- Links jobs to their respective issues

**Example:**
```bash
# Test all open bugs
npx runhuman github bulk-test \
  --repo myorg/myapp \
  --url https://myapp.com \
  --labels bug \
  --limit 5
```

---

## Initialization Command

### init

Initialize a new Runhuman project with interactive configuration.

```bash
npx runhuman init [options]
```

**Options:**
- `--name <name>` - Project name (skip prompt)
- `--url <url>` - Default URL (skip prompt)
- `--github-repo <owner/repo>` - GitHub repository (skip prompt)
- `--yes` - Accept all defaults (non-interactive)
- `--json` - Output as JSON

**What it does:**
- Creates `.runhumanrc` configuration file in current directory
- Optionally creates a new project
- Sets up project defaults (URL, duration, screen size, etc.)
- Configures GitHub integration if desired

**Interactive prompts:**
1. Project name
2. Default URL
3. Default duration
4. Default screen size
5. GitHub repository (optional)
6. Create API key? (optional)

**Example:**
```bash
# Interactive setup
npx runhuman init

# Non-interactive with options
npx runhuman init \
  --name "My App" \
  --url https://myapp.com \
  --github-repo myorg/myapp
```

**Creates `.runhumanrc`:**
```json
{
  "project": "project-id",
  "defaultUrl": "https://myapp.com",
  "defaultDuration": 5,
  "defaultScreenSize": "desktop",
  "githubRepo": "myorg/myapp"
}
```

---

## Configuration Files

### .runhumanrc

Project-level configuration file (JSON format).

**Location:** Current directory (project root)

**Example:**
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
- `apiKey` - API key (format: `qa_live_` + 40 hex chars)
- `apiUrl` - API server URL (default: https://runhuman.com)
- `project` - Default project ID
- `defaultUrl` - Default test URL
- `defaultDuration` - Default test duration in minutes (1-60)
- `defaultScreenSize` - Default screen size (desktop|laptop|tablet|mobile)
- `githubRepo` - Linked GitHub repository (owner/repo)

---

### Global Config

**Location:** `~/.config/runhuman/config.json`

**Same format as .runhumanrc**

**Created by:** `runhuman login` and `runhuman config set --global`

---

## Environment Variables

All configuration can be overridden with environment variables:

```bash
RUNHUMAN_API_KEY=qa_live_xxxxxxxxxxxxx
RUNHUMAN_API_URL=https://runhuman.com
RUNHUMAN_PROJECT=my-project-id
RUNHUMAN_DEFAULT_URL=https://example.com
RUNHUMAN_DEFAULT_DURATION=5
RUNHUMAN_DEFAULT_SCREEN_SIZE=desktop
```

**Useful for CI/CD:**
```bash
export RUNHUMAN_API_KEY=${{ secrets.RUNHUMAN_API_KEY }}
npx runhuman create https://staging.app.com -d "Test deployment" --sync
```

---

## Configuration Priority

Configuration is loaded in this order (highest to lowest priority):

1. **CLI flags** - `--api-key`, `--project`, etc.
2. **Environment variables** - `RUNHUMAN_API_KEY`, etc.
3. **Project config** - `.runhumanrc` in current directory
4. **Global config** - `~/.config/runhuman/config.json`
5. **Defaults**

---

## Exit Codes

All commands return standard exit codes:

| Code | Meaning | Commands |
|------|---------|----------|
| 0 | Success | All commands |
| 1 | General error | All commands |
| 2 | Authentication failed | login, create, etc. |
| 3 | Resource not found | show, delete, etc. |
| 4 | Validation error | create (missing fields) |
| 5 | Timeout | wait (exceeded timeout) |

**Usage in scripts:**
```bash
npx runhuman create ... --sync
if [ $? -eq 0 ]; then
  echo "Test passed"
else
  echo "Test failed"
  exit 1
fi
```

---

## Quick Reference

### Most Common Commands (80/20 Rule)

```bash
# Setup (one-time)
npx runhuman login
npx runhuman projects switch <project-id>

# Daily usage
npx runhuman create <url> -d "description"
npx runhuman status <job-id>
npx runhuman results <job-id>

# CI/CD
npx runhuman create <url> -d "..." --sync

# Mobile testing
npx runhuman create <url> -d "..." --screen-size mobile

# Check config
npx runhuman config list
npx runhuman whoami
```

---

## Getting Help

For any command, use the `--help` flag:

```bash
npx runhuman --help                # List all commands
npx runhuman create --help         # Create command options
npx runhuman projects --help       # Projects subcommands
npx runhuman github test --help    # Specific command help
```

The CLI uses Commander.js with comprehensive, always-up-to-date help text.
