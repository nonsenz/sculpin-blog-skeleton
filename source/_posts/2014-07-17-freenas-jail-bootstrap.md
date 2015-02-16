---
title: create nice freenas base jail
description: default first steps to create a nice freebsd jail with freenas 9.2.1
category: administration
tags: 
    - freenas
    - freebsd 
    - jails
main_img:
    path: freenas_base.jpg
    copyright:
        name:  Lolla Moon
        url: https://www.flickr.com/photos/cherrysoda/

---

## whats the problem?

since freenas 9.2.1 does not support native jail cloning ([looks like](https://bugs.freenas.org/issues/5074) 9.3 will) and it is [not recommended](http://forums.freenas.org/index.php?threads/what-is-the-recommended-way-to-clone-a-jail.21156/#post-122791) to do it via warden in the background one has to manually setup the jail with the standard tools before doing anything else. here are the default things to do:

## setting it up

### create the jail

like always create a new jail (portjail), edit settings if you like, name it and press ok.

### enabling ssh

because we want to work in the jail over ssh, we have to enable it. click on the created jail in the webinterface and click the shell symbol on the bottom to open a shell in the browser. 

    > vi /etv/rc.conf

change `sshd_enable="NO"` to `"YES"` and savexit.

    > service sshd start

now you can login via ssh to the jails ip. 

### adding your user

because you want your personal user:

    > adduser

you can add the new user to the wheel group to give him root privileges. you'll be asked if you wan the new user to be invited to other groups. simply type in "wheel".

### enabling ports

the best way to install tools on your machine is via ports with some helpertools. because freenas 9 uses freebsd 9 we have to modify the ports config to be able to use the newer package format. besides this you add other things to your `/etc/make.conf` to optimize your build process:

    > vi /etc/make.conf

here is a small example:


    #Use clang instead of gcc, only needed for versions before 10.0
    CC=clang
    CXX=clang++
    CPP=clang-cpp
    #Compile everything in a ram disk and use ccache, see our tutorial
    WRKDIRPREFIX=/ram
    WITH_CCACHE_BUILD=yes
    #The newer pkgng format, only needed for versions before 10.0
    WITH_PKGNG=yes

now we can fetch all the makefiles we need later to install software via ports:

    > portsnap fetch extract

this may take a while. after that you could start installing software. but i recommend two other tools: [portmaster](https://github.com/freebsd/portmaster) and psearch.
portmaster allows easy installation and removal of multiple ports.

    > cd /usr/ports/ports-mgmt/portmaster
    > make install clean

now you can install ports like so:

    > portmaster ports-mgmt/psearch

now to psearch. it allows an easy way to search for ports. you also could search through the directories below `/usr/ports` but i think psearch makes it easier. lets install it via portmaster.

    > portmaster ports-mgmt/psearch
    
if you want to update your ports-data later use portsnap again:

    > portsnap fetch update
    
now you can check `/usr/ports/UPDATING` for special update info for some ports. if nothing special is noted here you simply can update all ports with

    > portmaster -ad

lots of the info here comes from [bsdnow](http://www.bsdnow.tv/tutorials/ports). check it out for more info about updating and more.

### tools

now for some standard tools. install it all via portmaster:

    > portmaster shells/zsh editors/vim ftp/wget
    
as all the software and its dependencies will be compiled on your machine, this can take while...
now set up zsh as your default shell with

    > chsh -s zsh USERNAME
    
now you have a nice base jail to start working with. have fun.

