# Cosmovisor Configuration Guide

This guide explains the key Cosmovisor configuration parameters recommended for cheqd node operators. These settings can be applied as environment variables, in a `config.toml` file under `$DAEMON_HOME/cosmovisor/`, or via a custom path using the `--cosmovisor-config-file` flag at startup.

> **Note:** Environment variables always take precedence over values set in the config file. The interactive installer for cheqd nodes sets most of these parameters for you in both the daemon service configuration file (`cheqd-cosmovisor.service`) and as a system-wide environment variable. Understanding these settings helps with troubleshooting and advanced setups.

---

## Configuration Parameters

| Parameter | Default Value | Required | Description | Set by Installer | Comments/Recommendations |
|:---------:|:-------------:|:--------:|:------------|:----------------:|:-------------------------|
| `DAEMON_HOME`| `/home/cheqd/.cheqdnode` | Yes | Location of the `cosmovisor/` directory. | Yes | Unless you installed your cheqd node at different location, you should stick to default value. |
| `DAEMON_NAME` | `cheqd-noded` | Yes | Name of the node binary. | Yes | For most users, the default value should be fine. |
| `DAEMON_ALLOW_DOWNLOAD_BINARIES` | `true` | No | Enable/disable auto-download of upgrade binaries. | Yes | Set to `true` for smoother, unattended upgrades. |
| `DAEMON_DOWNLOAD_MUST_HAVE_CHECKSUM` | `true` | No | Require binary checksums in upgrade plans. | Yes | By default, we include checksums in our upgrade plans. |
| `DAEMON_RESTART_AFTER_UPGRADE` | `true` | No | Automatically restart after upgrade. | Yes | Leave the default value in case you want fully-automated upgrades. |
| `DAEMON_RESTART_DELAY` | `30s` | No | Delay (in seconds) between upgrade and restart. | Yes | `0` is fine for most setups. |
| `DAEMON_SHUTDOWN_GRACE` | `30s` | No | Grace period (in seconds) for shutdown to allow cleanup before force kill. | Yes | For safer undattended upgrades. |
| `DAEMON_POLL_INTERVAL` | `300ms` | No | How often to poll for upgrade plans (locally - looking for upgrade-info.json file). | No | Default is frequent; `60s` is often sufficient. |
| `DAEMON_DATA_BACKUP_DIR` | `DAEMON_HOME`| No | Custom backup directory. | No | Set if you want to enable backups at specific locations. Note that this will require a lot of additional storage, since it will backup whole data directory before upgrade is attempted. |
| `UNSAFE_SKIP_BACKUP` | `true` | No | Skip backup before upgrade. | Yes  | Set to false to enable automatic backups before each upgrade. Note that this will take a lot of time and storage, especially on bigger, non-pruned nodes. |
| `DAEMON_PREUPGRADE_MAX_RETRIES` | `0` | No | Max retries for pre-upgrade handler after exit status 31. | No | If not changed, the daemon will retry upgrades until succeeds or gets stopped. |
| `COSMOVISOR_DISABLE_LOGS` | `false`| No | Disable Cosmovisor logs (not the node logs). | No | For most users, the default value should be fine. |
| `COSMOVISOR_COLOR_LOGS` | `true` | No | Enable colored Cosmovisor logs. | No. | For most users, the default value should be fine. |
| `COSMOVISOR_TIMEFORMAT_LOGS` | `kitchen` | No | Timestamp format for logs. | No | `kitchen` = `3:04PM`; other options: `ansic`, `unix`, `ruby`, `rfc822`, `rfc3339`, `rfc3339nano`.  For most users, the default value should be fine. |
| `COSMOVISOR_CUSTOM_PREUPGRADE` | '' | No | Path to script you want to run before upgrade (`$DAEMON_HOME/cosmovisor/$COSMOVISOR_CUSTOM_PREUPGRADE`). | No | Use this for setting up some custom pre-upgrade actions (like backups or state exports) |
| `COSMOVISOR_DISABLE_RECASE` | `false` | No | If `true`, upgrade directory must match plan name exactly (case-sensitive). | No | For most users, the default value should be fine. |

---

## Additional Notes

- **Backups:** Cosmovisor can back up your node data before upgrades. The backup size depends on your node's data directory. Ensure you have enough disk space.
- **Pre-upgrade Handler:** The `DAEMON_PREUPGRADE_MAX_RETRIES` parameter is for advanced use cases where the Cosmos SDK app implements a pre-upgrade handler.
- **Log Time Formats:** The `COSMOVISOR_TIMEFORMAT_LOGS` parameter supports several formats. `kitchen` is a human-readable time (e.g., `3:04PM`). For more, see [Go time formats](https://pkg.go.dev/time#pkg-constants).

## Example: Modifying the Cosmovisor Systemd Service File

To set Cosmovisor parameters at the service level, you can edit the systemd service file, typically located at `/usr/lib/systemd/system/cheqd-cosmovisor.service`. Here is an example with custom values:

```ini
[Unit]
Description=Service for running cheqd-noded daemon
After=network.target
Documentation=https://docs.cheqd.io/node

[Service]
Environment="DAEMON_HOME=/home/cheqd/.cheqdnode"
Environment="DAEMON_NAME=cheqd-noded"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_POLL_INTERVAL=30s"
Environment="UNSAFE_SKIP_BACKUP=false"
Environment="DAEMON_RESTART_DELAY=30s"
ENVIRONMENT="DAEMON_DOWNLOAD_MUST_HAVE_CHECKSUM=true"
ENVIRONMENT="DAEMON_SHUTDOWN_GRACE=30s"

Type=simple
User=cheqd
ExecStart=/usr/bin/cosmovisor run start
Restart=on-failure
RestartSec=30
StartLimitBurst=5
StartLimitInterval=60
TimeoutSec=120
StandardOutput=journal
StandardError=journal
SyslogIdentifier=cosmovisor
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

> **Important:** If you manually modify the service file, ensure these changes are not overwritten by the interactive installer at future executions. When prompted by the installer to update the cosmovisor daemon file, **decline** to preserve your custom settings.

---

## Example: Cosmovisor config.toml file

```toml
daemon_home = '/home/cheqd/.cheqdnode'
daemon_name = 'cheqd-noded'
daemon_allow_download_binaries = true
daemon_download_must_have_checksum = true
daemon_restart_after_upgrade = true
daemon_restart_delay = 30s
daemon_shutdown_grace = 30s
daemon_poll_interval = 300000000000
unsafe_skip_backup = true
daemon_data_backup_dir = '/home/cheqd/.cheqdnode'
daemon_preupgrade_max_retries = 0
daemon_grpc_address = 'localhost:9090'
cosmovisor_disable_logs = false
cosmovisor_color_logs = true
cosmovisor_timeformat_logs = 'kitchen'
cosmovisor_custom_preupgrade = ''
cosmovisor_disable_recase = false
```

> **Important:** If you manually modify the config.toml file, ensure these changes are not overwritten by the interactive installer at future executions. When prompted by the installer to update the cosmovisor config.toml file, **decline** to preserve your custom settings.

For more details, see the [Cosmovisor documentation](https://docs.cosmos.network/main/tooling/cosmovisor).
