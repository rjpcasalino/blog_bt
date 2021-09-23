---
title: 'Pis, Perl Scripts, and LCDs'
layout: post
---

> **TL;DR** I wanted to resurrect my old LCDs and use Perl for producing output. I wanted to try out Alpine on ARM but switched over to Raspberry Pi OS since it offered better ease of use for interacting with the LCD.

<figure>
![Pi with LCD and breadboard](/static/imgs/Pi_LCD.jpg)
<figcaption>_sorry, messy cable work_ :-)</figcaption>
</figure>

### Pi GPIO

After wiring my breadboard I grabbed HiPi:

> http://hipi.znix.com/install.html

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
