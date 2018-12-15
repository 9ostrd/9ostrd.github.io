---
layout: post
title: TinyOS 2.1.2 on Ubuntu installation
excerpt: ""
categories: [installation]
comments: true
---

This article has re-created an instruction to run TinyOS on currently Linux OS. I was tested with Ubuntu Desktop 14.04.5 (It's possible to use on Ubuntu 16.XX if all of the packages was supported)(I was tested on lasted Ubuntu version, but still not working) 32 bit on VirtualBox version 5.2.22 r126460 (Qt5.6.3).

I skip an installation of Ubuntu on VirtualBox. You can find the installation on the internet and do it by youself. I'll give only an important informations that uesed in this article.

# VM specifications

Type: Linux 32 bit

Version: Ubuntu 

Set the Memory & size to 1024 MB (1 GB)

Dynamically allocated 10 GB

# Get prerequisites software

We will be installing vim, an editor that is a bit more intuitive than the default vi  that comes with Linux. Open a Terminal window and type the following command  (not need to change directory) from the default one you start in.

```bash
$ sudo apt-get install vim
```

Make sure that the installation completed without any warnings. This is true for  anything else we do here. 
 
A command summary of vim is available here and in many other places:  <http://www.fprintf.net/vimCheatSheet.html>

# Install TinyOS compilers for various uProcessors

We will edit <strong>/etc/apt/sources.list.d/tinyprod-debian.list</strong> and add two  repositories location. These repositories contain the compilers we need. Using vim:

```bash
$ sudo vim /etc/apt/sources.list.d/tinyprod-debian.list
```

go to the end of the file and add the following: 

```bash
deb http://tinyprod.net/repos/debian wheezy main
deb http://tinyprod.net/repos/debian msp430-46 main
```

Save the file and exit back to the Terminal prompt.

Install the repository security key by running the commands: 

```bash
$ gpg  --keyserver keyserver.ubuntu.com  --recv-keys DBCA24B8A9B913B9 
```

Sometimes you need to exit terminal here, so I recommend that you will just do it  and open Terminal again. Now run the command: 

```bash
$ gpg -a  --export DBCA24B8A9B913B9 | sudo apt-key add - 
```

You must see an Exit message: OK. If not you need to investigate why not.
 
Exit the terminal again and start a new one to make the key take affect. 
 
Now we can install the packages. In Terminal we run: 

```bash
$ sudo apt-get update
```

In this state, If you found an Errors of gpg-keys. You need to set a gpg-keys with a new keys that system report as follows:

```bash
$ gpg  --keyserver keyserver.ubuntu.com  --recv-keys <new key>
$ gpg -a  --export <new key> | sudo apt-key add - 
```

then,

```bash
$ sudo apt-get install msp430-46 nesc tinyos-tools
```

# Get TinyOS

Working in your home directory, run in Terminal: 

```bash
$ wget http://github.com/tinyos/tinyos6release/archive/tinyos62_1_2.tar.gz
```

unzip the file and move it to a more suitable location (here we just rename it  because we want to be able to have all read/write to the source code): 

```bash
$ tar xf tinyos-2_1_2.tar.gz
$ mv tinyos-release-tinyos-2_1_2 tinyos-main
```

# Setup TinyOS dev. environment

You will need to add some environment variables to your shell. The following are  the necessary ones. Using <strong>vim</strong> edit <strong>.bashrc</strong> and add the following lines at the end of  the file:

```bash
export TOSROOT="$HOME/tinyos-main"
export TOSDIR="$TOSROOT/tos"
export CLASSPATH=$CLASSPATH:$TOSROOT/support/sdk/java
export MAKERULES="$TOSROOT/support/make/Makerules"
export PYTHONPATH=$PYTHONPATH:$TOSROOT/support/sdk/python
```

You have to logout/login

# Test the installation

Start by testing the environment by running: 
 
```bash
$ tos-check-env 
```

If you did everything properly you will see only one WARNING about graphviz not  being version 1.10 (it will appear twice). This is actually OK because you will have a  newer version (likely 2.x). To check run: 
 
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

If you've programmed successfully the Z1 you should see the leds blinking

# Known Issues

## Permission denied errors

It may not be possible to compile programs with non privileged users. To solve this issue there are 2 easy solutions:

```bash
$ sudo usermod -a -G tty yourname
```

You have to logout/login to get group addition happens.

# References

1. [Tinyos on recent version of ubuntu](https://askubuntu.com/questions/483916/installing-tinyos-on-recent-version-of-ubuntu/483956#483956)
2. [Errors of packages 'nesc', 'tinyos-tools' and 'msp430-46'](https://github.com/tinyos/tinyos-main/issues/308)
3. [Troubles installing TinyOs on ubuntu karmic](http://mail.millennium.berkeley.edu/pipermail/tinyos-help/2010-April/045890.html)
4. [Cannot open /dev/ttyUSB0: Permission denied](https://github.com/esp8266/source-code-examples/issues/26)
5. [Automatic installation](http://tinyos.stanford.edu/tinyos-wiki/index.php/Automatic_installation)
6. [Installing TinyOS](http://tinyos.stanford.edu/tinyos-wiki/index.php/Installing_TinyOS)
