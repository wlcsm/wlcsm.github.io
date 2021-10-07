---
weight: 5
title: "Mail Setup"
date: 2019-10-01T17:55:28+08:00
lastmod: 2019-10-01T17:55:28+08:00
draft: false
author: "Will Cashman"
authorLink: "https://dillonzq.com"
description: "Documentation of my Email setup"
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["emoji"]
categories: ["Markdown"]

lightgallery: true
---

# Email

My email setup is a little complicated because I maintain a local copy of my mail on my computer for backup reasons. It is also nice because it is faster.
I use Neomutt which is a terminal based email client. This is because I like the minimalism and I just think that email is something that is so fundamental that it should be in a terminal. I have tried other lightweight clients like Sylpheed and I still prefer it. (I do like Apple Mail though).

- picture

However, in case you don't want to go down that rabbit hole, I will present my configuration in layers of difficulty. These being

* Using Neomutt (a terminal-based client) without downloading locally
* Storing local mail
* Automatic synchronisation and desktop notifications

This works on Linux and Apple, not a clue whether it works on windows

## Neomutt

Here we will setup a minimum example


## Storing local mail

I spent way too long trying to do this before and about a month after I finally started using it properly, I discovered Luke Smith's [mutt-wizard]() which handles all of it for you as well as providing sane defaults and color schemes

I will briefly go over what mutt-wizard sets up.

Previously we used IMAP to carry out all of our mailbox operations. Now we will be performing all of them locally (which is much faster!), and then using `isync` (or `mbsync` as it was previously known) to synchronise the local changes with your remote mail server. This goes the other way as well, when changes happen to the remote server, like receiving new mail, that also gets downloaded.

Once all the mail is nice and synchronised in our local computer, we need to index the mail in order to actually access it in a reasonable way. The indexer does things such as searching for mail, filtering by different tags and so on. Neomutt acts mainly as a front-end for it, though Neomutt also has its own built-in capability. Since so much work is done by notmuch you can just use a more [lightweight front-end](). For simplicity we will just be using Neomutt here because thats what mutt-wizard uses.

So now lets see what happens

1. Synchronise mail (`isync`)
2. Index mail (`notmuch`)
3. View mail (`neomutt`)
4. Send mail (`msmpt`)

So to synchronise our mailboxes (and hence get new mail) we can run `mbsync -a` to synchronise all mailboxes, or `mbsync <your.email@address>"` to synchronise a specific mailbox.

To index mail, we just need to run the `notmuch new` command to update the notmuch database

To view mail we just type

```
neomutt
```

To start the Neomutt client. It has Vim navigation (thanks to the mutt-wizard script). You can type `h` to find all the other commands

Though the exact key binding may be different with some mail providers offering different mailboxes, some common useful ones are:

* `o` - Get new mail for this mailbox
* `O` - Get new mail for all mailboxes
* `ga` - Go to archive (if your mailbox has one by default. Gmail doesn't)
* `gt` - Go to Trash
* `gs` - Go to Sent
* `Ma` - Move to archive
* `Mt` - Move to trash


## Automatically poll for mail and desktop notifications

I first note that you can make it so that it runs without polling whenever you get new mail using the IMAP IDLE API, a good program for this is `goimapnotify` [arch wiki](). However after trying this method, I decided to only poll every 30min otherwise I found myself dashing to answer every email that popped up and it was distracting my work flow.

Let make a little script that: synchronises our mailboxes, indexes the mail, and makes a desktop notification if there is new mail

get-mail.sh

```zsh
#!/usr/bin/env bash

# Sync with remote mailbox
mbsync -a

# Index new mail locally and save the final line of standard output
NEWMAIL=$(notmuch new | tail -1)

# Make a notification with the final line of output with no expiry time
if [[ "$NEWMAIL" =~ No\ new\ mail.* ]]; then
	notify-send -t 100000 "$NEWMAIL"
fi
```

In the second line we take the last line of the `notmuch new` command which indicates the number of new files (as well if any other changes occurred) and if there was mail, then we make a desktop notification which expires in 100000 seconds with the contents of the last line. 

Then make this file executable with `chmod +x get-mail.sh`

### Automatically calling it

Now we need to autmatically call the program every X mins. This is specific to your operating system, Mac has launchd, Linux has crontab or systemd. We are using linux so the simplest option is probably to make a cron job, however since I have been meaning to become better at linux server administration, I decided to use systemd's timers.

First we make a service /etc/systemd/user/get-mail.service 

```systemd
[Unit]
Description=Get mail
Type=oneshot

[Service]
ExecStart=/bin/sh /usr/bin/getmail
```

Then a following timer to call the service every 30mins

```systemd
[Unit]
Description=Run my "get-mail" service every 30 mins

[Timer]
OnBootSec=30min
OnUnitActiveSec=30min

[Install]
WantedBy=timers.target
```

we can then start this with

```
systemctl --user daemon-reload
systemctl --user enable get-mail.timer
systemctl --user start get-mail
```

we can see the active timers with 

```
systemctl --user list-timers
```
to get an output like

```
NEXT                        LEFT      LAST                        PASSED        UNIT                         ACTIVATES
Sun 2020-12-20 02:17:06 EST 5min left Sat 2020-12-19 02:17:06 EST 23h ago       systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
Sun 2020-12-20 02:19:06 EST 7min left Sun 2020-12-20 02:09:06 EST 2min 42s ago  get-mail.timer               get-mail.service
n/a                         n/a       Fri 2020-12-18 02:13:47 EST 1 day 23h ago grub-boot-success.timer      grub-boot-success.service

3 timers listed.
Pass --all to see loaded but inactive timers, too.
```

We can see the next time the get-mail.timer to run. If you see `n/a` in the `NEXT` section then something has gone wrong. You can check the logs using
```
journalctl --user -u get-mail
```
