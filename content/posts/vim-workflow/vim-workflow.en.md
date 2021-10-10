---
weight: 5
title: "A Vim Purist's Workflow"
date: 2019-10-01
draft: false
author: "Will Cashman"
authorLink: "https://wlcsm.github.io"
description: "A Vim Purist's Workflow"
resources:

catagories: ["emoji"]
categories: ["Markdown"]

lightgallery: true
---

I have been using Vim for about 1.5 years now and I am constantly surprised about how many features it has.

Despite using it for many smaller projected, I wasn't sold on using Vim as my main programming environment, and even then, it was only with CoC installed to essentially replicate VS Code in Vim. Until I stumbled across this video by Leeren at VimConf [](). This completely blew my mind (as did his other videos on Vim).

Its amazing to me just how many features Vim has built in and that have been nearly forgotten. Nowhere before had I seen people talk about the `compiler` setting and `include search`. I knew Vim was good at managing tags, though I hadn't even thought to learn it as I assumed that it would be default to setup, rather than the two steps it takes in reality.

Since then I have ridiculously pruned my Vim configuration, noting that I am actually using Neovim rather than Vim; its the same thing but the codebase is more actively maintained and has a more positive direction for Vim. Fortunately for me, it makes many sane defaults that Vim doesn't have which means my vimrc can be even shorter!

Now note that I am claiming Vim is a great *text editor* akin to VS Code or Sublime Text, I am not claiming that it should be used in hardcode Java/C++/C# developement (of which I don't have experience in).

Most of my work is medium sized Python, Rust, C or Go. Which don't need such environments (at least small C programs don't).

I like having a pure Vim configuration because I spend a lot of time in SSH or Docker containers and so having many key mappings gets annoying very quickly. 

Probably one of the biggest things you should do is use the Vim manual, type `:h <something>` and press tab to show a list of all help pages that match the string provided. Once inside a help page, you can press `gO` to get an overview of the page, and `Ctrl-[` over a keywork (one that is highlighted) to jump to its definition in the help pages

Here are the non-programming related 

```vim
set hidden
inoremap jk <ESC>
autocmd Filetype markdown,tex,mail setlocal spell spelllang=en_au
```

The first is the dumbest setting that I have not seen a single Vim purist not put in their config.
The second is the only keymapping that I can't seem to shake off
The last just turns on spelling when in markdown, latex, and mail

Now I don't use any plugins at the moment but there are many plugins that I would use if I were in something like web development like `vim-surround`

## `vim-commentary`
Anyway, here are some replacements for common plugins that I use

```vim
vnoremap gc :norm! i<C-R>=substitute(&commentstring, '%s.*', '', '')<CR><CR>
vnoremap gC :norm! <C-R>=len(substitute(&commentstring, '%s.*', '', ''))<CR>x<CR>
```
This is a crude implementation of commenting similar to Tim Pope's `vim-commentary`. Just select the text you wish to comment in visual mode, and press `gc` to add the comment, or `gC` to delete a comment in the first column.

This works with most filetypes however some (like Rust) it doesn't choose the correct commenting character. For that I suggest you put this in a ftplugin

```vim
vnoremap gc :norm! i#<CR>
vnoremap gC :norm! x
```
if you wanted to comment a `#` sign. Note that you need to write the same number of `x`'s as the cha TODO


## `Ultisnips`

Ultisnips is great, and is especially useful for things like web developement when you have templates where you want to edit multiple fields in the template at once or give default values. However, I don't need to do that, I just need to get a snippet and be able jump arround to edit each of the columns

```vim
inoremap ;s <ESC>:-1r ~/.config/nvim/snippets/<C-R>=&ft<CR>/
inoremap ;; <ESC>:call search('{%[^%]*%}', 'zW')<cr>c%
```

TODO mention reddit post

Then put snippets in the `~/.config/nvim/snippets/` directory assorted by file type. Then when you press `;s` it will partially complete a filepath for you. Type the snippet that you want or press tab to make Vim show a menu of all possible options.

Once it is inserted, navigate to the beginning of the snippet and press `;;` to jump to the next item. TODO should make it so that you don't need to navigate back yourself TODO the thoughtbot video I think does this partially?


## Background Make

I think this was taken from Drew DeVault?

```vim
command! -nargs=* BgMake
    \ silent execute ":!(make " . "<args>" . " > /tmp/make.output 2>&1;"
                   \ "notify-send 'make finished' 'make <args> finished') &" |
    \ redraw! |
    \ cfile /tmp/make.output | copen
```


## Basic Goto-Definition

In basic python projects, we might want to find some stuff so just use the `include search` functionality.

## VS Code like definitions

Just use a tags file. All you need to is install `ctags` then use tags.
If you want intellisense, just use the preview window for tags

## Vimtex

Vimtex is seriously great, but I found that I was only using two of its (many) features:

1. Compiling and error navigation
2. Concealing

The first I replaced by just using a make file and using the Vim `make` command and telling it what `compiler` to user

Concealing was important to me at the time since I was doing a Honour in Mathematics, and as you can image, Vim simplifies it sooo much, but I just copied the concealing that vimtex does out of its source code and put the ones I needed into put this into my `ftplugin/tex.vim` file
```vim
syn match texMathSymbol '\\C' contained conceal cchar=ℂ
syn match texMathSymbol '\\F' contained conceal cchar=𝔽
syn match texMathSymbol '\\N' contained conceal cchar=ℕ
syn match texMathSymbol '\\Q' contained conceal cchar=ℚ
syn match texMathSymbol '\\R' contained conceal cchar=ℝ
syn match texMathSymbol '\\Z' contained conceal cchar=ℤ
```
In fact this was an improvement since I got so simplify vimtex's `mathbb{C}` to just `\C`.

Also, Vim has its own latex syntax highlighting which doesn't do some environment by default, but you can add them yourself. For instance, to add the `align` environment you can add this to your `ftplugin/tex.vim`

```vim
call TexNewMathZone("MyAlignGroup","align",1)
```

Again, I highly reccoment reading Vim's builtin `tex.vim` file in the `$VIMRUNTIME` which already does a lot for you!

## Markdown and Latex

I began to start using Markdown to write Beamer presentations, so I added this to include latex syntax highlighting in my markdown documents

```vim
command! -bang TexMarkdown :call TexMath(<bang>)
function! TexMath(full_tex_highlighting) abort

    unlet b:current_syntax

    if a:full_tex_highlighting ==# '!'

        " Source the entire Tex syntax file
        source $VIMRUNTIME/syntax/tex.vim
        " Make highlighting work with the align environment
        call TexNewMathZone("MyAlignGroup","align",1)
        " Make it so that % doesn't begin a latex comment
        syn clear texComment

    else

        syn include @tex syntax/tex.vim
        syn region mkdMath start="\\\@<!\$" end="\$" skip="\\\$" contains=@tex keepend
        syn region mkdMath start="\\\@<!\$\$" end="\$\$" skip="\\\$" contains=@tex keepend

    endif

    let b:current_syntax='markdown'

    " Get all of my Latex concealments
    source /home/wlcsm/.config/nvim/ftplugin/tex.vim

endfunction
```
Note that running `TexMarkdown` included all Tex syntax highlighting, whereas `TexMarkdown!` includes only the equation syntax highlighting


## Plasticboy markdown

```
let g:markdown_folding=1
let g:markdown_fenced_languages = ['rust', 'bash=sh', 'vim']
```

This uses Vim's built-in features for folding and syntax highlighting for fenced languages. The only thing that would be nice is better support for the YAML header that can be provided. I don't write documents with that header often though (only these blog posts actually` in which case I didn't feel the need to include it

## Todo Vim

Managing todo lists. I used to use a plugin for it. Now I just grep in the project directory
```vim
:lgrep -R "TODO" .
```
assuming you are in your project directory. This populates your location list with all the TODOs I've left in my project and allows me to just to each one. You can map it to a key if you plan on using it often


# Include Search

I need to say, if you do a `[I`, then to go to the Nth item, just press `N[\t`. Side note: I literally cannot find this documented in the manual, I may have missed it though. I only know because it is used in a suggested key mapping in the manual.

# Oldfile

You can also use old files

# Marks

Workflow


# NerdTree

TODO
Input the default settings and the `20vs .` command

# Session management

Just use `mksession`, the only problem it that it doesn't remember unlisted buffers. i.e. the quickfix list, location list, netrw, and help pages. If you want to keep netrw open just execute `setlocal bl` to list the buffer, then it will remain when you same the session.

TODO see if that works with the quickfix list
