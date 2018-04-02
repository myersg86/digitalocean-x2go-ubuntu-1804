## Original
---

### Introduction

MediaWiki is an open source wiki platform. In this guide, we will be setting up the latest version of MediaWiki on a Ubuntu 14.04 server.

### Prerequisites

To complete this guide, you should have access to a clean Ubuntu 14.04 server instance with a sudo non-root user.

### Server Components

We will start off by installing all of the components necessary to serve our new wiki. Luckily, we can find all of these within Ubuntu's default repositories.

```command
sudo apt-get update
sudo apt-get install lighttpd mysql-server php5-fpm php5-mysql php5-intl php5-curl php5-gd php5-mcrypt php5-xcache
```

During the installation, you will be asked to select and confirm an password for MySQL. Note that we are not installing MediaWiki in this step.

### MySQL

To complete the initial MySQL setup, we need to initialize the database and improve security. So type the following command.

```command
mysql_install_db
```

Afterwards, we need to lock down a few insecure settings that MySQL has enabled by default. We can do this by walking through a security script.

```command
mysql_secure_installation
```

Press ENTER to accept the suggested changes.

Next, we will create a database and user for our MediaWiki installation to use. Start a MySQL session by logging in with the MySQL `root` user:

```command
mysql -u root -p
```

You will be ask to provide the MySQL administrative password that you set up during installation. Once authenticated, you will be dropped into a MySQL prompt, which looks like this: `mysql>`.

### First, we can create a dedicated database for our MediaWiki installation to use. We will call our database `mediawiki` for clarity, but you can use whatever you'd like.

---

### WRONG: ```custom_prefix(mysql)
CREATE DATABASE mediawiki;
```

Next, we can set up the MySQL user account that will be used to enter and manage our data. In our example, we will call our user `mediawikiuser`. We will use the password `password`, but you should change that to something more secure.

​```mysql
GRANT INDEX, CREATE, SELECT, INSERT, UPDATE, DELETE, DROP, ALTER, LOCK TABLES ON mediawiki.* T 'mediawikiuser'@'localhost' IDENTIFIED BY 'password';
```
---
### CORRECT:
```
CREATE DATABASE my_wiki;
```

You should see the output:

```
Query OK, 1 row affected (0.00 sec)
```

Next, we will create a database user for the MediaWiki installation:

```
GRANT INDEX, CREATE, SELECT, INSERT, UPDATE, DELETE, ALTER, LOCK TABLES ON my_wiki.* TO 'sammy'@'localhost' IDENTIFIED BY 'password';
```
---
Next, we will flush the changes so that they will be available immediately.

```custom_prefix(mysql&gt;)
FLUSH PRIVILEGES;
```

Finally, we can exit out of the MySQL to get back to our regular shell session.

```custom_prefix(mysql&gt;)
exit
```

### PHP-FPM and Lighttpd

The only `lighttpd` configuration we will need to do is related to PHP processing, so it makes sense to take care of both components at the same time.

First, we will edit the `php.ini` file for PHP-FPM. Open the file in your text editor with `sudo` privileges.

```command
sudo nano /etc/php5/fpm/php.ini
```

Find and uncomment the `cgi.fix_pathinfo` line, then restart PHP-FPM in order to implement our changes.

```command
sudo service php5-fpm restart
```

Next, we'll set up our web server so that it can pass PHP requests to PHP-FPM for processing.

Start off by moving into the `lighttpd` available configuration directory where optional configuration snippets are stored.

```command
cd /etc/lighttpd/conf-available
```
Among the available configurations, we can find one called `15-fastcgi-php.conf`. We will use this as a basis for our configuration. Since we will be modifying the file, start off by making a backup so that we can get back to the original if necessary.

```command
sudo cp 15-fastcgi-php.conf 15-fastcgi-php.conf.bak
```

Next, open the file up for editing.

```command
sudo nano 15-fastcgi-php.conf
```

Inside, you will find a configuration that defines how PHP files will be procesed. We will keep the configuration structure itself, but we won't need most of the options within this configuration context. The only lines we *do* need are the "socket" and "broken-scriptfilename" options. Remove the other options you see so that your file looks like this:

```
fastcgi.server += ( ".php" =>
    ((
        "socket" => "/var/run/lighttpd/php.socket",
        "broken-scriptfilename" => "enable"
    ))
)
```

Since we will be passing PHP requests to PHP-FPM, we will have to point `lighttpd` to the correct socket location so that these two components can communicate:

```
fastcgi.server += ( ".php" =>
    ((
        "socket" => "/var/run/php5-fpm.sock",
        "broken-scriptfilename" => "enable"
    ))
)
```

Next, we need to explicitly turn off PHP processing for URLs beginning with "/images/", since that is the one location where user uploaded content will be accessible. Below the existing block, turn PHP processing off for these URLs by adding:

```
fastcgi.server += ( ".php" =>
  ((
    "socket" => "/var/run/php5-fpm.sock",
    "broken-scriptfilename" => "enable"
  ))
)
$HTTP["url"] =~ "^/images/" {
  fastcgi.server = ()
}
```

Save and close the file when you are finished with this change. Enable the PHP processign configuration we were just working on.

```command
sudo lighty-enable-mod fastcgi-php
```

Afterwards, we can reload `lighttpd` to implement our changes.

```command
sudo service lighttpd force-reload
```

### MediaWiki

We will download the latest version of MediaWiki from the [project's website](http://www.mediawiki.org/wiki/Download). Visit the site, right-click on the download link for the latest release, and select "copy link address". Then download and extract the file.

```command
cd ~
wget http://releases.wikimedia.org/mediawiki/1.24/mediawiki-1.24.2.tar.gz
sudo tar xzvf mediawiki* -C /var/www/ --strip-components=1
```

MediaWiki is now ready to be configured. To access the web configuration interface, visit `http://server_domain`.

You should see a page like this which indicates that the wiki has not been configured yet:

![MediaWiki not setup](https://assets.digitalocean.com/articles/mediawiki_lighttpd_homework/mediawiki_not_set_up.png)

Click on the provided link to begin the configuration process.

On the first page, select the configuration and content languages and click "Continue". The next page will check your installation to see that MediaWiki has everything it needs. You should see a line in green indicating that all of the required components are available:

Click "Continue" to move on. On the next page, you will have to add your database credentials. Use localhost for the host, mediawiki for the name, blank for the table prefix, mediawikiuser for the username, and your password for the password.

Fill out the database details so that MediaWiki access the database we created, then click "Continue" to move on. On the next page, you can accept the default values to use the pre-selected account, storage engine, and character set. Click "Continue" to advance.

On the following page, select a name for the wiki. Create an administrator account by selecting a username, choosing and confirming a password, and providing an email address. Select the "Ask me more questions" option and click the "Continue" button again.

On the next page, you'll have the opportunity to configure various policies for your wiki. At the top, you can select whether you want to require accounts and authorization for certain functionality. This also gives you the option to select a license for the wiki's content. Several other settings are available as well.

Some options you might want to select if you plan on allowing images are "Enable file uploads" and "Enable Instant Commons". We also installed PHP XCache, so you will probably want to select that option to enable that feature.

Click "Continue" to skip through the rest of the screens.

### LocalSettings.php

To complete the installation, you will need to upload LocalSettings.php from your local computer to your server. Move into the location where you downloaded the file on your local computer:

```custom_prefix(local$)
cd /path/to/download/directory
```

From here, use the `scp` command to upload the file to your server:

```custom_prefix(local$)
scp LocalSettings.php username@server_domain_or_IP:
```

Now, move the file from your home directory to the document root and give the web server ownership of the file:

```command
sudo mv ~/LocalSettings.png /var/www
sudo chown www-data:www-data /var/www/LocalSettings.png
```

Now, visit your server's domain name or IP address and you should see your installed wiki.

### Conclusion

Your new wiki should now be up and running.
