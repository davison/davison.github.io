---
layout: post
title: Escaping the Cult of Google
abstract: For a long time now I've been getting more and more concerned about the damage being done to me through my membership of the "Cult of Google". I'd long shunned the Chrome browser and the Google search engine, as well as GMail for personal use. But weaning myself off the rest of the chocolate factory product suite is overdue.
comments: true
tags: privacy security openrights
---

For a long time now I've been getting more and more concerned about the damage being done to me through my membership of the "Cult of Google". I'd long shunned the Chrome browser and the Google search engine, as well as GMail for personal use. But weaning myself off the rest of the chocolate factory product suite is overdue.

For the unwary, a brief explanation. Google, like all big tech companies (Meta, Twitter, Apple, Microsoft and others) collect data from you. A lot of data. In fact, more than you would believe possible. It's what powers their businesses and drives the enormous, eye-watering [piles of cash](https://www.statista.com/statistics/266206/googles-annual-global-revenue/) they all make. Typically you're not paying them for the products you use right? Gmail, maps, facebook, insta, ichat, whatever. They generate their revenue by selling you.. where you are and where you've been, what you eat, everything you ever searched for on the web and most of the web pages you visit (yes, those too), what notes you take, the pictures you store, who your friends and family are, who else you chat to, the films and TV you watch, the music you listen to. It's endless. You're the product that they sell to their millions of advertising clients and other - often more [nefarious](https://en.wikipedia.org/wiki/Facebook%E2%80%93Cambridge_Analytica_data_scandal) - orgs. This is all well documented and you can search (not Google!) it anywhere on the web.

So much for the why, how about the what and the how.

## Search

This is the biggie for Google.. we even started using their name as a verb to describe the very action of searching the web. It can't get any better (worse) than that for Pete's sake.

But I don't want something that is spying from the shadows on all my possible searches for medical conditions, political literature, information about sexual orientation, domestic abuse, racism, totalitarian regimes or any other number of things that could get me into trouble, imprisoned, tortured or killed depending on where I am when I do that. Or will maybe just cause my next medical insurance renewal to be declined for unexplained reasons 6 months after I went searching for information about a friend's bowel cancer. These are no crackpot conspiracy theories. I switched my default engine to [DuckDuckGo](https://duckduckgo.com) years ago, and more recently to [Brave Search](https://https://search.brave.com/) but there are other privacy engines. Did the same on my phone, mac, PC and everything else I use to search.

## Chat

Well I guess no-one really uses Google services for this, that battle was long lost to WhatsApp and other Meta owned tools. Although WhatsApp is end to end encrypted, Meta (facebook) can still see all the (erm) meta data of who you converse with, when and how often. And do I trust Meta any further than I could spit them? I do not. I totally expect them to cave in to governments and add back doors to the product and then bury the change announcement 600 pages deep in a "Minor T&C update" that gets published the next time there's a global catastrophe to dominate the news. Because that's the kind of friendly, neighbourhood scum they are.

So I moved to [Signal](https://signal.org/en) which has a perfectly good mobile app and can be used on desktop too for convenience. It has voice and video calling all with the same level of encryption and protection. It can do everything WhatsApp can except spew all your data into Meta's bottomless collection buckets.

## Web browsing

Like many people, I used Google Chrome when it first arrived on the scene. But even at the first signs of Google progressively erasing the privacy and adding to the blatant tracking and spying, I shifted to the open source version - [chromium](https://www.chromium.org/Home/) - and then later to [Brave](https://brave.com/) which is a secure/private-by-default chromium based browser. Brave is my only browser on Linux, mac (work) and phone with outstanding tracker and ad blocking features. [Firefox](https://www.mozilla.org/en-GB/firefox/new/) is also an excellent multi-platform choice for privacy and security.

## Mail

I briefly flirted with gmail when it first arrived as a secondary email option, but soon just set it to forward all my email to my personal domain where for many years I had operated my own mail server at home. I did this in order to avoid my mail being stored or passing through 3rd party MTAs, including my [own ISP](https://aaisp.net) even though I have a very high level of trust in them and their operation. However, I've now switched to [Proton](https://proton.me/mail) mail. Proton is based in Switzerland and has strong privacy guarantees and end to end encryption. They cannot read my mail and keep no logs of my activity. I'd already had a free account with them for a while but at this point went all in and paid for a multi-user account that I could move my primary personal domain to. The web mail and Android app are pretty good, but I get seriously irritated with the fact that their [IMAP bridge](https://proton.me/mail/bridge) is a _desktop_ GUI application for LINUX. Jeez fellas.. come on.. I want to run it on a RasPi and connect clients to it from other machines. This is Linux, not some toytown OS.

## Drive and Docs

Drive is easy.. [proton drive](https://proton.me/drive). This will work for me as I am only a very light user of GDrive, but I'd prefer to get something running in order to be able to utilise the massive NAS I have at home (mostly stores media but obviously some docs too). Something like Resilio Sync could work for this - Android, web and NAS server side all supported, but I've not had huge success with it to date.

Google Docs is a lot harder, there are no functionally equivalent substitutes (Office365.. LOL) with the performance and ease of use and also the docs are backed by Drive. Annoying but will need to stick with this for limited applications for now.


## Calendar

My latest move. Uninstalled Google Calendar from my phone for the first time ever after importing the calendar events into [Proton Calendar](https://proton.me/calendar). Not quite as easy to add other contacts and locations to calendar events, but in these early days, I'm happy enough with it that I'm sure I won't ever need to switch back. Bonus now is that I can receive calendar invites at my primary email address and people don't need to know my gmail address. Nice.

## Notes

I'm not a heavy user of Google Keep Notes, never have been. I don't take many notes anyway but always found Keep to be randomly featured and strangely disconnected from the rest of the chocolate factory. Check lists and free text not allowed in the same note? Really? No calendar or gmail integration.. huh? But I do have a few things there, the main feature for me was sharing notes with my wife - things like Christmas shopping lists so we were always up to date on what we'd got for who. I'm struggling a bit on this one, there are a multitude of note sharing apps around but I want one that I can 

1. collaborate with others on (sync'ing)
1. use across Android and my browser on all platforms (clipping notes)
1. have stuff like lists and free text together :) 

Maybe [Joplin](https://joplinapp.org/)..? but any suggestions would be great.

## Maps

Tricky.. Google Maps and Waze (also now owned by Google) are somewhat indispensible for navigation and finding places. There are some [alternatives](https://surfshark.com/blog/google-maps-alternative) but feature comparison wise only Navmii might come close and they charge &pound;12.99 a month but STILL collect your data, though they promsie not to share it. Hmm.. so why collect it?

## Historic Data

Google does offer a way to limit the collection of data and delete some of the history they hold for you, which might be enough for some people (though not me). If you do, go to your account's [data and privacy](https://myaccount.google.com/data-and-privacy) section and remove, reduce, pause, turn off and limit everything you can in all the sections on that page.

## Other

The difficult one is going to be stock, or OEM Android and the Google Play services framework that runs as a system service. Lots of apps piggyback on this to avoid firewalling and network permission guards you may have added to your phone so they're a pretty insidious vector for privacy and security miscreants. I've repurposed a couple of old phones and installed [LineageOS](https://lineageos.org/) on them which is great - totally Google free - but I need my main phone to work with GPay and native banking apps and these won't work unless those services and the Play Store are installed :( 

## Summary

Getting better, but a distance to go. Would love to hear from anyone that has successfully replaced some of these harder to do services with plausible and functionally similar alternatives.
