---
title: 'Using the calm window manager'
layout: post
---

The calm window manager (cwm) is a "lightweight and efficient window manager for X11" written mostly by [Marius Eriksen](https://monkey.org/~marius/). It purports to "maintain the simplest and most pleasant aesthetic." Having used it for nearly two weeks it's hard to disagree. 

The control keys are fairly simple (ctrl, meta, shift):

```bash
CM-Return	Spawn a new terminal.
...
M-Return	Hide current window
...
M-question	Spawn “exec program” dialog.
M-period	Spawn “ssh to” dialog.
```

There are 35 odd-ish key bindings. That's not too bad. cwm's "GROUPS" concept makes it easy to simulate "virtual desktops". Groupings can be traversed via arrow keys. 

One can move windows about with key bindings one would find in vim (i.e., hjkl).

Configuration is pleasant and straightforward (.cwmrc):

```bash
command firefox  firefox
command shot	 scrot
command pad 	 "xterm -e ed /home/rjpcasalino/yellowpad.md"

borderwidth 1
# Inherit groups 
sticky yes

# Define a “gap” in pixels at the edge of the screen. Maximized windows will not overlap gap area.
gap 24 0 0 0
# Ignore programs by name, not drawing borders around them.
ignore showip 
ignore showtime

bind-key XF86MonBrightnessUp	"light -A 10"
bind-key XF86MonBrightnessDown	"light -U 10"
```

<hr>

Here's some links that are helpful if you're interested in cwm:

* [cwm(1)](https://man.openbsd.org/cwm.1)
* [cwmrc(5)](https://man.openbsd.org/cwmrc.5)
* [Getting started with cwm](https://undeadly.org/cgi?action=article&sid=20090502141551)






