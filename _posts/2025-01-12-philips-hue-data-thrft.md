---
layout: post
title: Philips - another company that wants to steal your data
abstract: Philips, via their Hue hub, have recently decided they have a right to steal your data, weaken your security (while lying that it improves it) and cripple products you bought from them in good faith if you refuse to comply with the lowlife they employ to make such decisions. I chose not to of course and now this company that I previously had no beef with is the latest addition to my list of "never spend another penny on any product of theirs for the rest of my life" list.
comments: true
tags: privacy security smarthome
---

Philips, via their [Hue hub](https://www.philips-hue.com/en-gb), have recently decided they have a right to steal your data, weaken your security (while lying that it improves it) and cripple products you bought from them in good faith if you refuse to comply with the lowlife they employ to make such decisions. I chose not to of course and now this company that I previously had no beef with is the latest addition to my list of "never spend another penny on any product of theirs for the rest of my life" list.

## What happened? 

It actually happened some time back, but I managed to avoid the impact for a while. Philips decided that it wasn't happy with not knowing what everyone was doing with their flagship smarthome product (Hue - the hub that manages their smart lighting of the same brand). So rather than simply content themselves with the hundreds of pounds/dollars/florins/whatever that we all spent on their kit, they chose to enforce creation of the ubiquitous 'cloud account' in order to continue to use the app that controls the hub. Once you create the account, it links to your hub and Philips can grab who whatever they like from your device - maybe other parts of your network too, who knows (not us because their source code is proprietary).

My exposure to the Hub app is limited anyway, I use it only to add new devices (bulbs) to my hub and then everything from that point is controlled and managed via [HomeAssistant](https://home-assistant.io/) and its Hue integration. But I recently had a bulb fail, bought a new one and discovered I was blocked from getting it included into my setup. Here's what the Hue app looks like when I open it now (note - no possibility of access to any of my devices or to add new ones until I comply):

![home_screen]({{ site.url }}/public/images/philips/home.jpg)

..and if you click "Tap to learn more" you get this:

![readmore_screen]({{ site.url }}/public/images/philips/readmore.jpg)

Bit weird - the home page said "Hue accounts are designed to enhance your system's security" but the screen you get to talks about product features that you may or may not want. I don't want Spotify pairing with my light bulbs, don't have voice assistants, can already control my lights away from home with a VPN, don't need Philips to control who has access to my devices and couldn't give a monkeys about "signing in" on multiple mobile devices when I don't even need to do that on a single one of them. So I shouldn't be forced to create an account to get access to stuff I don't want, right? Not according to the mouth breathers at Philips. The only mention of security on the "Read more" page was the "Use secure products and features" bullet point in the middle of the other crap. OK - let's continue the magical mystery tour and "Learn more" - this button now links to [this page](https://www.philips-hue.com/en-gb/explore-hue/accounts) from which I shall save you the bother of having to figure out how it explains that my security will be improved: it doesn't explain that. At all. Sure, it talks about all the faux security features you get with the account, but compared with not having anyone access my hue hub and it remaining entirely private behind my firewalled network with no Internet access into or out of it, this is NOT A FUCKING IMPROVEMENT.

Why did they do all this? You'd have to ask them. "Improved Security" is just a common phrase employed by those trying to scare you into being complicit in their data rape activity. 

## Avoiding the whole mess

The basic connectivity of Hue products uses an IEEE network/radio specification ([Zigbee](https://en.wikipedia.org/wiki/Zigbee)), this is what made it possible to add a number of significantly cheaper, but still very high quality non-Philips bulbs to my Hue hub (they're made by [Innr](https://www.innr.com/en/), a company created by two ex-Philips execs). It means that not only can you interchange bulbs on Hue, you can use a different hub altogether. I bought a [branded Home Assistant one](https://www.home-assistant.io/connectzbt1/) - so happy that these kinds of products are on the market now because they weren't when I originally got my Hue hub. The £30 I spent on it and the sunk cost of the now redundant Hue hub was more than worth it to be able to stick a middle finger up to Philips in response to their bullshit.

I plugged the thing in to a USB 2.0 (not 3.0!!) port on my HA machine and of course it was instantly recognised and subsequently setup and configured in a matter of seconds.

![zigbee_config]({{ site.url }}/public/images/philips/zigbee.png) It was then a case of moving from bulb to bulb, starting with those physically closest to the Zigbee receiver and gradually moving further out so that the mesh network builds up in the best way possible. Each bulb was reset using [power cycling](https://www.youtube.com/watch?v=DBc1fIgVXXo) since I had no access to them any more in the Hue app (thanks Philips) and once reset to pairing mode, added into the ZHA network through the simple discovery mechanism. There was a bit of tedious search/replace of my HA YAML config files to ensure that the groups, scenes and automations were updated to match the new names of my bulbs, but the whole exercise probably took me about 3 hours for the 20 or so smart bulbs that I have. 

![kitchen_lamp]({{ site.url }}/public/images/philips/lamp.png)

And here's my Zigbee mesh (a handy live network view created by the ZHA integration - nice)

![mesh]({{ site.url }}/public/images/philips/mesh.png)

Lastly, after testing all my bulbs individually and testing all the scenes/automations they were involved in, I took great delight in deleting the Hue integration from HA, factory-resetting my physical Hue hub and removing it from my cabinet and then uninstalling the Hue app from my phone.

Hey Philips.. 

![finger]({{ site.url }}/public/images/philips/finger.png)