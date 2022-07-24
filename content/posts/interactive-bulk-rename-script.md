---
title: "Bulk Rename Script"
date: 2021-01-01
draft: false
author: "Will Cashman"
tags: ["shell"]
---

I spend pretty much all my time in the terminal. After some time, I find I can navigate just as fast, if not faster there than in a GUI file browser. 

I used to use the ultra-lightweight [nnn](https://github.com/jarun/nnn) terminal file browser, but even that I found unnecessary other than being very nice to look at. 

The one feature that I missed however, was bulk renaming files in my editor. When you press `r` in `nnn`, it brings up the files in the directory in the program specified in your `$EDITOR` variable, with numbers next to them. Here you can delete files an rename them, which if like me, you are using Vim, can be very fast. 

```bash
1 a.txt
2 b.txt
3 c.txt
```

Rather than download the 10,000 lines of C code, I decided to write a small (POSIX compliant) shell script to automate this procedure. 


```bash
#!/bin/sh

# Output a list of files in the current directory
get_files () { ls -A; }

get_files | awk '{print NR " " $0}' > /tmp/stuff.txt

$EDITOR /tmp/stuff.txt

if [ "$1" == "-d" ]; then
	echo "Running in dry mode"
fi

i=0
(read -r num new_line
for line in $(get_files); do
	 i=$((i + 1))
	if [ -z "$new_line" ] || [ "$num" -gt "$i" ]; then
		[ "$1" == '-d' ] &&  echo "Removing $line" || rm -rf "$line"
	else 
		if [ "$new_line" != "$line" ]; then
			[ "$1" == '-d' ] &&  echo "Moving $new_line to $line" || mv "$new_line" "$line"
		fi
		read -r num new_line
	fi
done) < /tmp/stuff.txt
```
It can take one argument "-d" which runs it in "dry mode", showing the changes it would make without actually doing it

You might be wondering why I did simply use `ls` rather than create the `get_files` function. This is because apparently, the `ls` function is not guaranteed to output the correct files all the [time](https://github.com/koalaman/shellcheck/wiki/SC2045) (literally why tho?) and should instead iterate over globs, however unless the `nullglob` option is set, POSIX shell will not expand the glob if there are no files in the directory which would lead to an error. 

The first line gets the files and uses the `awk` utility to attach numbers to the lines, this is so that we can detect deletions of files.

If you wish to debug the program, you could simply replace the `rm -rf "$line"` with `echo "Removing $line"` and similarly with `mv "$new_line" "$line"`. If you want you could even make this is into a debug command by passing in a command line flag and

```bash
if [ "$1" = "-d" ]; then
	echo "Removing $line" 
else
	rm -rf "$line"
fi
# Or more succinctly as
[ "$1" = "-d" ] && echo "Removing $line" || rm -rf "$line"
```

Then when you call the program as `rename.sh -d` it will only echo the changes rather than make them
