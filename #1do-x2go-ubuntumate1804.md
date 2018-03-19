# How To Set Up and Configure a Remote Desktop with X2Go on Ubuntu 18.04

### Introduction

In this guide, you will learn how to install and configure a remotely accessible Linux desktop environment on a Digital Ocean Ubuntu 18.04 Droplet.

Your droplet will have many same utilities and functionality as having an [Ubuntu Mate Desktop](https://ubuntu-mate.org/what-is-ubuntu-mate/) installation on a physical computer, except it will live on a DigitalOcean droplet, making it accessible from anywhere with internet access.

The typical solution to interacting with a GUI( graphical user interface) on a remote Linux desktop is Virtual Network Computing (VNC). VNC connections can be sluggish or unresponsive and often have default settings that create security concerns when made used over the internet. 

For situations in which you'd like to remotely access a full Linux desktop with a GUI interface, X2Go is a great solution..

X2Go works with your existing SSH daemon, encrypting all traffic between the client and the server while relying on the same well-tested and secure mechanism of authentication. It either avoids or optimizes the most latency-intensive parts of X-forwarding safely and without complex manual configuration. The end result is a highly-responsive and near-native desktop experience accessible from anywhere with internet connectivity.

 Such a setup is useful when:

- You need this desktop environment but can't install a Linux-based operating system locally.
- You need some combination of graphical desktop, high-speed Internet, reliable power source, and ability to scale resources up and down quickly.

Remote access clients are available for Windows, Mac, and Linux.

## Prerequisites

Before you begin this guide you'll need the following:

- An Ubuntu 18.04 instance with at least 4GB of RAM. (4GB is great to start with, 8GB+ RAM is optimal).
  Choose a server location that is as close as possible to the location where you intend to connect from to reduce latency.
- One Ubuntu 18.04 server with a sudo non-root user, SSH key, and firewall enabled, which you can set up by following [this Initial Server Setup tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04).

Remember that if you run out of RAM, the application will be terminated by the Linux kernel, and you might lose your work. If you know or notice that the programs you use require a lot of RAM, you can power off your droplet and resize as needed.

## Step 1 — Add the X2Go repository on Launchpad to APT

Introduction to the step. What are we going to do and why are we doing it?

First we will need to add the official X2Go software repository to our list of packages.

Next...

Finally...

Now transition to the next step by telling the reader what's next.

Use the following commands to add the `ppa:x2go/stable` as a package source to your local system. Before doing so, you might like to learn about [apt-get](https://help.ubuntu.com/community/AptGet) and [ppa](https://help.ubuntu.com/community/Repositories/CommandLine#Adding_Launchpad_PPA_Repositories)s

Alternatively you can also use the Ubuntu [software center](https://help.ubuntu.com/community/Repositories/Ubuntu#Adding_PPAs).


To to install `add-apt-repository` on Ubuntu 18.04:

```command
sudo apt-get install software-properties-common
```

Afterwards you can add our ppa:

```command
sudo add-apt-repository ppa:x2go/stable
sudo apt-get update
```

The X2Go related packages should now be listed calling:

```command
apt-cache search x2go
```

You are now able to access the X2Go packages.

## Step 2 — Firewalling the Server

Installing an entire desktop environment pulls in a lot of additional software dependencies and recommendations, some of which may try to open up network ports. For maximum security, the only port that we'll want to leave open will be port 22, so that we're able to connect with SSH and X2Go.

To secure our server, we'll be using Uncomplicated Firewall (UFW), because it's less error-prone to beginner mistakes, easier to understand and manage, and fits better with our goal of only allowing connections to one port. iptables and other more sophisticated firewalls are better suited for advanced and complex rules that require more fine-grained detail. (See [UFW Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands) for a quick reference guide to common commands.)

First, install UFW:

```command
sudo apt-get install ufw
```

By default, the firewall should be inactive at this point. You can check with:

```command
sudo ufw status verbose
```

The result should be:

```
OutputStatus: inactive
```

Verifying the status at this point is important to avoid locking ourselves out if `ufw` is active when we block all incoming connections later.

If UFW is already active, disable it with:

```command
sudo ufw disable
```

Now, set the default firewall rules to deny all incoming connections and allow all outgoing ones:

```command
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

And, allow SSH connections to the server (port 22):

```command
sudo ufw allow 22
```

With the rules in place, let's activate `ufw`:

```command
sudo ufw enable
```

This will output:

```
OutputCommand may disrupt existing ssh connections. Proceed with operation (y|n)?
```

Type `y` and press `ENTER` to activate the firewall.

If you run into a problem and discover that SSH access is blocked, you can follow [How To Use the DigitalOcean Console to Access your Droplet](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-console-to-access-your-droplet) to recover access.

With our firewall in place, there's only one point of entry to our server, and we're ready to install the graphical environment for the X2Go server.

## Step 3 — Installing the Desktop Environment

In this tutorial, you'll install the XFCE desktop environment. There are two ways to achieve this, but you only need to **choose one**—either the Minimal Desktop Environment *or* the Full Desktop Environment.

**Minimal Desktop Environment**: If you want to install a small, core set of packages and then build on top of them by manually adding whatever you need afterward, you can use the `xfce4` *metapackage*.

A metapackage doesn't contain software of its own, it just depends on other packages to be installed, allowing for an entire collection of packages to be installed at once without having to type each package name individually at the command line.

Install `xfce4` and all of the additional dependencies needed to support it:

```
sudo apt-get install xfce4
```

**Full Desktop Environment:** If you don't want to handpick every component you need and would rather have a default set of packages, like a word processor, web browser, email client, and other accessories pre-installed, then you can choose `task-xfce-desktop`.

Install and configure a complete desktop environment that's similar to what you would get with Debian XFCE from a bootable DVD on your local PC:

```
sudo apt-get install task-xfce-desktop
```

Now that our graphical environment is installed and configured, we need to set up a way to view it from another computer.

## Step 3 — Installing X2Go on the Server

X2Go comes with two main components: the server, which starts and manages the graphical session on the remote machine, and the client, which we install on our local computer to view and control the remote desktop or application.

Since Debian does not include the X2Go server in its default repositories, we have to add an extra repository to the package manager's configuration.

First, import the X2Go's developers' public key as a security measure to ensure that we can only download and install packages properly signed with their private keys.

```
sudo apt-key adv --recv-keys --keyserver keys.gnupg.net E1F958385BFE2B6E
```

Now, add the repository to the package manager's config files:

```
echo 'deb http://packages.x2go.org/debian jessie main' | sudo tee /etc/apt/sources.list.d/x2go.list
```

This creates the file `/etc/apt/sources.list.d/x2go.list` and adds the line `deb http://packages.x2go.org/debian jessie main` to it, telling the package manager where to find the supplementary packages.

To refresh the database of available software packages, enter the following command:

```
sudo apt-get update
```

And, finally, install X2Go on the server:

```
sudo apt-get install x2goserver x2goserver-xsession
```

At this point, no further setup is required on your server. However, keep in mind that since SSH password authentication is disabled for increased security, you'll need to have your SSH private key available on any machine that you want to log in from.

We are now done setting up the server and can type `exit` or close the terminal window. The rest of the steps will focus on the client for your local machine.

## Step 3 — Title Case

Another introduction

Your content

Transition to the next step.

## Conclusion

In this article you [configured/set up/built/deployed] [something]. Now you can....
