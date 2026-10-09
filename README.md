# nvim

[![CI](https://github.com/wuzting/wznvim/actions/workflows/ci.yml/badge.svg)](https://github.com/wuzting/wznvim/actions/workflows/ci.yml)

[English](README.en.md) | **简体中文**

一个使用 Lua 编写、由 [lazy.nvim](https://github.com/folke/lazy.nvim) 管理的模块化
Neovim 个人配置，设计思路参考 [NvChad](https://github.com/NvChad/NvChad)，
开箱即用地支持编辑、LSP、补全、Git、终端与 AI 助手。

## 环境要求

| 依赖 | 说明 |
| --- | --- |
| Neovim **>= 0.11** | 使用 `vim.lsp.config` / `vim.lsp.enable` 与 `vim.treesitter.foldexpr` |
| Git | 插件引导与更新 |
| [Nerd Font](https://www.nerdfonts.com/) | 界面、补全菜单与状态栏图标 |
| `ripgrep` | `Telescope live_grep` |
| C 编译器 + `tree-sitter` CLI | 为 nvim-treesitter 的 `master` 分支编译解析器 |
| Node.js / npm | 通过 Mason 安装的部分语言服务器 |
| `opencode` 可执行文件 | AI Agent 集成（见[说明](#说明与定制)） |

## 安装

```bash
# 先备份已有配置
mv ~/.config/nvim ~/.config/nvim.bak 2>/dev/null

git clone git@github.com:wuzting/wznvim.git ~/.config/nvim
nvim
```

首次启动时 `lazy.nvim` 会自动完成引导并安装锁定版本的插件。随后执行：

```vim
:Lazy sync          " 安装 / 更新 / 清理插件
:MasonInstallAll    " 安装语言服务器与格式化工具
:TSUpdate           " 编译 / 更新 treesitter 解析器
:checkhealth        " 检查环境
```

## 项目结构

```
.
├── init.lua                     # 入口：核心配置 → 引导 lazy.nvim → 加载插件
├── lazy-lock.json               # 锁定的插件版本
├── .luarc.json                  # Lua Language Server 配置
├── AGENTS.md                    # 面向 AI 编程助手的说明
├── CHANGELOG.md                 # 版本变更记录
├── LICENSE                      # MIT 许可证
└── lua/
    ├── core/
    │   ├── bootstrap.lua        # 缺失时自动安装 lazy.nvim
    │   ├── default_config.lua   # 汇总默认配置（ui + lazy + mappings）
    │   ├── init.lua             # 编辑器选项、诊断与自动命令
    │   ├── mappings.lua         # 全部键位映射，按插件分组
    │   └── utils.lua            # 配置加载、懒加载与映射辅助函数
    └── plugins/
        ├── init.lua             # 传给 lazy.nvim 的插件清单
        └── configs/             # 各插件的配置模块
            ├── aerial.lua       ├── lspconfig.lua   ├── osc52.lua
            ├── alpha.lua        ├── mason.lua       ├── supermaven.lua
            ├── blankline.lua    ├── nvimtree.lua    ├── telescope.lua
            ├── catppuccin.lua   ├── others.lua      ├── toggleterm.lua
            ├── cmp.lua          ├── plantuml_preview.lua
            ├── gitsigns.lua     ├── treesitter.lua
            └── lazy_nvim.lua
```

## 功能特性

### 界面
- **[tokyonight.nvim](https://github.com/folke/tokyonight.nvim)** — 当前配色。
  如果更喜欢 catppuccin，`configs/catppuccin.lua` 中也保留了相应配置。
- **alpha-nvim** — NvChad 风格的启动页。
- **lualine.nvim** + **bufferline.nvim** — 状态栏与缓冲区标签页。
- **indent-blankline.nvim** — 缩进参考线。
- **which-key.nvim** — 键位提示。
- **render-markdown.nvim** — 缓冲区内的 Markdown 渲染。

### 文件与编辑
- **nvim-tree.lua** — 文件树。
- **telescope.nvim** — 模糊查找（文件、全文搜索、缓冲区、Git、帮助等）。
- **Comment.nvim**、**nvim-autopairs** — 注释与括号自动配对。
- **mini.diff** + **gitsigns.nvim** — Git 差异标记、hunk 操作与 blame。
- **hop.nvim** — 快速光标跳转。

### LSP 与补全
- **mason.nvim** — 安装与管理语言服务器、格式化工具。
- **nvim-lspconfig** — 已为下表语言启用 LSP，提供补全、定义跳转、悬停、
  引用、重命名与诊断。
- **nvim-cmp** 搭配 `cmp-nvim-lsp`、`cmp-buffer`、`cmp-path`、`cmp-nvim-lua`、
  `cmp_luasnip` — 自动补全。
- **LuaSnip** + **friendly-snippets** — 代码片段。
- **nvim-treesitter** — 语法解析、高亮、缩进与折叠。
- **supermaven-nvim** — 行内 AI 补全。

#### 支持的语言（补全 / 跳转）

| 语言 | 文件类型 | Language Server | 补全 | 定义跳转 | 备注 |
| --- | --- | --- | --- | --- | --- |
| HTML | `html` | `html-lsp` | ✅ | ✅ | |
| CSS / SCSS / LESS | `css` / `scss` / `less` | `css-lsp` | ✅ | ✅ | |
| JavaScript / TypeScript | `js` / `jsx` / `ts` / `tsx` | `vtsls` | ✅ | ✅ | 基于 tsserver |
| Vue | `vue` | `vue_ls` + `vtsls` | ✅ | ✅ | 经 `@vue/typescript-plugin` 共享 ts 能力 |
| C / C++ | `c` / `cpp` / `h` / `hpp` | `clangd` | ✅ | ✅ | `<A-o>` 源文件 ↔ 头文件切换 |
| Lua | `lua` | `lua_ls` | ✅ | ✅ | 已预置 `nvim` 全局 |
| Python | `python` | `pyright` | ✅ | ✅ | |
| CMake | `cmake` | `cmake-language-server` | ✅ | ✅ | |

> 跳转/查看相关按键：`gd` 定义、`gD` 声明、`gi` 实现、`gr` 引用、
> `K` 悬停文档、`<leader>D` 类型定义。

Mason 会安装：`lua-language-server`、`stylua`、`css-lsp`、`html-lsp`、
`typescript-language-server`、`vue-language-server`、`deno`、`prettier`、
`clangd`、`clang-format`。

> ⚠️ **一致性提示**：`vtsls` 已在 `lspconfig.lua` 中启用，但**不在**
> `:MasonInstallAll` 的安装列表里，新机器需补充执行：
>
> ```vim
> :MasonInstall vtsls
> ```
>
> 另外 `deno` 虽被 Mason 安装，但 `lspconfig.lua` 未调用
> `vim.lsp.enable("deno")`，当前不会生效。

### 终端与 AI
- **toggleterm.nvim** — 浮动 / 水平 / 垂直终端。
- **opencode.nvim** — 在 Neovim 内驱动 `opencode` AI Agent
  （依赖 **snacks.nvim**）。

## 键位映射

Leader 键为 **`<Space>`**。

### 通用
| 模式 | 按键 | 功能 |
| --- | --- | --- |
| i | `jj` | 退出插入模式 |
| i | `<C-b>` / `<C-e>` | 行首 / 行尾 |
| n | `<Esc>` | 清除搜索高亮 |
| n | `<C-h/j/k/l>` | 在窗口间移动 |
| n | `<C-s>` | 保存文件 |
| n | `<C-c>` | 复制整个文件 |
| n | `<A-j>` / `<A-k>` | 向下 / 向上滚动 4 行 |
| n | `<S-u>` | 重做 |
| n | `<leader>b` | 新建缓冲区 |
| n | `<leader>n` / `<leader>rn` | 切换绝对 / 相对行号 |
| n | `<leader>y` / `<leader>p` | 通过系统剪贴板复制 / 粘贴 |
| n | `;` | 进入命令模式 |

### 文件与搜索
| 按键 | 功能 |
| --- | --- |
| `<C-n>` | 打开 / 关闭文件树 |
| `<leader>e` | 聚焦文件树 |
| `<leader>ff` / `<leader>fa` | 查找文件 / 查找全部（含隐藏与被忽略文件） |
| `<leader>fw` | 实时全文搜索 |
| `<leader>fb` / `<leader>fh` / `<leader>fo` / `<leader>fz` | 缓冲区 / 帮助 / 历史文件 / 当前缓冲区 |
| `<leader>cm` / `<leader>gt` | Git 提交 / Git 状态 |
| `<leader>pt` / `<leader>th` | 选择终端 / 主题切换 |

### LSP
| 按键 | 功能 |
| --- | --- |
| `gd` / `gD` / `gi` / `gr` | 定义 / 声明 / 实现 / 引用 |
| `K` | 悬停文档 |
| `<A-o>` | C/C++ 源文件 ↔ 头文件切换（`clangd`） |
| `<leader>ls` | 函数签名帮助 |
| `<leader>D` | 类型定义 |
| `<leader>ca` | 代码操作 |
| `<leader>fm` | 格式化缓冲区 |
| `<leader>f` / `<leader>q` | 浮动诊断 / 诊断写入 loclist |
| `<leader>wa` / `<leader>wr` / `<leader>wl` | 添加 / 移除 / 列出工作区目录 |

### 缓冲区、注释与 Git
| 按键 | 功能 |
| --- | --- |
| `<C-Right>` / `<C-Left>` | 下一个 / 上一个缓冲区 |
| `<leader>bb` / `<leader>bd` | 选择缓冲区 / 选择并关闭 |
| `<leader>/` | 切换注释（普通与可视模式） |
| `]c` / `[c` | 下一个 / 上一个 Git hunk |
| `<leader>rh` / `<leader>ph` / `<leader>gb` / `<leader>td` | 重置 hunk / 预览 hunk / blame 当前行 / 切换已删除内容 |

### 终端与 AI
| 按键 | 功能 |
| --- | --- |
| `<A-i>` / `<A-h>` / `<A-v>` | 切换浮动 / 水平 / 垂直终端 |
| `<leader>h` / `<leader>v` | 新建水平 / 垂直终端 |
| `<leader>oa` / `<leader>os` / `<leader>ot` | 询问 opencode / 选择操作 / 开关 |
| `<leader>or` / `<leader>ol` | 将选区 / 当前行发送给 opencode |
| `<PageUp>` / `<PageDown>` | 滚动 opencode 输出 |

### 补全（插入模式）
| 按键 | 功能 |
| --- | --- |
| `<C-n>` / `<C-p>` | 下一个 / 上一个候选项 |
| `<C-Space>` | 触发补全 |
| `<CR>` | 确认选择 |
| `<Tab>` / `<S-Tab>` | 下一个 / 上一个，或在片段中跳转 |
| `<C-e>` | 关闭菜单 |
| `<C-d>` / `<C-f>` | 滚动文档 |

### 跳转与提示
| 按键 | 功能 |
| --- | --- |
| `s` / `ss` | Hop 跳转到单词 / 字符 |
| `<leader>wK` / `<leader>wk` | which-key 全部 / 查询键位 |
| `<leader>cc` | 跳转到当前缩进上下文 |

## 命令

| 命令 | 说明 |
| --- | --- |
| `:Lazy` | 插件管理界面 |
| `:Mason` | 语言服务器 / 工具管理界面 |
| `:MasonInstallAll` | 安装 `configs/mason.lua` 中列出的全部工具 |
| `:TSUpdate` | 更新 treesitter 解析器 |
| `:ToggletermList` | 列出当前终端 |
| `:ClangdSwitchSourceHeader` | 在 C/C++ 源文件与头文件间切换 |
| `:checkhealth` | 检查 Neovim 环境 |

## 定制方式

- **编辑器选项** — `lua/core/init.lua`
- **键位映射** — `lua/core/mappings.lua`；映射按插件分组，通过
  `require("core.utils").load_mappings("name")` 按需加载。
- **插件** — 在 `lua/plugins/init.lua` 中添加条目；将配置放到
  `lua/plugins/configs/<plugin>.lua`，并在条目中 require。
- **默认配置** — `lua/core/default_config.lua` 汇总了 UI 配置、lazy.nvim
  选项与映射表。

```lua
-- lua/plugins/init.lua
{
    "author/plugin.nvim",
    cmd = "SomeCommand",          -- 按需懒加载
    opts = function()
        return require "plugins.configs.plugin"
    end,
    config = function(_, opts)
        require("plugin").setup(opts)
    end,
}
```

```lua
-- lua/core/mappings.lua
M.myplugin = {
    plugin = true,
    n = {
        ["<leader>x"] = { "<cmd>SomeCommand<CR>", "do something" },
    },
}
```

## 说明与定制

- **`opencode.nvim`** 在 `lua/plugins/init.lua` 中硬编码了可执行文件路径
  `/home/wdc/.opencode/bin/opencode`，请按本机实际情况修改。
- **supermaven** 当前在运行时被关闭，因为 `configs/supermaven.lua` 中的
  `condition` 返回 `false`。将其改为 `true` 即可启用行内 AI 补全。
- **剪贴板** 使用 OSC52 进行复制，粘贴则回退到默认寄存器
  （`lua/core/init.lua`），因此在 SSH 环境下也能正常工作。
- **保存时** 会自动删除行尾空白，并保证文件末尾只有一个换行。
- **折叠** 使用 treesitter 表达式折叠，默认全部展开。

## 许可证

本项目基于 [MIT License](LICENSE) 发布，第三方插件仍遵循其各自的许可证。
