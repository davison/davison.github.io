---
layout: post
title: Home Automation First Steps
abstract: I dipped my toe in the smart home automation waters after I came across OpenHAB while researching options for the multi-room audio system that I wanted to build. As a strong advocate of privacy, I wanted to avoid cloud based solutions like Alexa, Google Home and others.
comments: true
tags: pi hardware smarthome openhab
---

I dipped my toe in the smart home automation waters after I came across [OpenHAB](https://www.openhab.org) while researching options for the [multi-room audio system]({{ site.url }}/2019/03/09/multi-room-audio-I.html) that I wanted to build. As a strong advocate of privacy, I wanted to avoid cloud based solutions like Alexa, Google Home and others. 

If you're unaware, these kinds of services work by sending your voice commands captured through smart speakers to Amazon (or Apple/Google/Microsoft) where the voice is analysed by artificial intelligence services. Unfortunately, that means the recordings are available not just to computer software AI at those companies, but to [employees](https://www.telegraph.co.uk/technology/2019/04/11/amazon-employees-listen-thousands-customer-alexa-recordings/) too. There's even evidence that mistakes made in picking up the trigger words (like "Alexa" or "OK Google...") can lead to anything being [recorded and actions being taken](https://www.techspot.com/news/74820-amazon-explains-how-alexa-secretly-recorded-couple-conversation.html) that you certainly didn't want.

That worries me enough that I won't have them in the house.

Back to OpenHAB then. This looked appealing to me for a number of reasons;

 - it's open source and coded in a language I know well (Java) so I should be able to tinker and troubleshoot as needed.
 - it installs to Raspberry Pis easily enough, which is hardware that I also know well.
 - there are a large number of supported (officially or by an active community) plugins and integrations with lots of smart-home technology products. This offers the opportunity of managing many smart home products from a single app instead of lots of disparate apps.
 - rules can be written to create interactions between different techs without using cloud based integration apps like [IFTTT](https://ifttt.com/) which seems like a terrible architecture to me.
 - though not easy, there is the possibility of introducing voice commands without the use of commercial cloud providers.

 So it seemed like a worthwhile thing to try out and a fun project getting all my (soon to be bought!) smart tech installed and working in a way that suits my desire for digital privacy and freedom.


## Install and Setup

I used a RasPi and a pre-baked [OpenHAB Raspbian image](https://www.openhab.org/docs/installation/openhabian.html) to install the server. 

## Lighting & Power

Initially I added a Phillips Hue hub, but didn't buy any of their bulbs due to the price of them. Once I'd added the Hue binding from the UI, OpenHAB recognised the hub instantly and added it to the list of "Things" (in OpenHAB parlance) through the "inbox". The bulbs I decided to get were ones made by Innr (Amazon sell them) as they had good reviews from people using them with the Phillips Hue Hub. My experience has been similar, they just work perfectly as they're Zigbee network devices, same as Hue, and they look to be of very good build quality.

Each new bulb is added to the hub using the native Hue app available from the app stores, and after that I don't need to use their app for anything else. Once the bulb is known to the Hub, it appears in the OpenHAB inbox where you can configure its properties and events as "Items" and control it via OpenHAB. I've now got around a dozen bulbs of various types in various places and rooms that can be managed from the app. Most are just standard dimmable bulbs, a couple are colour spectrum capable but it's a bit gimmicky.

At present I have about 5 TP-Link power plugs in use. They're too big and very ugly but they work with OpenHAB. Similar to the bulbs, I just use the Kasa app to set them up and then never have to touch that app again.

## Other Stuff

OpenHAB works with Squeezebox, so yay! it works with my [multi-room audio system]({{ site.url }}/2019/03/09/multi-room-audio-I.html) although the player controls are lacking in some major features (browsing music). Mostly I have them configured in OH to be able to control play/pause, often as part of rules (see below).

Both my TVs can be controlled to an extent too (one LG WebOS TV and one Samsung UXsomethingOrOther) but they're a bit erratic. Again, mostly handy for power on/off or mute from a distance and definitely not a replacement for any standard remote control device. You can send popup notifications to the LG screen though which is cool, but I haven't found a really practical use for that in my setup yet.

I've played with some other random integrations like ChromeCast and Plex server, but they weren't great and I don't really use either of them any more, so.. meh.

## Sitemap

Took me a while and some experimentation to figure out how the various groups of items would interact and how best to lay them out in what OH terms a "sitemap". This is a UI defined in 'code' that works with the OH mobile apps and with their "basic" UI through a web browser. I've also created a complete set of custom icons for my sitemap by downloading and editing (in [Inkscape](https://inkscape.org/)) freely available material design sets such as [these](https://material.io/resources/icons/?style=baseline). This makes a lot more sense after you get your Things and Items coded. 

![main site map]({{ site.url }}/public/images/openhab/hillside-sitemap.png)

The nice thing is that you can specify Group items in the sitemap and allow OH to lay them out in some default order with controls that match the item types, or you can exercise full control over each entry, overriding labels, icons and ordering as you choose.


## Coding Things and Items

OH becomes more powerful if you take the time to code the Things and Items in your network. You can tag them with various attributes that make them available to (for example) voice activated systems like [Mycroft](https://mycroft-ai.gitbook.io/docs/) and group them up in ways that make it easier to lay out a sitemap. Another benefit is that it means you have a record of your devices and config making it a lot easier to setup again if you damage the Pi or just want to do a clean re-install.

Here's an example of where it helps; I have hall lights that consist of 2 separate fixtures controlled by a single wall switch, one light on a wall, the other a ceiling fixture. Most of the time you do want to manipulate them as a pair (though handily, having smart bulbs in them means you now have a choice that you never used to have). In my `lights.items` file I have the following entries for them

```
Dimmer BU_HallLightKitchen_Brite        "Wall light"                    <light>             (RM_Hall, gHallLights)           ["Light","Lighting"] {channel="hue:0100:001738ae9bcf:bulb9:brightness"}
Dimmer BU_HallLightDoor_Brite           "Main light"                    <light>             (RM_Hall, gHallLights)           ["Light","Lighting"] {channel="hue:0100:001738ae9bcf:bulb7:brightness"}

Group:Dimmer gHallLights	            "Hall lights"                   <light>             (RM_Hall, gDownstairsLights)     ["Light","Lighting"]

```

Each of the individual lights is configured to belong to two groups - the `RM_Hall` defined elsewhere and `gHallLights` defined in this file. As you can see, the hall lights group also belongs to the room group and to a larger group named `gDownstairsLights`. This grouping allows a switch on the group to control the switch on all items in the group. In my sitemap file, I have
```
Group item=RM_Hall {
    Slider item=gHallLights
    Slider item=BU_HallLightKitchen_Brite
    Slider item=BU_HallLightDoor_Brite
}
```
![hall site map]({{ site.url }}/public/images/openhab/hall-lights-sitemap.png)

.. where the sliders can be adjusted individually or as a group with the "Hall Lights" slider.
![hall lights on]({{ site.url }}/public/images/openhab/hall-lights-on.png)


## Rules

This is the interesting bit: OH has a neat DSL (like a macro language) for coding ["Rules"](https://www.openhab.org/docs/tutorial/rules.html).. basically more complex automations built from simple interactions with your integrated things combined with events, messaging and conditional logic. For this it probably does help to have a bit of coding experience, even if just using macros in office applications or something like that.

First thing this allows me to do is to set a rule that will simply turn a bunch of lights and power switches on when activated - saves going from room to room doing it as it starts to get dark, right?


So in a file I named `automation.rules` I have the following;

```
rule "Lights on evening"

when 
    Item sLightsOn received command

then
    BU_OfficeLamp_Brite.sendCommand(15)
    BU_HallLightDoor_Brite.sendCommand(100)
    BU_HallLightKitchen_Brite.sendCommand(50)
    FishTankLamp_Switch.sendCommand(ON)
    LoungeLamp_Switch.sendCommand(ON)
end

```
(actually there are a bunch of others that also get switched on too, but you get the idea from here). As you can see, some bulbs are turned up full, some not so high and some power switches are activated for the fish tank light and a lamp in the lounge. What's the `sLightsOn` item though and how does it get activated?

`sLightsOn` is just an item defined in one of the items files as follows;
```
String sLightsOn   "Turn stuff on (getting dark)" <sunset>   (Home)
```

and that enables a button to be assigned to it in the site map with `Switch item=sLightsOn mappings=[ON="Select"]` which renders like this in the app:

![lights-on]({{ site.url }}/public/images/openhab/lights-on-action.png)

Now when I press that button, about 12 lights and power switches come on in various parts of the house. Neat. I also have very similar setups to turn everything off (when going to bed at night) and an intermediate one that turns quite a few things off and dims some of the other bulbs - we use this one after the hustle and bustle of early evening and dinner is done when we're mostly relaxing in front of the TV. Thanks to the power of groups in OH, the late evening power off is even simpler and powers off a number of other things that might have been used, such as TVs and audio players.

```
rule "Lights, music & TV off"

when 
    Item sBedtime received command

then 
    gDownstairsLights.sendCommand(0)
    gAudioStop.sendCommand(ON)
    gDownstairsPower.sendCommand(OFF)
    SamsungTV_Power.sendCommand(OFF)
    LGTV_Power.sendCommand(OFF)
end
```

But of course, having to press an actual button on the phone to turn things on when it starts getting dark is just so tedious. Surely we can do better?

We can. If you add the 'astro' binding to OH, it will fire events at the official times for sunrise and sunset, in addition to a few others. So with a small addendum to my rules file..

```
 *
 * switch lights on at sunset
 */
rule "Evening switch on"
when
    Channel 'astro:sun:home:set#event' triggered START
then
    sLightsOn.sendCommand(ON)
end
```

... I have the sunset event trigger my sLightsOn item just as if I'd pressed the button in the UI myself. Nice. 

## Summary

I quite like OH but there are some warts. It feels a little clunky to configure - both through the UIs it makes available and the code files. You can do a lot with them quite easily, but why not some standard config format like yaml? There is also some opaqueness to the setup where it's not always clear how to translate things you might configure through the paper UI into code in the various files. Being Java based, it can be a bit of a memory hog and several times I've had to reboot the pi because of lock ups caused by out-of-memory issues in the OH processes.

On the other hand, it has a good native app for Android and iOS, is fun to play with and mostly it works pretty well. I'm sticking with it for now, but I may look more closely at one of the alternatives in the future just for comparison.
