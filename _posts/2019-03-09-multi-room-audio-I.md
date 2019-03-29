---
layout: post
title: Multi Room Audio - part I
abstract: Before Christmas last year, I started playing around with some tech that I hoped would eventually become a multi-room audio system. Although I got a bit distracted with some smart-home stuff (which I might eventually write about), I then came back to the original goal of listening to music and radio in various places around the house.
---

Before Christmas last year, I started playing around with some tech that I hoped would eventually become a multi-room audio system. Although I got a bit distracted with some smart-home stuff (which I might eventually write about), I then came back to the original goal of listening to music and radio in various places around the house.

My ambitions for the project were as follows:

1. Re-purpose as much as I could of existing gear that we already owned, especially speakers because we had two very good pairs of passive speakers and an [excellent sound bar](https://www.techradar.com/uk/reviews/samsung-hw-ms650-soundbar) under the main TV
1. Make it possible to play different music or streams in each location, __OR__ synchronise any or all of the locations so that they all played the same music in perfect lag free sync
1. Control all of it from any of our phones
1. Make it expandable for future room additions
1. Do it all at a fraction of the cost of [Sonos](https://www.sonos.com). I'm not having a pop at Sonos, they make great sounding equipment that's beautifully designed and have opened their API's so that developers can extend it. That makes them a company I'd normally like to support, but I already had nice kit which I didn't want going to waste, and Sonos *is* expensive.

## Planning

I didn't really do a lot, being more of a JFDI kind of person, but I spent some time researching what I needed so that I could meet as many of the project ambitions as I could.

The two main things to decide were hardware for the passive speakers, and the software that would manage the whole system.

### Hardware

Having already built a few things from standard Raspberry Pi boards, I was pretty familiar with them and there are plenty of specialised OS images available, so that seemed like an obvious choice for the base. Having a spare one kicking its heels in the drawer helped. ![RasPi]({{ site.url }}/public/images/mra/pi.jpg) 

Just needed to find a compatible DAC/Amp as the sound from the Pi's headphone jack is widely known to be pretty woeful. IQAudIO and HiFiBerry were the likely candidates for this. Mostly because I prefer black to red, I went with [IQaudIO's DigiAmp+](http://iqaudio.co.uk/hats/9-pi-digiamp.html) ... ![IQAudio's DigiAmp+]({{ site.url }}/public/images/mra/iq.jpg) ...which sits neatly on top of the Pi with some standoffs and plugs into the 40 pin GPIO header. This means the amp can supply power to the Pi and therefore only the amp needs a power supply via its barrel connector, the Pi doesn't need separate USB power. Nice.

At around £33 for a Pi, £5 for a storage card and £54 for the amp, the combined unit for each room with passive speakers was £92 before considering power, networking and cabling costs.


### Software

Choosing the software platform wasn't as easy. Initially, I wanted to base it around [MPD](https://musicpd.org) because I'd used it on my PC for years and it was pretty flexible in terms of defining outputs to various sinks like PulseAudio servers or HTTP streams - locally or across the network. But previous test attempts to make MPD work in the kind of configuration I was after hadn't yielded much success and it seemed that most of my web searches were turning up similar stories elsewhere.

As far as alternatives go however, there were certainly no shortages. A good number of modern music players all with varying capabilites as potential multi-room systems can be found. Many of them have gorgeous web interfaces, modern looking Android/iOS apps and pluggable feature-rich services. But one of my key ambitions was lag-free sync'd up rooms, and disappointingly none of these great systems appeared to have this designed in. [Some](https://roonlabs.com/) couldn't manage it at all, [one or two](https://volumio.org/) had workarounds or unofficial plugins that made some strides in that direction and [one in particular](http://strobe.audio/) looks like it's going to be the future but isn't quite there yet.

In the end, the oldest system with possibly the poorest web interface and API connectivity was the one I picked, purely because it had been designed from the ground up with the multi-player sync feature. [SqueezeBox (LMS)](http://wiki.slimdevices.com/index.php/Logitech_Media_Server) seemed to be the only system to have got this aspect right from the research that I did. When Logitech decided not to continue making the hardware devices, they open-sourced the software for the server and this made it possible to use as the heart of the system I wanted. Handily, it's also supported by the [smart-home bus](https://www.openhab.org) I'm using (also open source of course!) and lastly, it can be installed as a [standalone player](http://wiki.slimdevices.com/index.php/Squeezelite) on Linux desktop systems.

The final decision was whether to custom build an image for the Pi's that included squeezebox, or go with a ready-made one. I chose to prototype with the [Max2Play](https://www.max2play.com/en/) image first because it was simple enough to install and supoprted all of the features I wanted, including explicit support for the IQaudIO boards.


## Planned system overview

This was the plan then. One machine acts as a Squeezebox server, this is the hub of the system that receives command and control messages from the client software - web browsers or mobile apps. The server listens for connections from players which are the machines that generate the signals to connected audio sinks like speakers and sound bars. It manages the connected players and displays them to the clients which can select them to play music to. The server also connects to music sources locally (I use NFS to enable it to mount a drive on another machine where all my music files are stored) and to external sources such as Internet radio stations or services like Spotify.

![architecture]({{ site.url }}/public/images/mra/architecture.jpg)

Right, most of the important decisions made and the parts ordered, it was time to try and build something and see if it might work!
