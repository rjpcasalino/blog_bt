---
layout: post
title: 'Nemo in the Network Dreams of Computers'
---

<hr>

I often wonder if my machines are happy being resurrected as they were. They've gone through myriad configurations, so many that I can't recall. They've slumbered for long periods in between. With a heave and a ho, their  names have been moved about and shuffled around while their innards remain the same.

Why resurrect old Sun machines? I think it's because I wanted to feel closer to the past. It's rescinding very quickly now, with memories stretched so thin they appear as gossamer. I wanted to feel close to Bill, also. What a joy that would be, uh?

In any case, as they say, here we go:

<img src="https://upload.wikimedia.org/wikipedia/commons/3/3e/Little_Nemo_1906-02-11.jpg"/>

This document will guide you through netbooting into either OpenBSD (on a SPARC V9 or an AMD64 box) or NixOS  or anything that netboot.xyz offers (on an AMD64 box). We'll also cover a bit more of netboot.xyz later as well as  iPXE (an open-source implementation of the  Preboot eXecution Environment), Open Firmware (also known as Open Boot; the terms "OBP", "OpenBoot PROM" and "PROM" (programmable read-only memory) are often used interchangeably in this context), and a tiny amount of Forth programming.

Manual documents you'll want to have handy for reference &mdash;

     netintro(4)
     diskless(8)
     pxeboot(8)
     dhcpd(8)
     arp(4)
     arp(8)
     rarpd(8)
     eeprom(8)
     cu(4) - maybe

All of which can be found at [https://man.openbsd.org/](https://man.openbsd.org/)

Files you'll encounter &mdash;

     /etc/bootparams     Client root and swap pathnames.
     /etc/dhcpd.conf     DHCP daemon configuration file.
     /etc/ethers         Ethernet addresses of known clients.
     /etc/exports        Exported NFS mount points.
     /etc/fstab          Static information about the filesystems.
     /etc/hostname.$if   Interface-specific configuration file.
     /etc/hosts          Host name database.
     /etc/myname         Default hostname.
     /etc/mygate         Default gateway.
     /tftpboot           Location of boot programs loaded by the Sun PROM.

A short search on eBay will present you with many choices concerning Ultras. I snagged an Ultra 5, 10, and 30 for less than $500.

You'll likely encounter a message about IDPROM contents being invalid during the power-on self-test (POST). Seeing as the unit's [NVRAM](https://en.wikipedia.org/wiki/Non-volatile_random-access_memory) chip probably died at some point in the last 20 odd years, this makes sense. At first, as I searched for ways to repair the chip, I only found posts from hardware hackers explaining how to retrofit (piggyback) a lithium coin battery onto it. I didn't have any time for that nonsense, so I snagged the [M48T58Y-70PC1](https://www.digikey.com/en/products/detail/stmicroelectronics/M48T58Y-70PC1/361258?s=N4IgTCBcDaILIBYAcAVArEgmgWgOwAYAFAYQEYACEAXQF8g) from DigiKey and swapped it out. The chip is easy to find on the Ultra 5 and 10. It'll be resting in a plastic cradle that is either black or green depending. The new chip doesn't have to be put into this cradle, but there's no harm. In the Ultra 30, the chip is hidden behind the power supply, but the supply is easy to slide out of the way for easy access to the chip.

<sub>Below is an example of a likely error message you'll encounter. Debugging can be disabled</sub>
![IDPROM contents invalid](/static/imgs/IDPROM_contents_invalid.jpg)

![replacement ultra 10 nvram](/static/imgs/nvram_ultra10.jpg)
<sub>Replacement M48T58Y-70PC1 Timekepper placed in an Ultra 10; upper left</sub>

Once you've replaced the chip and booted the machine, you should connect either via VGA and keyboard or serial console (you'll need a Null modem which can be found on DigiKey or Amazon; you might want to have a gender changer handy as well), and once connected you'll have the opportunity to program in the machine's ethernet address:

    ok set-defaults
    1 0 mkp
    80 1 mkp
    8 2 mkp
    0 3 mkp
    20 4 mkp
    XX 5 mkp
    YY 6 mkp
    ZZ 7 mkp
    0 8 mkp
    0 9 mkp
    0 a mkp
    0 b mkp
    XX c mkp
    YY d mkp
    ZZ e mkp
    0 f 0 do i idprom@ xor loop f mkp

where "XX:YY:ZZ" are the last 3 bytes of the media access control &mdash; MAC address for the machine. If you examine the NVRAM chip from the machine, it should have a yellowish sticker with a bar code and six hex digits (as in the image below). That's about all the Forth programming you're gonna have to do.

We `set-defaults` just to be sure. You can print the environment out via `printenv` while `setenv` does what one would guess (e.g., `setenv auto-boot? false`).

You'll most likely find yourself having to stop the boot process and enter the PROM. This can be done with a Sun Keyboard (Type 5 or 6, 8 PIN) by pressing the key combo: `STOP + A` which sends a break and will drop you in at the `ok` prompt. If you are using `screen` to connect to the serial adapter (e.g., `screen /dev/ttyUSB0 <baud rate> <other stuff>`) then sending a break would consist of pressing the key combo: <samp>CTRL a b</samp>.

![NVRAM Ultra 30](/static/imgs/NVRAM_ULTRA30.jpg)
<sub>NVRAM in cradle within an Ultra 30 amidst the dust</sub>

OpenBoot provides a programmable user interface that gives you access to an extensive set of functions for hardware and software development, fault isolation, and debugging. Asking for `help` is always a good first step when learning something new.

> `help` without any specifier, displays instructions on how to use the help system and lists the available help categories. Because of the large number of commands, help is available only for commands that are used frequently.

Since you're hoping to netboot, it might be wise to test the network connection &mdash;

![watch-net test](/static/imgs/watch-net_test.jpg)
<sub>you can test _devices_ and issue commands<sub>

#### References

* https://www.openbsd.org/papers/bsdcan2019_netboot.pdf
* http://cholla.mmto.org/computers/sun/ultra/nvram.html
* https://docs.oracle.com/cd/E19683-01/816-1177-10/overview.html#pgfId-1007990
