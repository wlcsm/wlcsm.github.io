---
title: "Neovim Async Jobs (No Plugins)"
date: 2021-11-06
draft: false
author: "Will Cashman"
tags: ["vim"]
---

Neovim introduced asynchronous job control which load to plugins enabling you to run shell commands without blocking Vim, most notably the fantastic ([Neomake](https://github.com/neomake/neomake)) plugin. For my development however, I only really need to run shell commands asynchronously and load the data into the quickfix list.

Neomake is great, but it's also 13000+ lines of code, this is one:

```vim
command! -nargs=+ AsyncRun call jobstart(['sh', '-c', '<args> > /tmp/run.log 2>&1'],
	\ {'on_exit': {-> execute('cfile /tmp/run.log')}})
```


1. `command! -nargs=+ Run` - Define a command called 'Run' which must be given at least one argument
2. `call jobstart`
	- `['sh', '-c', '<args> > /tmp/run.log 2>&1']` - Execute the arguments of the command and redirect the output to `/tmp/run.log`
	- `{'on_exit': {-> execute('cfile /tmp/run.log')}})` - When the program exits, load the file `/tmp/run.log` into the quickfix list


Now `:call AsyncRun('go build')` will run `go build` in the shell asyncrohnously and load the errors in the quickfix list. It you find that you can't jump to the errors, make sure the `errorformat` variable is set correctly. For example if you were to run `go build` you should set your compiler to `go` with `compiler go`.

There is one problem with this: since we are writing to the same file every time, running multiple commands at once will cause problems, but this is something that I don't really need at the moment and could be easily added later (an example is given at the end)

## Extensions

This is pretty sufficent for my needs by it could be extended in many ways. Here are a couple of fairly common quality-of-life improvements. First, a quick refactor it to make it easier to add changes

```vim
let s:logfile = '/tmp/run.log'

function AsyncRun(cmd) 
	return call jobstart(['sh', '-c', a:cmd . ' > ' . s:logfile . ' 2>&1'],
		\ {'on_exit': function('s:OnExit')})
endfunction

function s:OnExit() dict
	execute 'cfile ' . s:logfile
endfunction
```

### Open the quickfix list if errors exist

```vim
function s:OnExit() dict
	" ... rest of code
	cwindow
endfunction
```

### Print a message if there aren't any messages

```vim
function s:OnExit() dict
	" ... rest of code
	if len(getqflist()) == 0
		echom "Finished"
	endif
endfunction
```
### Managing multiple asynchronous tasks

The `jobstart` function returns a job id which you can use to keep track of it. We could change the function to not allow you to run the same command while the previous command is still running.

```vim
let s:jobs = {}
let s:logfile = '/tmp/run'
let s:num = 0

" Submit a job into the dictionary
function! AsyncRun(cmd)
	if has_key(s:jobs, a:cmd)
		echo "Job '" . a:cmd . "' is already running!"
	else
		let s:num += 1
		let outfile = s:logfile . s:num
		let id = jobstart(['sh', '-c', a:cmd . ' > ' . outfile . ' 2>&1'],
			\ {'on_exit': function('s:OnExit')})
		let s:jobs[a:cmd] = [id, s:logfile . s:num]
	endif
endfunction

function! s:OnExit(id, code, event) dict
	let cmd  = filter(items(s:jobs), {i,v -> v[1][0] == a:id})[0][0]

	execute 'cfile ' . s:jobs[cmd][1]

	" ... rest of code

	" Delete the log file and remove the job from the dictionary
	call system('rm ' . s:jobs[cmd][1])
	call remove(s:jobs, cmd)
endfunction

function! PrintJobs()
	for [key, value] in items(s:jobs)
		echo value[0] . ': ' . key
	endfor
endfunction
```

This also added the `PrintJobs()` function to print all the running jobs

This doesn't even include things like stopping jobs prematurely, and we haven't even touched the `on_stdin` and `on_stderr` callbacks for the `jobstart` function. At the moment, I've found our original one line function along with printing the result when finished to be all I want for now. 
