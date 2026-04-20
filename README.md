# WP Debug Toolkit CLI

An agent skill that gives your AI coding assistant CLI-based WordPress performance profiling and debugging.

![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)

---

## What This Is

This is an installable agent skill for AI coding assistants. Once installed, your agent can profile WordPress REST endpoints, discover APIs across plugins and themes, manage debug mode, and call authenticated endpoints — all from the terminal, without touching a browser.

The headline feature is `--profile` on any REST endpoint: one command returns timing, query counts, memory usage, duplicate query detection, and per-plugin attribution. It requires [WP Debug Toolkit Pro](https://wpdebugtoolkit.com) to be installed on the target WordPress site.

---

## Installation

```bash
npx skills add WP-Debug-Toolkit/wpdt-cli
```

Supported agents: **Claude Code**, **Cursor**, **VS Code Copilot**, **Codex**, **Gemini CLI**

---

## What You Get

- Profile any REST endpoint — timing, query counts, memory delta, slow queries, N+1 detection, and per-plugin query attribution
- Discover all REST APIs registered by plugins, themes, and WordPress core
- Persistent endpoint annotations — your agent's notes about cryptic third-party endpoints (safety, auth, return shape) survive across sessions and can be exported as a pack to share with a team or load on another site
- Bootstrap a fresh agent session with a Markdown brief of any plugin's API
- Enable or disable debug mode and query logging from the terminal
- Call any endpoint with automatic authentication (no manual token setup)
- Search and inspect endpoint details
- Auto-detected WP-CLI environment — bridges the gap between your AI agent's terminal and local dev tools like Local by Flywheel, where `wp` works in the site shell but not from external terminals

---

## Quick Example

Profile a WooCommerce endpoint and see exactly where the queries are coming from:

```bash
wp dbtk api call GET /wc/v3/products --params='{"per_page":5}' --profile
```

Output (abridged):

```json
{
  "status": 200,
  "profile": {
    "execution_time_ms": 142.5,
    "memory": {
      "delta_mb": 2.4,
      "peak_mb": 14.2
    },
    "queries": {
      "total": 23,
      "slow": 2,
      "duplicates": 3,
      "by_component": {
        "woocommerce": 18,
        "wordpress-core": 5
      },
      "slowest": [
        {
          "sql": "SELECT * FROM wp_postmeta WHERE post_id IN (...)",
          "time_ms": 34.2,
          "component": "woocommerce"
        }
      ]
    },
    "php_errors": []
  }
}
```

---

## Advanced Workflows

### Record queries while browsing

Start a tagged recording, browse the site manually, stop when done, then analyze the captured queries with filters:

```bash
wp dbtk query-log start --tag=checkout-flow
# Browse the site, test checkout, etc.
wp dbtk query-log stop
wp dbtk query-log read --tag=checkout-flow --slow --duplicates
wp dbtk query-log read --tag=checkout-flow --component=woocommerce --format=json
```

### Compare before and after a code change

Profile an endpoint, make your change, profile again, and compare the results:

```bash
# Before
wp dbtk api call GET /wc/v3/products --params='{"per_page":10}' --profile > /tmp/before.json

# Make your code change...

# After
wp dbtk api call GET /wc/v3/products --params='{"per_page":10}' --profile > /tmp/after.json

# Compare query counts, timing, and duplicates between the two files
```

### Track performance while vibecoding

When you're building features or fixing bugs with an AI assistant, use tagged recordings to catch regressions as you go:

```bash
# Record a baseline before your change
wp dbtk query-log start --tag=before-fix
# Browse the site, test the flow
wp dbtk query-log stop
wp dbtk query-log read --tag=before-fix --summary

# Make your code change, then record again
wp dbtk query-log start --tag=after-fix
# Test the same flow
wp dbtk query-log stop
wp dbtk query-log read --tag=after-fix --summary

# Compare memory usage between the two
wp dbtk query-log read --tag=before-fix --memory
wp dbtk query-log read --tag=after-fix --memory
```

Record before, record after, compare. A simple habit that keeps performance smooth while developing.

### Bootstrap an agent session with a plugin's API

Before doing API work against a specific plugin, hand your agent a brief instead of asking it to rediscover routes from training data:

```bash
wp dbtk api discover                          # Run once per site
wp dbtk api bootstrap --source=woocommerce    # Markdown brief: namespace, routes, safety, annotations
wp dbtk api bootstrap --source=rankmath --annotated-only --max-routes=20
```

Pipe it into a file your agent reads at startup, or paste it into the chat. The brief is faster and more accurate than rediscovery, and it picks up any annotations you (or previous sessions) have saved.

### Annotate cryptic endpoints so the next session knows them

When you (or your agent) figures out what an unfamiliar endpoint does, save it. Annotations persist across `discover` runs and surface in `list`, `show`, `search`, and `bootstrap`:

```bash
# Method-level annotation with safety classification
wp dbtk api edit /wc/v3/orders --method=POST \
  --description="Create a new order" \
  --safety=mutates-data \
  --auth-note="Requires shop_manager or administrator" \
  --returns="Created order object with id and status"
```

Valid `--safety` values: `read-only`, `mutates-data`, `destructive`, `unknown`. Agents that read annotated schemas treat `mutates-data` and `destructive` as a strong hint to require explicit user confirmation before calling.

### Share annotations across sites

Annotations are stored as a portable JSON pack:

```bash
# On the source site
wp dbtk api export --source=woocommerce --annotated-only > woocommerce-annotations.json

# On another site
wp dbtk api import woocommerce-annotations.json --dry-run    # Preview
wp dbtk api import woocommerce-annotations.json              # Merge
```

Use `--mode=replace-source` to wipe and replace all annotations for a source. Use `--template` on export to generate empty fields for batch annotation by a human or AI.

### Investigate a slow page

Profile the endpoint behind a slow page, find the slowest queries, and check for N+1 patterns:

```bash
# Profile the endpoint
wp dbtk api call GET /wp/v2/posts --params='{"per_page":20}' --profile

# Check the profile output for:
#   - queries.slow — queries exceeding the slow threshold
#   - queries.duplicates — repeated queries (N+1 problems)
#   - queries.by_component — which plugin is responsible
#   - queries.slowest — the top offenders with full SQL

# If you see duplicates, drill into the query log for details
wp dbtk query-log read --slow --duplicates --format=json
```

---

## Supported Environments

| Environment | Status |
|---|---|
| Standard WP-CLI (in PATH) | Supported |
| Local by Flywheel — macOS (Apple Silicon) | Supported |
| Local by Flywheel — macOS (Intel) | Supported |
| MAMP | Coming soon |
| Docker / Docker Compose | Coming soon |
| DDEV | Coming soon |
| Lando | Coming soon |
| Local by Flywheel — Linux | Coming soon |

PRs welcome — please test on your environment before submitting.

---

## Requirements

- [WP Debug Toolkit Pro](https://wpdebugtoolkit.com) installed and active on the WordPress site
- Site must be running (start it in Local by Flywheel, DDEV, Docker, etc. before running commands)
- Query logging features require a Pro license; basic profiling works for all users

---

## Documentation

Full documentation and plugin download: [wpdebugtoolkit.com](https://wpdebugtoolkit.com)

---

## Manual Usage

The wrapper script can also be used directly without installing the skill:

```bash
bash path/to/scripts/wp dbtk api list
bash path/to/scripts/wp dbtk api call GET /wp/v2/posts --profile
```

This is useful for one-off debugging or for environments where the skill manager is not available.

---

## Contributing

Environment support contributions are welcome. If you get the wrapper working on MAMP, Docker, DDEV, Lando, or another environment, please open a PR with your changes and test results.

[Open an issue](https://github.com/WP-Debug-Toolkit/wpdt-cli/issues) to report bugs or request new environments.

---

## License

MIT
