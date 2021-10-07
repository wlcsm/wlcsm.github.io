# Vim Tricks: Building and Running code



Vim has a built in support for building files through the `:make` command.

First you will want to set your compiler so that Vim knows what command it should run with `make` and what the error format should be when it parses it in the quickfix/location list.

## Build/Run/Test Code

I would say this is one of the other big reasons people don't use Vim. However Vim actually has fairly good support for this as well. Of course you aren't going to get the same level as an IDE, but TODO

We use the `make` command. This defaults to running `make` in the terminal and outputting the errors into the Quickfix list for you to easily navigate. If you want to change the make program, simply set a different `makeprg`. For rust I have it set to `set makeprg=cargo\ check\ %` Note that you need to escape spaces with a backslash, and that '%' denotes the current file's name in Vim.

Also if you want the LSP-like experience where you want Vim to be constantly telling you about the errors, then you can just make an autocomand to tell Vim to run the `make` command every time you write to the buffer, however, I'd recommend first reading the next paragraph

The most annoying thing about this, that I'm still surprise Vim has is that this runs the make command in the foreground... i.e. the rest of Vim is suspended until it finished. That is just crazy, the idea is mainly just to do syntax check which should be very quick but still, its quite annoying especially if you wanted it to run after every write. Now there are plugins for this, however, I stole this code from someone TODO who

```vim
command! -nargs=* BgMake
    \ silent execute ":!(make " . "<args>" . " > /tmp/make.output 2>&1"
    \ redraw! | cfile /tmp/make.output | copen
```

Which runs the come in the background. You can also make it send a notification when its done if you are doing a very long build 

```vim
command! -nargs=* BgMake
    \ silent execute ":!(make " . "<args>" . " > /tmp/make.output 2>&1;"
                   \ "notify-send 'make finished' 'make <args> finished') &" |
    \ redraw! | cfile /tmp/make.output | copen
```

Just run `:BgMake <args>` instead of `make <args>` and you are good to go.

## Custom error formats

