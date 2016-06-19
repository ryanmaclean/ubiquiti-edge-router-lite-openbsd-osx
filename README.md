# ubiquiti-edge-router-lite-openbsd-osx
Install OpenBSD on your Ubiquiti Edge Router Lite from MacOS (OSX)

Retrieve the Shimmery Wondrous USB Stick via Ubiquiti Surgery
=============================================================

* Open your Edge Router Lite. There are three small screws on the back. I used a size 4 phillips screwdriver to get his done.
* Once opened, remove the USB stick
* On your Mac, open the Terminal app
* Type `diskutil list` in the terminal window - this shows you disks currently inserted
* On your Mac, insert the USB stick
* You'll probably only see the FAT32 boot partition show up. LEAVE IT ALONE FOR NOW. 

We'll begin by taking a backup with diskutil
============================================

* Again in the terminal, type `diskutil list` - you should see a new disk entry, take note of the `/dev/disk#`
* Unmount the disk that contains the `Windows_FAT_32` partiton that is 148.9MB using this command: `diskutil unmountDisk /dev/disk#`
* You should be rewarded with the following: `Unmount of all volumes on disk4 was successful` (if not, please close all windows you have open relating to this disk)
* Take a backup of the USB disk to your Desktop with the following command: `cd ~/Desktop && sudo dd if=/dev/disk# of=erl-backup$(date +"%m_%d_%Y").img.dd bs=512`
* Please *patiently* wait for about 10 minutes or so (534 seconds on my MBP) - if you get impatient, you can press `ctrl+t`
* Confirm the file was created: `ls -alh ~/erl-backup*`
* You should receive output similar to the following: `-rw-r--r--  1 root  staff   3.7G 18 Jun 20:05 /Users/erlang/erl-backup06_18_2016.img.dd`
