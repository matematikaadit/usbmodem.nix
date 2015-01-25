# Examining the Installer

## Download the Archieve

If you wanna follow this tutorial, but don't have the modem or are currently
can't access it, as an alternative you could download the archieve in this repo
[release pages][release]. Here the command you could use in the terminal.

    $ wget https://github.com/matematikaadit/usbmodem.nix/releases/download/v0.0.1/usbmodem.tar.xz

If you wanna check the integrity of the files, you could also download the
accompanied md5sum files and check that files against your downloaded archieve.

    $ wget https://github.com/matematikaadit/usbmodem.nix/releases/download/v0.0.1/usbmodem.tar.xz.md5sum
    $ md5sum -c usbmodem.tar.xz.md5sum
    usbmodem.tar.xz: OK

Using `-c` flag to check the md5sum files, we would get an OK messages like
above if our downloaded files was indeed correct. Also don't forget to
execute the aboves command in the same directory of your downloaded archieve.

Next, we want to extract those archieve and working with it. For that purpose,
execute command below.

    $ tar -xaf usbmodem.tar.xz

Extract the tar using `-x` switch, autoguessing the compression method using
`-a` and provide the archieve name using `-f` switch.

Now, you'll got a `usbmodem` folder in your current directory.

## Starting to Examine

From this point on, we will be working inside the mounted directory, or the
extracted directory if you're using the step above.

    $ cd usbmodem
    $ ls
    ArConfig.dat  AUTORUN.INF  HiLink.app     linux_mbb_install  Startup.ico
    AutoRun.exe   autorun.sh   install_linux  MobileBrServ

Meanwhile, we also wanna determine the type of the files. Hence, we will use
this more robust command.

    $ file *
    ArConfig.dat:      ASCII text, with CRLF line terminators
    AutoRun.exe:       PE32 executable (GUI) Intel 80386, for MS Windows
    AUTORUN.INF:       ASCII text, with CRLF line terminators
    autorun.sh:        Bourne-Again shell script, ASCII text executable
    HiLink.app:        directory
    install_linux:     Bourne-Again shell script, ASCII text executable
    linux_mbb_install: directory
    MobileBrServ:      directory
    Startup.ico:       MS Windows icon resource - 14 icons, 48x48, 16-colors

Owh, what we got now. It seems that we have two bash script here.

    autorun.sh:        Bourne-Again shell script, ASCII text executable
    install_linux:     Bourne-Again shell script, ASCII text executable

Let's start from `autorun.sh` file since it's name implies that this
file must be executed automatically when the storage mounted. Though
there's no Linux distro as far as I know that could do something like
that.

## Examining `autorun.sh`

    $ cat autorun.sh
    #!/bin/bash


    CURRENT_PATH=`echo $0|sed 's/autorun.sh//'`

    "${CURRENT_PATH}"/install_linux

I've ommited extra whitespace in the end of the files.

Now, let see. It's rather a simple script, isn't it. This script just
set the current path based on it's own path, hence it called `echo $0`.
Next, it called `install_linux` script which located in the current path
above.

Well, it seems we must move to `install_linux` script this time.

## Examining `install_linux`

Let see how many lines we have in `install_linux`

    $ wc -l install_linux
    142 install_linux

Ouch, that too much. Let's examine part by part then.

    $ head install_linux
    #!/bin/bash

    #VERSION=22.001.03.01.03

    install_exit()
    {
        echo "Preass any key to exit. "
        read COMMAND
        exit
    }

First, this file declared as bash script indeed. As can be seen
from that first line. Next, there's a comment which mentions
this script version. But for now, we wouldn't care for it.
Now, the last, we got a function definition called `install_exit()`
which will output "Preass any key to exit. " and asking us
to supply a key input then exit.

Ho..ho..ho... Do you see something strange? Yes! it's **Preass**, not
press. I do wonder what it's mean by that. Maybe we must really preass
the key? How do you preass the key again?

Anyway, it's a rather standard command. Let's move to the next part.
We will use sed with 'n,m!d' script to show lines n to m (or in this
script semantics, it means delete other lines that's not between n to
m).

    $ sed '11,20!d' install_linux

    WHEREWHOAMI="`which whoami`"
    ROOTORNOT="`$WHEREWHOAMI`"
    #echo "$ROOTORNOT"
    if [ "$ROOTORNOT" != "root" ]
    then
        echo "You must run the install process by root."
        install_exit
    fi

What's with that superfluous declaration? Why not something simple like

    if [ "$(whoami)" != "root" ]
    then
        echo "You must run the install process by root."
        install_exit
    fi

Ok. We really don't care. Let's continue.

    $ sed '21,30!d' install_linux
    CURRENTPATH="$(dirname "$0")"
    #echo "$CURRENTPATH"
    TMP_FILE_PATH="/tmp/MobileBrServ_AutoRun"
    INSTALL_FILE="/linux_mbb_install"
    SYSCONFIG_PATH="/ArConfig.dat"
    INSTALL_SHELL="/install"
    DATACARD_VERIFY="/DataCard_Verify"
    MBB_BIN_FILE="/mbbservice.bin"
    MBB_FILE="/mbbservice"
    SYSCONFIG_VALUE="VALUE"

Some variable declarations. Oh, also if you note it. The way CURRENTPATH declared
here is inconsistent with how CURRENTPATH declared in `autorun.sh`. Here they
use dirname instead. Though, I would agree with this method compared using sed in
previous script.

Let's go next.

    $ sed '31,40!d' install_linux

    INSTALL_PATH="/usr/local/MobileBrServ"

    LOG_PATH="/tmp/MobileBrServ_autoinstall_log"

    #define the install variable
    FIRST_INSTALL="NO"
    INSTALL_OR_NOT="NO"

    echo "Current path = ${CURRENTPATH}" > ${LOG_PATH}

Another variable declarations and some logging. Anyway, I'm a bit hating that
declaration above, since we couldn't guess which variables point out to an
relative path and which variable point to absolute path. You'll see this in
the next part.

    $ sed '41,64!d' install_linux

    check_ISO()
    {
        echo "begin to verify ISO ..." | tee -a ${LOG_PATH} #> /dev/null 2>&1
        #TESTFILE="${CURRENTPATH}"
        #echo "$TESTFILE"
        #read COMMAND
        if [ ! -f "${CURRENTPATH}${INSTALL_FILE}${MBB_BIN_FILE}" ]
        then
        echo "${CURRENTPATH}${INSTALL_FILE}${MBB_BIN_FILE}"
        echo "the .bin file is not exist! " | tee -a ${LOG_PATH}
        install_exit
        fi

        if [ ! -f "${CURRENTPATH}${INSTALL_FILE}${SYSCONFIG_PATH}" ]
        then
        echo "the ArConfig.dat file is not exist! " | tee -a ${LOG_PATH}
        install_exit
        fi
        echo "verify the ISO succeed !" | tee -a ${LOG_PATH}
    }

[release]: https://github.com/matematikaadit/usbmodem.nix/releases/

<!--
vim:sw=4:sts=4:et:ai:bs=indent,eol,start:
-->
