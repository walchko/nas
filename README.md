# Raspberry Pi Network Attached Storage (NAS)

Setup a simple NAS for home. There are numerous references around the web, I am just trying to document my setup. Some good references are:

- [alexellis.io](http://blog.alexellis.io/hardened-raspberry-pi-nas/)

## Rasbian

I am using Raspbian Jessie Lite (2017-01-11) as the base. Once setup, do (this took a while for me):

	sudo apt update
	sudo apt upgrade

**Note** I am using the newer package manager `apt` instead of the older ones `apt-get` and `apt-cache`. Now get tools to upgrade the kernel (raspbian always has an old kernel).

	sudo apt install rpi-update
	sudo rpi-update

Now your kernel and software should be good to go.

### Customization

`raspi-config`:

- host name
- resize the your micro SD card
- change the GPU/memory split
- correct time zone
- set local to `en_US.UTF-8 UTF-8` (default is en_GB because rpi was made in the UK)

When you login via `ssh` you get a bunch of licensing garbage. You can change the message of the day (motd) by editing `/etc/motd` to something else ... I like ascii art!

	nina@Dalek ~ $ ssh pi@raspberrypi.local
	pi@raspberrypi.local's password: 

				  a8888b.   
				 d888888b. 
				 8P"YP"Y88 
				 8|o||o|88 
				 8'    .88 
				 8`._.' Y8. 
				d/      `8b. 
			   dP   .    Y8b. 
			  d8:'  "  `::88b 
			 d8"         'Y88b 
			:8P    '      :888 
			 8a.   :     _a88P 
		   ._/"Yaa_:   .| 88P| 
		   \    YP"    `| 8P  `.
		   /     \.___.d|    .' 
		   `--..__)8888P`._.' 
	Last login: Sat Feb 18 20:12:10 2017 from fe80::14f5:a870:7365:b0e1%eth0
	pi@raspberrypi:~ $ 

### Software

Other useful software can be installed with `sudo apt install <package>`:

- build-essential
- cmake
- nmap
- python-dev
- htop

- [kernel ref](http://walchko.github.io/posts/2014/10/linux-kernel/)

## Static IP Address

Since this is a server, I don't want the IP address to change. I also use a wired interface, but I also show a wifi example below:

	[kevin@raspberrypi ~]$ more /etc/network/interfaces
	auto lo
	iface lo inet loopback

	# dynamic interface
	#iface eth0 inet dhcp

	# static interface
	iface eth0 inet static
	  address 192.168.1.112
	  netmask 255.255.255.0
	  gateway 192.168.1.1

	auto wlan0
	iface wlan0 inet static
	  address 192.168.1.111
	  wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
	  netmask 255.255.255.0
	  gateway 192.168.1.1

## Node.js

Node is historically old on debian servers, so let's get updates from the source.

	curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
	sudo apt-get install -y nodejs

This will install it to `/usr/lib` instead of `/usr/local` unfortunately.

### NPM

Other useful npms can be installed with `sudo npm install -g <package>`:

- httpserver
- archeyjs

- [node ref](http://walchko.github.io/posts/2015/09/nodejs/)

## Samba

This is the preferred way to communicate between macOS and linux. First let's install the software:

	sudo apt-get update
	sudo apt-get install samba samba-common-bin

Setup a login for user `pi` (or whatever user you want):

	sudo smbpasswd -a pi

Now let's set it up:

	sudo cp /etc/samba/smb.conf /etc/samba/smb.bak
	sudo pico /etc/samba/smb.conf

Change the config file to look like this:

	#======================= Global Settings =======================

	[global]
	   workgroup = WORKGROUP
	   wins support = yes
	   dns proxy = no

	#### Debugging/Accounting ####
	   log file = /var/log/samba/log.%m
	   max log size = 1000
	   syslog = 0
	   panic action = /usr/share/samba/panic-action %d


	####### Authentication #######

	   server role = standalone server
	   passdb backend = tdbsam
	   obey pam restrictions = yes
	   unix password sync = yes
	   passwd program = /usr/bin/passwd %u
	   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
	   pam password change = yes
	   map to guest = bad user

	############ Misc ############

	# Allow users who've been granted usershare privileges to create
	# public shares, not just authenticated ones
	   usershare allow guests = yes

	#======================= Share Definitions =======================

	[homes]
	   comment = Home Directories
	   browseable = yes
	   read only = no
	   create mask = 0700
	   directory mask = 0700
	   valid users = %S


## Hard Drive

[hdparam](http://www.htpcguides.com/spin-down-and-manage-hard-drive-power-on-raspberry-pi/)

## Plex.tv

**issues:**

- I tried to add a movie but it wouldn't show up. **Solution:** plex is run from `/etc/init.d` by user `plex` and I put the movie in a place owned by `pi` ... permission!!! Make sure your movies are located where user `plex` has access.

[instructions](https://www.element14.com/community/community/raspberry-pi/raspberrypi_projects/blog/2016/03/11/a-more-powerful-plex-media-server-using-raspberry-pi-3)

## SSH

To ease logins, from macOS, do:

	ssh-copy-id pi@raspberrypi.local

This will copy over your `id_rsa.pub` key so you can authenticate without having to use a password. Now if you are really security minded, you can disable passwords over ssh and **only** allow cryptographic keys to login. My setup doesn't need to be that secure.

# License

MIT
