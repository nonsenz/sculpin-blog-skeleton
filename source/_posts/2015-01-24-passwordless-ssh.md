---
layout: post
title: passwordless ssh
description: how to use keypairs to authenticate against ssh, scp, and more
category: administration 
tags: 
    - security
    - shell

main_img:
    path: passwds.jpg
    copyright:
        name:  jakeliefer
        url: https://www.flickr.com/photos/jakeliefer/
---

### what is better: keys or passwords?

interesting thoughts on that topic can be found [here](http://lwn.net/Articles/369703/). after all i thinks its good to know how to use it.

### 1. step: create keypair

`ssh-keygen` is the tool of choice. `-t` sets the prefered keytype and `-b` the size. some things to consider choosing the type are listed [here](http://security.stackexchange.com/a/23385). so after all

    ssh-keygen -t rsa -b 2048

does the trick. you'll be asked for passphrase (which you'll be asked for from time to time) and that's it.
don't forget to check out the ssh-keygen manpage for all the gory details. 

### 2. copy the public key to your destination host

    ssh-copy-id -i ~/.ssh/id_rsa.pub user@remote-system

### 3. deny connection via password on the destination host (if you want to)

normally you just have to set the following sshd_config options:

	# /etc/ssh/sshd_config
	PasswordAuthentication no
	ChallengeResponseAuthentication no
	UsePAM yes

and don't forget to restart your sshd.
