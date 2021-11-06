---
weight: 4
title: "Neovim Async Jobs (No Plugin)"
date: 2021-11-06
draft: true
author: "Will Cashman"
authorLink: "https://wlcsm.github.io"
description: "How to run shell commands asynchronously in Neovim without plugins"
resources:

catagories: ["vim"]

lightgallery: true
---

Neovim introduced the jobstart command which can run jobs asynchronously as provide a channel to interact with the job. It also provides callbacks to be called on events from the stdout and stderr channels.

There are some fantastic plugins that will achieve this for you and much more such as neomake, but for all my development, the only features I need is to run it asynchronously and load the data into the quickfix list. Neomake is 1000+ lines, this is one:

```vim
command! -nargs=+ AsyncRun call jobstart(['sh', '-c', '<args> > /tmp/run.log 2>&1'], {'on_exit': {-> execute('cfile /tmp/run.log')}})
```

1. `command! -nargs=+ Run` - Define a command called 'Run' which must be given at least one argument
2. `call jobstart(
	a. `['sh', '-c', '<args> > /tmp/run.log 2>&1']` - Execute the arguments of the command and redirect the output to `/tmp/run.log`
	b. `{'on_exit': {-> execute('cfile /tmp/run.log')}})` - When the program exits, load the file `/tmp/run.log` into the quickfix list

Now a `:call AsyncRun('go build')` will run `go build` in the shell asyncrohnously and read the errors in quickfix list. Though make sure your `errorformat` is set correctly. For example if you were to run `go build` you should have `compiler go` somewhere

You can add various quality of life improvements such as opening the quickfix list if errors exist, or printing a message if there aren't any messages, but i'll leave that up to you to decide ;)

*Sidenote*: You could also compress this into a glorious one line command:


Absolute beauty
