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
wp dbtk api list [--namespace=<ns>] [--method=<method>] [--source=<slug>] [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--namespace` | string | — | Filter by REST namespace (e.g., `wc/v3`, `wp/v2`) |
| `--method` | string | — | Filter by HTTP method: `GET`, `POST`, `PUT`, `DELETE` |
| `--source` | string | — | Filter by source plugin slug (e.g., `woocommerce`, `wpdebugtoolkit`) |
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
wp dbtk api show <route> [--format=<format>]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<route>` | Yes | REST route path, e.g., `/wp/v2/posts` or `/wc/v3/products` |

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
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
wp dbtk api search <keyword> [--format=<format>]
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<keyword>` | Yes | Search term to match against routes, descriptions, and parameters |

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
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

> **Note on query capture:** Query analysis (total, slow, duplicates, by_component) requires SAVEQUERIES to be enabled. There is a known issue where SAVEQUERIES may already be defined as `false` during WP-CLI bootstrap, preventing query capture for that request. When query capture works, the full query breakdown is returned. If queries are not captured, query fields will be absent or zero.

---

## `wp dbtk api edit <route>`

Annotates an endpoint with a custom description. Annotations persist across `discover` runs and appear in `list` and `show` output.

**Syntax**

```
wp dbtk api edit <route> --description=<text>
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<route>` | Yes | REST route path to annotate |

**Options**

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `--description` | string | Yes | Description text to save for this route |

**Example command**

```bash
wp dbtk api edit /elementor/v1/globals --description="Fetch global design settings (colors, fonts)"
```

**Example output**

```
Success: Annotation saved for /elementor/v1/globals.
```

Use this when you figure out what a cryptic third-party endpoint does. The note shows up in `list` and `show` output on future runs.
