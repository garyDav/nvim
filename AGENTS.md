# AGENTS.md

Neovim config built on [LazyVim](https://www.lazyvim.org/). Most behavior comes from upstream LazyVim defaults; this repo only overrides/extends them, so prefer overriding upstream specs instead of reinventing plugin setup.

## Structure

- `init.lua` → bootstraps lazy.nvim, defines global `dd()` debug helper (`require("util.debug").dump`), then calls `config.lazy`. `vim.print` is aliased to `dd()`.
- `lua/config/options.lua`, `keymaps.lua`, `autocmds.lua` — global config hooks auto-loaded by LazyVim.
- `lua/config/lazy.lua` — lazy.nvim setup; the only place to add LazyVim extras (`import = "lazyvim.plugins.extras.*"`), global plugin defaults, `dev.path`.
- `lua/plugins/*.lua` — every file is auto-imported by lazy.nvim as a plugin spec. Custom specs are merged onto upstream LazyVim specs.
- `lua/craftzdog/*` — hand-written helper modules (`discipline`, `hsl`, `lsp`), required from keymaps/config.
- `lua/plugins/example.lua` — inert (`if true then return {} end`). Ignore it; don't treat as a template.

## Verify / debug

No build/test/lint pipeline exists (it's a config only). Verify by launching `nvim` and watching for startup errors:
- `nvim 2>&1 | tee /tmp/nvim.log`, then `:messages`, `:checkhealth`, `:Lazy log`.
- To test a Lua snippet headlessly: `nvim --headless "+lua ..." +qa`.
- Use the global `dd(...)` inside nvim to inspect values.

## Gotchas

- **blink.cmp must stay pinned to a release** (`version = "*"` in `lua/plugins/editor.lua`). Do NOT remove this: latest `main` of blink.cmp is v2, which requires `saghen/blink.lib`, a native build needing a Rust toolchain. No `cargo` is installed on this machine. The repo uses `blink.cmp` as the completion engine (`vim.g.lazyvim_cmp = "blink.cmp"` in `options.lua`).
- **Spec merge semantics**: `vim.tbl_deep_extend` merges tables but **overwrites lists**. To extend a list-valued option (e.g. treesitter `ensure_installed`), use `opts = function(_, opts) ... vim.list_extend(...)` rather than assigning the list. Documented in `lua/plugins/example.lua:141`.
- `lua/plugins/example.lua`, `lua/plugins/editor.lua`, and `lua/craftzdog/*` use **tab indentation**, while `stylua.toml` specifies 2-space/120-col. Preserve each file's existing indentation; don't run stylua on tab-indented files.

## Conventions (existing, keep)

- Colorscheme: `craftzdog/solarized-osaka.nvim` (set as `colorscheme` in `config.lazy.lua`).
- Picker: telescope (`vim.g.lazyvim_picker = "telescope"`).
- Prettier requires a config file present (`vim.g.lazyvim_prettier_needs_config = true`).
- Local plugin dev path: `dev.path = "~/.ghq/github.com"` — plugins under there load as dev versions.
- `lazy-lock.json` is gitignored (but currently tracked/committed); don't rely on it or commit updates for it.
