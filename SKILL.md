---
name: wpdt-cli
description: "Use when debugging WordPress performance, profiling REST endpoints, investigating slow queries, managing debug mode, or working with WP Debug Toolkit. Provides CLI commands for endpoint profiling, API discovery, query analysis, and debug configuration."
---

# WP Debug Toolkit CLI

WP Debug Toolkit Pro adds CLI commands for debugging, profiling, and interacting with WordPress REST APIs from the terminal. The headline feature is `--profile` on any REST endpoint — one command gives you full query analysis, timing, memory usage, and component attribution.

## Running WP-CLI

**Always use the wrapper script bundled with this skill instead of bare `wp`:**

```bash
WP="bash ${CLAUDE_SKILL_DIR}/scripts/wp"
```

The wrapper auto-detects your environment (Local by Flywheel, standard WP-CLI, etc.) and handles PHP binary paths, MySQL sockets, and WP-CLI location automatically.

**Never run bare `wp` commands.** They will fail in most local development environments.

> **Note for non-Claude agents:** If `${CLAUDE_SKILL_DIR}` doesn't resolve in your environment, use the path to this skill's directory instead (e.g., `.claude/skills/wpdt-cli/scripts/wp` or `.cursor/skills/wpdt-cli/scripts/wp`).

## Key Workflows

### Profile any REST endpoint

Check performance of any endpoint — yours or third-party:

```bash
$WP dbtk api call GET /wc/v3/products --params='{"per_page":5}' --profile
```

The profile output includes:
- **execution_time_ms** — total time for the endpoint
- **memory.delta_mb / peak_mb** — memory consumption
- **queries.total** — number of database queries executed
- **queries.slow** — queries exceeding the slow threshold (50ms)
- **queries.duplicates** — repeated queries (N+1 problems)
- **queries.by_type** — breakdown by SELECT, INSERT, UPDATE, DELETE
- **queries.by_component** — which plugin/theme triggered each query
- **queries.slowest** — the top 5 slowest queries with SQL and component
- **php_errors** — any PHP errors/warnings captured during execution

### Measure your code's impact

Before your change:
```bash
$WP dbtk api call GET /wp/v2/posts --profile > /tmp/before.json
```

Make your change, then:
```bash
$WP dbtk api call GET /wp/v2/posts --profile > /tmp/after.json
```

Compare query counts, timing, and duplicates between the two.

### Check if your plugin adds overhead

Profile an endpoint and check the `by_component` field:

```bash
$WP dbtk api call GET /wc/v3/products --profile
```

The `profile.queries.by_component` object shows exactly how many queries each plugin contributes. If your plugin is listed with a high count, investigate those queries.

### Discover all REST APIs on a site

```bash
$WP dbtk api discover      # Scan all plugins/themes/core for REST routes
$WP dbtk api list           # Show all discovered routes
$WP dbtk api list --source=woocommerce  # Filter by plugin
$WP dbtk api search "order" # Find endpoints by keyword
$WP dbtk api show /wc/v3/products  # Full detail for a specific route
```

### Enable/disable debug mode

```bash
$WP dbtk debug on          # Enable WP_DEBUG + WP_DEBUG_LOG
$WP dbtk debug on --display # Also enable WP_DEBUG_DISPLAY
$WP dbtk debug off         # Disable all debug constants
$WP dbtk debug status      # Show current settings
```

### Call any REST endpoint

```bash
# GET request
$WP dbtk api call GET /wp/v2/posts --params='{"per_page":2}'

# POST request
$WP dbtk api call POST /wpdebugtoolkit/v1/query-logger/record --params='{"duration":60}'

# With profiling (three modes)
$WP dbtk api call GET /wp/v2/posts --profile          # Full profile
$WP dbtk api call GET /wp/v2/posts --profile=queries   # Query data only
$WP dbtk api call GET /wp/v2/posts --profile=summary   # Just timing + counts
```

### Manage query logging

```bash
$WP dbtk query-log on      # Enable query logging
$WP dbtk query-log stats   # Show log statistics
$WP dbtk query-log clear   # Clear the log
$WP dbtk query-log off     # Disable query logging
```

### Annotate third-party endpoints

When you figure out what a cryptic endpoint does, save it:

```bash
$WP dbtk api edit /elementor/v1/globals --description="Fetch global design settings (colors, fonts)"
```

Annotations persist across discovery runs.

## Quick Command Reference

| Command | Purpose |
|---------|---------|
| `dbtk api list` | List routes (--namespace, --method, --source, --format) |
| `dbtk api discover` | Scan all registered REST routes |
| `dbtk api call <METHOD> <route>` | Call any endpoint (--params, --profile, --format) |
| `dbtk api show <route>` | Show full detail for a route |
| `dbtk api search <keyword>` | Search routes by keyword |
| `dbtk api edit <route>` | Annotate an endpoint (--description) |
| `dbtk debug on/off/status` | Control WP_DEBUG, WP_DEBUG_LOG, WP_DEBUG_DISPLAY |
| `dbtk log clear/stats` | Manage debug.log file |
| `dbtk query-log on/off/clear/stats` | Control database query logging |
| `dbtk viewer setup/remove/status` | Manage standalone log viewer |
| `dbtk license activate/deactivate/status` | License management |

## Detailed References

- For complete `wp dbtk api` command documentation, see [references/api-commands.md](references/api-commands.md)
- For debug, log, query-log, viewer, and license commands, see [references/debug-commands.md](references/debug-commands.md)
- For profiling workflows with real output examples, see [references/profiling-guide.md](references/profiling-guide.md)

## Requirements

- **WP Debug Toolkit Pro** plugin must be installed and active on the WordPress site
- The site must be running (if using Local by Flywheel, start it in the Local app first)
- Query logging features require a Pro license (basic profiling works for all users)

## Troubleshooting

Run the probe command to see what environment was detected:

```bash
bash ${CLAUDE_SKILL_DIR}/scripts/wp --probe
```

Common issues:
- **"MySQL socket not found"** — The site isn't running. Start it in Local by Flywheel.
- **"Could not find WordPress root"** — Run from within a WordPress installation directory.
- **"Command not found: dbtk"** — WP Debug Toolkit plugin isn't installed or activated.
- **"Error establishing a database connection"** — Don't use bare `wp`. Use the wrapper script.
