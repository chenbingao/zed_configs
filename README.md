# Zed 配置

个人 [Zed](https://zed.dev) 配置，以 **Vim 为核心的全键盘工作流**：不依赖鼠标完成
项目管理、代码导航、Git 操作与分屏。

按键体系对齐同机的 Neovim 配置（`~/.config/nvim`），目标是两边的肌肉记忆一致。

- 目标 Zed 版本：**1.23.2**（所有 action 名与语义都对着该版本源码核实）
- 平台：macOS (arm64)

## 文件

| 文件 | 说明 |
|---|---|
| `settings.json` | 编辑器设置：Vim 模式、行号、搜索、缩进、rust-analyzer 路径 |
| `keymap.json` | 按键映射，10 个 context / 101 条绑定 |
| `tasks.json` | 任务定义（lazygit、cargo test） |
| `KEYMAP-MAPPING.md` | **Neovim ↔ Zed 完整键位对照表**，含语义差异与能力缺口说明 |

## 键位骨架

前缀式分组，与 Neovim 一致：

| 前缀 | 语义 | 例子 |
|---|---|---|
| `space` | 命令层 | `space f` 找文件 · `space s` 全局搜索 · `space g` 前缀进入 Git 组 · `space l` 前缀进入 LSP 组 |
| `,` | buffer / 面板层 | `,o` 找文件 · `,h ,l ,j ,k` 按方向关 buffer · `,r` 重载 |
| `Ctrl+h/j/k/l` | 窗口跳转（编辑器内） | 面板聚焦时改用 `Ctrl+w h/j/k/l` |
| 单键组 | LSP | `gd` 定义 · `gt` 类型 · `gi` 实现 · `gr` 引用 · `gn`/`gp` 上/下诊断 · `K` 悬浮 |
| `space g` | Git | `s` 暂存 hunk · `u` 取消暂存 · `r` 丢弃 hunk · `p` 预览 · `b` blame · `d` diff |
| `g c` / `g b` | 注释 | 行注释 / 块注释（Zed 原生） |
| `z a/c/o/M/R` | 折叠 | 切换 / 折叠 / 展开 / 全部折叠 / 全部展开（Zed 原生） |
| `[c` / `]c` | hunk 跳转 | 上一个 / 下一个 |

完整对照（含"哪些是近似而非等价"和"Zed 无对应能力"清单）见
[`KEYMAP-MAPPING.md`](./KEYMAP-MAPPING.md)。

## 使用

Zed 会自动读取 `~/.config/zed/` 下的配置，改完保存即生效（`keymap.json` 与
`settings.json` 支持热重载）。

若要把本仓库部署到新机器：

```bash
git clone <this-repo> ~/.config/zed
```

`keymap.json` / `settings.json` / `tasks.json` 均为 **JSONC**（带注释与尾逗号），
Zed 用 lenient 解析器读取，属于受支持格式。

## 依赖

部分按键依赖外部程序，缺失时对应功能不可用（不影响其他键位）：

| 依赖 | 用途 | 相关键位 |
|---|---|---|
| `lazygit` | Git TUI | task（命令面板 `task: spawn` 调起） |
| `cargo` | 跑测试 | `,tr`（`editor::SpawnNearestTask`） |
| `rust-analyzer` | Rust LSP | 见下方说明 |

### rust-analyzer

`settings.json` 用**绝对路径**显式指定了本机二进制：

```json
"lsp": { "rust-analyzer": { "binary": {
  "path": "~/.rustup/toolchains/stable-aarch64-apple-darwin/bin/rust-analyzer"
} } }
```

**为什么不能省**：Zed 在 PATH 里找不到可用的 rust-analyzer 时，会调用
`api.github.com` 查 release 下载，未认证限流是 60 次/小时/IP，很容易撞 403
（`status error 403 ... API rate limit exceeded`）。指定路径后
`get_language_server_binary` 第一步就命中并返回，完全不进下载逻辑。

另外要注意 Zed 从 Finder/Dock 启动时**不继承 shell 的 PATH**，所以依赖 PATH
自动发现并不可靠。若你的 toolchain 路径不同，改这一行即可；装了 rust-analyzer
组件后也可改用 `~/.cargo/bin/rust-analyzer`（rustup shim，会跟随默认 toolchain）。

## 维护注意

几处**容易踩且不会报错**的地方，改键位前建议先读：

1. **多键序列必须用空格分隔**：写 `", o"` 而不是 `",o"`。Zed 按
   `split_whitespace()` 切分（`crates/gpui/src/keymap/binding.rs`），`",o"` 会被当成
   **单个**键名，平台永远发不出这个键 —— 绑定静默失效且不报错。
2. **前缀键要小心 1 秒超时**：若某个键既是完整绑定又是更长绑定的前缀，裸按它会等待
   超时。比如项目面板里 `y` 是 `y y` 的前缀。
3. **`null` 才是解绑**：`"v": "escape"` 这种写法会被解析器拒绝（`escape` 不是
   action 名），正确写法是 `null` 或一个真实的 `ns::Action`。
4. **确认 action 在目标上下文有处理者**：Zed 里存在"action 名合法但只在某个面板
   注册"的情况，绑到编辑器里按下毫无反应。例如 `git::StageFile` 只在 GitPanel 注册，
   编辑器侧要用 `editor::ApplyAllDiffHunks`。

## .gitignore 说明

仓库只跟踪配置文件与文档；Zed 的运行时状态不入库：

- `prompts/` —— Zed 的提示词库 LMDB 数据库，运行时会持续改写
- `.backup-*/` —— 手工备份
- `themes/` —— **原本就是空目录**（Zed 首次运行建出的主题槽位，当前没有自定义主题）。
  用 `themes/.gitkeep` 占位，仅为了让 git 保留这个目录；以后放入自定义主题（`*.json`）
  会被正常跟踪

## 相关

- Neovim 配置：`~/.config/nvim`（本仓库键位的对齐基准）
- Zed 文档：<https://zed.dev/docs/configuring-zed>
