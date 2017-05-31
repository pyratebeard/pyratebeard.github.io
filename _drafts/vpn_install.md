---
layout: post
date: 2017-05-13
category: Notes
tags: openvpn raspberrypi
description: Configuring a Raspberry Pi as a VPN server
---

##### Fruity
As mentioned in my previous [post](log/2017/05/raspberry-pi-setup){:target="_blank" rel="noopener noreferrer"}, I want to use my Raspberry Pi as a VPN server. If you followed along with the install guide you should have a basic Raspbian OS running on your Pi. You may have installed other software, or configured the Pi however you prefer. If you have made any changes it _may_ alter the following VPN setup.

For this guide I will be continuing on with the basic install as it was after the previous post.

**Note** - Using a VPN from outside your home network will require port forwarding on your router. Through the router admin console you should be able to forward all traffic to your Pi's 'tun0' device, which is created later.

<br />
##### Open(VPN) your world
There are a number of VPN server options out there, but the most popular is [OpenVPN](https://openvpn.net/){:target="_blank" rel="noopener noreferrer"}. It is incredibly easy to install, and works with the default configuration. There is a web based management console which enables you to adjust the settings quickly.

First things first, let's make sure our Pi is up to date then we can install OpenVPN. We will also install `easy-rsa`, OpenVPN's CA utility
```
sudo apt update
sudo apt upgrade
sudo apt install openvpn easy-rsa
```

As root copy config file from OpenVPN's example files. If the command doesn't work with sudo, run as root as shown below (needs root password)
```
cd /usr/share/doc/openvpn/examples/sample-config-files/
su -c "gunzip -c server.conf.gz > /etc/openvpn/server.conf"
```

Open the config file for editing
```
sudo vi /etc/openvpn/server.conf
```

Change the Diffie hellman parameter from 1024 to 2048
Before
```
dh dh1024.pem
```
After
```
dh dh2048.pem
```

Remove comment (';') from the DHCP redirect line
```
push "redirect-gateway def1 bypass-dhcp"
```

Remove comment (';') from the two DNS lines. If you want to change the DNS servers edit the IP addresses on these lines. I have opted to stick with the defaults which are OpenDNS
```
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```

Finally reduce privileges by running as nobody, remove the comment (';') from the following lines
```
user nobody
group nogroup
```
Save and exit your text editor.

Now we need to set up the firewall. Firewall configuration on Linux is a sore subject for a lot of people, especially when trying to use the `iptables` commands. To make life easier you can opt to use a tool such as `ufw` which makes configurating the rules really easy. My guide will use the `iptables` commands because the only way to learn is by doing!

Forward IPv4 traffic
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Make persistant. In /etc/sysctl.conf, uncomment line
```
net.ipv4.ip_forward=1
```

Show iptables rules (should be blank)
```
sudo iptables -L
```

We are going to set up a number of rules. First we need to allow established outgoing connections. This makes life easier as we don't always know which port will be used for an outgoing connection, such as HTTP. If you prefer you can set this rule seperately for each port you open, but for ease we will set it globally.
The second rule is to allow incoming SSH connections. If you changed the port number used for SSH (see my Raspbian Install Guide) then you need to specify the port number.
The third rule we need to set if to allow OpenVPN traffic. This port number can also be changed in the OpenVPN config file, the default is 1194.
Then we need to allow TUN interface connections. We will also allow TUN connections to be forwarded through our other interface.
Finally we need to NAT the OpenVPN traffic using our TUN interface. 
```
sudo iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 2222 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i eth0 -p udp --dport 1194 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i tun+ -j ACCEPT
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
sudo iptables -I FORWARD -i tun0 -o eth0 -s 10.8.0.0/24 -d 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A FORWARD -i tun+ -j ACCEPT
sudo iptables -A FORWARD -i tun+ -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o tun+ -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Save the firewall changes in a backup file, then make the rules persistent after a reboot.
```
sudo iptables-save > 20170527_iptables_rules.bak
sudo invoke-rc.d iptables-persistent save
```

If you ever need to apply the saved rules, if iptables is flushed for example, run the following
```
sudo iptables-apply 20170527_iptables_rules.bak
```

You can now run `sudo iptables -L` again to see all the rules you have entered
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:2222 ctstate NEW,ESTABLISHED
ACCEPT     udp  --  anywhere             anywhere             udp dpt:openvpn ctstate NEW,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere            

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  10.8.0.0/24          192.168.0.0/24       ctstate NEW
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             ctstate ESTABLISHED
```

The next step is to generate the keys using `easy-rsa`. Copy the "easy-rsa" directory into our OpenVPN config directory, then create a new "keys" directory
```
sudo cp -r /usr/share/easy-rsa/ /etc/openvpn/
sudo mkdir /etc/openvpn/easy-rsa/keys
```

Before we generate the keys we need to change the following fields in "/etc/openvpn/easy-rsa/vars"
```
export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"
export KEY_NAME="EasyRSA"
```
Change the values to reflect your location and details. Also choose a name for your key (i.e. "server"), we will need this later. Save and quit your text editor

Generate the Diffie-Hellman pem file, this will take a while
```
sudo openssl dhparam -out /etc/openvpn/dh2048.pem 2048
```

When that has finished we can generate the certificates. It is easier to do this as root as we need to source the variables. When running `build-key-server` change the server name to the one you specified in the "vars" config file
```
sudo -i
cd /etc/openvpn/easy-rsa
source ./vars
./clean-all
./build-ca
./build-key-server server
exit
```

Copy the newly generated keys to your OpenVPN directory
```
sudo cp /etc/openvpn/easy-rsa/keys/{server.crt,server.key,ca.crt} /etc/openvpn
```

Open up the OpenVPN ("/etc/openvpn/server.conf") config file again and 
Now we can start our OpenVPN server
```
sudo systemctl start openvpn
sudo systemctl status openvpn
```

We now have to generate keys for the clients we want on our VPN. It is good practice to have individual key pairs for each client, and not to share one key pair. This makes life easier if a device is lost or stolen, we only have to revoke one device's key pair.

Generate the keys for the first client, changing the name to the device you will be using (run as root again)
```
sudo -i
cd /etc/openvpn/easy-rsa
source ./vars
./build-key client
exit
```

Create a new directory to keep things tidy then copy an example client configuration file, and the keys we have just created
```
mkdir ~/client
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client/client.ovpn
sudo cp /etc/openvpn/easy-rsa/keys/{client.crt,client.key,ca.crt} ~/client
```

Next open up the config file "client.ovpn". Uncomment the "nobody" and "nogroup" lines as before, also comment out the certificate and key lines
```
user nobody
group nogroup
;ca ca.crt
;cert client.crt
;key client.key
```
Save and quit your text editor.

Instead of copying the .ovpn file the two certs and the key across to our client, we can echo the contents into our .ovpn file and only copy the one file across to our client.

The syntax for this is in XML, for example
```
<tag_name>
contents
</tag_name>
```

So we can run the following commands (as root)
```
sudo -i
cd /home/pyratebeard/client
echo "<ca>" >> client.ovpn
cat ca.crt >> client.ovpn
echo "</ca>" >> client.ovpn
echo "<cert>" >> client.ovpn
cat client.crt >> client.ovpn
echo "</cert>" >> client.ovpn
echo "<key>" >> client.ovpn
cat client.key >> client.ovpn
echo "</key>" >> client.ovpn
exit
```

Securely copy the "client.ovpn" file across to your device. 
