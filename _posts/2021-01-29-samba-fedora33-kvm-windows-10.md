---
published: true
layout: post
title: File transfer between Linux and KVM Guest Windows 10
subtitle: Samba, SELinux Boolean, Firewall settings
cover-img:
thumbnail-img:
share-img:
tags: [linux, fedora]
---

# File transfer between Linux and KVM Guest Windows 10

OS: Fedora 33

## 1. Install samba
`sudo dnf install samba`

Expected result
```
 ❯ rpm -qa | grep samba
samba-common-4.13.3-0.fc33.noarch
samba-common-libs-4.13.3-0.fc33.x86_64
samba-libs-4.13.3-0.fc33.x86_64
samba-client-libs-4.13.3-0.fc33.x86_64
samba-common-tools-4.13.3-0.fc33.x86_64
samba-winbind-modules-4.13.3-0.fc33.x86_64
samba-winbind-4.13.3-0.fc33.x86_64
samba-winbind-clients-4.13.3-0.fc33.x86_64
samba-client-4.13.3-0.fc33.x86_64
samba-4.13.3-0.fc33.x86_64
```

## 2. Configure samba
The samba configuration file is located at `/etc/samba/smb.conf`.
There is also an example located at `/etc/samba/smb.conf.example` for your references.

First, let's backup the existing configuration.  
`sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.old`

Edit the configuration. See the example for the explaination of parameters.
Here I show my configuration for an example.
```
 ❯ cat /etc/samba/smb.conf
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.
#
# Note:
# SMB1 is disabled by default. This means clients without support for SMB2 or
# SMB3 are no longer able to connect to smbd (by default).

[global]
	workgroup = WORKGROUP
	security = user

	passdb backend = tdbsam

	printing = cups
	printcap name = cups
	load printers = yes
	cups options = raw

[homes]
	comment = Home Directories
	valid users = %S, %D%w%S
	browseable = No
	read only = No
	inherit acls = Yes

[printers]
	comment = All Printers
	path = /var/tmp
	printable = Yes
	create mask = 0600
	browseable = No

[print$]
	comment = Printer Drivers
	path = /var/lib/samba/drivers
	write list = @printadmin root
	force group = @printadmin
	create mask = 0664
	directory mask = 0775

# This is how I share my Documents directory.
# Replace the path parameter as you need.
# Simply repeat this pattern to share another one.
[Documents]
	comment = Documents
	path = /home/jeffshee/Documents
	read only = no
	browseable = yes
```

## 3. Enable samba
`sudo systemctl enable smb`
`sudo systemctl start smb`

>Sidenote:  
Always remember to reload samba once you modify the configuration.  
`sudo systemctl restart smb`  
Also you can check the samba's current status by:  
`sudo systemctl status smb`

## 4. Configure SELinux Boolean
There are a fews SELinux Boolean you will need to pay attention.
Here I show my configuration.
```
 ❯ getsebool -a | grep samba
samba_create_home_dirs --> off
samba_domain_controller --> off
samba_enable_home_dirs --> on
samba_export_all_ro --> off
samba_export_all_rw --> off
samba_load_libgfapi --> off
samba_portmapper --> off
samba_run_unconfined --> off
samba_share_fusefs --> off
samba_share_nfs --> off
sanlock_use_samba --> off
tmpreaper_use_samba --> off
use_samba_home_dirs --> off
virt_use_samba --> off
```
You can enable them by using `setsebool`, for example:  
`sudo setsebool -P samba_enable_home_dirs=1`

Refer to [samba_selinux](https://linux.die.net/man/8/samba_selinux) for detail explaination.

## 5. Configure firewall
```
sudo firewall-cmd --add-service=samba
sudo firewall-cmd --reload
```

Here I show my firewall configuration.
```
 ❯ sudo firewall-cmd --list-all 
FedoraWorkstation (active)
  target: default
  icmp-block-inversion: no
  interfaces: wlp4s0
  sources: 
  services: dhcpv6-client ipp ipp-client mdns samba samba-client ssh wsdd
  ports: 1025-65535/udp 1025-65535/tcp 137/tcp 138/tcp 139/tcp 445/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```
Note that I also opened up port 137/tcp 138/tcp 139/tcp 445/tcp.
These should be redundant, however, try adding these ports if it doesn't work.
To add port 445/tcp, for example:  
`sudo firewall-cmd --add-port=445/tcp`

>Sidenote:  
Always remember to reload firewall once you modify the configuration.
`sudo firewall-cmd --reload`

At this point, the samba should work, except for KVM (-> step 6).
Check your inet address using `ifconfig`.

For example, it is `192.168.0.197` in my case:
```
wlp4s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.197  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::58b9:f44d:ab4e:4d1e  prefixlen 64  scopeid 0x20<link>
        ether c0:e4:34:fb:2c:b1  txqueuelen 1000  (Ethernet)
        RX packets 3210056  bytes 4511880489 (4.2 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2167234  bytes 380765377 (363.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

You can test samba in followings method:
1. In GNOME
Launch `Nautilus > Other Locations`, your samba should be shown here.
Alternately, enter the address to connect, for example `smb://192.168.0.197/`
2. In Windows 10
Press Win Key + R, to launch the run dialog.
Enter the address to connect, for example `//192.168.0.197/`

>Sidenote:  
You wouldn't see your Linux samba shown in Windows' `File Explorer > Network`.  
Try [wsdd](https://github.com/christgau/wsdd) if you really want that functionality.
(I tried but haven't got it to work in my system yet...)

## 6. Configure firewall (libvirt)
You still need to allow samba for libvirt. These commands should do the tricks.
```
sudo firewall-cmd --zone=libvirt --add-service=samba
sudo firewall-cmd --zone=libvirt --add-service=samba-client
sudo firewall-cmd --reload
```

Here I show my firewall configuration.
```
 ❯ sudo firewall-cmd --list-all --zone=libvirt
[sudo] password for jeffshee: 
libvirt (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces: virbr0
  sources: 
  services: dhcp dhcpv6 dns samba samba-client ssh tftp
  ports: 
  protocols: icmp ipv6-icmp
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule priority="32767" reject

```

## That's all!! 🤩
