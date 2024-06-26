---
sidebar_position: 120
---

# 删掉Lazyvim自带的快捷键

> [!WARNING]
>
> **最新方法，自己 fork 一个 [Lazyvim](https://github.com/LintaoAmons/LazyVim)，把他里面的你不爽的快捷键全干掉!**

---

Lazyvim 自带的快捷键有时候不符合我的心意，直接绑定新的快捷键不一定会达到你想要
的效果，下面是我总结的如果disable默认快捷键的方法

## 粗暴方法

我会直接在绑定新的快捷键之前，删掉对应的已有的快捷键，但是这种方法并不总是有效

```lua
vim.keymap.del("n", "<leader>l", {})
```

## Lsp 相关的快捷键

因为我想要把 `gd` 绑定到 `Lspsaga` 提供的功能上，所以需要先 Disable 掉 Lazyvim
自带的绑定

> https://www.lazyvim.org/plugins/lsp#%EF%B8%8F-customizing-lsp-keymaps

这里可以理解为在 keys 这个自带的 array 里面 append 了一个 ele，这个 ele 会
disable 掉前面默认定义的快捷键

```lua
  {
    "neovim/nvim-lspconfig",
    init = function()
      local keys = require("lazyvim.plugins.lsp.keymaps").get()
      -- disable a keymap
      keys[#keys + 1] = { "gd", false }
    end,
  },
```

## 其他插件默认的快捷键

> https://www.lazyvim.org/configuration/plugins#%EF%B8%8F-customizing-plugin-specs

以 Flash 为例, 我需要把 `S` 在 visual 模式下作为 `surround` 插件的快捷键，但是Flash
现在会使用这个快捷键，所以需要先 Disable 掉它

```lua
keys = {
    ...
    -- highlight-next-line
    { "S", false, mode = { "v" } }, -- https://github.com/folke/flash.nvim/discussions/251
  }
}
```

## Debug 查看已有快捷键以及定义位置

```
:verbose map
```
