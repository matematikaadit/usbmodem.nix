# Examining the Installer

### Download the Archieve

If you wanna follow this tutorial, but don't have the modem or are currently
can't access it, as an alternative you could download the archieve in this repo
release pages. Here the command you could use in the terminal.

    $ wget https://github.com/matematikaadit/usbmodem.nix/releases/download/v0.0.1/usbmodem.tar.xz

If you wanna check the integrity of the files, you could download the
accompanied md5sum files and check that files agains your downloaded archieve.

    $ wget https://github.com/matematikaadit/usbmodem.nix/releases/download/v0.0.1/usbmodem.tar.xz.md5sum
    $ md5sum -c usbmodem.tar.xz.md5sum
    usbmodem.tar.xz: OK

Using `-c` flag to check the md5sum files, we would get an OK messages like
above if our downloaded files was indeed correct. Oh, and don't forget to
execute the aboves command in the same directory of your downloaded archieve.

Next, we want to extract those archieve and working with it. For that purpose,
execute commands below.

    $ tar -xaf usbmodem.tar.xz

Extract the tar using `-x` switch, autoguessing the compression method using
`-a` and providing the archieve name using `-f` switch.

Now, you'll got a `usbmodem` folder in your current directory.

### Starting to Examine

From this point on, we will be working inside the mounted directory, or the
extracted directory if you're using the step above.

    $ cd usbmodem
    $ ls
    ArConfig.dat  AUTORUN.INF  HiLink.app     linux_mbb_install  Startup.ico
    AutoRun.exe   autorun.sh   install_linux  MobileBrServ


<!--
vim:sw=4:sts=4:et:ai:bs=indent,eol,start:
-->
