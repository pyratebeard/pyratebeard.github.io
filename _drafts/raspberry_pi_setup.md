--- 
layout: post 
date: 2017-04-22 
category: Tech 
tags: raspberrypi 
description: Setting up an OS on a Raspberry Pi 
--- 
 
##### Mmmm Pi 
By now everybody has at least of the [Raspberry Pi](https://www.raspberrypi.org){:target="_blank" rel="noopener noreferrer"}. It's an affordable (~£30) little computer designed to enable anybody to learn to code and build projects. It has become a great hit in education and there have been some incredible uses from drones to (mini)supercomputers. Most people however seem to use it as a media server (based on people I know). 
 
I actually won my first Pi. My team was voted the regional winner in our category for the 2016 NASA Space Apps Challenge. The prize was a Raspberry Pi 3, which has some advantages over previous versions - namely a more powerful CPU, wireless LAN (Wifi) and bluetooth. 
 
There were some good intentions to build a drone, and I spent a long time looking at other people's projects. In the end my Pi sat on my desk collecting dust. Finally I decided I was going to do two things with it, first I was going to use it to run my own VPN, and second I wanted to set up my mail server on it. 
 
* Raspbian?! Yes, as some of you may know I am not a fan of the Debian based distros. I tend to choose Arch Linux or Fedora. As a Linux engineer my work is saturated with Red Hat so it's good for me to know Fedora. 
So Rasbian was not my first choice, but it is the choice that worked out of the box. I had some issues installing Arch Linux ARM, and post installation issues with Fedora ARM. 
As you'll see later Raspbian installs quickly and with no issues. This meant I could move on to the fun bit instead of fault finding. 
 
 
##### VPwhat? 
A VPN is a Virtual Private Network. It allows you to connect to your own 'private' network through any other 'public' network securely. If you imagine you home network is your private network, if you go to a friends house you can connect to their wifi (public network). If you then connect to you VPN everything you do is being tunneled through your own private network. 
 
The reasons for this are primarily security. By tunnelling your network activity through your VPN then you can be safe from any monitoring on the public network and even the ISP. Another advantage is that the VPN is basically an extension of your home network, which means you can access all the devices and files you have at home. 
 
VPNs are used extensively by corporations so their employees can access the company network from anywhere in the world. They are also used by people who travel alot and are therefore connecting to many different public networks. 
 
##### Mail electronically 
Some of you may have read my first post about [DeGooglefying](/log/2017/04/degoogle-part-1){:target="_blank" rel="noopener noreferrer"} (yes it's a word) my life. As part of this transition I moved to using my own domain for emails, and I though the Pi would be a great little mail server. 
 
 
##### Raspbian install 
Before we can set up our VPN we need to put an OS on the Pi. As mentioned above I have opted for [Raspbian](https://www.raspberrypi.org/downloads/raspbian/){:target="_blank" rel="noopener noreferrer"}. At the time of writing the stable release is 'Jessie', I chose the Lite version as I don't need a desktop for my uses. 
 
After downloading the zip archive extract the image file. If you're using Windows you will need to use an application such as [Etcher](https://etcher.io/){:target="_blank" rel="noopener noreferrer"} to write the image file to an SD card, which will be used in the Pi. For this guide I'm using Linux, so I can use the `dd` utility. 
 
A quick side note on SD cards. One thing that catches a lot of people out is the read and write speeds. Most cards will show you the read speed, which can be quite high. If you pay close attention to the small print the write speeds aren't always very high. I went for a card which had pretty high read AND write speeds so that I get the best I/O for my OS. The card I am using is a [PNY 32GB Elite-X microSDHC U3](https://www.pny.com/32GB_Elite-X_microSDHC_Card_CL_10_90MBs_with_Adapter?sku=P-SDU32U390EX-GE){:target="_blank" rel="noopener noreferrer"} (from around £20), which has read speed of ~90Mbps and benchmarked write speeds between 70 -> 85Mbps. These speeds vary depending on the devices but for it will be suitable for the Pi. 
 
Run the `lsblk` command to see the current devices, plug your SD card into your Linux machine the run again to get the device name of the SD card 
``` 
lsblk 
NAME                    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT 
sdd                       8:48   1  29.9G  0 disk 
``` 
 
If your machine automatically mounts the device you will need to unmount it 
``` 
umount /dev/sdd 
``` 
 
Copy the image file to our SD card. We are using a bytesize of 4M as recommended on the Raspbian site, if this doesn't work you can try 1M 
``` 
dd bs=4M if=2017-04-10-raspbian-jessie-lite.img of=/dev/sdd 
``` 
 
You can now see that the partitions have been created on the card (a 32GB card may seem like a waste of space but we will come back to that later) 
``` 
lsblk 
NAME                    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT 
sdd                       8:48   1  29.9G  0 disk 
├─sdd2                    8:50   1   1.2G  0 part 
└─sdd1                    8:49   1    41M  0 part 
``` 
 
As of November 2016 Raspbian does not enable ssh by default. This can be an issue if you don't have a monitor or TV with a HDMI port, or a HDMI cable! We can get around that by mounting the newly created boot partition and adding a file called "ssh". 
``` 
mount /dev/sdd1 /mnt 
touch /mnt/ssh 
umount /mnt 
``` 
If you are able to plug your Pi into a monitor or TV it is worth watching it boot, always nice to have "eyes on" in case of any errors. 
 
Once this has finished remove the SD card from your machine and plug into the slot on Pi. I will always use ethernet with my Pi, so plug it in and power it up. The first time you boot it is best to leave it for a few minutes. The system does some checks and then boots up. You should have a solid red light and a flashing green light. 
 
pi setup draft
