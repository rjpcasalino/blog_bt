---
title: 'Arthur Russell & The Art of the Unfinished'
layout: post
---

According to Wikipedia "[Arthur] Russell was prolific, but was also notorious for leaving songs unfinished and continually revising his music."[[1]](#fn:1)

I find myself continually stumbling into the unfinished: unfinished blog posts, programs, books, poems, walks. Why not vomit what I can as soon as I can and revise it later? Isn't that the glory of the internet, anyway? So if you find yourself lost, just hold on tight!

### https://man.sr.ht/installation.md

I'm a big fan of [source hut](https://git.sr.ht/) and support it's goals.[[2]](#fn:2) That being said, I thought it high time to actually use it locally before starting to hack on it. Last night I started the process of installing the man.sr.ht package on Arch.

#### things you might need...
- [srht-keygen](https://git.sr.ht/~sircmpwn/core.sr.ht/blob/master/srht-keygen)
- [arch signing key](https://man.sr.ht/packages.md#arch-linux)
- [postgres](https://wiki.archlinux.org/index.php/PostgreSQL)
- [redis](https://wiki.archlinux.org/index.php/Redis)
- [nginx](https://wiki.archlinux.org/index.php/nginx) 
- [pgpy](https://pgpy.readthedocs.io/en/latest/installation.html) <span>&mdash;</span> pretty good privacy 
- [python-yaml](https://security.archlinux.org/package/python-yaml) <span>&mdash;</span> probably installing the Python packages "incorrectly"; I have to check
- [python-redis](https://pypi.org/project/redis/)
- [the proper way to set /etc/hosts in nixos](https://unix.stackexchange.com/questions/489509/how-do-i-modify-my-hosts-file-in-nixos)

Okay, next you will add:

```
[sr.ht]
Server = https://mirror.sr.ht/archlinux/sr.ht
```
to your `/etc/pacman.conf` file.

Ah, wonderful. Continuing onward with:

```
$ pacman-key --recv-keys <signing key>
$ pacman-key --lsign-key <signing key>
```  

which you can find in source hut docs or link pasted above. Next,

```
$ pacman -S man.sr.ht
...
$ cd /etc
``` 

See, you've made it and now that you're sitting in `/etc` you'd maybe want to `mkdir sr.ht` and `touch sr.ht/config.ini`. Once that's done, `cat sr.ht/config.ini` and see the wondrous new creation. See the example config.ini files present in the official sr.ht repos.

In order for nginx to route correctly you must set your local `/etc/hosts` to contain something akin to `127.0.0.1  example.local`. Do this on the server as well.

Exhale.

Later on...

...just finished installing git.sr.ht. There were a few gotcha moments along the way, mainly related to permissions and database stuff. Some notes:

```
$ man chmod
$ man chown
```

It is wise to make sure that the web processes and the directories are owned by the user sshing into the machine. Have to double check that.

Further down the line you'd want to configure the rest of the services and secure postgres, redis, and all the rest. Then, good sir, you'd throw it all in a docker container and deploy it far and wide.

### The Shockwave Rider

I felt compelled to quickly write down my thoughts on John Brunner's *The Shochwave Rider* after having just completed it. I am looking to do a very Kerouac session which Truman Capote called "typing". <span>&mdash;</span> and look here I just had to go and verify all that information in the sentence prior. Three weeks on and off has to mean it was hard to revisit? But why? Maybe it has more reverence for the wrong reasons. Think back now, where did you read about it initially? <span>&mdash;</span>: The Unix Historical Society mailing list (how many flies on the wall there?) i.e., a bunch of old *white* men. ...but, god damn, *hippes* too. I dread saying it but "Thank God" for the hippies. At least, sometimes. 

What a wonderful tapestry this novel weaves! Here they all are gathered together for a common good, for what is Precipice CA if not this future of the USA? The town is Brunner's utopia. 

Some master "hacker" can unite all the aging intellectuals (dope smokers, no doubt...) of the world in this fantasy sci-fi. Nick, our hero, has been tortured and interrogated his entire life by his government facilitators at Tarnover. Brunner's fervent fantasies are scattered before us albeit with different names and phrases: `data-net` in 1975 (ARPANET came to us six years prior) is the Web now in 2019, tapeworms are the browser extensions of 2019, veephones <span>&mdash;</span> guess. This romp has all the cockamamie terms and phrases hazers will love. The capstone of thing is the Federal Government is now run by the mafia. RICO Act be damned! Are not gangsters and morons operating at the highest echelons of power now? Is it as in Hackers (1995) when someone said something like, "they are controlling the flow of information; they are corrupting data". All before us was created by the U.S. Department of Defense. Do warmongers still seek to sicken the airwaves? War never changes. 

<hr>
- [1] https://en.wikipedia.org/wiki/Arthur_Russell_(musician)
- [2] https://sourcehut.org/blog/2019-10-23-srht-puts-users-first/
