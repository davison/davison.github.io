---
layout: post
title: Encryption, Backups and Recovery
abstract: Encryption is a good thing, especially in my country where the government's desire to know and control every detail of private citizens' lives would make the Chinese and the Russians blush :) But don't lock yourself out of your own data as I thought I had yesterday.
comments: true
tags: linux encryption backup privacy ecryptfs
---

Encryption is a good thing, especially in my country where the government's desire to know and control every detail of private citizens' lives would make the Chinese and the Russians blush :) But don't lock yourself out of your own data as I thought I had yesterday.

I mostly use ubuntu/xubuntu based desktop machines and laptops at home, and always take the option to encrypt home directories (the entire disk on laptops), and always partition disks manually giving `/home` it's own. Ubuntu uses `ecryptfs` to manage the encryption and the data is automatically unlocked/decrypted when logging in; transparently to the user. So although my `/home/darren` directory looks perfectly normal to me when logged in and using the machine, to someone stealing the PC or disks and trying to mount them as slave drives, both the file names and the contents are unreadable;

```
darren@hepburn ~ $ ls -al /home/.ecryptfs/darren/.Private/
total 63980
drwx------  145 darren darren    49152 Apr  8 11:03 .
drwxr-xr-x    4 darren darren     4096 Apr  7 19:39 ..
drwxr-xr-x   86 darren darren    20480 Mar  1 00:43 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI01Sp1Y7Mbni.Gv5IWwPXFk--
drwxr-xr-x   11 darren darren    16384 Dec 11 01:15 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI0hV3GgFcVCr4j.37GTijik--
drwxrwxr-x    4 darren darren     4096 Jan  9  2017 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI2qeaiy7FAxmZ-uOnjdTHfk--
lrwxrwxrwx    1 darren darren      124 Nov  1  2015 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI2qI8IxpUtKovbFZLOS3WYE-- -> ECRYPTFS_FNEK_ENCRYPTED.FYb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeIpAMZD0J4V8wTCy4V9qGATRNRu0athRyAOGmqRvz8TNZ9E5JC8OFeRU.GiqPEP4ff
drwxrwxr-x    4 darren darren     4096 Nov  2  2015 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI2TBPVOoQu-9B662Qkm0f4k--
drwx------    2 darren darren     4096 Mar 10 11:36 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI2UvGRHd5W373h3TUCMgIO---
-rw-r--r--    1 darren darren     8192 Nov 13  2014 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI36yx0OBfD4BloNJv1XuD7U--
drwxrwxr-x    3 darren darren     4096 Nov  2  2015 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI3BypztnWrbxdQCZeBVvkkU--
drwxr-xr-x    2 darren darren     4096 Oct 23 13:47 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI3W6lWb1Jj.WUEt67ELAvWU--
-rw-r--r--    1 darren darren    12288 Feb  1  2014 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI44fBeh9bFGMuzrmJCIons---
-rw-r--r--    1 darren darren 13201408 Dec 28 00:53 ECRYPTFS_FNEK_ENCRYPTED.FWb.0NYQdohL4-RW4UL0G74SruUD2-J3nPeI458.0ZUYSR3s54SSxkI3v---
...
```

So, when it comes to backing up the contents of these directories, you should only be backing up the encrypted version and not the plain text versions. Seems obvious, but I've seen more than a few people suggesting that you do precisely that (rsync the plain text version somewhere while logged in) when answering queries about backing up encrypted home directories. Don't do that.

Whatever backup method you choose - dedicated backup software, simple rsync, [rsnapshot](https://www.rsnapshot.org) (which is the one I use) - target the `/home/.ecryptfs` directory as the source. This is the relevant bit from my `/etc/rsnapshot.conf` file for the machine named `hepburn` which is my main desktop.

```
backup	/home/.ecryptfs			hepburn/
backup	/etc/				hepburn/
backup	/usr/local/			hepburn/
backup	/root/				hepburn/
```

All good. Your backed up copies of `$HOME` are as safe on disk as your master copy. If you have a problem and need to restore data, simply mount a temporary decrypted version of your backup directory and copy out the files you need before unmounting the temporary directory again. And this is where I nearly had a heart attack yesterday because I forgot how to do that. Since Ubuntu does it for me on my home dir at login, and I'd not needed to recover files for some years, I just got completely lost with the steps to perform what I outlined above. Pretty stupid of me, so here I'll document the steps for my own benefit (maybe yours too if you're here reading this).

When you first encrypt your partition or directory at install time, ubuntu (ecryptfs if you're performing it manually) will generate a recovery password and ask you to store it safely. It's a 32-char hex key and is used as the passphrase for the private key to decrypt the data. If you do lose it, you can recover it with the `ecryptfs-unwrap-passphrase` utility, which is basically what ubuntu does on login in order to use it temporarily before re-wrapping it. I had kept mine safe in my [key and password manager](https:///www.keepassx.org/) so I had it ready to use.

Transcript of the salient commands below then as a reference for what to do next time I'm careless enough to delete stuff I *really* need and dumb enough to forget how to recover.

```
darren@hepburn ~ $ sudo ecryptfs-add-passphrase --fnek
[sudo] password for darren: 
Passphrase: 
Inserted auth tok with sig [aaaaaaaaaaaaaaaa] into the user session keyring
Inserted auth tok with sig [bbbbbbbbbbbbbbbb] into the user session keyring
```
... use the 32-char passphrase at the `Passphrase:` prompt. It needs to be added as root because root is the user that will eventually have to mount the directory. This could be a problem if you have non-root-capable users on your machine. Take note of both of the auth tokens inserted that I've masked above as a's and b's.

```
darren@hepburn ~ $ mkdir -p /tmp/recovery

darren@hepburn ~ $ sudo mount -t ecryptfs /path/to/your/backed-up-encrypted/home/.ecryptfs/darren/.Private/ /tmp/recovery
Passphrase: 
Select cipher: 
 1) aes: blocksize = 16; min keysize = 16; max keysize = 32
 2) blowfish: blocksize = 8; min keysize = 16; max keysize = 56
 3) des3_ede: blocksize = 8; min keysize = 24; max keysize = 24
 4) twofish: blocksize = 16; min keysize = 16; max keysize = 32
 5) cast6: blocksize = 16; min keysize = 16; max keysize = 32
 6) cast5: blocksize = 8; min keysize = 5; max keysize = 16
Selection [aes]: 
Select key bytes: 
 1) 16
 2) 32
 3) 24
Selection [16]: 
Enable plaintext passthrough (y/n) [n]: 
Enable filename encryption (y/n) [n]: y
Filename Encryption Key (FNEK) Signature [aaaaaaaaaaaaaaaa]: bbbbbbbbbbbbbbbb
Attempting to mount with the following options:
  ecryptfs_unlink_sigs
  ecryptfs_fnek_sig=bbbbbbbbbbbbbbbb
  ecryptfs_key_bytes=16
  ecryptfs_cipher=aes
  ecryptfs_sig=aaaaaaaaaaaaaaaa
Mounted eCryptfs
```
... ensure you use the `.Private` directory of the backup location that you want to recover from in the first argument to the mount command. Again, input the 32-char hex key (unwrapped passphrase) if prompted. Accept defaults for the first 3 answers but say `y` for filename encryption. I also found that I had to use the *2nd* of the auth tokens output earlier and not the first which it defaults to, but YMMV.

```
darren@hepburn ~ $ ls -al /tmp/recovery/
total 64012
drwx------  145 darren darren    45056 Apr  8 11:06 .
drwxrwxrwt   37 root   root      61440 Apr  8 13:22 ..
-rw-------   22 darren darren     1372 Apr 17  2018 1q
drwxrwxr-x    2 darren darren     4096 Nov  2  2015 .abook
-rw-r--r--   22 darren darren      524 Mar 22  2006 agp2.bin
drwxrwxr-x    2 darren darren     4096 Nov  2  2015 .android
-rw-rw-r--   22 darren darren       31 Sep 13  2017 .angular-cli.json
-rw-r--r--   22 darren darren      576 May 17  2011 .anyconnect
-rw-------   22 darren darren    64191 Jul 26  2007 appeal.tif
drwx------    3 darren darren     4096 Nov 15  2015 .arcv2
...
```
... et voila, your backed up files that you can now copy what you need from :) 

```
darren@hepburn ~ $ sudo umount /tmp/recovery 
```
... unmount it ASAP.
