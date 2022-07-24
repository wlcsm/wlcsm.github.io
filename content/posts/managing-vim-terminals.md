---
title: "Trick for Managing Neovim Terminals"
date: 2021-10-24
draft: false
author: "Will Cashman"
tags: ["vim"]
---

I've recently been use Neovim's terminals a lot more recently and I've found that I spend a lot of time doing
```
:term <TAB>
```
to find the terminal with my tests. Then <C-\><C-N><C-^> to get back to the file that I was editing, and if I have more than one terminal open it isn't easy to remember which terminal has my tests it in, which terminals had another thing, and so on. 

So I made a nice little Vim function to handle this:

```vim
let s:terminals = {}
function GoToTerminal(num)
	execute 'drop ' . get(s:terminals, a:num, 'term://bash')
	let s:terminals[a:num] = expand('%s')
	normal i
endfunction
```

Which I like to pair with these mappings

```vim
tnoremap <silent> <c-b> <cmd>execute 'drop ' . expand('#')<CR>
nnoremap <silent> <c-b> <cmd>call GoToTerminal(v:count)<CR>
```

If you really want, you can even avoid writing a function and do the entire thing in three lines
```vim
g:terms = {}
tnoremap <silent> <c-b> <cmd>execute 'drop ' . expand('#')<CR>
nnoremap <silent> <c-b> <cmd>exe 'drop ' . get(g:terms, v:count, 'term://bash') | let g:terms[v:count] = expand('%s') | norm i<CR>
```

So now if I want a terminal I just press [count]<C-B> where "count" is a number, or defaults to zero if not given. Then when I'm done with that terminal and want to go back to the main file, I just press <C-B>. 

Since I use the 'drop' command, if the terminal, or the original file are already present in another window, it will just move the cursor there.

If there is no terminal open for that [count], it will create a new one.
