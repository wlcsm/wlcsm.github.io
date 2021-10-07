---
weight: 5
title: "Vim for Software Development"
date: 2019-10-01T17:55:28+08:00
lastmod: 2019-10-01T17:55:28+08:00
draft: false
author: "Will Cashman"
authorLink: "https://dillonzq.com"
description: "Vim for Software Development"
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["emoji"]
categories: ["Markdown"]

lightgallery: true
---

Most people think Vim is an outdated piece of software development, useful perhaps editing small files over SSH, but surely no one uses it as their main code development tool right?

Here I am going to go through what I think are the key components to a normal software development. Of course, Vim doesn't give you the complex build environemtns of a proper IDE, but if you are working comfortably in another text editor like VS Code or Sublime Text, then theres a good chance that everything you are doing can be done in vanilla Vim. Thats right, I'm going to show all these things with a minimal vimrc with *no plugins*. 

Small note: I am actually talking about Neovim rather than Vim. Its the same thing but a better code base and friendlier community, and also sets many sane defaults (which means our vimrc can be even smaller!)

First, what do we want out of a text editor for software development:
		
1. Browse Files
2. Goto definition
3. Autocompletion
4. Build/Run/Test code

## Browse files

If you are used to VS Code's project drawer set-up where you have a line on the side to view your files, you can install the NerdTree plugin which is used by many, or the preinstalled `netrw` file viewer. Just type `:30vs .` to spawn a side window 30 columns wide, in a vertical split, in the current working directory.

My default it doesn't behave quite like VSCode's (but NerdTree does), but we can change that by using the tips in [this blog](https://shapeshed.com/vim-netrw/) putting the following lines in your vimrc
```vim
let g:netrw_banner = 0
let g:netrw_liststyle = 3
let g:netrw_browse_split = 4
let g:netrw_altv = 1
let g:netrw_winsize = 25
augroup ProjectDrawer
  autocmd!
  autocmd VimEnter * :Vexplore
augroup END
```
Now `netrw` will open in the split whenever you enter Vim and behave how the same as in VS Code.

Searching for files:

```
:find <file name>
```

Note that this will search in the current working directory *for Vim*, this is the current working directory in your terminal when you start Vim. If you want to see what it is currently set to, `:pwd`, if you want to change it `:cd <new directory>`

You can use wildcards to fuzzy match
```
:find *.txt
```
Will find all `txt` files etc.

If you want recursive searching for files just use

```
:set path+=**
```

To search inside files we can all use `grep`, which is a wrapper around the `grep` shell utility.

```
:grep <args>
```

This executes the shell command `grep -n <args>` and feeds the output into the quickfix list for you to easily navigate. You can use `lgrep` to put them into the location list. More here on how to use the Quickfix list

If you would prefer to use a different searching program, set the `grepprg` which is the shell command Vim will use to find the matches. It is set to `grep -n` by defualt. Just make sure the output includes the filename and the line number so that Vim can help you easily navigate to them, this is why Vim uses the `-n` option in `grep` to add the line numbers

For example, I prefer to use [ripgrep](TODO) because it's faster. I also do a fair amount of Rust programmings at the moment and so I don't want Vim to search in the `targets` directory (the build directory in rust), so I have this in my rust filetype plugin file

```vim
set grepprg=rg\ -rn\ -g\ '!target/**'
```

So that it uses ripgrep `rg` with line numbers, and ignores the `target` directory


## Goto Definition/Hover

There are actually many actions for this in Vim because Vim doesn't perform lexical analysis TODO is it lexical?

There are several main mechanisms:

* `gd`: Goes to the first occurrence on the object in the current method. Note that this only really works with some language that have the concept of a 'method' defined, namely bracket delimited languages
* Include search: This actually searches through the current file and all files that this file imports (provided that the `includepath` variable has been correctly configure). Pressing `[I` will list all the occurrences of the word under the cursor in these files. If you want to go to the Nth line in the list it shows, press `N[\t`, that it, the number, then a `[`, then a tab. (at can't even find this is in the manual lol). There are actually many more navigation commands with include search, to see the all, use `:h include-search`
* Tags: This is probably the greatest thing. There is also a preview window which you could use kind of like the hover functionality given by LSPs. TODO
* `vim-lsp`: Technically in Neovim 5.0, this is no longer a plug-in. It is definitely very useful, however I haven't really needed to use it as I've found that using tags for definitions of: functions, type, etc. and include search for variables has been more than I need for all of my development. Note: If you use `cscope` then you can also use tags for local variable names too.


Also note the use of marks when you want to return to older positions TODO expand

## Autocompletion

Vim has some really great autocompletion. You can press `Ctrl-n` in insert mode to get the generic completion, which will actually match on things in the dictionary as well as words in any open buffer. You can also press, `Ctrl-I` to get completion from include search (even if the included files are not in a buffer), `Ctrl-f` to autocomplete file name, I use this so much..., there's a tag completions I thing as well.

The only thing that is missing is method completion, (TODO is it though?)

