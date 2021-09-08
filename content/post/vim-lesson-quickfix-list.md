---
author: "Will Cashman"
title: "Vim's Quickfix/Location list"
date: "2021-03-13"
description: "Vim's Quickfix/Location list"
series: ["vim"]
---

People often complain that Vim doesn't have good support for project-wide searches or error navigation without the use of plugins or even LSPs. These people have clearly never learned about Vim's inbuilt quickfix and location list.


## Quickfix list

The quickfix list is a buffer in Vim which provides fast navigation to locations in files. 


## Differences between the Quickfix and Location list

The two are exactly the same, however there is only one quickfix list per Vim session, but there is one location list for every window. For this reason, the quickfix list is normally used for the errors of your program or build results, whereas the location list is normally used for things like `grep`. However you can use either for anything.

Every command for the quickfix list has an equivalent for the location list. 

Command | Quickfix | Location 
Open | `copen` | `lopen`
Next entry | `cnext` | `lnext`
Make | `make` | `lmake`
Grep | `grep` | `lgrep`
Older list | `colder` | `lolder`

There are heaps more, but for the rest of this post I will just be presenting the quickfix variant with the understanding that everything has a location list equivalent

## Using

So here's an example problem. I need to build my program and find the errors.

I've set `make` to run `go build` which presents me with a list of errors. 

Side note: to make that initial pop-up display of the errors disappear, just run `:silent make` (I'd recommend mapping this to a key).

Use the output of `make` to build a file. More on that [here](). Depending on the code you are running you may have to set a `compiler` so that Vim knows how it should parse the output. 

Notice that if there were errors, Vim will automatically jump to the location of the first error (you can stop that by running `:make!`). To go to the next error press `:cnext`. Now type `:clist` to get a pop-up of which error you are located at in the list. To get an interactive list, type `:copen` to open an interactive buffer for the quickfix list. Pressing `Enter` on any entry in this buffer will make you jump to the error.

Tim Pope has a plug-in which makes some good default mappings for this, or you could just do `:cn` and then press `@:` which repeats the last ex command

Press `:clist` to get a little pop-up of the quickfix lists and a 

