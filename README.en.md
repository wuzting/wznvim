# nvim

[![CI](https://github.com/wuzting/wznvim/actions/workflows/ci.yml/badge.svg)](https://github.com/wuzting/wznvim/actions/workflows/ci.yml)

**English** | [简体中文](README.md)

A personal, modular Neovim configuration written in Lua and managed by
[lazy.nvim](https://github.com/folke/lazy.nvim), inspired by
[NvChad](https://github.com/NvChad/NvChad). It ships an out-of-the-box setup
for editing, LSP, completion, Git, terminals and AI agents.

## Requirements

| Dependency | Notes |
| --- | --- |
| Neovim **>= 0.11** | Uses `vim.lsp.config` / `vim.lsp.enable` and `vim.treesitter.foldexpr` |
| Git | Plugin bootstrap and updates |
| A [Nerd Font](https://www.nerdfonts.com/) | Icons in the UI, completion menu and statusline |
| `ripgrep` | `Telescope live_grep` |
| C compiler + `tree-sitter` CLI | Building parsers for the `master` branch of nvim-treesitter |
| Node.js / npm | Several language servers installed by Mason |
| `opencode` binary | AI agent integration (see [Notes](#notes--customization)) |

## Installation

```bash
# Back up any existing config first
mv ~/.config/nvim ~/.config/nvim.bak 2>/dev/null

git clone git@github.com:wuzting/wznvim.git ~/.config/nvim
nvim
```

`lazy.nvim` bootstraps itself on the first launch, then installs the pinned
plugin set. Finish the setup with:

```vim
:Lazy sync          " install / update / clean plugins
:MasonInstallAll    " install the language servers & formatters
:TSUpdate           " build / update treesitter parsers
:checkhealth        " verify the environment
```

## Project Structure

```
.
├── init.lua                     # Entry point: core → bootstrap lazy.nvim → plugins
├── lazy-lock.json               # Pinned plugin revisions
├── .luarc.json                  # Lua Language Server config
├── AGENTS.md                    # Guidance for AI coding assistants
├── CHANGELOG.md                 # Release history
├── LICENSE                      # MIT license
└── lua/
    ├── core/
    │   ├── bootstrap.lua        # Installs lazy.nvim when missing
    │   ├── default_config.lua   # Aggregated default config (ui + lazy + mappings)
    │   ├── init.lua             # Editor options, diagnostics, autocommands
    │   ├── mappings.lua         # All keymaps, grouped by plugin
    │   └── utils.lua            # Config loading, lazy-loading and mapping helpers
    └── plugins/
        ├── init.lua             # Plugin specs passed to lazy.nvim
        └── configs/             # Per-plugin configuration modules
            ├── aerial.lua       ├── lspconfig.lua   ├── osc52.lua
            ├── alpha.lua        ├── mason.lua       ├── supermaven.lua
            ├── blankline.lua    ├── nvimtree.lua    ├── telescope.lua
            ├── catppuccin.lua   ├── others.lua      ├── toggleterm.lua
            ├── cmp.lua          ├── plantuml_preview.lua
            ├── gitsigns.lua     ├── treesitter.lua
            └── lazy_nvim.lua
```

## Features

### UI
- **[tokyonight.nvim](https://github.com/folke/tokyonight.nvim)** — active colorscheme.
  A [catppuccin](https://github.com/catppuccin/nvim) config is also kept in
  `configs/catppuccin.lua` if you prefer it.
- **alpha-nvim** — NvChad-style dashboard.
- **lualine.nvim** + **bufferline.nvim** — statusline and buffer tabs.
- **indent-blankline.nvim** — indent guides.
- **which-key.nvim** — discoverable keymap hints.
- **render-markdown.nvim** — in-buffer Markdown rendering.

### Files & Editing
- **nvim-tree.lua** — file explorer.
- **telescope.nvim** — fuzzy finder (files, grep, buffers, Git, help, …).
- **Comment.nvim**, **nvim-autopairs** — commenting and bracket pairing.
- **mini.diff** + **gitsigns.nvim** — Git diff signs, hunks and blame.
- **hop.nvim** — fast cursor motions.

### LSP & Completion
- **mason.nvim** — installs/manages servers and formatters.
- **nvim-lspconfig** — enables LSP for the languages below, providing
  completion, go-to-definition, hover, references, rename and diagnostics.
- **nvim-cmp** with `cmp-nvim-lsp`, `cmp-buffer`, `cmp-path`, `cmp-nvim-lua`,
  `cmp_luasnip` — completion.
- **LuaSnip** + **friendly-snippets** — snippets.
- **nvim-treesitter** — parsing, highlighting, indentation and folding.
- **supermaven-nvim** — inline AI completion.

#### Supported languages (completion / navigation)

| Language | Filetype(s) | Language Server | Completion | Go-to-definition | Notes |
| --- | --- | --- | --- | --- | --- |
| HTML | `html` | `html-lsp` | ✅ | ✅ | |
| CSS / SCSS / LESS | `css` / `scss` / `less` | `css-lsp` | ✅ | ✅ | |
| JavaScript / TypeScript | `js` / `jsx` / `ts` / `tsx` | `vtsls` | ✅ | ✅ | backed by tsserver |
| Vue | `vue` | `vue_ls` + `vtsls` | ✅ | ✅ | shares TS via `@vue/typescript-plugin` |
| C / C++ | `c` / `cpp` / `h` / `hpp` | `clangd` | ✅ | ✅ | `<A-o>` switches source ↔ header |
| Lua | `lua` | `lua_ls` | ✅ | ✅ | `nvim` global preconfigured |
| Python | `python` | `pyright` | ✅ | ✅ | |
| CMake | `cmake` | `cmake-language-server` | ✅ | ✅ | |

> Navigation keys: `gd` definition, `gD` declaration, `gi` implementation,
> `gr` references, `K` hover, `<leader>D` type definition.

Mason installs: `lua-language-server`, `stylua`, `css-lsp`, `html-lsp`,
`typescript-language-server`, `vtsls`, `vue-language-server`, `prettier`,
`clangd`, `clang-format`, `cmake-language-server`, `pyright`.

> **Manual install**: to add a server outside this list, open the manager with
> `:Mason` and pick `Install`, or run `:MasonInstall <package>` (e.g.
> `:MasonInstall pyright`); then enable it in `lspconfig.lua` with
> `vim.lsp.enable("<server>")`.

### Terminal & AI
- **toggleterm.nvim** — floating / horizontal / vertical terminals.
- **opencode.nvim** — drive the `opencode` AI agent from inside Neovim
  (backed by **snacks.nvim**).

## Keymaps

The leader key is **`<Space>`**.

### General
| Mode | Key | Action |
| --- | --- | --- |
| i | `jj` | Exit insert mode |
| i | `<C-b>` / `<C-e>` | Beginning / end of line |
| n | `<Esc>` | Clear search highlights |
| n | `<C-h/j/k/l>` | Move between windows |
| n | `<C-s>` | Save file |
| n | `<C-c>` | Copy whole file |
| n | `<A-j>` / `<A-k>` | Scroll 4 lines down / up |
| n | `<S-u>` | Redo |
| n | `<leader>b` | New buffer |
| n | `<leader>n` / `<leader>rn` | Toggle absolute / relative numbers |
| n | `<leader>y` / `<leader>p` | Yank / paste via system clipboard |
| n | `;` | Enter command mode |

### Files & Search
| Key | Action |
| --- | --- |
| `<C-n>` | Toggle nvim-tree |
| `<leader>e` | Focus nvim-tree |
| `<leader>ff` / `<leader>fa` | Find files / find all (hidden + ignored) |
| `<leader>fw` | Live grep |
| `<leader>fb` / `<leader>fh` / `<leader>fo` / `<leader>fz` | Buffers / help / oldfiles / current buffer |
| `<leader>cm` / `<leader>gt` | Git commits / git status |
| `<leader>pt` / `<leader>th` | Pick terminal / theme switcher |

### LSP
| Key | Action |
| --- | --- |
| `gd` / `gD` / `gi` / `gr` | Definition / declaration / implementation / references |
| `K` | Hover documentation |
| `<A-o>` | Switch C/C++ source ↔ header (`clangd`) |
| `<leader>ls` | Signature help |
| `<leader>D` | Type definition |
| `<leader>ca` | Code action |
| `<leader>fm` | Format buffer |
| `<leader>f` / `<leader>q` | Floating diagnostic / diagnostics to loclist |
| `<leader>wa` / `<leader>wr` / `<leader>wl` | Add / remove / list workspace folders |

### Buffers, Comments, Git
| Key | Action |
| --- | --- |
| `<C-Right>` / `<C-Left>` | Next / previous buffer |
| `<leader>bb` / `<leader>bd` | Pick buffer / pick and close |
| `<leader>/` | Toggle comment (normal & visual) |
| `]c` / `[c` | Next / previous Git hunk |
| `<leader>rh` / `<leader>ph` / `<leader>gb` / `<leader>td` | Reset hunk / preview hunk / blame line / toggle deleted |

### Terminal & AI
| Key | Action |
| --- | --- |
| `<A-i>` / `<A-h>` / `<A-v>` | Toggle floating / horizontal / vertical terminal |
| `<leader>h` / `<leader>v` | New horizontal / vertical terminal |
| `<leader>oa` / `<leader>os` / `<leader>ot` | Ask opencode / select action / toggle |
| `<leader>or` / `<leader>ol` | Send range / line to opencode |
| `<PageUp>` / `<PageDown>` | Scroll opencode output |

### Completion (insert mode)
| Key | Action |
| --- | --- |
| `<C-n>` / `<C-p>` | Next / previous item |
| `<C-Space>` | Trigger completion |
| `<CR>` | Confirm selection |
| `<Tab>` / `<S-Tab>` | Next / previous, or jump in snippet |
| `<C-e>` | Close menu |
| `<C-d>` / `<C-f>` | Scroll documentation |

### Motions & Hints
| Key | Action |
| --- | --- |
| `s` / `ss` | Hop to word / character |
| `<leader>wK` / `<leader>wk` | which-key all / query keymaps |
| `<leader>cc` | Jump to current indent context |

## Commands

| Command | Description |
| --- | --- |
| `:Lazy` | Plugin manager UI |
| `:Mason` | Language server / tool manager |
| `:MasonInstallAll` | Install every tool listed in `configs/mason.lua` |
| `:TSUpdate` | Update treesitter parsers |
| `:ToggletermList` | List active terminals |
| `:ClangdSwitchSourceHeader` | Switch between C/C++ source and header |
| `:checkhealth` | Diagnose the Neovim environment |

## Customization

- **Editor options** — `lua/core/init.lua`
- **Keymaps** — `lua/core/mappings.lua`; mappings are grouped per plugin and
  loaded on demand via `require("core.utils").load_mappings("name")`.
- **Plugins** — add specs to `lua/plugins/init.lua`; put their configuration in
  `lua/plugins/configs/<plugin>.lua` and require it from the spec.
- **Defaults** — `lua/core/default_config.lua` aggregates the UI config, the
  lazy.nvim options and the mappings table.

```lua
-- lua/plugins/init.lua
{
    "author/plugin.nvim",
    cmd = "SomeCommand",          -- lazy-load when needed
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

## Notes & Customization

- **`opencode.nvim`** hardcodes the binary path
  `/home/wdc/.opencode/bin/opencode` in `lua/plugins/init.lua`. Update it to
  match your machine.
- **supermaven** is currently disabled at runtime because its `condition`
  returns `false` (`configs/supermaven.lua`). Flip it to `true` to enable inline
  AI completion.
- **Clipboard** uses OSC52 for copying and falls back to the default register
  for pasting (`lua/core/init.lua`), so it works well over SSH.
- **On save**, trailing whitespace is stripped and a single trailing newline is
  enforced.
- **Folding** uses treesitter expression folding, expanded by default.

## License

Released under the [MIT License](LICENSE). Third-party plugins remain under
their own respective licenses.
