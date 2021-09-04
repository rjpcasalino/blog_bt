---
layout: post
title: 'Nemo in Network Dreamland'
---

<img src="https://upload.wikimedia.org/wikipedia/commons/3/3e/Little_Nemo_1906-02-11.jpg"/>

This document will guide you through netbooting into either OpenBSD (on a SPARC V9 or an AMD64 box) or NixOS  or anything that netboot.xyz offers (on an AMD64 box). We'll also cover a bit more of netboot.xyz later as well as  iPXE (an open source implementation of the  Preboot eXecution Environment), Open Firmware (also known as OpenBoot), and a tiny amount of Forth programming.
<aside>
<small>
the terms "OBP", "OpenBoot PROM" and "PROM" (programmable read-only memory) are often used interchangeably in this (and other) context.
</small>
</aside>

Manual documents you'll want to have handy for reference — 

     arp(4)
     arp(8)
     boot_sparc64(8)
     biosboot(8) — amd64-specific first-stage system bootstrap
     cu(4) 
     dhcpd(8)
     dhcpd.conf(5)
     diskless(8)
     eeprom(8)
     ethers(5)
     hosts(5)
     mountd(8)
     netintro(4)
     pxeboot(8)
     rarpd(8)
     rpcinfo(8)
     tftpd(8)

All of which can be found at [https://man.openbsd.org/](https://man.openbsd.org/)

Files you'll encounter — 

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
     /usr/mdec/ofwboot.net  Network bootstrap
     /bsd                Default system kernel
     /bsd.rd             Standalone installation kernel, suitable for disaster recovery

A short search on eBay will present you with many choices concerning Ultras. I snagged an Ultra 5, 10, and 30 for less than $500.

You'll likely encounter a message about IDPROM contents being invalid during the power-on self-test (POST). Seeing as the unit's [NVRAM](https://en.wikipedia.org/wiki/Non-volatile_random-access_memory) chip probably died at some point in the last 20 odd years, this makes sense. At first, as I searched for ways to repair the chip, I only found posts from hardware hackers explaining how to retrofit (piggyback) a lithium coin battery onto it. I didn't have any time for that nonsense, so I snagged the [M48T58Y-70PC1](https://www.digikey.com/en/products/detail/stmicroelectronics/M48T58Y-70PC1/361258?s=N4IgTCBcDaILIBYAcAVArEgmgWgOwAYAFAYQEYACEAXQF8g) from DigiKey and swapped it out. The chip is easy to find on the Ultra 5 and 10. It'll be resting in a plastic cradle that is either black or green depending. The new chip doesn't have to be put into this cradle, but there's no harm. In the Ultra 30, the chip is hidden behind the power supply, but the PSU is easy to slide out of the way for easy access to the chip.

<sub>Below is an example of an error message you'll likely encounter. Debugging can be disabled.</sub>
![IDPROM contents invalid](/static/imgs/IDPROM_contents_invalid.jpg)

Once you've replaced the chip and booted the machine, you should connect either via VGA and keyboard or serial console. If connecting via the latter, you'll need a null modem cable or adapter both of which can be found at DigiKey or Amazon. You might want to have a gender changer handy as well.

You'll most likely find yourself having to stop the boot process and enter the PROM. This can be done with a Sun Keyboard (Type 5 or 6, 8 PIN) by pressing the key combo: `STOP+A` which sends a break and will drop you at the `ok` prompt. If you are using `screen` to connect to the serial adapter (e.g., `screen /dev/ttyUSB0`) then sending a break would consist of pressing the key combo: <samp>CTRL a b</samp>. Yet another way is to use `cu` and "call up" the UNIX system of your choice. Once you've connected, you can ascertain what key sequence is needed to send a break by issuing commands such as `~?` which prints a list of other commands.

Once you've gotten yourself to the PROM, it's time to program in the machine's ethernet address:

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

where "XX:YY:ZZ" are the last 3 bytes of the media access control — MAC address for the machine. If you examine the NVRAM chip from the machine, it should have a yellowish sticker with a bar code and six hex digits (as in the image below). Having carefully entered the above commands, you'll have done about all the Forth programming you'd have to do. Issue the `reset` command and check via `banner` after the machine comes back up whether the ethernet address stuck.

We `set-defaults` just to be sure. You can print the environment to get a sense of what's what via `printenv` while `setenv` does what one would guess (e.g., `setenv auto-boot? false`).

![NVRAM Ultra 30](/static/imgs/NVRAM_ULTRA30.jpg)
<sub>NVRAM in cradle within an Ultra 30 amidst the dust</sub>

OpenBoot provides a programmable user interface that gives you access to an extensive set of functions for hardware and software development, fault isolation, and debugging. Asking for `help` is always a good first step when learning something new:

> `help` without any specifier, displays instructions on how to use the help system and lists the available help categories. Because of the large number of commands, help is available only for commands that are used frequently.

Since you're hoping to netboot, it might be wise to test the network connection &mdash;

![watch-net test](/static/imgs/watch-net_test.jpg)
<sub>you can test _devices_ and issue commands<sub>

If you're receiving good packets, this means the client is ready. The boot server, in my case, became the Ultra 10's new job. Keep in mind that some what is to be discussed further below depends on what architecture you are trying to netboot. Unless otherwise noted, you can assume the client is a SPARC64 box.

Luckily, the Ultra 10's CD-ROM drive is in decent shape and it was rather painless to download, verify, and install OpenBSD after burning the ISO (International Organization for Standardization) to a CD-R. Sadly, the 5 and 30's respective drives had given up the ghost. Replacement parts are easy to come by but it shouldn't matter since we plan to netboot them anyway!

Let's take a look at what our client is hoping to see once we've issued the `boot net` command at the `ok` prompt. By the way, you can adjust the boot order by setting the `boot-device` environment variable via `setenv`. 

Okay, let's go ahead and boot from the network...

![rarp client error](/static/imgs/rarp_error.jpg)
<sub>Retrying...Check TFTP server and network setup</sub>

Thankfully, the client offers advice as to what to do next.

Setting up a reverse ARP server is dead simple with `rarpd`. By default this process forks and runs in the background (it's a daemon after all) but if one were to run it in the foreground and pass it the `-d` flag they'd be better equipped to debug issues. A bit more on `rarpd` from its manual page:

> Upon receiving a request, rarpd maps the target hardware address to an IP address via its name, which must be present in both the ethers(5) and hosts(5) databases.

As mentioned earlier, <samp>/etc/ethers</samp> contains addresses of known clients such that an example entry appears thus:

    08:00:20:00:02:CD     SNAFU # or the client's actual name

According to the kind editors of Wikipedia:

> MAC addresses are primarily assigned by device manufacturers, and are therefore often referred to as the burned-in address, or as an Ethernet hardware address, hardware address, or physical address.

Keep the above in mind as you try to make sense of why we have so how many names for the one _thing_.

#### but, I don't want to worry about the link layer...

Address resolution provides dynamic mapping between two different forms of addresses: 32-bit IP addresses and whatever type of address the network interface uses. This network interface's hardware address (often an 48-bit ethernet address) is what the device driver software will need to know in order to route the incoming IP datagram. The device driver never looks at the destination IP address of an incoming IP datagram. Again, the function of ARP is to map two addresses (the _logical_ IP address and the _physical_ hardware address).

A diskless system would read its unique hardware or physical address from the interface card (device) and send an RARP request (a broadcast to the network) asking for someone to reply with that diskless system's IP address. This is the job of the DHCP server. Below is an example of `rarpd` getting a packet and sending a reply:
<aside>
<small>
> Most routers are incapable of forwarding RARP requests. BOOTP was meant to improve upon RARP and address its shortcomings. DHCP was meant to improve upon BOOTP and address its shortcomings, especially the proliferation of "vendor extensions" as defined in RFC 1497.
</small>
</aside>

    server# rarpd -a -d 
    hme0: 8:00:20:00:01:AB # server ether
          192.168.0.1 # server IP
    rarpd: got a packet
    rarpd: 08:00:20:00:02:CD # client ether
    rarpd: reply sent

Once the diskless has its IP said system should then probe the newly received address with ARP.

![Little Nemo 3](https://upload.wikimedia.org/wikipedia/commons/0/0b/Little_Nemo_1906-10-21.jpg)

#### dhcpd setup
<small>
<aside>
> DHCP is built directly on UDP and IP which are as yet inherently insecure. Furthermore, DHCP is generally intended to make maintenance of remote and/or diskless hosts easier. While perhaps not impossible, configuring such hosts with passwords or keys may be difficult and inconvenient. Therefore, DHCP in its current form is quite insecure.
</aside>
</small>

The manual can be cryptic and nothing works better than an example to clear things up:
   
    option  domain-name "my.domain";
    option  domain-name-servers 192.168.1.3, 192.168.1.5;

     subnet 192.168.1.0 netmask 255.255.255.0 {
        option routers 192.168.0.13;

        range 192.168.1.32 192.168.1.127;

        host static-client {
                hardware ethernet 22:33:44:55:66:77;
                fixed-address 192.168.1.200;
        }

        host pxe-client {
                hardware ethernet 02:03:04:05:06:07;
                filename "pxeboot";
                option-34 1; # see below
                next-server 192.168.1.1;
        }
     }

(you'd find this in <samp>/etc/examples/dhcpd.conf</samp>).

It'll behoove you to have a look over [RFC 2132](https://datatracker.ietf.org/doc/html/rfc2132).

I found myself tumbling around in `dhcpd.conf` some days annoying my poor wife when her many machines (some even strapped to her wrist, if you can believe that!) were calling and demanding their IPs but receiving none.

If you make changes to `/etc/dhcpd.conf` (as you surely will) be sure to have those changes reflected in reality by restarting the daemon with `rcctl restart dhcpd`. Bare in mind that `dhcpd` can be run with the `-n` flag which will only test the configuration, but will not run `dhcpd`.

> The documentation for the various options mentioned [in the actual documentation] is taken from the IETF draft document on DHCP options, RFC 2132. Options which are not listed by name may be defined by the name option-_nnn_, where _nnn_ is the decimal number of the option code. These options may be followed either by a string, enclosed in quotes, or by a series of octets, expressed as two-digit hexadecimal numbers separated by colons.

#### bootparamd or rpc.bootparamd

An example: 

> When the client named "mockingbird" requests the pathname for its logical "root" it will be given the server name "server" and the pathname `/export/mockingbird` as the response to its RPC request.

#### creating a root filesystem

You can create a root filesystem with base70.tgz and `tar`. You can add extra sets if you so desire, but base is the bare minimum needed.

#### References

* https://www.openbsd.org/papers/bsdcan2019_netboot.pdf
* http://cholla.mmto.org/computers/sun/ultra/nvram.html
* https://docs.oracle.com/cd/E19683-01/816-1177-10/overview.html#pgfId-1007990
* http://www.cs.columbia.edu/~sedwards/presentations/2019-vcf-netboot.pdf (wonderfully helpful!)

<hr>
![Littel Nemo in Slumberland 2](https://upload.wikimedia.org/wikipedia/commons/c/ce/Little_Nemo_1908-09-20.jpg)

