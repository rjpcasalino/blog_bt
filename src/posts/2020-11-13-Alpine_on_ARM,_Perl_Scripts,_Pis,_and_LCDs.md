---
title: Alpine on ARM, Perl Scripts, Pis, and LCDs
layout: post
---

> **TL;DR** I wanted to resurrect my old LCDs and use Perl for producing output. I wanted to try out Alpine on ARM but switched over to Raspberry Pi OS since ease of use for interacting with the LCD was more pleasant.

<figure>
![Pi with LCD and breadboard](/static/imgs/Pi_LCD.jpg)
<figcaption>_sorry, messy cable work_ :-)</figcaption>
</figure>

I started with the AArch64 image from:

> [https://alpinelinux.org/downloads/](https://alpinelinux.org/downloads/)

From there I read the instructions for installing Alpine on a Pi:

> [https://wiki.alpinelinux.org/wiki/Raspberry_Pi](https://wiki.alpinelinux.org/wiki/Raspberry_Pi)

After, I paused and adjusted my back because I have such poor posture.

Soon thereafter I followed this link:

> [https://wiki.alpinelinux.org/wiki/Create_a_Bootable_USB](https://wiki.alpinelinux.org/wiki/Create_a_Bootable_USB)

I used [GParted](https://en.wikipedia.org/wiki/GParted) to create a bootable FAT32 partition and then created a file system by issuing the command:

	mkdosfs -F 32 /dev/sdX1

Where X is the correct reference to the new partition. Next, I extracted the contents of the AArch64 tarball to the newly mounted disk:
	
	mkdir -p your-drive

	mount /dev/sdX1 /your-drive

	tar -xvf alpine.tar.gz /your-drive

I inserted that newly formatted disk into the Pi 3. Alpine booted quickly and once at the user prompt I was greeted with:
	
	localhost login:

I knew very little about Alpine before installing it. I did have some idea about its minimalist tendencies and use in lightweight containers.

Things that have so far caught my eye:

- OpenRC
- run-from-RAM
- apk

I'll start with apk, which is the package manager. Seeing as Alpine is lightweight and free of all the fat of other distributions one will not have access to the man pages right off the bat. The FAQ's "Why don't I have man pages or where is the 'man' command?" actually contains outdated information: as of Nov 2020 (might need a PR?) the package `man-db` didn't seem to work for me. I tried some educated guesses and landed on:

	apk add mandoc

Note the absence of the hyphen between man and doc. Keep in mind that apk has a search command:

	apk search <SNAFU>

Afterward, I then had the ability to snag man pages:

	apk add perl

	apk add perl-doc

	man perl

It's important to remember to save changes one has made:

	lbu commit -d 

At this point I wanted to ensure I had a basic toolbox with the tools I needed. The newbie guide suggests certain packages to install:

	apk add lsof lsof-doc nano nano-doc curl curl-doc git git-doc

I ended up also adding:

	apk add make gcc

Next, adding users

	useradd 

	ash: useradd: not found

whoops, please excuse my dyslexia.

	adduser 

	Usage: adduser [OPTIONS] USER [GROUP]

The newbie guide offers up some options but I decided to ignore most and just create a new user called remote:

	adduser -D --home /opt/remote --shell /bin/ash remote

There are issues with the above as /opt/remote will result in a 'no such file' error on reboot. Recall that one must commit.

Next,

	passwd remote

That's the way I've always done it but the newbie guide does it this way:

	echo "your password here" | chpasswd

but that spat back at me:

	chpasswd: missing new password

so I just did it how I normally would...maybe not the best idea but one I feel comfortable with being a newbie and all.

	lbu commit -d

	reboot

	...time passes...

	ssh remote@alpine-box

	alpine-box:~$

Some other small tasks included adding `sudo`:

	apk add sudo

	visudo

## run-from-RAM

I installed via diskless mode (I think?):

> This is the default boot mode of the .iso images. setup-alpine configures this if selecting to install to "disk=none", and it means that the whole operating system and the applications run extremely fast from within RAM (saving unnecessary disk spin-ups, power and wear). A customized configuration and package selection may still be completely preserved on permanent storage media by using the "local backup utility" lbu and a local package cache.

But data mode is interesting as well:

> This mode is still accelerated by running the system from RAM, however swap storage and the whole /var directory tree gets mounted from a persistent storage device (two newly created partitions). This location holds e.g. all log files, mailspools, databases, etc., as well as lbu backup commits and the package cache.

## OpenRC

As far as init systems go this is one I hadn't yet used and seems to be just fine. In any case, OpenRC is part of the Gentoo Linux ecosystem, which is one I haven't spent much time in.

### Pi GPIO

After wiring my breadboard I grabbed HiPi:

> http://hipi.znix.com/install.html

What followed was a few hours of trying to get `cpan -i HiPi` to work...

I think my "failure" here was more or less a lack of understanding regarding Alpine's limitations (namely space) based on installation method. And the whole @INC Perl stuff just didn't worth the effort/time. 
- - - 
The next morning I pondered a bit on some of the troubles I was having the night previous. Whereas I am glad I took the time to try something new via Alpine, the big immediate take away was that Alpine isn't meant for doing anything GPIO related with _ease_. I would have loved to spend more time installing gcc and make and others tasks of that ilk but what I really wanted to do was play around with my LCD after years of disuse. Accordingly, I switched to Raspberry Pi OS (formerly Raspbian).

The foregoing mentioned HiPi:

> The HiPi Modules provide access to the Raspberry Pi for your Perl scripts.

> The distribution contains modules that allow you to access the Raspberry Pi GPIO pins directly and modules that wrap access to the kernel device drivers for the GPIO, SPI, I2C and 1 Wire devices.

> The distribution also provides high level wrappers for some common ICs and peripherals

Besides HiPi, getting started with an LCD requires wiring the GPIO correctly, ensuring that the [I2C](https://en.wikipedia.org/wiki/I%C2%B2C) interface is enabled via [`raspi-config`](https://www.raspberrypi.org/documentation/configuration/raspi-config.md), and ensuring the Pi can see the LCD device via [i2cdetect](https://linux.die.net/man/8/i2cdetect).

Some code:
	
	#!/usr/bin/env perl

	use v5.10;

	use IO::Socket qw(:DEFAULT :crlf);
	use HiPi qw( :lcd );
	use HiPi::Interface::LCDBackpackPCF8574;
	use HiPi::RaspberryPi;

	use constant MY_ECHO_PORT => 2007;

	my ($bytes_out, $bytes_in) = (0,0);

	my $quit = 0;
	$SIG{INT} = sub { $quit++ };

	my $port = shift || MY_ECHO_PORT;

	my $sock = IO::Socket::INET->new( Listen => 20,
					  LocalPort => $port,
					  Timeout => 60*60,
					  Proto => 'tcp',
					  Reuse => 1)
					  or die "Can't create listening socket: $!";

	my $lcd = HiPi::Interface::LCDBackpackPCF8574->new(
	    width            => 16,
	    lines            => 2,
	    devicename       => '/dev/i2c-1',
	    address          => 0x27
	);

	warn "waiting for incoming connections on $port...\n";
	while (!$quit) {
		next unless my $session = $sock->accept;

		$lcd->backlight(1);
		$lcd->init_display();
		$lcd->enable(1);
		$lcd->clear;
		$lcd->set_cursor_position(0,0);
		$lcd->set_cursor_mode( HD44780_CURSOR_BLINK );

		my $peer = gethostbyaddr($session->peeraddr, AF_INET) || 
		$session->peerhost;
		my $port = $session->peerport;
		warn "Connection from [$peer,$port]\n";

		while (<$session>) {
			$bytes_in += length($_);
			chomp;
			my $msg_out = (scalar reverse $_) . CRLF;
			print $session $msg_out;
			$lcd->send_text($_);
			sleep 2;
			$lcd->clear;
			$bytes_out += length($msg_out);
		}
		warn "Connection from [$peer, $port] done\n";
		close $session;
	}

	$lcd->send_text("bytes_sent = $bytes_out, bytes_received = $bytes_in");
	$lcd->clear;
	close $sock;

This implements a basic echo server that displays its output on an LCD device using the common PCF8574 based backpacks. More here:

> [http://hipi.znix.com/module/interface/lcdbackpackpcf8574.html](http://hipi.znix.com/module/interface/lcdbackpackpcf8574.html)
