# Neovim ↔ Zed 键位对照表

生成: 2026-10-12（第 2 版，依据独立核查报告全面修正后重写）
Zed 版本: 1.23.2 —— 所有 action 名与语义均对着该版本源码核实

> **本版与第 1 版的重要差异**：第 1 版有 4 处硬伤（逗号键位写法非法导致 9 条绑定永不触发、
> `space g S` 指向编辑器里没有处理者的 action、视觉 `v` 退出能力丢失、插入模式 `ctrl-p/n`
> 变成滚动并吞掉补全菜单导航），已全部修复。下面标注的"依据"都是源码行号，可复核。

---

## 一、对照基准

| 侧 | 文件 | 说明 |
|---|---|---|
| nvim | `lua/keymaps.lua` | **84 行**（14:55 被 git 还原后的当前版本） |
| nvim | `lua/lsp/common.lua` | LSP on_attach，buffer-local |
| nvim | `lua/lsp/rust.lua` / `swift.lua` | `K` 覆盖为 RustLsp hover / `,ss` `,sr` |
| nvim | `lua/config/gitsigns.lua` | 16 条 hunk 操作 |
| nvim | `lua/config/ufo.lua` | `zR` / `zM` / `zp` |
| nvim | `lua/config/nvim-cmp.lua` | 插入模式菜单 |
| nvim | `lua/config/telescope.lua` | picker 内键位 |
| nvim | **nvim-tree 插件默认键位表** | `nvimtree.lua` 未设 `on_attach`，故走 `nvim-tree/keymap.lua:53-116` |
| Zed | `~/.config/zed/keymap.json` | 10 个 context / **101 条绑定** |
| Zed | `~/.config/zed/settings.json` | vim 段 + rust-analyzer 路径 |

`<leader>` = 空格（`config/lazy.lua:21`）。**`,` 不是 leader**，是普通字面键。

---

## 二、一一对应（行为等价）

### 窗口 / buffer

| 键 | nvim | Zed |
|---|---|---|
| `Ctrl+h/j/k/l` | `<C-w>h/j/k/l` | `workspace::ActivatePane*` |
| `Ctrl+i` | 跳转列表向前 | `pane::GoForward` |
| `Ctrl+o` | 跳转列表向后（内置） | `pane::GoBack` |
| `Tab` / `Shift+Tab` | 下一个 / 上一个 buffer | `pane::ActivateNextItem` / `ActivatePreviousItem` |
| `q` | 禁用宏录制 | `null` |
| `[c` / `]c` | 上一个 / 下一个 hunk | `editor::GoToPreviousHunk` / `GoToHunk` |
| `gc` / `gb` | 行 / 块注释 | `vim::PushToggleComments` / `PushToggleBlockComments`（Zed 原生） |
| `zR` / `zM` | 全部展开 / 折叠 | `editor::UnfoldAll` / `FoldAll`（Zed 原生） |

### LSP

| 键 | nvim | Zed |
|---|---|---|
| `K` | `vim.lsp.buf.hover` | `editor::Hover` |
| `gd` / `gt` / `gi` / `gr` | 定义 / 类型 / 实现 / 引用 | `editor::GoToDefinition` / `GoToTypeDefinition` / `GoToImplementation` / `FindAllReferences` |
| `gn` / `gp` | `vim.diagnostic.goto_next` / `goto_prev` | `editor::GoToDiagnostic` / `GoToPreviousDiagnostic` |
| `<leader>lc` | actions-preview 代码操作 | `editor::ToggleCodeActions` |
| `<leader>lr` | `vim.lsp.buf.rename()` | `editor::Rename` |

### `<leader>` 主命令

| 键 | nvim | Zed | 备注 |
|---|---|---|---|
| `<leader>fs` | `:w` | `workspace::FormatAndSave` | ⚠️ 见注 1 |
| `<leader>fS` | `:wa` | `workspace::SaveAll` | |
| `<leader>e` | `NvimTreeFindFile`（树中定位当前文件） | `pane::RevealInProjectPanel` | 已修正（原为 `ToggleFocus`，只开关不定位） |
| `<leader>fw` | `live_grep` | `pane::DeploySearch` | |
| `<leader>fb` | telescope buffers | `tab_switcher::Toggle` | |
| `<leader>ds` | telescope 诊断 | `diagnostics::Deploy` | |
| `<leader>lo` | Navbuddy 大纲 | `outline::Toggle` | |
| `<leader>S` | spectre 搜索替换 | `pane::DeploySearch {replace_enabled: true}` | |

### `,` 层（**本版修正**：键位串必须写 `", x"` 带空格）

| 键 | nvim | Zed |
|---|---|---|
| `,o` | telescope `find_files` | `file_finder::Toggle` |
| `,r` | 重载文件（`:e`） | `editor::ReloadFile` |
| `,h` / `,l` | 关左侧 / 右侧所有 buffer | `pane::CloseItemsToTheLeft` / `CloseItemsToTheRight` |
| `,j` / `,k` | 关其他 / 当前 buffer | `pane::CloseOtherItems` / `CloseActiveItem` |
| `,tt` | neotest 输出面板 | `terminal_panel::Toggle`（含义不同，见注 3） |
| `,tr` | 跑最近测试 | `editor::SpawnNearestTask`（见注 3） |

> **为什么必须带空格**：Zed 用 `split_whitespace()` 切分多键序列
> （`crates/gpui/src/keymap/binding.rs:56`），写成 `",o"` 会被解析成**单个**键名 `",o"`，
> 平台永远发不出这个键 → 绑定静默失效且不报错。shipped `vim.json:66` 的裸逗号也是写 `","`。
> 另外裸 `,` 默认被绑给 `vim::RepeatFindReversed`，不清掉会让每次按逗号等 1 秒前缀超时，
> 所以本文件把 `","` 置 `null`。

### gitsigns 组

| 键 | nvim | Zed | 备注 |
|---|---|---|---|
| `<leader>gs` | `stage_hunk` | `git::StageAndNext` | ✅ 编辑器有处理者（`editor/src/git.rs`） |
| `<leader>gu` | `undo_stage_hunk`（**只取消**暂存） | `git::UnstageAndNext` | **本版修正**（原用 `ToggleStaged`，那是切换，未暂存 hunk 会被反向暂存） |
| `<leader>gt` | `toggle_current_line_blame` | `editor::ToggleGitBlameInline` | ✅ |
| `<leader>gr` | `reset_hunk` | `git::Restore` | ✅ |
| `<leader>gR` | `reset_buffer` | `git::RestoreFile` | ✅ |
| `<leader>gS` | `stage_buffer`（整文件） | `editor::ApplyAllDiffHunks` | **本版修正**（原用 `git::StageFile`，它只在 GitPanel 注册，编辑器里按下无反应） |
| `<leader>gb` | `blame_line {full=true}`（当前行弹窗） | `editor::BlameHover` | **本版修正**（原用 `git::Blame`，那是切换整文件 blame gutter） |
| `<leader>gd` | `diffthis`（当前文件 vs index） | `git::DiffHead` | ⚠️ 近似：Zed 是**整个工作区** vs HEAD（`git_ui/src/project_diff.rs:45-50`） |
| `<leader>gp` | `preview_hunk` | `editor::ToggleSelectedDiffHunks` | **本版修正**（原名是 `ToggleHunkDiff`，那只是弃用别名） |

### 插入模式

| 键 | nvim | Zed | 备注 |
|---|---|---|---|
| `Ctrl+p` / `Ctrl+n` | 光标上 / 下一行 | `vim::Up` / `vim::Down` | **本版修正**：原用 `LineUp/LineDown`，那是**滚动**（`scroll.rs:11-17` 文档即 "Scrolls..."） |
| `Ctrl+e` / `Ctrl+a` | 行尾 / 行首 | `editor::MoveToEndOfLine` / `MoveToBeginningOfLine` | ✅ |
| `Ctrl+.` | `cmp.mapping.complete()` | `editor::ShowCompletions` | ✅ |
| `Ctrl+f` / `Ctrl+b` | **单字符**右 / 左移 | `vim::NextWordStart` / `PreviousWordStart` | ⚠️ 词级近似，见注 5 |
| 菜单可见时 `Ctrl+p/n` | 选上 / 下一候选 | Zed 原生 `ContextMenuPrevious/Next` 接管 | **本版修正**：加了 `!showing_completions` 守卫 |

### 视觉模式

| 键 | nvim | Zed | 备注 |
|---|---|---|---|
| `v` | 退出视觉模式 | `vim::SwitchToNormalMode` | **本版修正**：原写 `"v": null`，解绑后按 v 完全没反应 |
| `,y` | 复制选区（系统剪贴板） | `vim::Yank` | |
| `<` / `>` | 缩进并保持选中 | `SendKeystrokes "< g v"` / `"> g v"` | |
| `<leader>gs` / `<leader>gr` | 视觉范围暂存 / 重置 | `git::StageAndNext` / `git::Restore` | |

### 项目面板（基准 = nvim-tree 插件默认表）

| 键 | nvim-tree | Zed | 备注 |
|---|---|---|---|
| `o` / `<CR>` | `open.edit` | `project_panel::OpenPermanent` | nvim 的"打开"，目录则展开 |
| `<Tab>` | `open.preview` | `project_panel::Open` | 预览标签 |
| `a` / `r` | `fs.create` / `fs.rename` | `NewFile` / `Rename` | |
| `d` | `fs.remove`（删除） | `project_panel::Delete` | ⚠️ **本版改为删除**（Zed 原生 `d` 是"新建目录"）。**Zed 面板删除无撤销** |
| `c` / `x` / `p` | `copy.node` / `cut` / `paste` | `Copy` / `Cut` / `Paste` | |
| `y` | `copy.filename` | `workspace::CopyRelativePath` | ⚠️ Zed 无"复制文件名"能力，退化为相对路径（≈ nvim-tree 的 `Y`） |
| `y y` | （`gy` 绝对路径） | `workspace::CopyPath` | 见注 6 |
| `<C-v>` / `<C-x>` | 垂直 / 水平分割 | `OpenSplitVertical` / `OpenSplitHorizontal` | 键位挪到 `Shift+v` / `Shift+x` |
| `<C-r>` | `rename_sub`（省略扩展名） | `Rename` | ⚠️ Zed 无此能力，退化为普通重命名 |
| `h` / `l` | `parent_close` / `parent_open` | `CollapseSelectedEntry` / `ExpandSelectedEntry` | |
| `W` / `E` | 全部折叠 / 展开 | `CollapseAllEntries` / `ExpandAllEntries` | 键位挪到 `Shift+w` / `Shift+e` |
| `-` | `change_root_to_parent`（改树根） | `SelectParent` | ⚠️ 语义不同：Zed 只是移动选中项，不能改树根 |

---

## 三、Zed 侧额外绑定（nvim 没有）

| 键 | action | 为何加 |
|---|---|---|
| `space c` | `command_palette::Toggle` | Zed 命令面板（nvim 用 `:`） |
| `space t` | `terminal_panel::Toggle` | 终端面板（nvim 当前基线已无 `<leader>tt`） |
| `space g B` | `git::Blame` | 整文件 blame gutter |
| `space g p` | `editor::ToggleSelectedDiffHunks` | 内联展开 hunk diff |
| `Ctrl+w h/j/k/l`（`!Editor && !Terminal`） | `ActivatePane*` | 面板聚焦时裸 `Ctrl+hjkl` 会被面板子上下文抢走 |
| `w/b/e/ge` | `vim::*Subword*` | camelCase / snake_case 逐段跳（**替换**了 vim 原生的词运动，见注 7） |
| `s` / `S` | `vim::PushSneak*` | 两字符跳转（**替换**了 vim 原生的 `s`=substitute，见注 7） |
| `S`（视觉） | `vim::PushAddSurrounds` | **替换**了 vim 原生视觉 `S`=SubstituteLine |

---

## 四、缺口（Zed 无对应能力）

| nvim | 说明 |
|---|---|
| `<leader>fh` | telescope `help_tags` —— 无帮助标签搜索（最接近 `zed::OpenDocs`） |
| `<leader>gD` / `<leader>gx` | 与上一提交 diff / 切换显示已删除行 —— 编辑器可达 action 里都没有 |
| `ih`（gitsigns hunk 文本对象） | 编辑器 action 里 Hunk 系列只有 GoTo/Apply/Expand/Collapse/Toggle，无"选中 hunk" |
| `zp`（ufo peek） | 无 Peek action；Zed 折叠键位反而更全（`za/zA/zc/zC/zo/zO/zM/zR/zf`） |
| cmp 的 `Ctrl+b/f` 滚补全文档 | 无补全文档滚动 action |
| options `cursorcolumn` / `hlsearch=false` / `completeopt` | Zed 无对应设置（搜索始终高亮） |
| nvim-tree `R` / `g?` / `u` / `e` / `ge` / `q` / `<C-t>` | 刷新（靠 watcher）· 帮助 · 两种重命名 · 复制基础名 · 关树 · 新标签打开 |
| nvim-tree 其余 | `O P J K > < s S f F H I B C M U m .`、`bd/bt/bmv`、`]c/[c`、`]e/[e`、`<BS>`、`<C-e>`、`<C-k>`、`<C-]>`、`L` |
| `,ss` / `,sr` | Swift `:XcodebuildSetup` / `:XcodebuildBuildRun` —— 需写成 task 才有意义，未加 |

---

## 五、语义翻译说明（不是直接对应）

1. **`<leader>fs` → `FormatAndSave`**：nvim 侧 conform 配了保存时格式化；Zed 的 `format_on_save` 默认 `"off"`，故用 `FormatAndSave` 强制格式化。想要纯保存改 `workspace::Save`。
2. **`<leader>gd` → `git::DiffHead`**：Zed 没有"当前文件 diff 视图"的 action，`DiffHead` 是**整个工作区** vs HEAD 的 ProjectDiff，属近似。
3. **`,tt` / `,tr`**：nvim 是 neotest 的输出面板与"跑最近测试"。Zed 没有测试面板 action（`editor::RunTest` 不存在），只能用任务系统近似，需在 `tasks.json` 定义带 `tags` 的任务才真正可用。
4. **`<leader>gu`**：即使换成 `UnstageAndNext`，它也是"取消暂存**并移到下一个 hunk**"，多一步移动；gitsigns 的 `undo_stage_hunk` 没有完全等价的 action。
5. **`Ctrl+f` / `Ctrl+b`**：nvim 是**字符级**左右移，Zed 侧用词级移动近似。要严格一致可改 `editor::MoveRight` / `MoveLeft`。
6. **项目面板 `y`**：`y` 本身是前缀（`y y` 更长），所以按 `y` 会等 1 秒前缀超时才触发。这是"想保留 `gy` 语义"的代价。
7. **`w/b/e` 与 `s/S`**：这两组是我用 Zed 原生能力**替换**了 vim 语义 —— `s` 原本是 substitute（改写单字符），现在变成 sneak。若你依赖 `s`，把 `"s"`/`"shift-s"` 两行删掉即可恢复。
8. **`d`（项目面板）**：Zed 原生 `d` 是"新建目录"，本文件按"以 nvim 为准"改成了删除。**Zed 面板删除没有撤销**，误按即丢文件。

---

## 六、需要实际运行才能确认

1. **`Ctrl+hjkl` 跨窗跳转**：只在 `Editor && VimControl` 绑了裸键；面板聚焦需用 `Ctrl+w hjkl`。
2. **视觉 `<` / `>` 保持选区**：用 `SendKeystrokes "> g v"` 实现，依赖"自引用取次高优先级"规则。
3. **插入模式 `Ctrl+f/b/p/n`**：`vim::NextWordStart` / `Up` / `Down` 在 `vim_mode == insert` 上下文没有官方默认绑定，属未文档化用法。
4. **sneak（`s`/`S`）与 subword（`w/b/e`）**：你 nvim 侧无对应插件，是新引入的手感。
5. **前缀超时**：`space` 与 `,` 都是前缀，裸按会等 1 秒。`which_key.enabled` 已开（延迟 300ms），按住会弹提示。
6. **终端内的键位**：已把 `Ctrl+w` 段限制在 `!Editor && !Terminal`，终端里 `Ctrl+w` 应正常删词，需实测。

---

## 七、已知遗留项（未处理，明确记录）

以下问题已由独立核查确认存在，但**有意未修**，列在这里以免被遗忘。

### 优先级较高

| 项 | 问题 | 影响 |
|---|---|---|
| `space s` / `space r` 变成前缀 | 这两个键原本是完整绑定，但 `space x`/`space X` 之外还存在 `space s`… 与 `space g*`/`space l*`，加上 `space t` 也是前缀。Zed 里"某键既是完整绑定又是更长绑定的前缀"会导致裸按等待 1 秒超时 | 按 `space s` 会延迟约 1 秒才触发全局搜索 |
| 项目面板 `d` | 按"以 nvim 为准"改成 `project_panel::Delete`，而 Zed 面板删除**没有撤销** | 误按即丢文件 |

### 优先级较低

| 项 | 问题 |
|---|---|
| `timer` 式 which_key | `which_key.enabled` 已开（300ms），但只在"按住前缀"时提示，不改变超时行为 |
| `tab`（普通模式） | 覆盖了 `editor::AcceptEditPrediction`（编辑预测接受键），未在注释中声明 |
| `g i` / `g t` | 覆盖了 vim 原生的 `vim::InsertAtPrevious` / `vim::GoToTab`，未在注释中声明；且 `g shift-t` 仍是 `vim::GoToPreviousTab`，导致 `g t`/`g T` 不再配对 |
| `space x` | 是 `FormatAndSave`（保存前强制格式化），Zed 侧**没有纯保存键**。nvim 的 `<leader>fs` 是纯 `:w` |
| 项目面板 `z c` | 覆盖了 vim 原生的 `project_panel::ScrollCursorCenter`（把光标行滚到中间），未在注释中声明 |

### 文档层面的已知不精确

- `space g u` 即使用 `UnstageAndNext`，仍多一步"移到下一个 hunk"，与 gitsigns 的 `undo_stage_hunk` 不完全等价（见第五节注 4）。
- 项目面板 `y` 是 `y y` 的前缀，裸按会等 1 秒超时（见第五节注 6）。

