# How To Set Up a Remote Desktop with X2Go on Ubuntu 17.10

### Introduction

In this guide, you will learn how to install and configure an remotely accessible desktop enviornment on a Digital Ocean Ubuntu 17.10 Droplet. 
Your droplet will have the same utilities and functionality  as having an Ubuntu Mate Desktop Installation on a physical computer, except it lives on DigitalOcean servers and is accessible from anywhere that you have internet access. 
Remote access client are available for Windows, Mac, and Linux.

For a variety of reasons, most servers don't use or have any need for a graphical user interface.
In some use cases however it can be helpful or desirable to have a lightweight desktop environment accessible from anywhere you have an internet connection.

The typical solution to interacting with a GUI( graphical user interface) on a remote Linux desktop is Virtual Network Computing (VNC). Unfortunately VNC connections can be sluggish or unresponsive and most VNC server packages available require significant tweaking to run them securely on the internet. X2Go was created as a solution to this problem. 
X2Go works with your existing SSH daemon, encrypting all traffic between the client and the server while relying on the same well-tested and secure mechanism of authentication.
X2Go either avoids or optimizes the most latency-intensive parts of X-forwarding without complex manual configuration, it and sends only compressed updates back and forth.
Simply put, X2Go compresses and optimizes the stuff the updates sent back and forth to a point where it seems just as responsive as if you had your monitor plugged directly into your droplet server. 

 

Such a setup is useful when:

- You need this desktop environment but can't install a Linux-based operating system locally.
- You need some combination of graphical desktop, high-speed Internet, reliable power source, and ability to scale resources up and down quickly.


## Prerequisites

Before you begin this guide you'll need the following:

- An Ubuntu 16.04 instance with at least 2GB of RAM. (2GB is minimal, 4GB is better to start with, and 8GB+ would be optimal). Choose a server location that is as close as possible to the area where you intend to connect from.
-   A CentOS 7 droplet with SSH access. For more information, visit [this tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)
-   A LAMP stack, which you can install by following [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7)
- https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04

Remember that if you run out of RAM, the application will be terminated by the Linux kernel, and you might lose your work. If you know or notice that the programs you use require a lot of RAM, you can power off your droplet and resize as needed.

## Step 1 — Install Let’s Encrypt SSL Certificate

## Add the X2Go repository on Launchpad to APT

Use the following commands to add the `ppa:x2go/stable` as a package source to your local system. Before doing so, you might like to learn about [apt-get](https://help.ubuntu.com/community/AptGet), [sudo](https://help.ubuntu.com/community/RootSudo) and [ppa](https://help.ubuntu.com/community/Repositories/CommandLine#Adding_Launchpad_PPA_Repositories)s

Alternatively you can also use the Ubuntu [software center](https://help.ubuntu.com/community/Repositories/Ubuntu#Adding_PPAs).


To to install `add-apt-repository` on Ubuntu 16.04:

```
sudo apt-get install software-properties-common
```

Afterwards you can add our ppa:

```
sudo add-apt-repository ppa:x2go/stable
sudo apt-get update
```

The X2Go related packages should now be listed calling:

```
apt-cache search x2go
```

Congratulations, you are now able to access the X2Go packages. You may continue by installing x2goserver, x2goclient or pyhoca-gui or any other of the available packages.

Introduction to the step. What are we going to do and why are we doing it?

First....

Next...

Finally...

Now transition to the next step by telling the reader what's next.

## Step 2 — Title Case

Another introduction

Your content

Transition to the next step.

## Step 3 — Title Case

Another introduction

Your content

Transition to the next step.

## Conclusion

In this article you [configured/set up/built/deployed] [something]. Now you can....
