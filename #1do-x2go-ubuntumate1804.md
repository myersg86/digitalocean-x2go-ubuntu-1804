# How To Set Up and Configure a Remote Desktop with X2Go on Ubuntu 16.04

### Introduction

In this guide, you will learn how to install and configure a remotely accessible Linux desktop environment on a Digital Ocean Ubuntu 16.04 Droplet.

Your droplet will have many same utilities and functionality as having an [Ubuntu Mate Desktop](https://ubuntu-mate.org/what-is-ubuntu-mate/) installation on a physical computer, except it will live on a DigitalOcean droplet, making it accessible from anywhere with internet access.

The typical solution to interacting with a GUI( graphical user interface) on a remote Linux desktop is Virtual Network Computing (VNC). VNC connections can be sluggish or unresponsive and often have default settings that are not secure if used outside of local networks. In situations where you need to remotely connect to and access a Linux desktop securely and with minimal latency, X2Go is an excellent solution.
X2Go works with your existing SSH daemon to encrypt all traffic between the client and the server, relying on well-tested and secure mechanism of authentication. It either avoids or optimizes the most latency-intensive portions of X-forwarding safely and without complex manual configuration. The end result is a highly-responsive and near-native desktop experience accessible from anywhere with internet connectivity.

Such a setup is useful when:

- You need this desktop environment but can't install a Linux-based operating system locally.
- You need some combination of graphical desktop, high-speed Internet, reliable power source, and ability to scale resources up and down quickly.

You can connect from a computer running Linux, Windows or Mac OS X by using the free and open-source X2Go Client.

## Prerequisites

Before you begin this guide you'll need the following:

- An Ubuntu 16.04 instance with at least 4GB of RAM. (4GB is great to start with, 8GB+ RAM is optimal).
  Choose a server location that is as close as possible to the location where you intend to connect from to reduce latency.
- One Ubuntu 16.04 server with a sudo non-root user, SSH key, and firewall enabled, which you can set up by following [this Initial Server Setup tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04).

Recall that if you run out of RAM, applications will be terminated by the Linux kernel, and you might lose your work. If you notice that the programs you use require a lot of RAM, you can power off your droplet and resize as needed.

## Step 1 — Add the X2Go repository on Launchpad to APT

Introduction to the step. **What are we going to do and why are we doing it?** [apt-get](https://help.ubuntu.com/community/AptGet)  [ppa](https://help.ubuntu.com/community/Repositories/CommandLine#Adding_Launchpad_PPA_Repositories)s

First we will need to add the official X2Go software repository to our list of packages. 

To do this, we will first to install`add-apt-repository`, a  is a command line utility for adding Personal Package Arcives (PPAs).
To to install `add-apt-repository` on Ubuntu 16.04:

```command
sudo apt-get install software-properties-common
```

**Next...**

Next, use the following commands to add the `ppa:x2go/stable` as a package source to your local system.

```command
sudo add-apt-repository ppa:x2go/stable
sudo apt-get update
```

**Finally...**

The X2Go related packages should now be listed calling:

```command
apt-cache search x2go
```

You are now able to access the X2Go packages.

**Now transition to the next step by telling the reader what's next.**


## Step 2 — Firewalling the Server

Installing a desktop environment can pull-in a lot of additional software dependencies and recommendations, some of which may try to open up network ports. For maximum security, the only port that we'll want to leave open will be port 22, so that we're able to connect with SSH and X2Go.

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

In this tutorial, you'll install the lightweight [MATE Desktop Environment](http://mate-desktop.org/). There are two ways to achieve this, but you only need to **choose one**—either the Minimal Desktop Environment *or* the Full Desktop Environment.

**Add Required Repositories for Desktop Environment**

```
sudo apt-add-repository ppa:ubuntu-mate-dev/ppa
```

```
sudo apt-add-repository ppa:ubuntu-mate-dev/xenial-mate
sudo apt update
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install --no-install-recommends ubuntu-mate-core ubuntu-mate-desktop
sudo apt full-upgrade
```

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

```command
apt-get update
apt-get install python-software-properties
add-apt-repository ppa:x2go/stable
sudo apt-get update
sudo apt-get install x2goserver x2goserver-xsession
```

## Step 4 — Installing the X2Go Client Locally

If you're using Windows on your local machine, you can download the client software from <http://code.x2go.org/releases/X2GoClient_latest_mswin32-setup.exe>.

After you select your preferred language and agree to the license, a wizard will guide you through each step of the installation process. Normally, there shouldn't be any reason to change any of the default values pre-filled or pre-selected in these steps.

If you're running macOS locally, you can find the client software at <http://code.x2go.org/releases/X2GoClient_latest_macosx_10_9.dmg>.

Double-click the .dmg file to open a folder containing the copyright, license, and X2Go client executable, then double-click the executable to start the client.

And, if you're using Debian or Ubuntu you can install the X2Go client with:

```
sudo apt-get install x2goclient
```

If you'd like additional information about the clients or you'd like to build from the source, you can visit [X2Go's official documentation](http://wiki.x2go.org/doku.php/download:start).

Now that the desktop client is installed, we can configure its settings and connect to the X2Go server to use our remote XFCE desktop.

## Step 5 — Connecting to the Remote Desktop

When you first open the X2Go client, the following window should appear. If it doesn't, click **Session** in the top-left menu and then select **New session ...**.

![X2Go Client Screenshot - Creating a New Session](http://assets.digitalocean.com/articles/how-to-setup-a-remote-desktop-with-x2go-on-debian-8/create-new-session.png)

In the **Session name** field, enter something to help differentiate between servers. This can be particularly useful if you plan on connecting to multiple machines, since all of the names will be listed in the program's main window once you save your settings.

Enter your server's IP address or hostname in the **Host** field under **Server**.

Enter the username you used for your SSH connection in the **Login** field.

And, since it's what we installed in Step Two, choose `XFCE` as your **Session type**.

Finally, because we log into the server with SSH keys, click the folder icon next to **Use RSA/DSA key for ssh connection** and browse to your private key.

The rest of the default settings should suffice for now, but as you get more familiar with the software, you can fine tune the client based on your individual preferences.

After pressing the **OK** button, you can start your graphical session by clicking the white box that includes your session name on the top-right side of the screen.

![X2Go Main Window - Session List](http://assets.digitalocean.com/articles/how-to-setup-a-remote-desktop-with-x2go-on-debian-8/main-window.png)

In a few seconds, your remote desktop will be displayed, and you can start interacting with it. At first login, XFCE will ask if you want to **Use default config** or **One empty panel**. The first option will create a rectangular panel docked at the bottom of the screen, containing a few useful application shortcuts (e.g. a file manager, a terminal emulator, a browser, etc.). This option will also add a top panel to the desktop that includes utilities like an application launcher, a clock, a shutdown menu, and more.

Unless you're already familiar with XFCE, opting for an empty panel can be more complicated since you'll be starting from scratch. There will be no taskbar, no clock, no pre-configured start menu; it will be up to you to add everything to an empty panel on your own.

Additionally, on Windows and Linux-based operating systems, there are a few useful keyboard shortcuts you can use for a better experience:

`CTRL+ALT+F` will toggle full-screen mode on and off. Working in full-screen mode can feel more like a local desktop experience. Plus, other keyboard shortcuts will be grabbed by the remote OS instead of the local one.

`CTRL+ALT+M` will minimize the remote view, even if you are in full-screen mode

`CTRL+ALT+T` will disconnect from the session but leave the GUI running on the server. It's just a quick way of disconnecting without logging off or closing applications on the server. The same will happen if you click the window's close button.

Lastly, there are two ways you can end the remote session, closing all of the graphical programs running in it. You can log off remotely from XFCE's **start menu**, or you can click the button marked with a circle and a small line (like a power/standby icon) in the bottom right corner of the main portion of the screen.

The first method is cleaner but may leave programs like session managing software running. The second method will close everything but may do so forcefully if a process can't cleanly exit. In either case, be sure to save your work before proceeding.

![X2Go Main Window - Terminate Session Button](http://assets.digitalocean.com/articles/how-to-setup-a-remote-desktop-with-x2go-on-debian-8/terminate-session.png)

On a final note, although it's not required, let's go into XFCE's control panel, navigate to power settings and disable all standby features. If you don't use the desktop for a while, XFCE will try to trigger a standby. Although the server normally won't do anything when it receives this request, it's better to avoid any unforeseen behaviors altogether.

## Conclusion

You now have a working "cloud desktop," complete with all the advantages of an always-online, remotely-accessible, computing system.

To go a step further, you could centralize your development work by [creating a git repository](https://www.digitalocean.com/community/tutorials/how-to-create-a-pull-request-on-github), installing a remote code editor like Eclipse, or [configuring a web server](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-debian) for testing web applications. You could also couple your remote desktop with [a good backup scheme](https://www.digitalocean.com/community/tutorials/how-to-choose-an-effective-backup-strategy-for-your-vps) to make sure that your work environment is available from anywhere and that it's safe from data loss.

If you'd like to learn more, visit [X2Go's official documentation website](https://wiki.x2go.org/doku.php).

In this article you configured/set up/built/deployed something. Now you can....
