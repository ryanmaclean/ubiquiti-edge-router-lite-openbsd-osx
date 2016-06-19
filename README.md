# Ubiquiti Edge Router Lite OpenBSD Upgrade on OSX

Install OpenBSD 5.9 on your Ubiquiti Edge Router Lite from MacOS (OSX)

Note that this assuredly will void warranties, guarantees and might be a tax on your sanity: PROCEED AT YOUR OWN PERIL. 

That being said, for roughly $100 USD as of this writing, it's not cheap, but certainly possible to have a backup ERL (Edge Router Lite) handy should something go awry. 

I should also note that the Ubiquiti devices themselves have a pretty darn good WebUI, and have an OK install that includes fun stuff like Python 2.7 stock - you might just want to play around with it before putting OpenBSD on it. 

There is in fact a guide for this already, but as it is not specific to the ERL (as well taking some TFTP detours you may prefer), I decided to make a more lengthy, focused guide. The official Octeon OpenBSD Instructions are here: http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/INSTALL.octeon 

Tools Required
==============

You'll need some stuff in order to get started. I've added links to Amazon sellers, but note that these aren't affiliate links and that you're probably going to be able to find these cheaper elsewhere, or possibly in a store local to you.

* Serial connection to your Ubiquiti Edge Router Lite. I'd suggest a FTDI Cisco knock-off cable like this: https://www.amazon.com/Cisco-Console-Cable-Windows-Linux/dp/B00RY3ELKG 
* Phillips size 0 screwdriver: https://www.amazon.com/Stahlwille-4752-0-Phillips-Recess-Cross-Head-Screwdriver/dp/B00C12AY7O
* Your local admin password (most likely the same one you use on your phone for the App Store these days...)
* Some patience
* A beverage

Retrieve the Shimmery Wondrous USB Stick via Ubiquiti Surgery
=============================================================

* Open your Edge Router Lite. There are three small screws on the back. I used a size 000 phillips screwdriver to get his done, though upon further examination a 0 would have been the correct size (it's also much more common).
* Once opened, remove the USB stick
* On your Mac, open the Terminal app
* Type `diskutil list` in the terminal window - this shows you disks currently inserted
* On your Mac, insert the USB stick
* You'll probably only see the FAT32 boot partition show up. LEAVE IT ALONE FOR NOW. 
* Again in the terminal, type `diskutil list` - you should see a new disk entry, take note of the `/dev/disk#`
* Unmount the disk that contains the `Windows_FAT_32` partiton that is 148.9MB using this command: `diskutil unmountDisk /dev/disk#`
* You should be rewarded with the following: `Unmount of all volumes on disk4 was successful` (if not, please close all windows you have open relating to this disk)

We'll begin by taking a backup with diskutil
============================================

* Take a backup of the USB disk to your Desktop with the following command: `cd ~/Desktop && sudo dd if=/dev/disk# of=erl-backup$(date +"%m_%d_%Y").img.dd bs=512`
* Chances are, you'll need to enter your local admin password at the prompt
* Please *patiently* wait for about 10 minutes or so (534 seconds on my MBP) - if do happen to get impatient, you can press `ctrl+t` in order to see what's going on
* Confirm the file was created: `ls -alh ~/erl-backup*`
* You should receive output similar to the following: `-rw-r--r--  1 root  staff   3.7G 18 Jun 20:05 /Users/erlang/erl-backup06_18_2016.img.dd`

Download OpenBSD from a Mirror
==============================

You'll want a copy of the miniroot for OpenBSD 5.9 prior to moving forward. I got mine from here: 
http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/

* Via commandline: `curl -O http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/miniroot59.fs`

You'll get output similar to the following, letting you know it's done (100%):

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12.0M  100 12.0M    0     0   461k      0  0:00:26  0:00:26 --:--:--  622k
```

Write the Mini Root Image to the USB Stick
==========================================

Remember that disk # we retrieved oh so long ago? Yep, we'll need it again:

* Use the following command to overwrite the disk with our miniroot: `sudo dd if=miniroot59.fs of=/dev/disk#`
* Chances are good that you won't need to enter your password again, but in case you've taken a bio-break, be prepared to enter it once more

Once completed, you'll see output probably identical to the following:

```
24576+0 records in
24576+0 records out
12582912 bytes transferred in 21.667341 secs (580732 bytes/sec)
```

Huzzah! You've got a miniroot image ready to go!

Move the USB Stick From Your Mac to the ERL
===========================================

Our next step is to remove the USB stick from our Mac; it's currently un-mounted, so there'll be no messing about with unmounting and getting warnings, thank you very much. You can simply yank it straight out and slap it in the ERL. 

* Gently but firmly grasp the USB stick and remove it from your Mac
* Insert the USB stick into the Edge Router Lite once more
* Screw the whole thing back together - we're being optimistic here :smirk:
* Attach the console cable to your ERL in the "console" port
* Attach the other end of said cable into your Mac in a free USB port

Connect Your Mac to Your Router
===============================
Next we'll need to get our connection to the Ubiquiti ERL working via the console cable. I'm assuming you've only got one FTDI cable connected. If you have more than one connected, I'll also assume you've got an app to do this, or may prefer to use `minicom`... The salient thing to remember here is that `9600` won't suffice: you'll need to connect at a rate of `115200`. The following command is a one-liner that worked for me. 

* Still in the terminal, type the following: `screen $(ls -ltr /dev/*usb* | grep tty | cut -d " " -f 16) 115200`

You should get a blank screen - that's a good thing! Don't panic, we'll put some text in that scary void shortly.

_NOTE: If you don't get a blank screen and instead get some error, chances are some other things are going on, you can troubleshoot with: `ls -ltr /dev/*usb*`. Do you see a "usbserial" device there, or nothing? If nothing... there might be an issue with your cable, unfortunately, and you'll need to get that working prior to moving forward._ 

Get the Ubiquiti Router Up and Running
======================================

Now it's time to apply power to your ERL. If you had somehow jumped ahead and plugged in the power already, please remove it - we'll need to interrupt the boot process before moving forward. 

* Plug in the power to the Ubiquiti router
* In screen, press the "enter" key on your keyboard a few times in order to interrupt the boot process

If successful, you should see the following prompt:

```
Octeon ubnt_e100#
```

This is fantastic news - we're ready load our OpenBSD 5.9 mini root image!

Boot The Image on Your Edge Router Lite
=======================================

* To load your miniroot: `fatload usb 0 $loadaddr bsd.rd`

This should result in the following output:

```
reading bsd.rd
..........
........................
.......

8379834 bytes read
```

Are you ready? It's go time!

* Boot the image: `bootoctlinux`

Tons of output will result, the last of which is the following prompt:

```
Welcome to the OpenBSD/octeon 5.9 installation program.
(I)nstall, (U)pgrade, (A)utoinstall or (S)hell?
```

Complete the OpenBSD Install
============================

The rest of this will involve installing OpenBSD to your USB stick. Remember that you can back up the install at any time using our process above. 

If you're OK using eth0 as your primary interface, you can follow along with my config, but otherwise I'd highly recommend the official install instructions: http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/INSTALL.octeon

OpenBSD Install
===============

* Press "i" on your keyboard to continue along with the instal
* Press "enter" to select the default vt220 
* System hostname: this is up to you, I used "octeon1"
* Press "enter" to use the default "cnmac0" (this is eth0 as printed on the ERL)
* Press "enter" to not use IPv6 (unless you use it, of course)
* Press "enter" to complete the network interface configuration
* Enter a password for your root account, note that there will not be any feedback in screen as you type it in
* Repeat the password, again, no feedback
* Press "enter" to start sshd by default
* Setup a local user - I recommend this instead of using ssh as root - type the name in
* Either type the full name for this user, or hit enter to keep it the same
* Enter the password for the user, then repeat it
* Hit "enter" to disallow root login (highly recommended)
* Enter your timezone - mine was `America/Vancouver`, but you can hit "?" to list them
* Hit "enter" to select your root disk `sd0` (there should only be one :smile:
* Press "enter" once more to select the whole disk
* Again, "enter" to use auto layout
* Once the disk preparation is done, hit enter to use HTTP as the protocol
* Press "enter" again in order to not use a proxy
* As usual, press enter to use the default BSD mirror (mine was `ftp.OpenBSD.org`
* Select the default server directory by pressing "enter"
* Press "enter" to select the default sets ( you *may* want to remove the `game59.tgz` set, I kept it!)
* Wait while the sets are grabbed from the mirror, extracted, then installed (about 20 minutes on a 100mbit link for me)
* Press "enter" for *almost* the last time to accept that you're done
* Finally, hit enter again to set the time! 

We've installed the OS at this point, but will need to reboot in order to load it from scratch. 

* Type `reboot` to reboot your ERL into OpenBSD mode!
