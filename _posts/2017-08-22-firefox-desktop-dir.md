---
layout: post
date: 2017-08-22
category: Notes
tags: firefox directory
description: Fixing Firefox's obsession with ~/Desktop
---

This is probably really picky, but I'm not a fan of the Desktop directory that Firefox defaults to for downloads. As soon as I install Firefox I change the default download dir, but I had noticed that Firefox still created the dir when it started, and it would be the default location when an upload window was opened.

This had been one of those things that I had lived with, never really looking in to a fix. For some reason today I decided to have a look to see if this behaviour could be fixed, and thankfully I found an answer straight away.

The file `$HOME/.config/user-dirs.dirs` sets a number of default directories, one of which is the Desktop directory. At first I attempted to change the value in the file, and this worked until I rebooted my system. After a reboot the file was regenerated and the Desktop value was back.

If you have the `xdg-users-dirs` package installed you can run the following
```
xdg-user-dirs-update --set DESKTOP $HOME/
```

If you don't have the package installed you can stop XDG from regenerating the directory by doing the following
```
cat >> $HOME/.config/user-dirs.conf << EOF
enabled=False
EOF
```

Solutions found at the following links
- [Arch Linux forums](https://bbs.archlinux.org/viewtopic.php?pid=996905#p996905){:target="_blank" rel="noopener noreferrer"} 
- [AskUbuntu](https://askubuntu.com/questions/48446/how-to-make-permanent-change-to-config-user-dirs-dirs){:target="_blank" rel="noopener noreferrer"}
- [UNIX StackExchange](https://unix.stackexchange.com/questions/207216/user-dirs-dirs-reset-at-start-up){:target="_blank" rel="noopener noreferrer"}

