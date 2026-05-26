# KeyStats — Project Handoff

**Date:** 2026-05-26
**Branch:** `feat/linux-gnome`
**Commit:** `2b11ff2` feat(linux): add Linux/GNOME port with Rust daemon and GNOME Shell extension

## Project Purpose

KeyStats 是隐私优先的键盘/鼠标使用统计工具，已有 macOS (Swift) 和 Windows (C#) 版本。
本分支新增 Linux/GNOME 移植，使用 Rust 守护进程 + GNOME Shell 扩展架构。

## Repo Layout

```
keyStats/
├── KeyStats/              # macOS (Swift/AppKit)
├── KeyStats.Windows/       # Windows (C#/WPF)
├── KeyStats.Linux/         # Linux (Rust workspace)        ← 新增
│   ├── Cargo.toml
│   ├── crates/
│   │   ├── keystats-core/       # 共享模型、格式化、导入导出
│   │   ├── keystats-daemon/     # evdev 输入 + SQLite + D-Bus
│   │   └── keystatsctl/        # CLI (status, doctor)
│   └── packaging/               # systemd, udev, 文档
├── KeyStats.GNOME/         # GNOME Shell 扩展 (GJS)        ← 新增
│   └── keystats@debugtheworldbot.github.io/
│       ├── extension.js          # 面板 + 弹窗 UI
│       ├── prefs.js              # AdwPreferencesWindow
│       ├── stylesheet.css        # 双主题 (dark/light)
│       ├── metadata.json
│       └── schemas/
├── KeyStatsHelper/         # macOS helper (Swift)
└── docs/superpowers/       # 设计规格 + 实施计划
```

## Architecture — Linux Daemon

```
GNOME Shell extension ← D-Bus session bus → keystats-daemon → /dev/input/event*
                                    ↓
                              ~/.local/state/keystats/stats.sqlite3
```

**关键决策：**
- Daemon 使用 Rust (evdev + zbus + rusqlite)
- 扩展使用 GJS (GNOME Shell 45+ ES modules)
- 扩展不直接读输入设备，只通过 D-Bus 通信
- StatsManager 用 `Arc<Mutex<StatsManager>>` 跨线程共享
- 权限：普通用户 + `input` 组（Ubuntu 默认不加入）
- Schema v2 (含 middle_clicks 列 + v1→v2 迁移)

**D-Bus API (session bus):**
- Service: `io.github.debugtheworldbot.KeyStats`
- Interface: `io.github.debugtheworldbot.KeyStats1`
- Methods: GetTodayStats, GetRates, GetTopKeys, GetHistory, GetPermissionStatus, ResetToday, ClearAllData, ExportData, ImportData
- Property: Version (readonly)

## Tech Stack

| 层 | 技术 |
|---|---|
| Daemon | Rust 2024, evdev 0.13, zbus 5.x, rusqlite 0.32 (bundled SQLite), chrono, tracing |
| CLI | Rust + clap 4 derive |
| Extension | GJS ES modules, St/Clutter, Gio.DBus, Adw (prefs) |
| Build | cargo workspace in KeyStats.Linux/ |
| Tests | 37 (12 core + 25 daemon), `cargo test --workspace` |
| Lint | clippy `-D warnings`, rustfmt |

## Implementation Rules (from CLAUDE.md)

- **Privacy**: never log raw keystrokes, text content, mouse positions — only aggregate counts
- **Minimal Intrusion Principle**: new features in new files, narrow extension boundaries
- **Code style**: `#[allow(dead_code)]` on pre-wired methods, `#[cfg(test)]` for test helpers
- **No emojis as structural icons** in the extension

## Major Completed Capabilities

- evdev input pipeline: keyboard, mouse clicks (L/M/R/side), movement, scroll
- KPS/CPS sliding window + peak tracking, midnight auto-reset
- SQLite persistence with WAL, schema v2, XDG_STATE_HOME
- D-Bus service: 9 methods, gdbus + GJS verified
- systemd user service + udev rule + packaging docs (EN/ZH)
- GNOME extension: dual-theme panel + popup with hero cards, click detail, key breakdown (3×5 grid), history bar chart, connection indicator, KPS badge
- Preferences: AdwPreferencesWindow with Display/Refresh/Data Management
- Data Management: Reset Today (clears daily_stats + key_counts), Clear All Data (with confirmation dialog)
- CLI: `keystatsctl status` / `keystatsctl doctor` with real device scan
- CI: build-linux job in release.yml (build + test + clippy + fmt + tarball)
- README.md / README_ZH.md Linux install sections

## Known Issues & Constraints

- 弹窗透明度功能已放弃（St CSS background-color 会影响子元素渲染，set_background_color 在 St 上无效）
- Dynamic Accent Color 设置开关占位但未实现（三处 TODO 注释标记）
- Middle click 硬编码为 0（daemon 已支持但需重启重建）
- 不支持按应用统计（Wayland 限制）
- 扩展仅支持 GNOME 45+，未在 KDE/其他 DE 测试

## Local Verification

```bash
# Build & test
cd KeyStats.Linux && cargo test --workspace && cargo clippy --workspace --all-targets -- -D warnings && cargo fmt --all --check

# Run daemon
cargo run -p keystats-daemon

# Test D-Bus
gdbus call --session --dest io.github.debugtheworldbot.KeyStats --object-path /io/github/debugtheworldbot/KeyStats --method io.github.debugtheworldbot.KeyStats1.GetTodayStats

# Install GNOME extension
cd KeyStats.GNOME/keystats@debugtheworldbot.github.io
cp -r * ~/.local/share/gnome-shell/extensions/keystats@debugtheworldbot.github.io/
glib-compile-schemas ~/.local/share/gnome-shell/extensions/keystats@debugtheworldbot.github.io/schemas/
# Alt+F2 → r → Enter → gnome-extensions enable keystats@debugtheworldbot.github.io
```
