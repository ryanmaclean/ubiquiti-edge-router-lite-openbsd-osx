# Ubiquiti Edge Router Lite 3 OpenBSD Upgrade on OSX

Install OpenBSD 5.9 on your Ubiquiti Edge Router Lite 3 (ERL3) from MacOS (OSX)

Note that this assuredly will void warranties, guarantees and might be a tax on
your sanity: PROCEED AT YOUR OWN PERIL.

That being said, for roughly $100 USD as of this writing, it's not cheap, but
certainly possible to have a backup ERL (Edge Router Lite) handy should
something go awry.

I should also note that the Ubiquiti devices themselves have a pretty darn good
WebUI, and have an OK install that includes fun stuff like Python 2.7 stock -
you might just want to play around with it before putting OpenBSD on it.

There is in fact a guide for this already, but as it is not specific to the
ERL3 (as well taking some TFTP detours you may prefer), I decided to make a
more lengthy, focused guide. The official Octeon OpenBSD Instructions are here:
<http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/INSTALL.octeon>

## Tools Required

You'll need some stuff in order to get started. I've added links to Amazon
sellers, but note that these aren't affiliate links and that you're probably
going to be able to find these cheaper elsewhere, or possibly in a store local
to you.

* Serial connection to your Ubiquiti Edge Router Lite.
* Phillips size 0 screwdriver
* Your local admin password
* Some patience
* A beverage

## Retrieve the Shimmery Wondrous USB Stick via Ubiquiti Surgery

* Open your Edge Router Lite. There are three small screws on the back.
* Size 0 phillips screwdriver will fit snugly.
* Once opened, remove the USB stick.
* On your Mac, open the Terminal app
* Type `diskutil list` in the terminal - this shows you disks currently inserted
* On your Mac, insert the USB stick
* You'll probably only see the FAT32 boot partition show up. LEAVE IT ALONE.
* Again in the terminal, type `diskutil list` - take note of the `/dev/disk#`
* Use this command to unmount 150MB fat disk: `diskutil unmountDisk /dev/disk#`
* You should be rewarded with the following:
* `Unmount of all volumes on disk4 was successful`
* (if not, please close all windows you have open relating to this disk)

## We'll begin by taking a backup with diskutil

* Take a backup of the USB disk to your Desktop with the following command:
* `sudo dd if=/dev/disk# of=erl-backup$(date +"%m_%d_%Y").img.dd bs=512`
* Chances are, you'll need to enter your local admin password at the prompt
* Please *patiently* wait for about 10 minutes or so (534 seconds on my MBP)
* if you do happen to get impatient, press `ctrl+t` in order to see progress
* Confirm the file was created: `ls -alh ~/erl-backup*`
* You should receive output similar to the following: `-rw-r--r--  1 root * staff 3.7G 18 Jun 20:05 /Users/erlang/erl-backup06_18_2016.img.dd`

## Download OpenBSD from a Mirror

You'll want a copy of the miniroot for OpenBSD 5.9 prior to moving forward.
I got mine from here: <http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/>

* Via commandline: `curl -O http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/miniroot59.fs`

You'll get output similar to the following, letting you know it's done (100%):

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12.0M  100 12.0M    0     0   461k      0  0:00:26  0:00:26 --:--:--  622k
```

## Write the Mini Root Image to the USB Stick

Remember that disk # we retrieved oh so long ago? Yep, we'll need it again:

* Unmount the disk: `diskutil unmountDisk /dev/disk#`
* Use the following command to overwrite the disk with our miniroot:
* `sudo dd if=miniroot59.fs of=/dev/disk#`

Chances are good that you won't need to enter your password again, but in
case you've taken a bio-break, be prepared to enter it once more

Once completed, you'll see output probably identical to the following:

```
24576+0 records in
24576+0 records out
12582912 bytes transferred in 21.667341 secs (580732 bytes/sec)
```

Huzzah! You've got a miniroot image ready to go!

## Move the USB Stick From Your Mac to the ERL

Our next step is to remove the USB stick from our Mac. Once unmounted, you can
simply yank it straight out and slap it in the ERL.

* In the terminal: `diskutil unmountDisk /dev/disk#`
* Gently but firmly grasp the USB stick and remove it from your Mac
* Insert the USB stick into the Edge Router Lite once more
* Screw the whole thing back together - we're being optimistic here :smirk:
* Attach the console cable to your ERL in the "console" port
* Attach the other end of said cable into your Mac in a free USB port

## Connect Your Mac to Your Router

Next we'll need to get our connection to the Ubiquiti ERL working via the
console cable. I'm assuming you've only got one FTDI cable connected. If you
have more than one connected, I'll also assume you've got an app to do this,
or may prefer to use `minicom`... The salient thing to remember here is that
`9600` won't suffice: you'll need to connect at a rate of `115200`. The
 following command is a one-liner that worked for me.

Still in the terminal, type the following:

```
screen $(ls -ltr /dev/*usb* | grep tty | cut -d " " -f 16) 115200`
```

You should get a blank screen - that's a good thing! Don't panic, we'll put
some text in that scary void shortly.

### NOTE

If you don't get a blank screen and instead get some error, chances are
some other things are going on, you can troubleshoot with: `ls -ltr /dev/*usb*`.
Do you see a "usbserial" device there, or nothing? If nothing... there might be
an issue with your cable, unfortunately, and you'll need to get that working
prior to moving forward._

## Get the Ubiquiti Router Up and Running

Now it's time to apply power to your ERL. If you had somehow jumped ahead and
plugged in the power already, please remove it - we'll need to interrupt the
boot process before moving forward.

* Plug in the power to the Ubiquiti router
* In screen, press the "enter" key a few times to interrupt the boot process

If successful, you should see the following prompt:

```
Octeon ubnt_e100#
```

This is fantastic news - we're ready load our OpenBSD 5.9 mini root image!

## Boot The Image on Your Edge Router Lite

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

## Complete the OpenBSD Install

The rest of this will involve installing OpenBSD to your USB stick. Remember
that you can back up the install at any time using our process above.

If you're OK using eth0 as your primary interface, you can follow along with
my config, but otherwise I'd highly recommend the official install
instructions: <http://ftp.openbsd.org/pub/OpenBSD/5.9/octeon/INSTALL.octeon>

## OpenBSD Install

* Press "i" on your keyboard to continue along with the instal
* Press "enter" to select the default vt220
* System hostname: this is up to you, I used "octeon1"
* Press "enter" to use the default "cnmac0" (this is eth0 as printed on the ERL)
* Press "enter" to not use IPv6 (unless you use it, of course)
* Press "enter" to complete the network interface configuration
* Enter a password for your root account (text will not show)
* Repeat the password, again, no feedback
* Press "enter" to start sshd by default
* Setup a local user - I recommend this instead of using ssh as root
* Either type the full name for this user, or hit enter to keep it the same
* Enter the password for the user, then repeat it
* Hit "enter" to disallow root login (highly recommended)
* Enter your timezone, but you can hit "?" to list them
* Hit "enter" to select your root disk `sd0` (there should only be one :smile:
* Press "enter" once more to select the whole disk
* Again, "enter" to use auto layout
* Once the disk preparation is done, hit enter to use HTTP as the protocol
* Press "enter" again in order to not use a proxy
* As usual, press enter to use the default BSD mirror
* Select the default server directory by pressing "enter"
* Press "enter" to select the default sets
* Wait while the sets are grabbed from the mirror, extracted, then installed
* Press "enter" for *almost* the last time to accept that you're done
* Finally, hit enter again to set the time!

We've installed the OS at this point, but will need to reboot in order to load
it from scratch.

* Type `reboot` to reboot your ERL

## Swap the Bootloader

First we'll backup the old bootloader, then set our own as the default:

* `setenv old_bootcmd "${bootcmd}"`
* `setenv bootcmd 'fatload usb 0 $loadaddr bsd;bootoctlinux rootdev=/dev/sd0'`
* `setenv bootdelay 5`
* `saveenv`

## Restart One Last Time

Now we'll need to give the router one last reboot in order to test the u-boot
booatloader chaining:

```
reset
```

Result:

```
Octeon ubnt_e100# reset

Looking for valid bootloader image....
Jumping to start of image at address 0xbfc80000


U-Boot 1.1.1 (UBNT Build ID: 4670715-gbd7e2d7) (Build time: May 27 2014 -
11:16:22)

BIST check passed.
UBNT_E100 r1:2, r2:18, f:4/71, serial #: 0418D6F15211
MPR 13-00318-18
Core clock: 500 MHz, DDR clock: 266 MHz (532 Mhz data rate)
DRAM:  512 MB
Clearing DRAM....... done
Flash:  4 MB
Net:   octeth0, octeth1, octeth2

USB:   (port 0) scanning bus for devices... 1 USB Devices found
       scanning bus for storage devices...
  Device 0: Vendor:          Prod.: USB DISK 2.0     Rev: PMAP
            Type: Removable Hard Disk
            Capacity: 3824.0 MB = 3.7 GB (7831552 x 512)
reading bsd
.......
.....
..............

5228781 bytes read
ELF file is 64 bit
Allocating memory for ELF segment: addr: 0xffffffff81000000 (adjusted to:
0x1000000), size 0x504890
Allocated memory for ELF segment: addr: 0xffffffff81000000, size 0x504890
Processing PHDR 0
  Loading 47e368 bytes at ffffffff81000000
  Clearing 86528 bytes at ffffffff8147e368
## Loading Linux kernel with entry point: 0xffffffff81000000 ...
Bootloader: Done loading app on coremask: 0x1
Total DRAM Size 0x0000000020000000
mem_layout[0] page 0x0000000000000542 -> 0x0000000000004000
mem_layout[1] page 0x0000000000104000 -> 0x0000000000108000
boot_desc->argv[1] = rootdev=/dev/sd0
Initial setup done, switching console.
boot_desc->desc_ver:7
boot_desc->desc_size:400
boot_desc->stack_top:0
boot_desc->heap_start:0
boot_desc->heap_end:0
boot_desc->argc:2
boot_desc->flags:0x5
boot_desc->core_mask:0x1
boot_desc->dram_size:512
boot_desc->phy_mem_desc_addr:0
boot_desc->debugger_flag_addr:0xa44
boot_desc->eclock:500000000
boot_desc->boot_info_addr:0x1001f0
boot_info->ver_major:1
boot_info->ver_minor:2
boot_info->stack_top:0
boot_info->heap_start:0
boot_info->heap_end:0
boot_info->boot_desc_addr:0
boot_info->exception_base_addr:0x1000
boot_info->stack_size:0
boot_info->flags:0x5
boot_info->core_mask:0x1
boot_info->dram_size:512
boot_info->phys_mem_desc_addr:0x24108
boot_info->debugger_flags_addr:0
boot_info->eclock:500000000
boot_info->dclock:266000000
boot_info->board_type:20002
boot_info->board_rev_major:2
boot_info->board_rev_minor:18
boot_info->mac_addr_count:3
boot_info->cf_common_addr:0
boot_info->cf_attr_addr:0
boot_info->led_display_addr:0
boot_info->dfaclock:0
boot_info->config_flags:0x8
Copyright (c) 1982, 1986, 1989, 1991, 1993
        The Regents of the University of California.  All rights reserved.
Copyright (c) 1995-2016 OpenBSD. All rights reserved.  http://www.OpenBSD.org

OpenBSD 5.9 (GENERIC) #9: Wed Mar  2 11:00:35 CET 2016
    jasper@erl.jasper.la:/usr/src/sys/arch/octeon/compile/GENERIC
real mem = 536870912 (512MB)
avail mem = 510427136 (486MB)
warning: no entropy supplied by boot loader
mainbus0 at root
cpu0 at mainbus0: Cavium OCTEON CPU rev 0.1 500 MHz, Software FP emulation
cpu0: cache L1-I 32KB 4 way D 8KB 64 way, L2 128KB 8 way
clock0 at mainbus0: int 5
iobus0 at mainbus0
dwctwo0 at iobus0 base 0x1180068000000 irq 56
usb0 at dwctwo0: USB revision 2.0
uhub0 at usb0 "Octeon DWC2 root hub" rev 2.00/1.00 addr 1
octrng0 at iobus0 base 0x1400000000000 irq 0
cn30xxgmx0 at iobus0 base 0x1180008000000 irq 48
cnmac0 at cn30xxgmx0: RGMII, address 04:18:d6:f1:52:11
atphy0 at cnmac0 phy 7: F1 10/100/1000 PHY, rev. 2
cnmac1 at cn30xxgmx0: RGMII, address 04:18:d6:f1:52:12
atphy1 at cnmac1 phy 6: F1 10/100/1000 PHY, rev. 2
cnmac2 at cn30xxgmx0: RGMII, address 04:18:d6:f1:52:13
atphy2 at cnmac2 phy 5: F1 10/100/1000 PHY, rev. 2
uar: ns16550, no working fifo
com0: console
com1 at uartbus0 base 0x1180000000c00 irq 35: ns16550, no working fifo
/dev/ksyms: Symbol table not valid.
umass0 at uhub0 port 1 configuration 1 interface 0 " USB DISK 2.0" rev
2.00/1.00 addr 2
umass0: using SCSI over Bulk-Only
scsibus0 at umass0: 2 targets, initiator 0
sd0 at scsibus0 targ 1 lun 0: <, USB DISK 2.0, PMAP> SCSI4 0/direct removable
serial.13fe41004CA98DCA8590
sd0: 3824MB, 512 bytes/sector, 7831552 sectors
vscsi0 at root
scsibus1 at vscsi0: 256 targets
softraid0 at root
scsibus2 at softraid0: 256 targets
boot device: sd0
root on sd0a (207d650e9076bd8f.a) swap on sd0b dump on sd0b
WARNING: No TOD clock, believing file system.
WARNING: CHECK AND RESET THE DATE!
Automatic boot in progress: starting file system checks.
/dev/sd0a (207d650e9076bd8f.a): file system is clean; not checking
/dev/sd0e (207d650e9076bd8f.e): file system is clean; not checking
/dev/sd0d (207d650e9076bd8f.d): file system is clean; not checking
setting tty flags
pf enabled
starting network
cnmac0: link is not up, the packet was dropped
DHCPREQUEST on cnmac0 to 255.255.255.255
DHCPACK from 10.0.111.1 (6d:73:9c:a2:25:88)
bound to 10.0.111.190 -- renewal in 15768000 seconds.
starting early daemons: syslogd pflogd ntpd.
starting RPC daemons:.
savecore: /bsd: kvm_read: version misread
checking quotas: done.
kvm_mkdb: can't open /dev/ksyms
clearing /tmp
kern.securelevel: 0 -> 1
creating runtime link editor directory cache.
preserving editor files.
starting network daemons: sshd smtpd sndiod.
starting local daemons: cron.
Sat Jun 18 23:44:47 PDT 2016

OpenBSD/octeon (octeon1) (console)

login:
```

## Update OpenBSD

Now that everything's all installed and working, you'll want to run an update:

```
su -
pkg_add -Uu
```

## Restoring Linux

Insert the old usb stick.

```
setenv oldbootcmd 'fatload usb 0 $loadaddr vmlinux.64;bootoctlinux $loadaddr
 coremask=0x3 root=/dev/sda2 rootdelay=15 rw rootsqimg=squashfs.img
 rootsqwdir=w mtdparts=phys_mapped_flash:512k(boot0),512k(boot1),64k@1024k(eeprom)'
setenv bootcmd 'sleep 10; usb reset; sleep 10; $(oldbootcmd)'
saveenv
reset
```
