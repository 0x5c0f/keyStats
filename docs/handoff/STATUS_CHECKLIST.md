# Status Checklist

**Date:** 2026-05-26 | **Branch:** `feat/linux-gnome` | **Commit:** `2b11ff2`

## Completed

- [x] Phase 0: evdev/权限/D-Bus/GNOME 扩展 spike 验证
- [x] Phase 1: Rust workspace bootstrap (3 crates, CLI)
- [x] Phase 2: SQLite schema v2, StatsManager, RateTracker, import/export
- [x] Phase 3: evdev 设备发现、事件循环、按键映射、权限诊断
- [x] Phase 4: D-Bus 服务 (9 methods, gdbus + GJS 验证)
- [x] Phase 5: systemd 用户服务、udev 规则、打包文档 (EN/ZH)
- [x] Phase 6: GNOME 扩展 MVP — 面板 + 弹窗 (hero/click/distance/key breakdown/history/actions)、双主题、连接状态
- [x] Phase 7: 隐私审计 (无原始事件日志)、安装流程文档
- [x] Phase 8: README (EN/ZH) 更新、CI build-linux job
- [x] 中间点击计数 (BTN_MIDDLE → middle_clicks → D-Bus middleClicks)
- [x] Key Breakdown 功能 (daemon key_counts → D-Bus GetTopKeys → 扩展 3×5 grid)
- [x] Key Breakdown 修饰键过滤 (Shift/Ctrl/Alt/Meta/CapsLock/NumLock 不显示)
- [x] Reset Today 同时清除 key_counts
- [x] Preferences: Data Management (Reset Today + Clear All Data + 确认对话框)
- [x] 面板断开状态显示 `K-- C--` + 红色连接点
- [x] Key Breakdown 行优先排列
- [x] 弹窗透明度功能已放弃（St CSS 限制，无法可靠实现）
- [x] 37 tests passing, clippy + fmt clean
- [x] 首次 commit 已提交

## Partially Complete

- [ ] Dynamic Accent Color — GSettings key + prefs.js switch 存在但未实现（三处 TODO）
- [ ] GNOME 扩展 UI polish — Key Breakdown badge 样式可进一步对齐 macOS/Windows pill 风格
- [ ] deb/rpm 打包 — cargo-deb/cargo-rpm 命令已记录但未实际测试产出

## Not Started

- [ ] Fedora GNOME smoke test
- [ ] KDE/其他 DE 兼容性评估
- [ ] 按应用统计（Wayland 限制，需研究方案）
- [ ] 扩展的本地化 (gettext .po/.mo)
- [ ] GNOME 扩展审查提交 (extensions.gnome.org)

## Priority — Next Sequence

1. **High**: 弹窗 UI 美化 — Key Breakdown pill badge 风格
2. **Medium**: Dynamic Accent Color 实现
3. **Medium**: Fedora GNOME smoke test
4. **Low**: deb/rpm 打包完善
5. **Low**: 扩展本地化
