# Profiling Guide

The `--profile` flag on `wp dbtk api call` gives you timing, memory, query counts, component attribution, and PHP error capture — all from a single CLI invocation, without touching a browser or adding instrumentation code.

---

## 1. What Profiling Captures

| Category | Fields |
|----------|--------|
| **Timing** | `execution_time_ms` — total wall-clock time for the endpoint |
| **Memory** | `memory.delta_mb` — memory consumed by the request; `memory.peak_mb` — PHP peak usage |
| **Queries** | `queries.total`, `.slow`, `.duplicates`, `.by_type`, `.total_time_ms`, `.by_component`, `.slowest` |
| **PHP errors** | `php_errors` — array of PHP warnings, notices, and errors captured during execution |

All measurements are scoped to the internal dispatch of that single endpoint call. They do not include WP-CLI bootstrap overhead.

---

## 2. Profile Output Format

Full profile output is returned as a `profile` key alongside the normal `status` and `data` fields.

**Example output (when queries are captured)**

```json
{
  "status": 200,
  "data": { "...endpoint response..." },
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

**Field-by-field reference**

| Field | Type | Description |
|-------|------|-------------|
| `execution_time_ms` | float | Total endpoint execution time in milliseconds |
| `memory.delta_mb` | float | Memory used by this request (peak minus baseline), in MB |
| `memory.peak_mb` | float | PHP peak memory usage at the end of the request, in MB |
| `queries.total` | int | Total number of database queries executed |
| `queries.slow` | int | Queries exceeding the slow query threshold (default: 50ms) |
| `queries.duplicates` | int | Number of queries that were run more than once with identical SQL |
| `queries.by_type` | object | Query counts by verb: `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| `queries.total_time_ms` | float | Sum of all query execution times in milliseconds |
| `queries.by_component` | object | Query counts keyed by plugin/theme slug that triggered them |
| `queries.slowest` | array | Top slowest queries, each with `sql`, `time_ms`, and `component` |
| `php_errors` | array | PHP errors, warnings, or notices caught during execution |

---

## 3. Three Profiling Modes

Use the `--profile` flag with an optional mode argument:

```bash
--profile           # Full profile (same as --profile=full)
--profile=full      # All data: timing, memory, queries, PHP errors
--profile=queries   # Query data only
--profile=summary   # Timing + counts only
```

**Mode comparison**

| Field | `full` | `queries` | `summary` |
|-------|:------:|:---------:|:---------:|
| `execution_time_ms` | yes | — | yes |
| `memory` | yes | — | yes |
| `queries.total` | yes | yes | yes |
| `queries.slow` | yes | yes | yes |
| `queries.duplicates` | yes | yes | — |
| `queries.by_type` | yes | yes | — |
| `queries.by_component` | yes | yes | — |
| `queries.slowest` | yes | yes | — |
| `php_errors` | yes | — | — |

Use `summary` for a quick sanity check. Use `queries` when you care about the database only. Use `full` (the default) when investigating.

---

## 4. Workflow: Is My Plugin Slowing Things Down?

Profile an endpoint that your plugin touches, then check `by_component`.

```bash
# Profile a WooCommerce product listing endpoint
wp dbtk api call GET /wc/v3/products --params='{"per_page":10}' --profile
```

In the output, look at `profile.queries.by_component`:

```json
"by_component": {
  "woocommerce": 18,
  "my-plugin": 12,
  "wordpress-core": 5
}
```

If your plugin slug appears with a high query count, those queries are being triggered by your code. Cross-reference with `queries.slowest` to find which specific queries are the worst offenders:

```json
"slowest": [
  {
    "sql": "SELECT * FROM wp_postmeta WHERE post_id IN (1,2,3,...)",
    "time_ms": 34.2,
    "component": "my-plugin"
  }
]
```

This tells you both what the query is and which plugin triggered it. Fix the query (add an index, batch the lookup, cache the result) and re-profile to confirm improvement.

---

## 5. Workflow: Before/After Comparison

Capture a baseline, make your change, then compare.

**Step 1: Capture baseline**

```bash
wp dbtk api call GET /wp/v2/posts --params='{"per_page":20}' --profile > /tmp/before.json
```

**Step 2: Make your change**

Deploy your code change (update a plugin, add an index, tweak a query, etc.).

**Step 3: Capture after**

```bash
wp dbtk api call GET /wp/v2/posts --params='{"per_page":20}' --profile > /tmp/after.json
```

**Step 4: Compare**

```bash
# Quick comparison of key metrics
jq '.profile.execution_time_ms, .profile.queries.total, .profile.queries.slow' /tmp/before.json /tmp/after.json
```

Or diff the full profiles:

```bash
jq '.profile' /tmp/before.json > /tmp/before-profile.json
jq '.profile' /tmp/after.json  > /tmp/after-profile.json
diff /tmp/before-profile.json /tmp/after-profile.json
```

Look for reductions in `execution_time_ms`, `queries.total`, and `queries.slow`. A meaningful improvement shows up as lower counts across all three.

---

## 6. Workflow: Finding Duplicate Queries

The `queries.duplicates` field counts how many queries were executed more than once with identical SQL. This is the N+1 indicator.

```bash
wp dbtk api call GET /wc/v3/orders --params='{"per_page":20}' --profile=queries
```

If `duplicates` is non-zero:

```json
"queries": {
  "total": 62,
  "slow": 0,
  "duplicates": 18,
  ...
}
```

18 duplicate queries on a 20-item list strongly suggests an N+1 problem: code is running the same query once per item in a loop rather than batching.

Switch to `--profile=full` and inspect `queries.slowest` — duplicated queries often appear in the slowest list when they accumulate across many items. Look for queries with `post_id = X` where X varies, or `meta_key = 'some_key'` running per row.

**Fixing N+1 problems**

The typical fix is replacing per-item queries with a single batch query using `IN (...)` or caching results in a static variable. After the fix, `duplicates` should drop to 0 or near 0.

---

## 7. Workflow: Component Attribution

`queries.by_component` maps each plugin/theme slug to the number of queries it triggered during the endpoint call. Component attribution works by inspecting the PHP call stack of each query and identifying the first frame that belongs to a known plugin or theme directory.

**Reading the by_component output**

```json
"by_component": {
  "woocommerce": 18,
  "wordpress-core": 5,
  "my-custom-plugin": 3,
  "some-theme": 1
}
```

This tells you:
- WooCommerce ran 18 queries (expected — it's the endpoint's plugin)
- WordPress core ran 5 queries (normal: options, user, nonce checks)
- Your custom plugin ran 3 queries (worth investigating if unexpected)
- The active theme ran 1 query (check if theme code is hooking into REST responses)

**Tracking down unexpected contributors**

If a plugin you didn't expect shows up, profile the endpoint without that plugin active (if safe to do so) and compare. If its query count drops to 0, that plugin is the source.

You can also use `queries.slowest` filtered by `component` to find which specific queries belong to that plugin.

---

## 8. SAVEQUERIES Requirement

Query capture (everything under `queries.*` except `queries.total` if it falls back to a counter) requires `SAVEQUERIES` to be `true` in WordPress.

**How to enable it**

```bash
wp dbtk query-log on
```

This enables both SAVEQUERIES and enhanced query logging.

**Known caveat**

There is a known issue where `SAVEQUERIES` may already be defined as `false` during WP-CLI bootstrap, before the plugin can override it. When this happens, query data will not be captured for that invocation and query fields will be absent or show zeros — even though `SAVEQUERIES` is technically enabled in the database setting.

**Workaround**

If you see missing query data despite having query logging enabled, try:

1. Confirm `wp dbtk debug status` shows `SAVEQUERIES: ON`
2. Re-run the profile command — the issue is intermittent
3. If it persists, check for other plugins or must-use plugins that define `SAVEQUERIES` early in the bootstrap sequence

Timing and memory data (`execution_time_ms`, `memory`) are always captured regardless of SAVEQUERIES state.
