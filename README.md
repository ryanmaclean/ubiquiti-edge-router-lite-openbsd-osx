# Ubiquiti Edge Router Lite OpenBSD Upgrade on OSX

Install OpenBSD 5.9 on your Ubiquiti Edge Router Lite from MacOS (OSX)

Note that this assuredly will void warranties, guarantees and might be a tax on your sanity: PROCEED AT YOUR OWN PERIL. 

That being said, for roughly $100 USD as of this writing, it's not cheap, but certainly possible to have a backup ERL (Edge Router Lite) handy should something go awry. 

I should also note that the Ubiquiti devices themselves have a pretty darn good WebUI, and have an OK install that includes fun stuff like Python 2.7 stock - you might just want to play around with it before putting OpenBSD on it. 

Tools Required
==============

You'll need some stuff in order to get started. I've added links to Amazon sellers, but note that these aren't affiliate links and that you're probably going to be able to find these cheaper elsewhere, or possible in a store local to you.

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

Boot The Image on Your Edge Router Lite
=======================================
Next we'll need to get our connection to the Ubiquiti ERL working via the console cable. 


