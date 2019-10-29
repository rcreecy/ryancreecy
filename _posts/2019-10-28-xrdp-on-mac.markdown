---
layout: post
title:  "xRDP on MacOS Mojave"
date:   2019-10-28 23:00:00 -0500
tags: tutorial technology
---
# What is xRDP
xRDP is an awesome utility developed by NeutinoLabs for allowing RDP protocol remote sessions to Linux/Unix operating systems. There's little to no documentation on the proper way to set it up on a MacOS machine, as I don't believe there is much demand for it. However, having a Mac in a remote lab can be extremely useful for many purposes, and this is a way to ditch the extra VNC viewer application while you're at it.
<!--more-->
# Step 1 - Setting up the build environment
Install xCode Command Line Tools (This may already be installed) [v2354]
```
xcode-select --install
```

Install Homebrew [v2.1.15]
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
> You can modify this string as needed, such as I needed to add a proxy for curl using the -x switch (Proxies will also need to be set at ALL_PROXY export and in git config --global)

Install OpenSSL (Brew) [LibreSSL 2.6.5]
```
brew install openssl
```

Once OpenSSL is installed properly, you must export CPPFLAGS to allow the compiler to find it.
```
export CPPFLAGS="-I/usr/local/opt/openssl/include"
```

Install (Brew) Dependencies
```
brew install automake
brew install libtool
brew install pkgconfig
brew install nasm
```

Install xQuartz [v2.7.11]
[https://www.xquartz.org/](https://www.xquartz.org/)
> After install, it will be required to log out and log back in

You may also want to make a folder to keep things organized and to work out of.
```
mkdir /Users/admin/Documents/xrdp/
```
Configure System Preferences for VNC Sharing
    - System Preferences
    - Sharing
    - Screen Sharing: On
    - Computer Settings
    - "Anyone may request permission to control screen": On
    - VNC viewers may control screen with password: {Your account password}

# Step 2 - Download the correct xRDP Packages
You will need both xRDP and xOrgxRDP from http://xrdp.org/ (tar.gz packages)

[XRDP Releases](https://github.com/neutrinolabs/xrdp/releases) - v0.9.11

[XORGXRDP Releases](https://github.com/neutrinolabs/xorgxrdp/releases) - v0.2.11

Once downloaded, pull them into your build directory and extract them.
```
cp /Users/admin/Downloads/xrdp-0.9.11.tar /Users/admin/Documents/xrdp/
tar -xvf xrdp-0.9.11.tar
rm xrdp-0.9.11.tar

cp /Users/admin/Downloads/xorgxrdp-0.2.11.tar /Users/admin/Documents/xrdp/
tar -xvf xorgxrdp-0.2.11.tar
rm xorgxrdp-0.2.11.tar
```
# Step 3 - Building xRDP
Run the xRDP build Bootstrapper and configure the build with OpenSSL. We will then run the make installer.
```
cd xrdp-0.9.11/
./bootstrap
./configure PKG_CONFIG_PATH=/usr/local/opt/openssl/lib/pkgconfig
make
sudo make install
```
> If you get a failure here, it is likely due to not setting the OpenSSL CPPFLAGS export properly.

# Step 4 - Building xOrgxRDP
Run the xOrgxRDP build Bootstrapper and configure the build with OpenSSL. We will then run the make installer again.
```
cd ../xorgxrdp-0.2.11/
./bootstrap
./configure PKG_CONFIG_PATH=/opt/X11/lib/pkgconfig
make
sudo make install
```
> If you get a failure here, make sure you restarted your session after installing your xQuartz X11 server

# Configure xRDP to function properly
You've made it this far. We now had xRDP installed and in a semi-ready state. The next step is to setup the configuration for Mac specific settings for how we will have to be connecting.

Let's open our xrdp.ini file to set everything up correctly!
```
sudo vi /etc/xrdp/xrdp.ini
```

First, find the line with all of your sessions
```
[Xorg]
name=Xorg
lib=libxup.so
username=ask
password=ask
ip=127.0.0.1
port=-1
code=20

[Xvnc]
name=Xvnc
lib=libvnc.so
username=ask
password=ask
ip=127.0.0.1
port=-1
#xserverbpp=24
#delay_ms=2000

[vnc-any]
name=vnc-any
lib=libvnc.so
ip=ask
port=ask5900
username=na
password=ask
#pamusername=asksame
#pampassword=asksame
#pamsessionmng=127.0.0.1
#delay_ms=2000

[neutrinordp-any]
name=neutrinordp-any
lib=libxrdpneutrinordp.so
ip=ask
port=ask3389
username=ask
password=ask
```

We are only worried about vnc-any in this instance, as that is the only one I have been able to get to function properly. We are also going to reference libvnc.dylib, not libvnc.so.
This is the changes sessions section. I just comment out any I won't be using and make the necessary changes for my environment.

```
#[Xorg]
#name=Xorg
#lib=libxup.so
#username=ask
#password=ask
#ip=127.0.0.1
#port=-1
#code=20

#[Xvnc]
#name=Xvnc
#lib=libvnc.so
#username=ask
#password=ask
#ip=127.0.0.1
#port=-1
#xserverbpp=24
#delay_ms=2000

[vnc-any]
name=RDP to VNC Connector
lib=libvnc.dylib
ip=127.0.0.1
port=5900
username=ask
password=ask
xserverbpp=24
#pamusername=asksame
#pampassword=asksame
#pamsessionmng=127.0.0.1
#delay_ms=2000

#[neutrinordp-any]
#name=neutrinordp-any
#lib=libxrdpneutrinordp.so
#ip=ask
#port=ask3389
#username=ask
#password=ask
```
> For vi, press 'i' to enter Insert/Edit mode. To save your changes our, press 'ESC' and enter :wq!

# Try to connect!
Now it is time to start the xRDP and XRDP Session Manager Daemons and attempt a preliminary connection to validate all settings are in place properly.

The daemons can be found and started out of the `/usr/local/sbin/' directory as below.
```
sudo /usr/local/sbin/xrdp
sudo /usr/local/sbin/xrdp-sesman
```

If you can authenticate through the xRDP session to get to the MacOS login screen, congrats, it's working!

Otherwise...

```
/var/logs/xrdp.log
/var/logs/xrdp-sesman.log
```

# Note about encryption
xRDP comes with a standard x509 2048bit RSA key/cert pair.
I highly encourage following their simple documentation to replace them with your own. I simply replace the current ones in `/etc/xrdp/` with my own, after renaming them to add a .bak extension with a self-signed 4096bit certificate. This prevents having to path out in the xrdp.ini (Since it appears to only honor certs and keys in the same directory anyways). The golden rule is, if an application provides you a certificate but allows you to substitute your own - DO IT.

https://github.com/neutrinolabs/xrdp/wiki/TLS-security-layer

# Final words
There is very little current documentation on doing this. The layout/tutorial here is what I found works for me, and I have tried it on a few systems. It is not very scalable, but I think that weighs into why this isn't documented out much anyways. This is something you may have in a small lab, but not a production or enterprise environment. But it is cool, and for the right use-case, fantastic!