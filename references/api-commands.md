# `wp dbtk api` Command Reference

Complete reference for all `wp dbtk api` subcommands. Use `wp dbtk api discover` first to populate the schema store with third-party and WordPress core routes — until then, only WPDT endpoints are shown.

---

## `wp dbtk api discover`

Scans all active plugins, themes, and WordPress core for registered REST routes and saves them to the schema store. Run this once before using `list`, `show`, or `search` to see the full picture.

**Syntax**

```
wp dbtk api discover
```

No options.

**Example output**

```
Success: Discovered 312 routes across 14 sources.
```

---

## `wp dbtk api list`

Lists REST routes from the schema store. Shows WPDT endpoints by default until `discover` has been run.

**Syntax**

```
wp dbtk api list [--namespace=<ns>] [--method=<method>] [--source=<slug>] [--annotated-only] [--tag=<tag>] [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--namespace` | string | — | Filter by REST namespace (e.g., `wc/v3`, `wp/v2`) |
| `--method` | string | — | Filter by HTTP method: `GET`, `POST`, `PUT`, `DELETE` |
| `--source` | string | — | Filter by source plugin slug (e.g., `woocommerce`, `wpdebugtoolkit`) |
| `--annotated-only` | flag | — | Only show routes that have saved annotations |
| `--tag` | string | — | Filter by route-level or method-level annotation tag |
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk api list --source=wpdebugtoolkit
```

**Example output**

```
+------------------+-------------------------------------------+---------+-----------------------------+
| Source           | Route                                     | Methods | Description                 |
+------------------+-------------------------------------------+---------+-----------------------------+
| wpdebugtoolkit   | /wpdebugtoolkit/v1/settings               | GET     | Get debug settings          |
| wpdebugtoolkit   | /wpdebugtoolkit/v1/settings               | POST    | Update debug settings       |
| wpdebugtoolkit   | /wpdebugtoolkit/v1/setup-viewer           | POST    | Install standalone viewer   |
| wpdebugtoolkit   | /wpdebugtoolkit/v1/query-logger/record    | POST    | Start query recording       |
| wpdebugtoolkit   | /wpdebugtoolkit/v1/debug-log/stats        | GET     | Get debug log file stats    |
+------------------+-------------------------------------------+---------+-----------------------------+
```

**Example with filter and JSON format**

```bash
wp dbtk api list --namespace=wc/v3 --method=GET --format=json
```

---

## `wp dbtk api show <route>`

Shows full detail for a single route: methods, parameters, auth requirements, and description.

**Syntax**

```
wp dbtk api show <route> [--method=<method>] [--format=<format>]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<route>` | Yes | REST route path, e.g., `/wp/v2/posts` or `/wc/v3/products` |

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--method` | string | — | Show details for a specific HTTP method only (e.g., `POST`) |
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk api show /wc/v3/products
```

**Example output**

```
+------------+--------------------------------------------------------------+
| Field      | Value                                                        |
+------------+--------------------------------------------------------------+
| Route      | /wc/v3/products                                              |
| Methods    | GET, POST                                                    |
| Description| List and create WooCommerce products                         |
| Auth       | wc_rest_check_post_permissions                               |
| Parameters |                                                              |
|   per_page | per_page (integer) - Maximum number of items to return       |
|   page     | page (integer) - Current page of the collection              |
|   search   | search (string) - Limit results to those matching a string   |
|   status   | status (string) - Limit result set to products with this status |
+------------+--------------------------------------------------------------+
```

If the route is not in the schema, run `wp dbtk api discover` first.

---

## `wp dbtk api search <keyword>`

Searches all routes in the schema store by keyword, matching against route paths, descriptions, and parameter names.

**Syntax**

```
wp dbtk api search <keyword> [--source=<slug>] [--annotated-only] [--format=<format>]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<keyword>` | Yes | Search term to match against routes, descriptions, and parameters |

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--source` | string | — | Filter results by source plugin slug |
| `--annotated-only` | flag | — | Only show results with annotations |
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk api search "order"
```

**Example output**

```
+-------------+-------------------------------+--------------+----------------------------------+
| Source      | Route                         | Methods      | Description                      |
+-------------+-------------------------------+--------------+----------------------------------+
| woocommerce | /wc/v3/orders                 | GET, POST    | List and create orders           |
| woocommerce | /wc/v3/orders/(?P<id>[\d]+)   | GET, PUT, DELETE | Get, update, delete an order |
| woocommerce | /wc/v3/orders/batch           | POST         | Batch update orders              |
+-------------+-------------------------------+--------------+----------------------------------+
```

---

## `wp dbtk api call <METHOD> <route>`

Calls any REST endpoint via internal WordPress dispatch (no HTTP round-trip). Authentication is handled automatically using the current WP-CLI user context.

**Syntax**

```
wp dbtk api call <METHOD> <route> [--params=<json>] [--profile[=<mode>]] [--format=<format>]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<METHOD>` | Yes | HTTP method: `GET`, `POST`, `PUT`, `DELETE` |
| `<route>` | Yes | REST route path, e.g., `/wp/v2/posts` |

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--params` | JSON string | — | Request parameters as a JSON object |
| `--profile` | flag or string | — | Enable profiling. Modes: `full` (default when flag is used), `queries`, `summary` |
| `--format` | string | `json` | Output format. Currently only `json` is supported |

**Profiling modes**

| Mode | What's included |
|------|----------------|
| `full` | Everything: timing, memory, full query breakdown, PHP errors |
| `queries` | Query data only: total, slow, duplicates, by_type, by_component, slowest |
| `summary` | Timing and counts only: execution_time_ms, memory, queries.total, queries.slow |

**Example: Basic GET request**

```bash
wp dbtk api call GET /wp/v2/posts --params='{"per_page":2}'
```

**Example output**

```json
{
  "status": 200,
  "data": [
    { "id": 42, "title": { "rendered": "Hello World" }, "status": "publish" },
    { "id": 41, "title": { "rendered": "Sample Post" }, "status": "publish" }
  ]
}
```

**Example: POST request**

```bash
wp dbtk api call POST /wpdebugtoolkit/v1/query-logger/record --params='{"duration":60}'
```

**Example: Full profile**

```bash
wp dbtk api call GET /wc/v3/products --params='{"per_page":5}' --profile
```

**Example output (with profiling)**

```json
{
  "status": 200,
  "data": [ ... ],
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
      "by_type": { "SELECT": 20, "INSERT": 2, "UPDATE": 1 },
      "total_time_ms": 89.3,
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

**Example: Summary profile only**

```bash
wp dbtk api call GET /wp/v2/posts --profile=summary
```

**Example output**

```json
{
  "status": 200,
  "data": [ ... ],
  "profile": {
    "execution_time_ms": 38.1,
    "memory": {
      "delta_mb": 1.1,
      "peak_mb": 12.4
    },
    "queries": {
      "total": 8,
      "slow": 0
    }
  }
}
```

---

## `wp dbtk api edit <route>`

Annotates an endpoint with semantic metadata. Annotations are stored separately from the discovered schema, so they persist across `discover` runs and appear in `list`, `show`, `search`, and `bootstrap` output.

Annotations can be route-level (apply to all methods) or method-level (specific to one HTTP method via `--method`). Provide at least one annotation field per call. Pass an empty string to clear a field.

**Syntax**

```
wp dbtk api edit <route> [--method=<method>] [--description=<text>] [--purpose=<text>] [--safety=<level>] [--auth-note=<text>] [--returns=<text>] [--example-params=<json>] [--tag=<tags>] [--verification=<state>]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<route>` | Yes | REST route path to annotate |

**Options**

| Option | Type | Description |
|--------|------|-------------|
| `--method` | string | HTTP method for method-level annotation (e.g., `POST`). Omit for route-level. |
| `--description` | string | Summary description of the endpoint |
| `--purpose` | string | When and why to use this endpoint |
| `--safety` | string | Safety level: `read-only`, `mutates-data`, `destructive`, `unknown` |
| `--auth-note` | string | Authentication/authorization notes (e.g., required role) |
| `--returns` | string | Summary of what the endpoint returns |
| `--example-params` | JSON string | Example parameters as a JSON object |
| `--tag` | string | Comma-separated tags |
| `--verification` | string | Verification state: `inferred`, `verified-source`, `verified-runtime`, `verified-docs` |

**Example: Route-level summary**

```bash
wp dbtk api edit /elementor/v1/globals --description="Fetch global design settings (colors, fonts)" --safety=read-only
```

**Example: Method-level annotation**

```bash
wp dbtk api edit /wc/v3/orders --method=POST \
  --description="Create a new order" \
  --safety=mutates-data \
  --auth-note="Requires shop_manager or administrator" \
  --returns="Created order object with id and status"
```

**Example: Mark verification and tag**

```bash
wp dbtk api edit /wc/v3/orders --tag=orders,wc --verification=verified-runtime
```

**Example: Clear a field**

```bash
wp dbtk api edit /wc/v3/orders --description="" --safety=""
```

**Example output**

```
Success: Annotation saved for /wc/v3/orders.
```

Use the `--safety` field aggressively. Agents that read annotated schemas treat `mutates-data` and `destructive` as a strong hint to avoid calling the endpoint without explicit user confirmation.

---

## `wp dbtk api export`

Exports annotations for a single source as a JSON pack. Packs are portable across sites and can be checked into version control or shared with a team. Combine with `import` on another site to replicate your annotation library.

**Syntax**

```
wp dbtk api export [--source=<slug>] [--namespace=<ns>] [--template] [--annotated-only] [--format=json]
```

**Options**

| Option | Type | Description |
|--------|------|-------------|
| `--source` | string | Source plugin slug to export (e.g., `woocommerce`). Required if `--namespace` is omitted. |
| `--namespace` | string | REST namespace to identify the source (e.g., `wc/v3`). Required if `--source` is omitted. |
| `--template` | flag | Export with empty semantic fields, ready to be filled in by a human or AI agent |
| `--annotated-only` | flag | Only include routes that already have annotations |
| `--format` | string | Output format. Currently only `json`. |

**Example: Export annotated routes for a plugin**

```bash
wp dbtk api export --source=woocommerce --annotated-only > woocommerce-annotations.json
```

**Example: Generate a template for AI/human completion**

```bash
wp dbtk api export --source=rankmath --template > rankmath-template.json
```

The exported pack contains the source slug, schema version, and per-route annotations. Use `import` on another site to load it.

---

## `wp dbtk api import <file>`

Loads an annotation pack into the local schema store. Supports merging into existing annotations or replacing all annotations for the pack's source.

**Syntax**

```
wp dbtk api import <file> [--mode=<mode>] [--dry-run]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<file>` | Yes | Path to the JSON pack file produced by `wp dbtk api export` |

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--mode` | string | `merge` | Import mode: `merge` (combine with existing annotations) or `replace-source` (replace all annotations for the source) |
| `--dry-run` | flag | — | Validate and report what would change without writing |

**Example: Preview an import**

```bash
wp dbtk api import woocommerce-annotations.json --dry-run
```

**Example: Merge into existing annotations**

```bash
wp dbtk api import woocommerce-annotations.json
```

**Example: Replace everything for the source**

```bash
wp dbtk api import woocommerce-annotations.json --mode=replace-source
```

**Example output**

```
Success: Imported 27 routes via merge mode.
```

---

## `wp dbtk api bootstrap`

Generates a compact API brief for a single plugin source, optimized for loading into a fresh AI agent session before any other API work. The Markdown format is designed to be pasted into a chat, attached to a system prompt, or piped into a file the agent reads at startup.

**Syntax**

```
wp dbtk api bootstrap [--source=<slug>] [--namespace=<ns>] [--annotated-only] [--max-routes=<n>] [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--source` | string | — | Source plugin slug. Required if `--namespace` is omitted. |
| `--namespace` | string | — | REST namespace to identify the source. Required if `--source` is omitted. |
| `--annotated-only` | flag | — | Only include routes that have annotations |
| `--max-routes` | integer | — | Maximum number of routes to include in the brief |
| `--format` | string | `markdown` | Output format: `markdown` or `json` |

**Example: Brief a plugin's full API**

```bash
wp dbtk api bootstrap --source=woocommerce
```

**Example: Brief only what's been annotated, capped at 20 routes**

```bash
wp dbtk api bootstrap --source=rankmath --annotated-only --max-routes=20
```

**Example: JSON for programmatic consumption**

```bash
wp dbtk api bootstrap --source=woocommerce --format=json
```

The brief includes the namespace, route counts, safety level distribution, and per-route summaries with method-level annotations. Run this at the start of any agent session that will work against a specific plugin's API — it's faster and more accurate than asking the agent to rediscover routes from scratch.
