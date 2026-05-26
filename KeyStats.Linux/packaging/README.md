# KeyStats Linux Packaging

All commands assume you are in the **repository root** (where `KeyStats.Linux/` and `KeyStats.GNOME/` are). Paths are relative to the repo root unless stated otherwise.

```
keyStats/                         ← you are here
├── KeyStats.Linux/               ← Rust workspace
│   ├── Cargo.toml
│   ├── crates/ (keystats-core, keystats-daemon, keystatsctl)
│   ├── packaging/ (systemd, udev)
│   └── target/  (build output)
├── KeyStats.GNOME/               ← GNOME extension source
│   └── keystats@debugtheworldbot.github.io/
└── ...
```

---

## Prerequisites

- **Rust** 1.70+ (`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`)
- **GNOME Shell 45+**
- **input group** membership

```bash
sudo usermod -aG input $USER
# Log out and back in, or: newgrp input
```

---

## Development Build & Run

All cargo commands run from `KeyStats.Linux/`:

```bash
cd KeyStats.Linux

# Build
cargo build -p keystats-daemon -p keystatsctl

# Run daemon (foreground, Ctrl+C to stop)
cargo run -p keystats-daemon

# Check permissions
cargo run -p keystatsctl -- doctor

# Check daemon stats
cargo run -p keystatsctl -- status
```

---

## Installation

### 1. Build and install daemon + CLI

```bash
cd KeyStats.Linux

# Release build
cargo build --release -p keystats-daemon -p keystatsctl

# Install to ~/.local/bin
mkdir -p ~/.local/bin
cp target/release/keystats-daemon ~/.local/bin/
cp target/release/keystatsctl ~/.local/bin/

# Ensure ~/.local/bin is on PATH
export PATH="$HOME/.local/bin:$PATH"
```

### 2. Install systemd user service

```bash
mkdir -p ~/.config/systemd/user
cp KeyStats.Linux/packaging/systemd/keystats.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now keystats.service

# Verify
systemctl --user status keystats.service
```

### 3. Install GNOME Shell extension

The extension source is at `KeyStats.GNOME/keystats@debugtheworldbot.github.io/`.

**Option A: pack as zip and install** (clean, recommended for distribution)

```bash
cd KeyStats.GNOME/keystats@debugtheworldbot.github.io

# Create the zip package
zip -r keystats@debugtheworldbot.github.io.zip \
    metadata.json extension.js prefs.js stylesheet.css schemas/

# Install
gnome-extensions install keystats@debugtheworldbot.github.io.zip

# The zip will be at: KeyStats.GNOME/keystats@debugtheworldbot.github.io/keystats@debugtheworldbot.github.io.zip
```

**Option B: copy directly** (no zip, quick dev iteration)

```bash
SRC=KeyStats.GNOME/keystats@debugtheworldbot.github.io
DST=~/.local/share/gnome-shell/extensions/keystats@debugtheworldbot.github.io

mkdir -p $DST
cp $SRC/metadata.json $SRC/extension.js $SRC/prefs.js $SRC/stylesheet.css $DST/
cp -r $SRC/schemas $DST/
glib-compile-schemas $DST/schemas/
```

After either option:

```bash
# Restart GNOME Shell: press Alt+F2, type r, press Enter

# Enable the extension
gnome-extensions enable keystats@debugtheworldbot.github.io
```

### 4. Verify

```bash
keystatsctl doctor       # check devices
keystatsctl status       # check daemon stats
# Click the "K... C..." indicator in the top bar → popup should open
```

---

## Packaging for Distribution

All packaging commands run from the **repo root**.

### tarball

```bash
cd KeyStats.Linux

# Build release binaries
cargo build --release -p keystats-daemon -p keystatsctl

# Assemble tarball
mkdir -p dist
cp target/release/keystats-daemon dist/
cp target/release/keystatsctl dist/
cp packaging/systemd/keystats.service dist/
cp packaging/udev/60-keystats-input.rules dist/
cd dist && tar -czf ../keystats-linux-x86_64.tar.gz *
# Tarball at: KeyStats.Linux/keystats-linux-x86_64.tar.gz
```

### GNOME extension zip

```bash
cd KeyStats.GNOME/keystats@debugtheworldbot.github.io
zip -r keystats@debugtheworldbot.github.io.zip \
    metadata.json extension.js prefs.js stylesheet.css schemas/
# Zip at: KeyStats.GNOME/keystats@debugtheworldbot.github.io/keystats@debugtheworldbot.github.io.zip
```

Move the zip to the repo root for release upload:

```bash
cp KeyStats.GNOME/keystats@debugtheworldbot.github.io/keystats@debugtheworldbot.github.io.zip .
```

### .deb

```bash
cargo install cargo-deb
cd KeyStats.Linux && cargo deb -p keystats-daemon
```

### .rpm

```bash
cargo install cargo-rpm
cd KeyStats.Linux && cargo rpm build -p keystats-daemon
```

---

## Permissions

Run `keystatsctl doctor` to check. If devices are blocked:

```bash
# Option A: input group (recommended)
sudo usermod -aG input $USER
# Re-login

# Option B: dedicated keystats group + udev rule
sudo cp KeyStats.Linux/packaging/udev/60-keystats-input.rules /etc/udev/rules.d/
sudo groupadd --system keystats
sudo usermod -aG keystats $USER
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## Troubleshooting

| Symptom | Check |
|---------|-------|
| Panel shows "…" or "offline" | Is daemon running? `systemctl --user status keystats.service` |
| `keystatsctl doctor` shows blocked | Not in `input` group. See Permissions |
| Extension not in `gnome-extensions list` | Restart GNOME Shell: **Alt+F2 → r → Enter** |
| Extension loads but method missing | Daemon needs rebuild. Run `cargo build -p keystats-daemon` and restart |
| Preferences Reset/Clear fails | Daemon not running or not rebuilt after schema changes |

---

## Uninstall

```bash
# Daemon
systemctl --user disable --now keystats.service
rm ~/.config/systemd/user/keystats.service
rm ~/.local/bin/keystats-daemon ~/.local/bin/keystatsctl

# GNOME extension
gnome-extensions uninstall keystats@debugtheworldbot.github.io

# Data
rm -rf ~/.local/state/keystats/

# Optional: remove udev rule
sudo rm /etc/udev/rules.d/60-keystats-input.rules
```

---

## File Locations

| What | Where |
|------|-------|
| Daemon binary | `~/.local/bin/keystats-daemon` |
| CLI binary | `~/.local/bin/keystatsctl` |
| systemd service | `~/.config/systemd/user/keystats.service` |
| GNOME extension | `~/.local/share/gnome-shell/extensions/keystats@debugtheworldbot.github.io/` |
| Stats database | `~/.local/state/keystats/stats.sqlite3` |
| Tarball (after build) | `KeyStats.Linux/keystats-linux-x86_64.tar.gz` |
| Extension zip (after build) | `KeyStats.GNOME/keystats@debugtheworldbot.github.io/keystats@debugtheworldbot.github.io.zip` |
| Journal logs | `journalctl --user -u keystats.service` |
