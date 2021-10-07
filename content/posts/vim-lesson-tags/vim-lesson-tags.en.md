---
weight: 5
title: "Vim Features: Tags list"
date: 2019-10-01T17:55:28+08:00
lastmod: 2019-10-01T17:55:28+08:00
draft: false
author: "Will Cashman"
authorLink: "https://dillonzq.com"
description: "Vim's Tags support"
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["emoji"]
categories: ["Markdown"]

lightgallery: true
---

This is how you get that IDE like feeling where you can navigate around codebases super fast.
This is built-in, no plugins, no configurations

## What are tags

Tags are an outline of your codebase, they record the location of important definitions like functions, type definitions, class definitions.

## Getting Started

Download ctags, also known as Exhuberant Tags, and run `ctags -R .` in your project root directory. Unless your codebase is quite large, this should be close to instaneous. You will notice that a file call `tags` has now been created in the root directory. You can open that file to see all the tags that have been generated, it should like something like this

TODO make tags for something


Now open Vim and position the cursor over the name of a function or type/class and press `Ctrl-]`. Vim should have jumped to the definition. Since Vim recognises this action as a 'jump', you can press `Ctrl-O` to return back you your original positions, and `Ctrl-I` to jump back forward.

You can also let Vim autocomplete tags with the `Ctrl-X Ctrl-t` command in insert mode, and if you want to say find the `BufferReader` struct, you can type `:tag Buf` and hit tab to let Vim show you all the matches, once you have found it, execute `:tag BufferReader` to jump to the tag.

Another interesting feature is the preview window, you can use `pts BufferReader` and Vim will open the definition in the 'preview window', this window behave pretty much exactly like any other, except you can do (TODO preview next) to cycle through multiple matches and `Ctrl-W z` to close it quickly. Its essentially Vim's solution to the hover preview that LSPs provide.

There is also a tag stack and also a `ts` command that I forgot.

You should probably make a keymapping for these and also for rebuilding the tags file
