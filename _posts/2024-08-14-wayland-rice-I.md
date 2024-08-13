---
layout: post
title: Wayland Rice - part I
abstract: Some recent updates to nvidia and mesa modules (maybe others) finally made it possible to switch to wayland for me full time. So I did.
comments: true
tags: linux desktop
---

Some recent updates to nvidia and mesa modules (maybe others) finally made it possible to switch to wayland for me full time. So I did.

A few months ago, I reinstalled an ancient laptop (originally bought in about 2011 I think) that I keep around for various bits of testing. It has an RJ45 port making it easy to connect to random other bits of networking equipment to configure them. Obviously by today's standards it's woefully spec'd and needs a very lightweight graphical environment if one can be used at all. I'd somehow bricked the thing weks ago but wanted to use it to test a camera, so I just reinstalled with Arch linux and decided to give [riverwm](https://codeberg.org/river/river) a try as I'd recently read about it and it looked like something that could possibly be used to replace my beloved [bspwm](https://github.com/baskerville/bspwm) on my main desktop. Everything's moving to wayland now, but bspwm won't so I needed something new for when I ditched Xorg.

The performance boost on the laptop was pretty significant, using only a fraction of the mammoth 4MB RAM (!) it has after starting up. I also liked the smoothness of wayland and the speed of river's window management - again bearing in mind the stone age on board graphics chip in the machine.

## Desktop install

So it looked promising, and I decided at this point to take a similar route on my main desktop - ditching Ubuntu for a return to Arch and hoping to use the same graphical stack as on the laptop. Alas, although river would start, my desktop used an nvidia card - also fairly old now as it happens, but I don't do any gaming and only need it to drive a 4k monitor, so it's perfectly fine for this box. But wayland barely worked with the combinatin of nvidia binaries, mesa, kernel version and/or whatever else was lacking at the time - lots of screen tearing, flickering, freezing and other unwanted artifacts. Sadly I installed Xorg again and continued with `bspwm`. Every time there was an update in the Arch repos to any of the key packages, I updated, restarted and held my breath as I tried entering `river` instead of `startx` at the console. But no joy.

Then mesa got an update to 1.24.1 which was a critical step and there was an immediate improvement - though not enough to make the desktop usable as a daily driver. Finally, a few weeks ago, the breakthrough updates to nvidia libs and kernels that took it from "experimental alpha" to "almost usable if you ignore the tearing on browser windows" to "perfectly stable and usable" in the space of a few days. Yay!

Now I could actually get to work configuring and customising the look and behaviour of the desktop in order to get as close as possible to the setup I'd spent so much time on in Xorg and which I really wanted to keep and replicate. Perhaps, per chance, improve. I'll add further parts to this series to cover the way I've configured river itself, the optional (non-default) tiler I picked, waybar and some custom modules, and the bling on top.
