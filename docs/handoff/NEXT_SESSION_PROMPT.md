# Next Session Prompt

Copy the following into a new Claude Code / agent session.

---

继续 KeyStats Linux/GNOME 移植工作。

**先读这些文件:**
- `docs/handoff/PROJECT_HANDOFF.md` — 完整架构和上下文
- `docs/handoff/STATUS_CHECKLIST.md` — 已完成/待完成/优先级
- `CLAUDE.md` — 项目级规范（隐私、线程安全、暗色模式、UI style）

**当前阶段:** MVP 核心功能已完成，进入 polish + 发布准备阶段。

**关键约束:**
- Rust workspace: `KeyStats.Linux/`，所有 cargo 命令在此目录执行
- GNOME 扩展源码: `KeyStats.GNOME/keystats@debugtheworldbot.github.io/`
- 扩展使用 GNOME 45+ ES module 语法 (`import X from 'gi://X'`)
- St CSS 不支持自定义属性；background-color 透明度会影响子元素渲染
- D-Bus: `io.github.debugtheworldbot.KeyStats` / `io.github.debugtheworldbot.KeyStats1`
- 修改扩展后需重启 GNOME Shell: **Alt+F2 → r → Enter**

**推荐下一步工作（按优先级）:**
1. 弹窗 UI 美化 — Key Breakdown 改为 macOS/Windows 风格的 pill badge，优化间距和视觉层次
2. 实现 Dynamic Accent Color — 三处 TODO 注释标记，读取 GSettings 后通过 St 动态主题应用到图表/KPS badge
3. Fedora GNOME smoke test — 在 Fedora 上验证安装流程和设备权限
4. deb/rpm 打包脚本完善 — 目前只有 cargo-deb/cargo-rpm 占位命令

**验证门禁:**
```bash
cd KeyStats.Linux
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all --check
```

**提交风格:** `feat(linux):`, `fix(linux):`, `chore(linux):`
