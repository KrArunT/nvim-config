To set Python IDE with LSP, Treesitter, autocompletion, and debugging.

Just copy this into your `~/.config/nvim/init.lua`.

---

# 📂 Neovim IDE Config for Python (`init.lua`)

```lua
-- ====================================
--   Neovim IDE Setup for Python
-- ====================================

-- Bootstrap lazy.nvim (plugin manager)
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- Load plugins
require("lazy").setup({
  -- LSP and Mason
  "neovim/nvim-lspconfig",
  "williamboman/mason.nvim",
  "williamboman/mason-lspconfig.nvim",

  -- Autocompletion
  "hrsh7th/nvim-cmp",
  "hrsh7th/cmp-nvim-lsp",
  "hrsh7th/cmp-buffer",
  "hrsh7th/cmp-path",
  "L3MON4D3/LuaSnip",

  -- Treesitter
  { "nvim-treesitter/nvim-treesitter", build = ":TSUpdate" },

  -- Debugging
  "mfussenegger/nvim-dap",
  "rcarriga/nvim-dap-ui",

  -- UI extras
  "nvim-lualine/lualine.nvim",
  "nvim-tree/nvim-tree.lua",
  "nvim-telescope/telescope.nvim",
})

-- ====================================
-- Mason & LSP (Pyright)
-- ====================================
require("mason").setup()
require("mason-lspconfig").setup({
  ensure_installed = { "pyright" },
})

local lspconfig = require("lspconfig")
lspconfig.pyright.setup({
  capabilities = require("cmp_nvim_lsp").default_capabilities(),
})

-- ====================================
-- Treesitter
-- ====================================
require("nvim-treesitter.configs").setup({
  ensure_installed = { "python", "lua" },
  highlight = { enable = true },
})

-- ====================================
-- Autocompletion
-- ====================================
local cmp = require("cmp")
cmp.setup({
  mapping = cmp.mapping.preset.insert({
    ["<Tab>"] = cmp.mapping.confirm({ select = true }),
    ["<C-Space>"] = cmp.mapping.complete(),
  }),
  sources = {
    { name = "nvim_lsp" },
    { name = "buffer" },
    { name = "path" },
  },
})

-- ====================================
-- Debugging (Python)
-- ====================================
local dap = require("dap")
local dapui = require("dapui")
dapui.setup()

dap.adapters.python = {
  type = "executable",
  command = "python3",
  args = { "-m", "debugpy.adapter" },
}

dap.configurations.python = {
  {
    type = "python",
    request = "launch",
    name = "Launch file",
    program = "${file}", -- current file
  },
}

-- Auto open/close dap-ui
dap.listeners.after.event_initialized["dapui_config"] = function()
  dapui.open()
end
dap.listeners.before.event_terminated["dapui_config"] = function()
  dapui.close()
end
dap.listeners.before.event_exited["dapui_config"] = function()
  dapui.close()
end

-- ====================================
-- UI Setup
-- ====================================
require("lualine").setup()
require("nvim-tree").setup()

-- Keymaps
vim.g.mapleader = " "
vim.keymap.set("n", "<leader>e", ":NvimTreeToggle<CR>", { desc = "Toggle File Explorer" })
vim.keymap.set("n", "<leader>f", ":Telescope find_files<CR>", { desc = "Find Files" })
vim.keymap.set("n", "<leader>r", ":!python3 %<CR>", { desc = "Run Python File" })
```

---

# ⚡ Setup Instructions

1. Install Neovim (v0.9+ recommended).
2. Create config directory:

   ```bash
   mkdir -p ~/.config/nvim
   ```
3. Save the above as `~/.config/nvim/init.lua`.
4. Open Neovim → it will auto-install plugins.
5. Install Python debug adapter:

   ```bash
   uv pip install debugpy
   ```
6. Done ✅ — You now have:

   * **Pyright LSP** (errors, hover docs, go-to-def)
   * **Treesitter** (highlighting, folding, AST parsing)
   * **nvim-cmp** (autocomplete)
   * **DAP** (debugging with debugpy)
   * **File explorer, fuzzy finder, status line**

---

