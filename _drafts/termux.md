---
layout: post
date: 2017-06-14
category: Tech
tags: terminal termux android linux
description: Installing and configuring termux on Android
---

Do you prefer working in the terminal than with horriblely flash GUIs? I prefer it so much I even do away with typical icon based launchers on my Android phone and use a [terminal](https://github.com/Andre1299/TUI-ConsoleLauncher){:target="_blank" rel="noopener noreferrer"}.

Sometime I need more than just a launcher, and this is where [termux](https://termux.com/){:target="_blank" rel="noopener noreferrer"} comes in. Termux is an "Android terminal emulator and Linux environment app". You can download it from [Play](https://play.google.com/store/apps/details?id=com.termux){:target="_blank" rel="noopener noreferrer"} or [F-Droid](https://f-droid.org/repository/browse/?fdid=com.termux){:target="_blank" rel="noopener noreferrer"} and install without rooting your device. It comes with a number of shells to choose from and install packages using `apt`.

Once installed there is no setup required, however there are a few things that you can do to improve the environment. These steps are personal preference, and as always use the commands at your own risk!

First make sure everything is up to date
```
apt update && apt upgrade
```

If you look in the current directory you will see there is nothing there
```
pwd
  /data/data/com.termux/files/home
ls -l
```

You can set up links to the shared internal storage by running
```
termux-setup-storage
```

This creates symlinks to a number of directories in your phone's storage
```
pwd
  /data/data/com.termux/files/home
ls -1
  storage
ls -1 storage/
  dcim
  downloads
  movies
  music
  pictures
  shared
```

Use the command `ls -l` to see the links.

A number of packages are provided
```
busybox --help
```

I tend to install a number of others (in no particular order)
```
apt install openssh vim zsh less irssi tmux git stow htop
```

Then we can pull down our dotfiles!
```
git clone https://github.com/pyratebeard/dotfiles.git
cd dotfiles
stow {vim,zsh,irssi,tmux}
chsh -s zsh
```

Now exit termux using `exit` or Ctrl-D and when you restart you should be in a more comfortable environment.

There we go. A rather quick and simple run through of termux. All this information can be found on their [help page](https://termux.com/help.html){:target="_blank" rel="noopener noreferrer"}. For more help or information contact me in the usual ways, or join the #termux IRC channel on freenode.

