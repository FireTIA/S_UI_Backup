# S-UI Backup

**Language:** English | [Русский](README-ru.md)

Personal backup / survival archive of **S-UI**.

The original upstream repository `alireza0/s-ui` became unavailable for me and started returning `404` for the repository, releases, raw install script and the author profile. This repository is kept as a personal backup of a known working S-UI installation.

This is **not an official S-UI repository**.

## Repository

```text
https://github.com/FireTIA/S_UI_Backup
```

## Status

Archived working installation:

```text
Panel: S-UI
Core: sing-box based
Backup date: 2026-05-11
Original install path: /usr/local/s-ui
Binary name: sui
Management command: s-ui
Systemd service: s-ui.service
```

## Repository layout

Expected structure:

```text
S_UI_Backup/
├── README.md
├── checksums.md5
├── checksums.hash
├── checksums.sha1
├── checksums.sha256
├── checksums.sha512
└── root/
    └── s-ui-survival/
        ├── file-info.txt
        ├── s-ui-command
        ├── s-ui.service
        ├── s-ui.sh
        ├── sui.bin
        ├── s-ui-systemd-cat.txt
        ├── s-ui-status.txt
        ├── tree-lite.txt
        └── s-ui-full-2026-05-11/
            └── usr/
                └── local/
                    └── s-ui/
                        ├── db/
                        ├── s-ui.service
                        ├── s-ui.sh
                        └── sui
```

The `db/` directory is intentionally kept without the real database.

## Important security warning

Do **not** upload a real `s-ui.db` to a public GitHub repository.

A real S-UI database may contain:

- admin panel settings
- users
- UUIDs
- Reality keys
- subscription data
- domains
- ports
- traffic limits
- other sensitive server configuration

This public backup should not contain:

```text
s-ui.db
s-ui.db-wal
s-ui.db-shm
*.sqlite
*.sqlite3
```

Recommended `.gitignore`:

```gitignore
*.db
*.db-wal
*.db-shm
*.sqlite
*.sqlite3
*.log
```

## Install / restore on a new VPS

Target system: Ubuntu/Debian with systemd.

Install required tools:

```bash
sudo apt update
sudo apt install -y git sqlite3 ca-certificates
```

Clone this repository:

```bash
git clone https://github.com/FireTIA/S_UI_Backup.git
cd S_UI_Backup/root/s-ui-survival
```

Stop old S-UI service if it exists:

```bash
sudo systemctl stop s-ui 2>/dev/null || true
```

Create install directories:

```bash
sudo mkdir -p /usr/local/s-ui/db
```

Copy S-UI files:

```bash
sudo cp -f sui.bin /usr/local/s-ui/sui
sudo cp -f s-ui.sh /usr/local/s-ui/s-ui.sh
sudo cp -f s-ui.service /usr/local/s-ui/s-ui.service
```

Fix permissions:

```bash
sudo chmod +x /usr/local/s-ui/sui
sudo chmod +x /usr/local/s-ui/s-ui.sh
```

Install the systemd service:

```bash
sudo cp -f /usr/local/s-ui/s-ui.service /etc/systemd/system/s-ui.service
```

Install the `s-ui` management command:

```bash
sudo cp -f s-ui-command /usr/bin/s-ui
sudo chmod +x /usr/bin/s-ui
```

Reload systemd and start S-UI:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now s-ui
```

Check service status:

```bash
sudo systemctl status s-ui --no-pager
```

Open the S-UI management menu:

```bash
sudo s-ui
```

## Alternative restore from the archived folder

If you prefer to restore from the archived directory tree instead of loose files:

```bash
cd S_UI_Backup/root/s-ui-survival
```

Copy the archived install tree:

```bash
sudo mkdir -p /usr/local/s-ui
sudo cp -a s-ui-full-2026-05-11/usr/local/s-ui/. /usr/local/s-ui/
```

Fix permissions:

```bash
sudo chmod +x /usr/local/s-ui/sui
sudo chmod +x /usr/local/s-ui/s-ui.sh
```

Install service and command:

```bash
sudo cp -f /usr/local/s-ui/s-ui.service /etc/systemd/system/s-ui.service
sudo cp -f s-ui-command /usr/bin/s-ui
sudo chmod +x /usr/bin/s-ui
```

Start S-UI:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now s-ui
sudo s-ui
```

## Restore a private database backup

Only do this if you have your own private `s-ui.db` backup.

Do **not** use a database from an untrusted source.

Stop S-UI:

```bash
sudo systemctl stop s-ui
```

Remove old database files:

```bash
sudo rm -f /usr/local/s-ui/db/s-ui.db
sudo rm -f /usr/local/s-ui/db/s-ui.db-wal
sudo rm -f /usr/local/s-ui/db/s-ui.db-shm
```

Copy your private database:

```bash
sudo cp -f s-ui.db /usr/local/s-ui/db/s-ui.db
```

Fix permissions:

```bash
sudo chown root:root /usr/local/s-ui/db/s-ui.db
sudo chmod 600 /usr/local/s-ui/db/s-ui.db
```

Check SQLite integrity:

```bash
sudo sqlite3 /usr/local/s-ui/db/s-ui.db "PRAGMA integrity_check;"
```

Start S-UI:

```bash
sudo systemctl start s-ui
```

Check status:

```bash
sudo systemctl status s-ui --no-pager
```

## Verify files

From the repository root:

```bash
cd S_UI_Backup
```

Check SHA256 sums:

```bash
sha256sum -c checksums.sha256
```

Other checksum files are also included:

```text
checksums.md5
checksums.sha1
checksums.sha512
checksums.hash
```

If your checksum tool uses different path formatting, manually compare the hashes for important files:

```bash
sha256sum root/s-ui-survival/sui.bin
sha256sum root/s-ui-survival/s-ui.sh
sha256sum root/s-ui-survival/s-ui.service
sha256sum root/s-ui-survival/s-ui-command
```

## Logs

Check S-UI service logs:

```bash
sudo journalctl -u s-ui -e --no-pager
```

Follow logs live:

```bash
sudo journalctl -u s-ui -f
```

## Useful commands

Open the management menu:

```bash
sudo s-ui
```

Restart S-UI:

```bash
sudo systemctl restart s-ui
```

Stop S-UI:

```bash
sudo systemctl stop s-ui
```

Start S-UI:

```bash
sudo systemctl start s-ui
```

Check service state:

```bash
sudo systemctl status s-ui --no-pager
```

## Notes

Do not use the built-in update option unless you know exactly where the update source points.

The original upstream disappeared for me, so the update function may fail or break the installation.

Recommended approach:

- keep this repository as an archive
- test restore on a separate VPS first
- do not overwrite a working production installation without a private backup
- keep real database backups private
- keep at least one offline copy of the original full backup

## Disclaimer

This repository is only an archival backup of a working S-UI installation.

It is not official, not maintained by the original author, and comes with no warranty.

Use at your own risk.
