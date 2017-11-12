---
layout: post
date: 2017-11-12
category: Tech
tags: android, termux, ssh, linux
description: SSH into your Android device with Termux
---

In a [previous post](/log/2017/06/termux-on-android){:target="_blank" rel="noopener noreferrer"} I talked about installing [termux](https://termux.com){:target="_blank" rel="noopener noreferrer"} on an Android device. This tool makes it easy to ssh into our other Linux systems, but what if we want to ssh into our Android device?

Unfortunately password login doesn't work on Android and if you haven't rooted your device you have limited permissions. Instead we can use ssh keys.

If you don't already have an ssh key pair then on you Linux system run
```
ssh-keygen -t rsa
```

If you accepted the defaults this will create two files under your user's .ssh directory, `id_rsa` and `id_rsa.pub`.

Make sure `sshd` is running on your Linux machine (requires the OpenSSH package)
```
systemctl status sshd
```

If it's not installed run the following (if it's just not running omit the first command)
```
sudo pacman -S openssh
sudo systemctl start sshd
```

Also make a note of the IP address 
```
ip a
```

Next, from termux on your Android device copy down the public key you just created
```
scp pyratebeard@192.168.1.3:.ssh/id_rsa.pub ./id_rsa.pub
```

Now add the public key to the `authorized_keys` list
```
cat id_rsa.pub >> .ssh/authorized_keys
```

Almost there. Finally we need to install OpenSSH on termux and start the daemon
```
apt install openssh
sshd
```

Make a note of the IP address of your Android device
```
ip a
```

That's it! You can now ssh from your Linux machine onto your Android device using port 8022
```
ssh 192.168.1.4 -p 8022
```

If you need to specify a user for the above command then from termux run
```
whoami
```
and add the user to the ssh command
```
ssh 192.168.1.4 -p 8022 -l u0_a161
```


