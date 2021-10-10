---
weight: 5
title: "Using nvim-lsp for Go"
date: 2021-01-01
author: "Will Cashman"
authorLink: "https://wlcsm.github.io"
description: "Using nvim-lsp for Go"
resources:

catagories: ["go", "vim"]

lightgallery: true
---

Now that nvim 0.5 is now stable I decided to finally try moving from CoC to the builtin LSP client nvim-lsp. This is mainly because its more minimalist and faster.

I previously trried the nvim-lsp back in unstable a year ago but I just couldn't get things to work properly. There was a lot of configuration, and after a day or so I still couldn't get it to reliably work. Now that 0.5 is stable I figured that they have solved the problems by now. I am now using a differnt language now (Go instead of Rust) though I doubt that was the problem. 

Overall I still find the experience not as smooth as I would have hoped but everything is working fairly nicely now so I'm happy. 

The main thing to note it that the configuration is still not as simple as with CoC which is probably why it hasn't caught on as much as CoC did. Yes it fairly straight forward for the knowledgable Vim user, to whom spending an hour setting up LSP support is considdered a victory. I don't see this being used by people just coming to vim. With CoC it was a VSCode-like, plug'n'play experience where you could copy a tutorial config (of which there are many compared to nvim-lsp tutorials) do a quick ':CocInstall' and you're done, everything works right out of the box.

Here, I still had many isses

# Requirements


This is only supported in Nvim 0.5. This is currently. You will need to install 'nvim-lspconfig' to enable 

Heres what we are going to setup. There is a link to the full config down below, but otherwise we are going to set it up in logically separate steps

* LSP - 'nvim-lspconfig'. Gives linting errors, go to definition/declaration/type declaration/rename etc
* nvim-cmp - Autocompletion engine. Press Ctrl-N to get a list of all possible values
* Telescope - Show off some fuzzy finding


## LSP


