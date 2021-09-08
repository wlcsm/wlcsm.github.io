---
author: "Will Cashman"
title: "Markdown to HTML Using sed"
date: "2020-02-15"
description: "Markdown to HTML Using sed"
series: ["shell"]
---

I recently discovered the `:TOhtml` command in Vim. It converts the current view of the file you are editing in Vim to HTML that can be viewed in the browser. TODO I think it has some other use cases too

However, it would be nice if it could actually render the Markdown file I was editing too. I looked at the common solutions like `pandoc` but I found that they were incredibly huge programs. I decided to ignore the extra features and see if I could make a minimalistic program to convert plain Markdown to html.

It isn't perfect, but most of what I need it to do can be done with the following sed script:

```sed
# Markdown to HTML render

1r header.html

# Wraps paragraphs in the <p></p> tags
/^\w/{H;$!d}
/^$/{x;/./s/\(.*\)/<p>\1<\/p>/}

# ###.* to <hN>
s/^# \(.*\)/<h1>\1<\/h1>/
s/^## \(.*\)/<h2>\1<\/h2>/
s/^### \(.*\)/<h3>\1<\/h3>/
s/^#### \(.*\)/<h4>\1<\/h4>/
s/^##### \(.*\)/<h5>\1<\/h5>/
s/^###### \(.*\)/<h6>\1<\/h6>/

# '*' or '-' or '+' to <li>
s/^\s*[\*-+]\s*\(.*\)/<li>\1<\/li>/

# [title](url) to <a href="url">title</a>
s/\[\(.*\)\](\(.*\))/<a href="\2">\1<\/a>/g

$r footer.html
```

You will also need the header and footer for the HTML file. This is up to you but I chose these simple templates

`header.sh`
```html
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
<title>Testing</title>
```

`footer.html`
```sh
</body>
</html>
```

To run it. Save the file as `markdown-to-html.sed` and execute it on a markdown file `file.md` as

```sh
sed -f markdown-to-html.sed file.md
```
