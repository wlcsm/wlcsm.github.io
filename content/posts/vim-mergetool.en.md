---
title: "Vim for Git Merging"
date: 2021-01-01
draft: false
catagories: ["vim"]
---

This is the config for my vim setup for resolving git merge conflicts.

```
[alias]
	mt = mergetool -t vimdiff

[mergetool]
	writeToTemp = true

[mergetool "vimdiff"]
 	cmd = nvim -d $MERGED $LOCAL $BASE $REMOTE -c 'wincmd J'
```

1. The alias mt means I can type `git mt` rather than the more verbose `git mergetool -t vimdiff`
2. `writeToTemp` makes git put the BASE, LOCAL, REMOTE files into your computer's temp directory.
 - BASE The file before the conflicting changes
 - LOCAL the 'local' changes
 - REMOTE the incoming changes

          BASE
         /    \
      LOCAL   REMOTE

By default git will put these files in the same directory as the merged file. So your editor may throw lots of errors. If you are using Golang like me, it will complain that that the functions have been defined more than once. With this option set, git will put the files in your computer's temp directory (varies with the operating system).
3. The `vimdiff` mergetool.
  * `nvim -d $MERGED $LOCAL $BASE $REMOTE` - Start vim in diff-mode with the $LOCAL $BASE $REMOTE $MERGED files
  * `-c `wincmd J` - execute the command <CTRL-J>. The cursor will be on the $MERGED window, and so <CTRL-J> will put this window on the very bottom.

Transforming

+--------+-------+------+--------+
|        |       |      |        |
| merged | local | base | remote |
|        |       |      |        |
+--------+-------+------+--------+

Into

+-------+------+--------+
|       |      |        |
| LOCAL | BASE | REMOTE |
|       |      |        |
+-------+------+--------+
|                       |
|        MERGED         |
|                       |
+-------+------+--------+


## Managing diffs


I should mention that my git workflow is quite simple and so I do not need a lot of support from Vim. If you wish to have proper git support checkout Tim Popes [vim-fugitive](https://github.com/tpope/vim-fugitive). I do all git commit, push, pull etc from the terminal, and so this means there are really only two things I need from Vim:

### 1. View `git blame`

```vim
command -range GBlame echo system("git blame -L <line1>,<line2> " . expand('%'))
```

Visually select some lines and run `:GBlame` to print the git blame

### 2. View and navigate diffs

```vim
command -nargs=* -complete=file GView enew | exe "r !git <args>" | setlocal ft=diff nomodified | norm mD
```

Explaination:
* `-nargs=\*` allow any number of arguments. Vim will replace the <args> placeholder
* `-complete=file` - pressing TAB will autocomplete file names
* `enew | exe "r !git <args>` - read the output of `git <args>` into a new buffer. e.g. `:GView diff HEAD` will run `git diff HEAD`
* `setlocal ft=diff nomodified` - set the filetype to `diff` and set nomodified. This is so that when we go to leave Vim, it won't complain that there are still unmodified buffers
* `norm mD` - Set the 'D' mark here. This is so that when I do into other files I can easily navigate back to the diff by pressing 'D

A habit I have is when I get to work in the morning, I run `:GView diff HEAD^` to see the changes that I made in the last commit to refresh my memory.

Then I run this function to jump to the actual position in the file that the cursor in on in the diff. Its not very readable, but basically it assumes the cursor is in a Universal diff format with
```diff
--- a/filename
+++ b/filename
@@ -1,<old line number> +1,<new line number> @@
-old data
+new data
```

And then jumps to the file 'filename' at <new line number>

```vim
" JumpIntoDiff: Jump to the actual file location the cursor is on
function JumpIntoDiff()
	" Get the line number for the start of the hunk in the file
	let s:diff_line_num = search('^@@', 'bn')
	let s:base_line_num = matchlist(getline(s:diff_line_num), '@@ .* +\(.*\),[0-9]* @@ .*')[1]

	" Count the number of lines deleted in this chunk
	let s:num_del = matchstr(execute(s:diff_line_num . ',.w ! grep "^-" | wc -l'), '\d\+' )

	" Get the file name. e.g. +++ b/file.name, the [6:] ignores the +++ b/
	let s:filename = getline(search('^+++', 'bn'))[6:]

	" Calculate the offset of the user's cursor from the beginning of the hunk
	let s:cursor_offset = line('.') - s:diff_line_num - 1

	" The real line number in the file
	let s:line_num_in_file = s:base_line_num + s:cursor_offset - s:num_del

	execute "edit +" . s:line_num_in_file . " " . s:filename
endfunction
```

Then make a simple keymapping to call the function e.g.

```vim
nnoremap <buffer> <leader>d <CMD>call JumpIntoDiff()<CR>
```

## Take the upper or lower diff
```vim
nnoremap <space>k :call search('^<<', 'b')<CR>dd:call search('^==')<CR>dV:call search('^>>')<CR>
nnoremap <space>j :call search('^<<', 'b')<CR>dV:call search('^==')<CR>:call search('^>>')<CR>dd
```


I also personally like to set the search as '<<<<' so I can immediately jump to the next diff, so I also put a 

```vim
call setreg('/', '<<<<')
```

in my diff.vim
