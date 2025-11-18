---
title: 'Alpine Up and Running on ASUS VivoBook E203MAS_L203MA or ASUS Chromebook C223'
Layout: post
---

> I said "dance", not "fuck", don't get it confused. — Cardi B

Whereas there are official docs and myriad guides to follow regarding setting up Alpine Linux I decided to record the process on a brand new ASUS VivoBook. Why the E203 VivoBook? I'm a big fan of the ASUS EeePC series and the E203 is a worthy successor. Simple and small, lightweight, and with a keyboard that pleases the fingertips: the E203 checked some of my boxes. Lack of back lit keys turned out to be more bothersome than I'd figured it would be. But coming equipped with USB-C, microSD, HDMI, and two USB 3.1 ports makes it a pretty robust offering from ASUS for a budget laptop. Slick and light with a price point that won't leave your wallet longing for that soft feel of greenback cotton, the VivoBook makes me shout "Republic of China all the way!". I also had an ASUS Chromebook hanging around burning a hole in my floor but the star is the VivoBook.

First, use [rpi-imager](https://github.com/raspberrypi/rpi-imager) to create a bootable USB drive. It just works.

<hr>

Second, turn off secure boot in the BIOS (get to the BIOS with F2 on the VivoBook; Chromebook requires [developer mode](https://www.chromium.org/chromium-os/chromiumos-design-docs/developer-mode/)).

Third, this is a [simple sys install](https://wiki.alpinelinux.org/wiki/Install_to_disk) (not running from RAM) so just follow steps given in installer.

Fourth, you will need the community repos so edit `/etc/apk/repositories` and remove the comment to allow downloading from community repos.

Finally, as root begin installing goodies:

	# become root or use doas
	su -
	# add basics
	apk add man mandocs openbox xterm xinit terminus-font doas

...and some other crap:

	apk add acpi git neofetch feh cwm polybar light
	# acpi is a client for battery, power, and thermal readings
	# git for duh, neofetch for hacker factor, and feh is for wallpaper
	# the calm window manager is my go to
	# polybar is a great status bar
	# light for screen backlight control

	# Getting bluetooth working requires dbus and bluez 
	# Check out dbus here: https://www.freedesktop.org/wiki/Software/dbus/
	apk add dbus dbus-openrc
	apk add bluez bluez-openrc bluez-alsa
	# oddly found that out here: https://wiki.postmarketos.org/index.php?title=Bluetooth

	# Remember to add bluetooth and others to runtime so it works at boot!
	rc-update add bluetooth boot
	rc-update add bluealsa boot
	rc-update add dbus default

	# audio is easy with alsa drivers and such
	# ("easy" - alas, alas...alsa doesn't see sound card on Chromebook; I hate being dyslexic)
	apk add alsa-utils alsa-utils-doc alsa-lib alsaconf alsa-ucm-conf
	# read more about alsa here: https://en.wikipedia.org/wiki/Advanced_Linux_Sound_Architecture

	# to change shell we need libuser
	apk add libuser
	# Alpine uses ash as default shell but one might want bash
	apk add bash

	# OK, done with apk for now

	touch /etc/login.defs
	# make sure /etc/default is around
	touch /etc/default/useradd

	# pick bash if you want
	doas lchsh <user>

	# do magic
	setup-xorg-base

	# add your user to all useful groups
	addgroup <user> audio
	addgroup <user> input
	addgroup <user> video
	# logout so the changes take effect

	# startx needs .xinitrc to load the Xresources file
	# and start cwm
	touch .xinitrc 
 
	# before starting X, install setxkbmap for compose key
	apk add setxkbmap
	# see .XCompose for more
	# then, startx or xinit!
	startx

OK, once that stuff is out of the way let's work on the touchpad. Install and read about xinput:

	apk add xinput xinput-doc

In order to discover the key mapping of the touchpad issue this command:

	xinput list
And from there once you've discovered what device, get the button map via:

	xinput get-button-map "the device"

And when you are ready, set the button map with:

	xinput set-button-map 1 2 3 4 5 6 7

The mapping is in physical order and setting one of the above values with "0" will disable the button. Thus setting 2 to "0" will disable that action.

The Chromebook doesn't have traditional function keys so use `xev` to find the right keycode or keysym name.
Bind the chosen key in `.cwmrc` to the action you'd like taken. For example, having the reduce brightness key actual reduce screen brightness.

To get suspend to work on LID close follow this guide: [https://wiki.alpinelinux.org/wiki/Suspend_on_LID_close](https://wiki.alpinelinux.org/wiki/Suspend_on_LID_close)

You might need to create the directories and don't forget to make the script you'll copy executable.

And, finally, here are the fonts that work best for most setups:

	apk add terminus-font ttf-inconsolata ttf-dejavu font-noto font-noto-cjk ttf-font-awesome font-noto-extra

##### Blogging and nix flakes and docker
Before starting let's install docker:

	apk add docker
	addgroup <user> docker
	
	# have docker start on boot
	rc-update add docker boot
	service start docker

I use a static site generator of my own (less than stellar) making called bss — here's how to get it working with Nix and Docker:

	# Once you've cloned bss, mount the dirs and expose the ports
	docker run -it --rm -v $(pwd)/bss/:/bss -v $(pwd)/blog_bt:/blog_bt -p 9000:9000 nixos/nix
	# bss is flakes based so we have to pass annoying flags but this can also be set in nix config
	nix build --extra-experimental-features nix-command --extra-experimental-features flakes
	# copy the resulting program into a dir on the PATH
	cp result/bin/bss  /nix/var/nix/profiles/default/bin/
