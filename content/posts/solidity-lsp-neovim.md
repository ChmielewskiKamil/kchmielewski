+++
title = 'Solidity LSP Neovim setup'
date = 2023-02-25
draft = false
tags = ["solidity", "neovim", "lsp"]
author = "Kamil Chmielewski"
description = "A simple configuration of the language server protocol (LSP) for Solidity in Neovim."
+++

Over the past few weeks, I've been trying to set up the Solidity LSP support in my AstroNvim configuration. I've tried the [Hyperledger](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#solang) Solang,
the official [Ethereum Foundation solc](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#solc), [Quixiang's solidity-ls](https://github.com/qiuxiang/solidity-ls) and the port of the [Juan Blanco Solidity VSCode extension](https://github.com/juanfranblanco/vscode-solidity) - [solidity language server](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#solidity_ls).
Some of them, like solidity-ls and solidity-language-server, were working partially with only the syntax highlighting and autocompletion but without the go-to definition functionality or the other way around. Some of them, like solc, gave me errors whenever I jumped between files and references too quickly. And solang did not work at all, lol.

It was until [@vex](https://twitter.com/vex_0x) suggested that there exists a [NomicFoundation's LSP](https://github.com/NomicFoundation/hardhat-vscode/pull/362), and it is the best right now. [You can read the Twitter thread](https://twitter.com/kamilchmielu/status/1628851108218040322?s=20) here if interested.

<!--more-->

## How to get the full LSP support for Solidity in Neovim

[Hardhat-VSCode extension](https://github.com/NomicFoundation/hardhat-vscode) provides official support for Vim and Neovim text editors via a standalone executable language server. It means that you will be able to achieve a complete LSP experience with syntax highlighting, autocompletion, go-to definitions, finding references, and marking warnings and errors in Neovim.

### Does the hardhat LSP work with Foundry, Truffle, Brownie, etc.?

Yes, it does. Initially, I was skeptical about the hardhat LSP since Foundry is my main framework for working with Solidity. But it turns out that it works perfectly fine. From my experience, there is no need to tweak the config to support different frameworks.

### How to install the Solidity LSP?

[Hardhat for VSCode's Solidity Language Server page](https://github.com/NomicFoundation/hardhat-vscode/tree/development/server) most likely has all the information you may need to set this up in both Vim and Neovim.

The first part - LSP installation:

1.  You have to install the language server with (I am assuming that you have `Node.js` and `npm` installed):  
    `npm install @ignored/solidity-language-server -g`
2.  Once you have the language server installed, you will have access to the standalone executable for the server: `nomicfoundation-solidity-language-server --stdio`. This will be passed to the Nvim config so that the `lspconfig` knows how to run the language server for Solidity.

Once you install the LSP, you can move to the Nvim config part.

### How to set up the Solidity LSP in Neovim?

The following steps may vary depending on how you have your Nvim set up. I am using [AstroNvim](https://astronvim.github.io/), which provides an excellent out-of-the-box experience and is a solid base for further configuration.

[The Solidity Language Server page mentioned before](https://github.com/NomicFoundation/hardhat-vscode/tree/development/server#neovim-lsp) provides a general guideline on configuring the `init.lua` file (or wherever you have your LSP config).

For Astronvim, I had to tweak it a little bit. The [custom LSP definition page](https://astronvim.github.io/Recipes/advanced_lsp#custom-lsp-definition) of the official Astronvim documentation has been helpful.

1.  Add the language server to the `servers` table. It has to be in a string format. You can name it however you like. I went with "solidity".
2.  In `["server-settings"]`, you have to add appropriate config options (the same as [defined on the Solidity LSP page](https://github.com/NomicFoundation/hardhat-vscode/tree/development/server#neovim-lsp)). The `cmd` is the most important setting. Neovim will know how to start the LSP. By additionally configuring the `filetypes` and `root_dir`, the language server will start automatically whenever a Solidity file is opened.
    ```lua
    -- Extend LSP configuration
    lsp = {
        -- enable servers that you already have installed without mason
        servers = {
            -- "pyright",
            "solidity"
        },
    -- Add overrides for LSP server settings. The keys are the name of the server.
        ["server-settings"] = {
            solidity = {
                cmd = { 'nomicfoundation-solidity-language-server', '--stdio' },
                filetypes = { 'solidity' },
                root_dir = require("lspconfig.util").find_git_ancestor,
                single_file_support = true,
            },
        },
    },
    ```

With this done, you should be ready to go. Remember that you might have to restart your terminal session to apply changes.

**Update:** After using this for a while, I've stumbled upon a couple of issues in big monorepos. In Foundry projects (or in a mix of Hardhat with Foundry), sometimes the `forge-std` library is not found in some of the files. Changing the `root_dir` line in the config above solves the problem most of the time.
    
```diff
solidity = {
    cmd = { 'nomicfoundation-solidity-language-server', '--stdio' },
    filetypes = { 'solidity' },
-	root_dir = require("lspconfig.util").find_git_ancestor,
+	require("lspconfig.util").root_pattern "foundry.toml",
    single_file_support = true,
},
```

**Update v2:** Recently, I migrated away from Astronvim to [my own tailor-made Nvim config](https://github.com/ChmielewskiKamil/nvim). When it comes to the LSP support, I went with [LSP Zero](https://github.com/VonHeikemen/lsp-zero.nvim). The Solidity configuration is similar to the one described above. In your `lsp.lua` file or wherever you do the lsp configuration, add these lines:

```lua
-- given: `local lsp = require('lsp-zero').preset({"recommended"})`
-- the snippet below will add Solidity lsp to your config

lsp.use('solidity', {
    cmd = { 'nomicfoundation-solidity-language-server', '--stdio' },
    filetypes = { 'solidity' },
    root_dir = require("lspconfig.util").find_git_ancestor,
    single_file_support = true,
})
```

## References

-   My current [Neovim config](https://github.com/ChmielewskiKamil/nvim)
-   You can check my [AstroNvim config on my GitHub](https://github.com/ChmielewskiKamil/astronvim_config/blob/main/init.lua)
-   [NomicFoundation's Solidity Language Server](https://github.com/NomicFoundation/hardhat-vscode/tree/development/server)
-   [AstroNvim documentation - Advanced LSP setup](https://astronvim.github.io/Recipes/advanced_lsp#custom-lsp-definition)
-   If you have any questions DM me on Twitter [@kamilchmielu](https://twitter.com/kamilchmielu).
