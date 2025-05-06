# Cosmovisor Configuration Guide

This guide explains the key configuration options for Cosmovisor when running a cheqd node. You can configure these settings via:

* Environment variables

* A `config.toml file` under `$DAEMON_HOME/cosmovisor/` (by default) or any other location passed to cosmovisor with `--cosmovisor-config` flag

> **Note:** Environment variables always take precedence over values set in the config file, if the `--cosmovisor-config` flag is not passed.

The cheqd node's [interactive installer](https://raw.githubusercontent.com/cheqd/cheqd-node/refs/heads/main/installer/installer.py) sets most of these parameters for you in both the daemon service configuration file (`cheqd-cosmovisor.service`) and as a system-wide environment variable. It will also create a config.toml file, for consistency purposes. Understanding these settings helps with troubleshooting and advanced setups.

## Configuration Parameters

| Parameter | Default Value | Required | Description | Set by Installer |
|:---------:|:-------------:|:--------:|:------------|:----------------:|
| `DAEMON_HOME`| `/home/cheqd/.cheqdnode` | Yes | Location of the `cosmovisor/` directory. | ✅ |
| `DAEMON_NAME` | `cheqd-noded` | Yes | Name of the node binary. Usually doesn’t need to change. | ✅ |
| `DAEMON_ALLOW_DOWNLOAD_BINARIES` | `true` | No | Allows Cosmovisor to auto-download upgrade binaries. Recommended to be true. | ✅ |
| `DAEMON_DOWNLOAD_MUST_HAVE_CHECKSUM` | `true` | No | Ensures downloaded binaries have checksums. Always true in cheqd upgrade plans. | ✅ |
| `DAEMON_RESTART_AFTER_UPGRADE` | `true` | No | Automatically restarts node after an upgrade. | ✅ |
| `DAEMON_RESTART_DELAY` | `30s` | No | Delay before restart after upgrade. 0s is fine for most setups. | ✅ |
| `DAEMON_SHUTDOWN_GRACE` | `30s` | No | Grace period for clean shutdown. Helps with safe unattended upgrades. | ✅ |
| `DAEMON_POLL_INTERVAL` | `300s` | No | How often to check for upgrade plans. | ❌ |
| `DAEMON_DATA_BACKUP_DIR` | `DAEMON_HOME`| No | Custom directory for pre-upgrade backups. Requires extra storage. | ❌ |
| `UNSAFE_SKIP_BACKUP` | `true` | No | Set to false to enable auto-backups (slower and storage-heavy). | ✅  |
| `DAEMON_PREUPGRADE_MAX_RETRIES` | `0` | No | Max retries for pre-upgrade hook (exit code 31). | ❌ |
| `COSMOVISOR_DISABLE_LOGS` | `false`| No | Disable Cosmovisor logs (not the node logs). | ❌ |
| `COSMOVISOR_COLOR_LOGS` | `true` | No | Enables colored logs for easier readability. | ❌. |
| `COSMOVISOR_TIMEFORMAT_LOGS` | `kitchen` | No | Time format for logs (e.g., 3:04PM). Other formats: rfc3339, unix, etc. | ❌ |
| `COSMOVISOR_CUSTOM_PREUPGRADE` | '' | No | Path to a custom pre-upgrade script. It should be located under `$DAEMON_HOME/cosmovisor/`. | ❌ |
| `COSMOVISOR_DISABLE_RECASE` | `false` | No | Enforces exact case matching for upgrade plan directories. | ❌ |

---

## 📝 Additional Notes

* **Backups**: Enabling backups can use significant disk space and time. Use with caution, especially on non-pruned nodes.
* **Custom Pre-upgrade Scripts**: Use COSMOVISOR_CUSTOM_PREUPGRADE for advanced automation (e.g., state export).
* **Log Time Format**: kitchen is human-readable. See [Go time formats](https://pkg.go.dev/time#pkg-constants) for more options.

## 🔧 Example: systemd Service File

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
Environment="DAEMON_POLL_INTERVAL=300s"
Environment="UNSAFE_SKIP_BACKUP=false"
Environment="DAEMON_RESTART_DELAY=30s"
Environment="DAEMON_DOWNLOAD_MUST_HAVE_CHECKSUM=true"
Environment="DAEMON_SHUTDOWN_GRACE=30s"

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

If you decide to use config.toml file instead, feel free to remove environment variables from daemon service file and add `--cosmovisor-config` to your config file, i.e. `ExecStart=/usr/bin/cosmovisor run start --cosmovisor-config /home/cheqd/.cheqdnode/cosmovisor/config.toml`.

> **⚠️ Important**: If you manually modify this file, the cheqd installer may overwrite your changes. When prompted during future installs, **decline the update** to preserve your custom settings.

## 🛠 Example: config.toml File

```toml
daemon_home = '/home/cheqd/.cheqdnode'
daemon_name = 'cheqd-noded'
daemon_allow_download_binaries = true
daemon_download_must_have_checksum = true
daemon_restart_after_upgrade = true
daemon_restart_delay = '30s'
daemon_shutdown_grace = '30s'
daemon_poll_interval = '300s'
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

> **⚠️ Reminder**: Like the service file, custom config.toml changes can be overwritten by the installer. **Decline updates** if you’ve made manual modifications.

For more details, see the [Cosmovisor documentation](https://docs.cosmos.network/main/tooling/cosmovisor).
