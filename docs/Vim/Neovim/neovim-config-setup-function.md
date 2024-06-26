---
sidebar_position: 40
---

# LazyVim 插件配置原理与插件载入代码执行顺序

> 代码可以在 https://github.com/LintaoAmons/print-config.nvim 找到

下面是这个示例插件的文件树

```bash
print-config.nvim on  main
❯ tree .
.
├── README.md
├── lua
│   └── print-config
│       ├── config.lua
│       └── init.lua
└── plugin
    └── print_config_plugin.lua # ⭐️

4 directories, 4 files
```

当插件开始加载时，位于插件repo下的 `plugin` 文件下的脚本首先被执行，在本示例中，这个文件就是 `plugin/print_config_plugin.lua` 

```lua title=plugin/print_config_plugin.lua
vim.print("This is the start point(can be consider as MainFunction/EntryPoint) when loading a plugin to neovim")

-- you can do anything you want by writing lua scripts
require("print-config").setup({
  called_by = "plugin/print_config_plugin.lua"
}) -- here I just call the setup method

vim.print("This is the end of Loading a plugin")
```

在代码中，我会执行一系列打印操作，这样，我们在加载插件的时候，就能够从中了解到代码执行的顺序

还需要注意的是这个 `setup` 方法，它是插件本身的一个方法，我这里强调的插件本身，是指你可以直接在 `require("插件名")` 之后拿到的方法，而不是 `require("插件名.其他包)"` 才能拿到的方法

因为这是一个 neovim 插件配置的 convention，这也是 `lazy.nvim` 配置插件的主要依据，下面就是我们如何在用 `lazy.nvim` 加载本插件的代码

```
  {
    dir = "/Volumes/t7ex/Documents/oatnil/beta/print-config.nvim", -- 1. dir 是因为我这个插件是本地的插件
    -- "LintaoAmons/print-config.nvim" -- 1. 你也可以直接声明 `github` 上的插件
    opts = {
        called_by = "lazy.nvim opts" -- 2. 注意这里的值，我们在最后加载插件的时候会通过这个值来辨别插件的代码执行顺序
    },
  },
```

好了，有了这一系列的代码，我们启动 `neovim` 的时候，就可以看到输出的信息

```
1: This is the start point(can be consider as MainFunction/EntryPoint) when loading a plugin to neovim
2: Setup method called by: plugin/print_config_plugin.lua
3: This is the end of Loading a plugin
4: Setup method called by: lazy.nvim opts
```

- 第 `1,2,3` 行都是来自于加载插件运行 `plugin/print_config_plugin.lua` 中的代码打印出来的
- 第 `4` 行是在 `lazy.nvim` 中声明插件的 `opts` 调用的

我们可以看看 `lazy.nvim` 插件的 [README](https://github.com/folke/lazy.nvim#-plugin-spec) 中是怎描述 `otps` 这个字段的: `opts should be a table (will be merged with parent specs), return a table (replaces parent specs) or should change a table. The table will be passed to the Plugin.config() function. Setting this value will imply Plugin.config()`

那这里又多一个 `Plugin.config()` function, 我们再来看看这个字段的描述: `config` is executed when the plugin loads. The default implementation will automatically run `require(MAIN).setup(opts)`. Lazy uses several heuristics to determine the plugin's `MAIN` module automatically based on the plugin's **name**. See also `opts`. To use the default implementation without `opts` set `config` to `true`.

- 其实我理解，这里就是说，其实最终默认就是调用声明的插件的 `setup` 方法
	- 如果你想要修改这个调用过程，你就需要去重写这里的 `config` 字段
	- 如果你只是想要用一个自己的配置，只需要修改 `opts` 字段就好了
