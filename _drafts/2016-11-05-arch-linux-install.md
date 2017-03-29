---
layout: post
date: 2016-11-05
category: Tech
tags: tech arch linux install guide
description: A guide to installing Arch Linux with LVM configuration and root partition disk encryption.
---
The first distro I ever installed was OpenSUSE, many years ago. At the time I didn't really understand what the different distros meant and so I just installed from a free disk I received with a magazine.

I quickly started playing around with a number of other distros including Fedora, Ubuntu, and Debian. Quickly I realised there was no "right" distro and it was all about choice. Fedora was the distro I ended up using for a number of years... until the introduction of Gnome3 and the GnomeShell. Straight away this wasn't for me, so I sought out alternative Desktop Environments. It was also at this opportunity that I decided to give Arch a try.

Even in the few short years I had used Linux I knew for certain this was for me and therefore I wanted to know more about it. Arch was a good choice because it enables you to get much more hands on.

Arch is not recommended for absolute beginners, but if you want to improve your understanding of how the OS works it is a good distro to play around with. Things may (will) break, but that is all part of the fun!

This guide details the steps I take to quickly run up an Arch install. Security is always something I consider and therefore I encrypt my root filesystem. A lot of the steps detailed below are taken straight out of the [official install guide](https://wiki.archlinux.org/index.php/Installation_guide){:target="_blank"}. These steps are not intended to replace the information on the Arch wiki, this is merly my adoption of the process.

Pre-reqs:
<ul class="sidebar">
<li>- You should be comfortable with using the Linux command line as there is no GUI for this installation</li>
<li>- I use vi for editing the files in this guide. Feel free to use nano if you prefer</li>
<li>- Grab the latest [Arch ISO](https://www.archlinux.org/downloads){:target="_blank"}</li>
<li>- Internet connection</li>
</ul>

Boot from the ISO and select "Boot Arch Linux (x86_64)"

After the ISO has loaded you will be presented with a prompt
```
root@archiso ~ #
```
The first thing I tend to do is load the keymap for my UK keyboard. This is an optional step, but will help if you use certain characters when setting passwords (#, @, /, \, etc).

First show a list of all available QWERTY keymaps, then load the desired map
```
ls -l /usr/share/kbd/keymaps/i386/qwerty/
loadkeys uk
```
The next step is to ensure you have a working internet connection. While this step is also optional I will be using an internet connection later on
```
systemctl start dhcpd
ip a
ping -c 3 archlinux.org
```
_If you are unsure how to get wifi working check out my guide [here]()_

It is advised to enable ntp (Network Time Protocol) to ensure the system clock is accurate
```
timedatectl set-ntp true
timedatectl status
```
