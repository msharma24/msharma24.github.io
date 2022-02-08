---
layout: post
title: My Neovim workflow for Terraform
---

# Demo First
  <iframe width="720" height="415" src="https://www.youtube.com/embed/hK9eI-EcB58" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
---

Here is a quick demo of using [terraformls](https://github.com/hashicorp/terraform-ls) and [tflint](https://github.com/terraform-linters/tflint) Language Server with the latest Neovim's native LSP support.

<br>
## Vim8 -> Neovim
I have recently switched from using Vim8 with [Conquer of Completion](https://github.com/neoclide/coc.nvim) for LSP configuration to the latest Neovim v0.6.1 in 
favour of the native [LSP](https://microsoft.github.io/language-server-protocol/) support, which means we can install any Language Server and a bunch of code completions and language features are just a `<Tab>` away.

Built in Language Server Configurations in nvim-lspconfig - [server configurations](https://github.com/neovim/nvim-lspconfig/tree/master/lua/lspconfig/server_configurations) 

Another killer feature which made me switch is that Neovim has an embedded [Lua 5.1 runtime](https://github.com/neovim/neovim/wiki/FAQ#why-embed-lua-instead-of-x) 
which is used to create faster and more powerful [plugins](https://github.com/rockerBOO/awesome-neovim) and provides beautiful Syntax highlighting with [Tree-sitter](https://github.com/nvim-treesitter/nvim-treesitter/wiki/Gallery) parsing library.

## Configuring Neovim for Terraform auto completions and tflint

I spend alot of time building large scale AWS infrastrcure using Terraform and integrating the tf language server and tflint boosts my productivity 10x.

**Steps to configure terraformls and tflint with Neovim LSP** 

1. Install `terraformls` [Installation link](https://github.com/hashicorp/terraform-ls#installation) 
2. Install `tflint` [Installation link](https://github.com/terraform-linters/tflint#installation) 

*Make sure the installation of these tools are in your `$PATH`* 

3. Install `vim-terraform` plugin to enforce `terraform fmt` on file save [Installation Link](https://github.com/hashivim/vim-terraform)

Once you have the nvim-cmp and lspconfig setup [Reference Link](https://github.com/neovim/nvim-lspconfig/wiki/Autocompletion) ,
Add the following to your `init.lua` file for Neovim to recognize the `hcl` and `terraform` [filetype](https://neovim.io/doc/user/filetype.html) 

```
vim.cmd([[silent! autocmd! filetypedetect BufRead,BufNewFile *.tf]])
vim.cmd([[autocmd BufRead,BufNewFile *.hcl set filetype=hcl]])
vim.cmd([[autocmd BufRead,BufNewFile .terraformrc,terraform.rc set filetype=hcl]])
vim.cmd([[autocmd BufRead,BufNewFile *.tf,*.tfvars set filetype=terraform]])
vim.cmd([[autocmd BufRead,BufNewFile *.tfstate,*.tfstate.backup set filetype=json]])

```

Add the following to automatically format *.tf and *.tfvars files with `terraform fmt` on save and align settings.
```
vim.cmd([[let g:terraform_fmt_on_save=1]])
vim.cmd([[let g:terraform_align=1]])
```

I use the following keymaps to quickly call the Terraform commmands in `Normal` mode.
```
keymap("n", "<leader>ti", ":!terraform init<CR>", opts)
keymap("n", "<leader>tv", ":!terraform validate<CR>", opts)
keymap("n", "<leader>tp", ":!terraform plan<CR>", opts)
keymap("n", "<leader>taa", ":!terraform apply -auto-approve<CR>", opts)
```

Call the terraformls and tflint PID to attach as client to the Neovim LSP 

*Reference Links to enable the language servers* 
- [terraformls](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#terraformls) 
- [tflint](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#tflint) 

```
require'lspconfig'.terraformls.setup{}
require'lspconfig'.tflint.setup{}
```

**Tree-sitter grammar for HCL language**

I would highly recommend installation of  HCL grammar for beautiful highlighting of all the `*.tf` and `hcl` files.

Run `:TSInstall hcl` as long as you have installed and configured the Tree-sitter plugin - [link](https://github.com/nvim-treesitter/nvim-treesitter) 


**Verify the config**

In your Terfaform templates git repository , open any `*.tf` file and run `:LspInfo`
and a pop-up will appear with the configured language servers.

<img src="{{site.baseurl}}/images/04/tf-ls.png">


### My Neovim Config
Feel free to take to peek at my [.dotfiles](https://github.com/msharma24/.dotfiles) 


