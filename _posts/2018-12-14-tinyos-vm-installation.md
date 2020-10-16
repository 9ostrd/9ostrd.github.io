---
layout: post
title: TinyOS 2.1.2 on Ubuntu installation
excerpt: ""
categories: [installation]
comments: true
---
This article has re-created an instruction to install TinyOS on currently Linux OS. The Linux that tested is Ubuntu Desktop 14.04 or Ubuntu 16.04 64 bit on VirtualBox version 5.2.22 r126460 (Qt5.6.3).

# VM specifications

Type: Linux 64 bit

Version: Ubuntu (14.04 or 16.04)

Set the Memory & size to 2048 MB (2 GB)

Dynamically allocated 20 GB

# Get prerequisites software

Vim is an editor that a bit more intuitive than the default Linux editor like vi. Open a Terminal window and type the following command.

```bash
$ sudo apt-get install vim
```

Make sure that the installation completed without any warnings. A command summary of vim is available here and in many other places:  <http://www.fprintf.net/vimCheatSheet.html>

# Install TinyOS compilers for various processors

Update the <strong>/etc/apt/sources.list.d/tinyprod-debian.list</strong> file and add two repositories location at the end of file. These repositories contain the compilers need. Using vim:

```bash
$ sudo vim /etc/apt/sources.list.d/tinyprod-debian.list
```

Add two repositories location at the end of files: 

```bash
deb http://tinyprod.net/repos/debian wheezy main
deb http://tinyprod.net/repos/debian msp430-46 main
```

Save the file and exit back to the prompt. then update package dependencies.

```bash
sudo apt-get update
```

In this process, the error of a repository security key will appear. So, the repository security key needs to install on the system by following commands: 

```bash
$ gpg  --keyserver keyserver.ubuntu.com  --recv-keys <new key>
$ gpg -a  --export <new key> | sudo apt-key add - 
```

After the installation, package dependencies should update without any warning or error.

```bash
$ sudo apt-get update
```
then,

```bash
$ sudo apt-get install git msp430-46 nesc tinyos-tools
```

# Get TinyOS

Working on home directory, download TinyOS: 

```bash
$ wget http://github.com/tinyos/tinyos-release/archive/tinyos-2_1_2.tar.gz
```

unzip the file and rename the directory to "tinyos-main": 

```bash
$ tar xf tinyos-2_1_2.tar.gz
$ mv tinyos-release-tinyos-2_1_2 tinyos-main
```

# Setup TinyOS dev. environment

Add environment variables to the terminal. Using <strong>vim</strong> edit <strong>.bashrc</strong> and add the following lines at the end of the file:

```bash
export TOSROOT="$HOME/tinyos-main"
export TOSDIR="$TOSROOT/tos"
export CLASSPATH=$CLASSPATH:$TOSROOT/support/sdk/java/tinyos.jar
export MAKERULES="$TOSROOT/support/make/Makerules"
export PYTHONPATH=$PYTHONPATH:$TOSROOT/support/sdk/python

echo "setting up TinyOS on source path $TOSROOT"
```

Instantly, update the terminal configuration:

```bash
source .bashrc
```

# Test the installation

Testing the environment: 
 
```bash
$ tos-check-env 
```

If everything properly, there are only warning about Graphviz shows on the terminal. Because it not being version 1.10 (Maybe the warning appear twice): 
 
```bash
$ dot –V 
```

To check if there is a Z1 mote connected:

```bash
$ motelist
Reference Device Description
---------- ---------------- --------------------------------------------- 
Z1RC1031 /dev/ttyUSB0 Silicon Labs Zolertia Z1
```

Note: due to the recent integration with TinyOS core at trunk, the scripts "motelist" and "tos-bsl" was formerly named "motelist-z1" and "z1-bsl". If you have an outdated version of the Z1 patch file offered above (tinyos-z1-2.1.1-18), you can either manually rename the old scripts. To see where the scripts are located and resolve any possible conflict look at the locations:

```bash
$ whereis motelist motelist-z1
```

In order to program motes you will need to access the serial ports. Joining the group that grants this privilege can grant you this access. The Linux command gpasswd administer the /etc/group file and dialout is the group that owns the serial ports,  so the following command will add you to this group. Run: 

```bash
$ sudo gpasswd -a <your-user> dialout 
```

This change only takes effect when logout from Ubuntu and login again. 

# Compiling the first TinyOS Program

To check if the installation succeeded we have to open a new terminal and execute the following commands:

```bash
$ cd $TOSROOT/apps/Blink 
$ make z1
```

If the setup is correct the output should be similar to this:

```bash
compiling BlinkAppC to a z1 binary
ncc -o build/z1/main.exe -Os -fnesc-separator=__ 
-mdisable-hwmul -Wall -Wshadow -Wnesc-all -target=z1 
-fn -board= -DDEFINED_TOS_AM_GROUP=0x22 -DIDENT_APPNAME=\"BlinkAppC\" 
-DIDENT_USERNAME=\"jordi\" -DIDENT_HOSTNAME -DIDENT_USERHASH=0x145d5669L 
-DIDENT_TIMESTAMP=0x4afab531L -DIDENT_UIDHASH=0x6fa552eaL BlinkAppC.nc -lm
    compiled BlinkAppC to build/z1/main.exe
            2286 bytes in ROM
54 bytes in RAM
msp430-objcopy --output-target=ihex build/z1/main.exe build/z1/main.ihex
    writing TOS image
```

# Programing Z1 for the first time

We can program the Blink application in our Z1 executing the following commands:

```bash
$ cd $TOSROOT/apps/Blink
$ make z1 install
```

If you've programmed successfully the Z1 you should see the LEDs blinking

# Known Issues

## Permission denied errors

It may not be possible to compile programs with non-privileged users. To solve this issue there are 2 easy solutions:

```bash
$ sudo usermod -a -G tty yourname
```

You have to logout/login to get group addition happens.

# References

1. [Tinyos on recent version of ubuntu](https://askubuntu.com/questions/483916/installing-tinyos-on-recent-version-of-ubuntu/483956#483956)
2. [Errors of packages 'nesc', 'tinyos-tools' and 'msp430-46'](https://github.com/tinyos/tinyos-main/issues/308)
3. [Troubles installing TinyOS on ubuntu karmic](http://mail.millennium.berkeley.edu/pipermail/tinyos-help/2010-April/045890.html)
4. [Cannot open /dev/ttyUSB0: Permission denied](https://github.com/esp8266/source-code-examples/issues/26)
5. [Automatic installation](http://tinyos.stanford.edu/tinyos-wiki/index.php/Automatic_installation)
6. [Installing TinyOS](http://tinyos.stanford.edu/tinyos-wiki/index.php/Installing_TinyOS)