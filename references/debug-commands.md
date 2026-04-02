# Debug, Log, Query-Log, Viewer, and License Commands

Complete reference for all `wp dbtk` commands except `api`. Covers debug mode, the debug log, query logging, the standalone viewer, and license management.

---

## `wp dbtk debug`

Controls WordPress debug constants in `wp-config.php`.

### `wp dbtk debug on`

Enables `WP_DEBUG` and `WP_DEBUG_LOG`. Optionally also enables `WP_DEBUG_DISPLAY`.

**Syntax**

```
wp dbtk debug on [--display]
```

**Options**

| Option | Type | Description |
|--------|------|-------------|
| `--display` | flag | Also enable `WP_DEBUG_DISPLAY` (shows PHP errors on screen). Not recommended for production. |

**Examples**

```bash
# Enable WP_DEBUG + WP_DEBUG_LOG
wp dbtk debug on

# Also show errors on screen
wp dbtk debug on --display
```

**Example output**

```
Success: Debugging enabled (WP_DEBUG + WP_DEBUG_LOG).
```

With `--display`:

```
Success: Debugging enabled (WP_DEBUG + WP_DEBUG_LOG + WP_DEBUG_DISPLAY).
```

---

### `wp dbtk debug off`

Disables all debug constants: `WP_DEBUG`, `WP_DEBUG_LOG`, and `WP_DEBUG_DISPLAY`.

**Syntax**

```
wp dbtk debug off
```

No options.

**Example output**

```
Success: Debugging disabled.
```

---

### `wp dbtk debug status`

Shows the current state of all debug-related constants and settings.

**Syntax**

```
wp dbtk debug status [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk debug status
```

**Example output**

```
+------------------+-------------------------------------------+
| Setting          | Value                                     |
+------------------+-------------------------------------------+
| WP_DEBUG         | ON                                        |
| WP_DEBUG_LOG     | ON                                        |
| WP_DEBUG_DISPLAY | OFF                                       |
| SAVEQUERIES      | OFF                                       |
| Enhanced logging | OFF                                       |
| Log path         | /var/www/html/wp-content/debug.log        |
+------------------+-------------------------------------------+
```

---

## `wp dbtk log`

Manages the `debug.log` file.

### `wp dbtk log clear`

Empties the debug log file. Prompts for confirmation unless `--yes` is passed.

**Syntax**

```
wp dbtk log clear [--yes]
```

**Options**

| Option | Type | Description |
|--------|------|-------------|
| `--yes` | flag | Skip the confirmation prompt |

**Examples**

```bash
wp dbtk log clear
wp dbtk log clear --yes
```

**Example output**

```
Are you sure you want to clear the debug log? [y/n] y
Success: Debug log cleared.
```

---

### `wp dbtk log stats`

Shows file size, path, and last-modified time for the debug log.

**Syntax**

```
wp dbtk log stats [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk log stats
```

**Example output**

```
+----------+-------------------------------------------+
| Field    | Value                                     |
+----------+-------------------------------------------+
| Path     | /var/www/html/wp-content/debug.log        |
| Exists   | Yes                                       |
| Size     | 2.4 MB                                    |
| Modified | 2026-04-02 14:32:01                       |
+----------+-------------------------------------------+
```

---

## `wp dbtk query-log`

Controls database query logging. Requires the Query Monitor module (Pro license).

### `wp dbtk query-log on`

Enables SAVEQUERIES and enhanced query logging.

**Syntax**

```
wp dbtk query-log on
```

No options.

**Example output**

```
Success: Query logging enabled.
```

> **Note:** If enhanced logging could not fully initialize for the current WP-CLI request (e.g., SAVEQUERIES was already bootstrapped as false), a warning is shown instead of a success. Query logging is still enabled in the database and will take effect on the next full WordPress request.

---

### `wp dbtk query-log off`

Disables query logging.

**Syntax**

```
wp dbtk query-log off
```

No options.

**Example output**

```
Success: Query logging disabled.
```

---

### `wp dbtk query-log clear`

Clears the query log file. Optionally clears all rotated log files. Prompts for confirmation unless `--yes` is passed.

**Syntax**

```
wp dbtk query-log clear [--all] [--yes]
```

**Options**

| Option | Type | Description |
|--------|------|-------------|
| `--all` | flag | Clear all rotated query log files in addition to the current log |
| `--yes` | flag | Skip the confirmation prompt |

**Examples**

```bash
# Clear current log (with confirmation)
wp dbtk query-log clear

# Clear all rotated logs without prompting
wp dbtk query-log clear --all --yes
```

**Example output**

```
Are you sure you want to clear the query log? [y/n] y
Success: Query log cleared.
```

---

### `wp dbtk query-log stats`

Shows query logging status, configuration, and log file statistics.

**Syntax**

```
wp dbtk query-log stats [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk query-log stats
```

**Example output**

```
+------------------+-------------------------------------------+
| Field            | Value                                     |
+------------------+-------------------------------------------+
| Enabled          | Yes                                       |
| Threshold        | 0.05s                                     |
| Log file exists  | Yes                                       |
| Log path         | /var/www/html/wp-content/query-log.json   |
| Log size         | 512 KB                                    |
| Total entries    | 1847                                      |
+------------------+-------------------------------------------+
```

---

## `wp dbtk viewer`

Manages the standalone log viewer — an independent React app that runs even when WordPress is broken.

### `wp dbtk viewer setup`

Installs the standalone viewer to the web root. Requires a Pro license with the Viewer or Query Monitor module.

**Syntax**

```
wp dbtk viewer setup --password=<password>
```

**Options**

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `--password` | string | Yes | Password for viewer authentication. Minimum 8 characters. |

**Example command**

```bash
wp dbtk viewer setup --password=MySecurePass123
```

**Example output**

```
Setting up viewer...
Success: Viewer installed at: https://example.com/wpdebugtoolkit/
```

If no valid license is found:

```
Error: No viewer module licensed. Activate a license first: wp dbtk license activate <key>
```

---

### `wp dbtk viewer remove`

Removes the standalone viewer from the web root. Prompts for confirmation unless `--yes` is passed.

**Syntax**

```
wp dbtk viewer remove [--yes]
```

**Options**

| Option | Type | Description |
|--------|------|-------------|
| `--yes` | flag | Skip the confirmation prompt |

**Examples**

```bash
wp dbtk viewer remove
wp dbtk viewer remove --yes
```

**Example output**

```
Are you sure you want to remove the standalone viewer? [y/n] y
Removing viewer...
Success: Viewer removed.
```

If the viewer is not installed:

```
Warning: Viewer is not currently installed.
```

---

### `wp dbtk viewer status`

Shows whether the viewer is installed, its URL, and password protection state.

**Syntax**

```
wp dbtk viewer status [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk viewer status
```

**Example output (viewer installed)**

```
+---------------------+---------------------------------------------+
| Field               | Value                                       |
+---------------------+---------------------------------------------+
| Installed           | Yes                                         |
| URL directory       | wpdebugtoolkit                              |
| Password protection | Enabled                                     |
| Full URL            | https://example.com/wpdebugtoolkit/         |
+---------------------+---------------------------------------------+
```

**Example output (viewer not installed)**

```
+---------------------+---------------------------------------------+
| Field               | Value                                       |
+---------------------+---------------------------------------------+
| Installed           | No                                          |
| URL directory       | wpdebugtoolkit                              |
| Password protection | Disabled                                    |
+---------------------+---------------------------------------------+
```

---

## `wp dbtk license`

Manages the WP Debug Toolkit Pro license key.

### `wp dbtk license activate <key>`

Activates a license key. Contacts the license server and stores the result locally.

**Syntax**

```
wp dbtk license activate <key>
```

**Arguments**

| Argument | Required | Description |
|----------|----------|-------------|
| `<key>` | Yes | The license key to activate (e.g., `XXXX-XXXX-XXXX-XXXX`) |

**Example command**

```bash
wp dbtk license activate XXXX-XXXX-XXXX-XXXX
```

**Example output**

```
Activating license...
Success: License activated. Tier: Pro. Modules: viewer, query, email.
```

On failure:

```
Error: License key is invalid or has reached its activation limit.
```

---

### `wp dbtk license deactivate`

Deactivates the currently active license. Contacts the license server to free up the activation slot.

**Syntax**

```
wp dbtk license deactivate
```

No options.

**Example output**

```
Deactivating license...
Success: License deactivated successfully.
```

If no license is active:

```
Error: No active license found.
```

---

### `wp dbtk license status`

Shows the current license state: active/inactive, tier, plan, expiry, and licensed modules.

**Syntax**

```
wp dbtk license status [--format=<format>]
```

**Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--format` | string | `table` | Output format: `table` or `json` |

**Example command**

```bash
wp dbtk license status
```

**Example output (active license)**

```
+---------+----------------------------------+
| Field   | Value                            |
+---------+----------------------------------+
| Status  | Active                           |
| Tier    | Pro                              |
| Plan    | Developer License                |
| Expires | Never                            |
| Modules | viewer, query, email             |
+---------+----------------------------------+
```

**Example output (no license)**

```
+---------+--------+
| Field   | Value  |
+---------+--------+
| Status  | Inactive |
| Tier    | N/A    |
+---------+--------+
```
