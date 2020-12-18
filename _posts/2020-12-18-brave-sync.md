---
layout: post
title: Fixing broken sync in Brave Browser
abstract: The sync functionality in the Brave browser is a useful way to keep settings, extensions and bookmarks synchronized on a number of different devices. But it can end up in a broken state where you are unable to leave or reset the sync chain on one or more of the sync'd devices. Here's how to fix that if it happens to you, without creating a new profile or re-installing the browser.
comments: true
tags: encryption backup privacy web
---

The sync functionality in the [Brave browser](https://brave.com/) is a useful way to keep settings, extensions and bookmarks synchronized on a number of different devices. But it can end up in a broken state where you are unable to leave or reset the sync chain on one or more of the sync'd devices. Here's how to fix that if it happens to you, without creating a new profile or re-installing the browser.

Brave is awesome - it's based on the open source [Chromium project](https://www.chromium.org/Home) (as is Google's Chrome browser too) but adds a lot of security and privacy by default. I use it as pretty much my only browser on Linux, Mac and Android. Brave has a feature to sync a number of settings across devices by adding them to a [sync chain](https://support.brave.com/hc/en-us/articles/360021218111-How-do-I-set-up-Sync-) where each device uses a shared encryption key based on a seed phrase to share data and settings. Unfortunately, perhaps for a variety of reasons, the sync chain can get messed up to the point where you can't even use the standard tools in Brave to reset it and force the device to leave the current sync chain in order to start or join a new one. [This post](https://community.brave.com/t/syncing-bookmarks-problem/172277) here is one example of someone struggling with this issue and even the support staff not really believing him, or at least not ever having come across the problem. Like others, his "solution" was to re-install Brave, others have created new user profiles, but the effect is the same - you lose all your settings, extensions, bookmarks, **history**, cookies, etc. just to get the sync function back. None of this is necessary.

In your profile directory for Brave (probably `$HOME/.config/BraveSoftware/Brave-Browser/Default` on Linux, `~/Library/Application Support/BraveSoftware/Brave-Browser/Default` on Mac assuming you're using the default user profile) there is a file called `Preferences` where settings, including sync settings, are held. It's a json formatted file, likely to be stored in a compressed (not pretty-print) format. There are 2 top level blocks in this file named `brave_sync` and `brave_sync_v2`, for example:

``` bash
$ jq .brave_sync < Preferences
{
  "enabled": true,
  "last_compact_time": {
    "bookmarks": "13226582376995998"
  }
}
```

Ensure you have quit Brave completely (or it will overwrite the changes you're about to make when you do) and then backup the `Preferences` file somewhere in case it goes wrong. Now edit the original with any editor. Locate the 2 blocks by name and empty them both, so that they appear in the file as `"brave_sync":{},"brave_sync_v2":{}` making sure not to mismatch the brackets by deleting too much or too little.

Save the file and re-open Brave, if all went well you should be able to go to the sync settings and see that sync is not in use and that you can create or join a new sync chain as normal.
