Huawei E303 USB Modem in NixOS
==============================

Huawei E303 USB Modem has official support for Linux. But we can't enjoy this
first class support in NixOS (yet), since the installer included in the
read-only memory storage of the modem isn't working as flawlessly as in other
Linux distro.

So, in order to handle this uncovenience, we will go through hardship of
finding the solution path. A bit long, since it'll mnention the process of how
we able to coming to an answer in each step. But hopefully, it'll become a good
learning source.

Now, let's get to the game.

Mount the modem's read-only storage
-----------------------------------

So, where was this storage's device file placed by our linux kernel?
Well, let's ask dmesg for that.

First, un-plug the usbmodem if it's already plugged. Then re-plug it again and
execute the command below.

    $ dmesg | tail
    [ 1911.985090] usb 2-1.5: new high-speed USB device number 6 using ehci-pci
    [ 1912.071507] usb 2-1.5: New USB device found, idVendor=12d1, idProduct=1f01
    [ 1912.071513] usb 2-1.5: New USB device strings: Mfr=2, Product=1, SerialNumber=0
    [ 1912.071516] usb 2-1.5: Product: HUAWEI HiLink
    [ 1912.071519] usb 2-1.5: Manufacturer: HUAWEI
    [ 1912.072980] usb-storage 2-1.5:1.0: USB Mass Storage device detected
    [ 1912.073168] scsi8 : usb-storage 2-1.5:1.0
    [ 1913.076738] scsi 8:0:0:0: CD-ROM            HUAWEI   Mass Storage     2.31 PQ: 0 ANSI: 2
    [ 1913.078225] sr1: scsi-1 drive
    [ 1913.078676] sr 8:0:0:0: Attached scsi CD-ROM sr1

Looks! it's been detected and the last two line said that a scsi CD-ROM was
attached for our usbmodem storage. I'll copy and paste that part below for
clarity.

    [ 1913.078225] sr1: scsi-1 drive
    [ 1913.078676] sr 8:0:0:0: Attached scsi CD-ROM sr1

It's gives us a clue about where did the device file created under our /dev/ directory.
For those who still unclear, it's in `/dev/sr1`, my buddy. Let's create a new directory
for mounting it, we will name it `usbmodem`.

    $ mkdir usbmodem
    $ sudo mount /dev/sr1 usbmodem
    mount: /dev/sr1 is write-protected, mounting read-only

Great! Let's see what's inside.

    $ ls -lF usbmodem
    total 583
    -r-xr-xr-x 1 root root   1057 Sep 27  2011 ArConfig.dat*
    -r-xr-xr-x 1 root root 144224 Sep 27  2011 AutoRun.exe*
    -r-xr-xr-x 1 root root     45 Jun 17  2011 AUTORUN.INF*
    -r-xr-xr-x 1 root root     94 Mar 31  2011 autorun.sh*
    dr-xr-xr-x 1 root root   2048 Sep  3  2011 HiLink.app/
    -r-xr-xr-x 1 root root   3262 Jun 18  2011 install_linux*
    dr-xr-xr-x 1 root root   2048 Sep 28  2011 linux_mbb_install/
    dr-xr-xr-x 1 root root   2048 Sep 27  2011 MobileBrServ/
    -r-xr-xr-x 1 root root 439926 Nov 26  2010 Startup.ico*

We using `-l` for long listing format, and `-F` for clarity about which one is a
normal file, executable, folder, or link by showing an extra character at the
end of the file name.  `*` means an executable, and `/` means directory. Though
you can always see that from the file's mode.

### Extra

Let's back up this storage, and put it somewhere online so that for the next
time we don't need to go through that step again. For this purpose, We must
choose a best compression strategy so that we could safe ourselves a few
megabytes for our archieve aboves.  Below, I've tried a few.

    $ for ext in gz xz bz2; do tar -caf usbmodem.tar.$ext usbmodem; done

Using `-c` flag for archieve creation, `-a` flag so that it'll automatically
guess the compression strategy by using the file's extension, and `-f` for
mention of the archieve's name. Now, let's compare the size.

    $ du -s usbmodem*
    4636    usbmodem
    3416    usbmodem.tar.bz2
    3540    usbmodem.tar.gz
    3156    usbmodem.tar.xz

The size above are in KB. We use `-s` (`--summarize`) options of du (disk
usage), so it won't recursively traverse the files inside the usbmodem
directory. Anyway, from the data above, we can see that usbmodem.tar.xz is the
winner. We able to save about 1.48M disk space (4636 - 3156 = 1480) for our
archieve. Now, delete the other archives and upload usbmodem.tar.xz somewhere.

    $ rm usbmodem.tar.{bz2,gz}
    $ # upload usbmodem.tar.xz somewhere

I've uploaded the archieve in [this repo release page][release], hence you
could just download that files for yourself. For now, we are going to go to the
part two of our journey.  I'll stop the intro here and pointing you out to
[1-examining.md](1-examining.md). 

Happy Hacking!

[release]: https://github.com/matematikaadit/usbmodem.nix/releases/download/v0.0.1/usbmodem.tar.xz

<!--
vim:sw=4:ts=4:et:ai:bs=indent,eol,start:
-->
