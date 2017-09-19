---
layout: post
date: 2017-07-11
category: Tech
tags: tech arch linux install guide
description: Setup a working environment on an Arch Linux installation
---

_Note_ This guide assumes you have a working Arch Linux installation. See my [previous post](/log/2016/11/arch-linux-install){:target="_blank" rel="noopener noreferrer"} for how to get started.

If you followed my Arch Linux install guide you should have rebooted your system and have a login prompt. If you enter the username and password you set you will log in to the system, but it's looking a bit plain. Now some people do work in the [TTY]() ([nixers](https://nixers.net/showthread.php?tid=1669){:target="_blank" rel="noopener noreferrer"} has an annual "Week in the TTY") but most of us prefer a graphical environment... even a minimal one.

To get started we need to install some packages. We will be installing the [X Window System](https://www.x.org/wiki/){:target="_blank" rel="noopener noreferrer"}, a window manager, and a terminal emulator.
```
sudo pacman -S xorg xorg-xinit awesome rxvt-unicode
```

I have chosen to install the [awesome](https://awesomewm.org/){:target="_blank" rel="noopener noreferrer"} window manager and the [urxvt](http://software.schmorp.de/pkg/rxvt-unicode.html){:target="_blank" rel="noopener noreferrer"} terminal emulator. There are so many combinations of software, which is one of the reasons I enjoy the Linux community, so don't worry if you prefer other tools. Install and use whatever works for you and don't be ashamed of your choices. Although I would say to try out other tools, you never know when you'll come across something better.

Now we need to get the window manager running. Copy some template configuration files and make some changes. The filenames begin with a dot '.' which in Linux indicates a hidden file. Most configuration files in your home directory will be hidden, and are known as 'dotfiles'. More on this later.
```
cp /etc/X11/xinit/xserverrc ~/.xserverrc
cp /etc/X11/xinit/xinitrc ~/.xinitrc
vi ~/.xinitrc
```

Comment out the following lines by entering a # at the start of the line
```
twm &
xclock -geometry 50x50-1+1 &
xterm -geometry 80x50+494+51 &
xterm -geometry 80x20+494-0 &
exec xterm -geometry 80x66_0_0 -name login
```

Write the following at the end of the file
```
exec awesome
```

Save and quit.

Now run the command
```
startx
```

And you should see the window manager start up. If you press the keys Win-r and type in _urxvt_ then a terminal will appear. If you ever need to drop out of the graphical environment for press the keys Win-Shift-Q to exit to the TTY.

The default awesome environment is nice enough, but I am what is known as a ricer. This means that I spend far too much time altering my dotfiles to customise my environment so that it works for me, and looks however I want it to. I store all my dotfiles in a git repository and use a tool called 'stow' to easily enable and disable them as required. A lot of this is personal preference, so as mentioned before don't feel compelled to copy exactly.

To set up my dotfiles I need a few more packages. I need git, so I can pull my repository down, and stow.
```
sudo pacman -S git stow
git clone "https://github.com/pyratebeard/dotfiles"
cd dotfiles
```

If you take a look at the README file it will quickly explain the tools I have files for and how to use stow to enable them. Let's install some more packages so we can get comfortable 
```
sudo pacman -S vicious zsh vim tmux qutebrowser ranger irssi mutt mpd ncmpcpp
```

Here is a quick run down of the applications that have just been installed
```
  vicious     >   plugins used by my awesome config
  zsh         >   z shell
  vim         >   text editor
  tmux        >   terminal multiplexer
  qutebrowser >   web browser
  ranger      >   file manager
  irssi       >   irc client
  mutt        >   email client
  mpd         >   audio player daemon
  ncmpcpp     >   audio player interface
```

Most of the applications I use are based in the terminal. If you're not use to working in the terminal it may be a big learning curve, but once you get the hang of it you may find it improves your workflow.

We're ready to enable our dotfiles.
```
stow {awesome,urxvt,zsh,vim,tmux,qutebrowser,ranger,irssi,mutt,mpd,ncmpcpp}
cd ~
ls -la
```

You should now see lots of hidden files which are pointing to the dotfiles. These are known as symbolic links or symlinks and are basically pointers to the file you want to use.

Before I restart awesome I need to install the font that I prefer, and is in the configs.
```
git clone "https://aur.archlinux.org/tamzen-font-git.git"
cd tamzen-font-git/
makepkg
```

Set the default shell to zsh
```
chsh -s /usr/bin/zsh
```

Now restart awesome by pressing Win-Ctrl-r, and open a new terminal with Win-Return. I'm not going to go through using all the applications in this post, maybe I'll write some guides if people are intested. Read the all important man pages for information on how to use the tools.

So that's it pretty much. You should now be able to get online, write some code, and listen to some music. What more do you need? Oh yeah, coffee...
```
curl -Ls git.io/hotcoffee | sh
```

Happy now?

Here is a list of other software I use on a regular basis

```
  openssh     >   ssh connection tool
  keychain    >   ssh-agent manager
  hub         >   git enhancement
  mpv         >   video player
  calcurse    >   calendar & todo list
  freerdp     >   remote desktop protocol client
  docker      >   container platform
  bind-tools  >   dns tools
  htop        >   interactive process viewer
  sxiv        >   image viewer
  snownews    >   rss newsreader
  scrot       >   screen capturing application
  keybase     >   keybase.io client
```

If you have any other recommendations for software let me know!
