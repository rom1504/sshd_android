# sshd_android
How to access your android phone from anywhere using ssh

## Motivation

Android is linux. So theoretically you can just run everything you can on a linux box.
It's not exactly convenient to type code or commands on a phone, so being able to write it on a computer is quite useful.
It's also useful to be able to transfer files easily between a computer and the phone.

## Requirements

So I decided to try and install a ssh server on my phone. It had to :
* run all the time
* be always available regardless of the network
* be performant
* run as root

## Existing apps
I tried the various available ssh application and they all have some problems : slow, full of ads, old encryption methods, ...

## Solution

### performant ssh server that can be run as root : openssh with termux

I then found https://github.com/termux/termux-app which is really useful to run gpu tools, including sshd.

Initially sshd runs as a basic user. In order to make it run as root, https://github.com/st42/termux-sudo is useful.
Using sshd -d can also provide useful debugging informations, as well as ssh -V.

That provides a good sshd server that supports ssh keys.

### run all the time : init.d

The next step was to make it run all the time. To do that, I tried various methods to setup init.d but it turned out it's different on every phone and it didn't work on mine (nexus 6). So to bypass this issue, I used the [init.d scripts support app](https://play.google.com/store/apps/details?id=com.ryosoftware.initd&hl=en) which simulates the init.d functionnalities by running some scripts (which can be run as root) some time after boot (which is convenient : at that time the phone probably connected to the network).

So I added a script to run sshd at boot time:
```
SYSBIN=/system/bin
SYSXBIN=/system/xbin
BB=$SYSXBIN/busybox
PRE=/data/data/com.termux/files
export LD_LIBRARY_PATH=$PRE/usr/lib
export PATH=$PATH:$SYSXBIN:$SYSBIN

sshd
```

The environment variables are similar to the one present in termux-sudo.

### be available regardless of the network

Phones usually change of network often, which can have firewalls that block the ssh ports, and it also means the public ip of the phone changes often. That is not convenient to access the phone.

To bypass that problem, I decided to connect my phone to my server using a reverse ssh tunnel.

To do that I created a simple script `tun`:
```
autossh -M 35478 -f -R 19995:localhost:8022 -N user@myserver.com
```

Two important parts :
* -R 19995:localhost:8022 : that means the ssh server of the phone will be available on port 19995 of your server
* autossh -M 35478 -f : autossh will keep the ssh connection open by reconnecting whenever needed

That script can then be added in your `init` folder.

The result is you can then connect to your server from anywhere, and then just run `ssh -p 19995 localhost` to connect to your phone.

## Related information

* run crond on your phone by following this [tutorial](http://www.imoseyon.com/2011/02/cron-on-android-is-awesome.html) 
