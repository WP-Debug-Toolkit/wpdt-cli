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
- Enable or disable debug mode and query logging from the terminal
- Call any endpoint with automatic authentication (no manual token setup)
- Search and inspect endpoint details
- Auto-detected WP-CLI environment — no manual configuration required for Local by Flywheel

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
- Site must be running (if using Local by Flywheel, start it in the Local app first)
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
