# Lab - Build a Nextcloud server with Ubuntu 22.04

## [Here's my YouTube playlist ←](https://youtube.com/playlist?list=PLuYfa_nCnOPVwyO_SzmOPdIe8X9rjyvBl)

<br>

## So, what is Nextcloud?

Nextcloud is open-source software, first developed in 2016, that lets you run a personal cloud storage service. It has features that are comparable to other services such as Dropbox or Google Drive.

The Nextcloud server software can be installed free of charge on Linux, and the client software can be installed on computers running Windows, OS X, or Linux. Mobile apps are also available for Android and iOS.

It’s a fork of the OwnCloud project, developed by many of the original members of the OwnCloud team. The two projects have many similarities, but differ in their interface and licensing agreements.

It provides a common file access layer through its Universal File Access. By leveraging existing management, security and governance tools and processes, deployment is made easier and faster. It brings data from cloud storage, Windows network drive and legacy data storage to users in a single, easy interface allowing them to access, sync and share files on any device, wherever they are, managed, secured and controlled. There’s also lots of collaboration tools like online document editing, audio/video chat and more.

<br>

<p align="center">
<img src="https://imgur.com/PbTQ8Ab.png" height="70%" width="70%" alt="nextcloud-diagram"/>
</p>

[Source](https://nextcloud.com/media/wp135098u/Architecture-Whitepaper-WebVersion-072018.pdf)

<br>

## Project overview:

<p align="center">
<img src="https://imgur.com/cLUhfAY.png" height="70%" width="70%" alt="lab-diagram"/>
</p>

<br>

- First, we’ll go ahead and create a Linode account, you can try out this [link](https://linode.gvw92c.net/c/2789508/939241/10906) that will give you 100 USD worth of credits which you can use for up to 3 months.

<br>

**Quick note:** You can use any cloud provider of your preference, we’ll stick to Linode because it’s what I’ve been using lately for hands-on Linux projects without having to worry about your local resources like RAM or CPU.

<br>

## Let’s get to the actual build then …

- We’ll generate our own virtual machine from the **Linodes** tab.

<br>

- Make sure to select **Ubuntu 22.04 LTS** as our image for this lab.

<br>

- Under **Region**, we’ll go with the closest to our actual location, in my case Dallas is my go to option.

<br>

- You can pick a shared CPU plan of course, however, I’d recommend going with **at least 2 GB of RAM** for this project.

<br>

<p align="center">
<img src="https://imgur.com/D6KfH2C.png" height="80%" width="80%" alt="linode"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/LgTtzyd.png" height="80%" width="80%" alt="linode"/>
</p>

<br>

- When you’re done with the settings, hit the **Create Linode** button.

<br>

<p align="center">
<img src="https://imgur.com/9fD2cug.png" height="80%" width="80%" alt="linode"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/VKPID23.png" height="80%" width="80%" alt="linode"/>
</p>

<br>

- Once provisioned, we’ll connect to our instance via SSH.

<br>

<p align="center">
<img src="https://imgur.com/8e42g8N.png" height="80%" width="80%" alt="ssh"/>
</p>

<br>

- Since Linode defaults our main account to root, we’ll need to create one for ourselves.

```
adduser <username>

sudo usermod -aG sudo <username>
```

<br>

<p align="center">
<img src="https://imgur.com/MBHicV2.png" height="80%" width="80%" alt="adduser"/>
</p>

<br>

- After creating your own user, be sure to log out from root and log back in as that user.

<br>

- We’ll ensure all installed packages are up to date.

```
sudo apt update

sudo apt dist-upgrade
```

<br>

- Once done, we’ll update our hostname by editing the following files:

```
sudo nano /etc/hostname

sudo nano /etc/hosts (only if you count with a registered DNS domain)
```

<br>

- Reboot the server so all our changes will take effect.

<br>

- Now, we’ll need to obtain the Nextcloud ZIP file from the [portal](https://nextcloud.com/).

<br>

<p align="center">
<img src="https://imgur.com/RABVTa7.png" height="80%" width="80%" alt="archive"/>
</p>

<br>

- Here’s the actual [link](https://download.nextcloud.com/server/releases/latest.zip) we’ll be using with **wget** from our terminal.

<br>

<p align="center">
<img src="https://imgur.com/p4ejGhp.png" height="85%" width="85%" alt="wget"/>
</p>

<br>

- Next, we’ll go through setting up the database server, so we’ll install **MariaDB**.

```
sudo apt install mariadb-server

systemctl status mariadb
```

<br>

<p align="center">
<img src="https://imgur.com/xWJKF8y.png" height="80%" width="80%" alt="mariadb"/>
</p>

<br>

- We’ll run the **mysql_secure_installation** script to improve the security of our database. Pay attention to the prompts and if uncertain, you can always reference the [documentation](https://mariadb.com/kb/en/mysql_secure_installation/).

```
sudo mysql_secure_installation
```

<br>

- Now, we’ll create the database for Nextcloud, we’ll need to access the **MariaDB** console.

```
sudo mariadb

CREATE DATABASE nextcloud;

SHOW DATABASES;

GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'mypassword';

FLUSH PRIVILEGES;
```

<br>

<p align="center">
<img src="https://imgur.com/xkIBYqH.png" height="90%" width="90%" alt="mariadb"/>
</p>

<br>

- Once we’re finished, we’ll exit by hitting **CTRL+D**.

<br>

- We’ll move on to installing **Apache** now.

<br>

> Apache HTTP Server is a free and open-source web server that delivers web content through the internet. It is commonly referred to as Apache and after development, it quickly became the most popular HTTP client on the web.

[Source](https://www.sumologic.com/blog/apache-web-server-introduction/)

- Let’s install the required packages to support our web server. Keep in mind **Apache** is part of the following dependencies so it’s not listed explicitly on our command.

```
**sudo apt install php php-apcu php-bcmath php-cli php-common php-curl php-gd php-gmp php-imagick php-intl php-mbstring php-mysql php-zip php-xml**
```

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2011.png)

- Once installed, we’ll need to check its status.

```
**systemctl status apache2**
```

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2012.png)

- We’ll enable the recommended **PHP** extensions and modules.

```
**sudo phpenmod bcmath gmp imagick intl

sudo a2enmod dir env headers mime rewrite ssl**
```

- Let’s install **unzip** and decompress our Nextcloud file now.

```
**sudo apt install unzip

unzip latest.zip**
```

- I’ll be renaming the directory just for the sake of consistency prior to relocating it and modifying its permissions.

```
**mv nextcloud/ nextcloud-build

sudo chown -R www-data:www-data nextcloud-build/

sudo mv nextcloud-build/ /var/www**
```

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2013.png)

- We’ll now go ahead and disable **Apache’s** default page.

```
**sudo a2dissite 000-default.conf**
```

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2014.png)

- We’ll setup our own config file where we’ll indicate **Apache** how to serve Nextcloud, please reference the actual content below.

```
**sudo nano /etc/apache2/sites-available/nextcloud-build.conf**
```

```
**<VirtualHost *:80>
    DocumentRoot "/var/www/nextcloud-build"
    ServerName nextcloud-build

    <Directory "/var/www/nextcloud-build/">
        Options MultiViews FollowSymlinks
        AllowOverride All
        Order allow,deny
        Allow from all
   </Directory>

   TransferLog /var/log/apache2/nextcloud-build_access.log
   ErrorLog /var/log/apache2/nextcloud-build_error.log

</VirtualHost>**
```

<aside>
💡 *These filenames need to match yours so be sure to update them as needed.*

</aside>

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2015.png)

- Now, we enable the site.

```
**sudo a2ensite nextcloud-build.conf**
```

- For our next stage we’ll update a couple of **PHP** options, we’ll edit the following file first.

```
**sudo nano /etc/php/8.1/apache2/php.ini**
```

- Let’s adjust these parameters, if you’re using nano, use **CTRL+W** as a search option to save yourself some time.

```
**memory_limit = 512M
upload_max_filesize = 200M
max_execution_time = 360
post_max_size = 200M
date.timezone = America/Mexico_City
opcache.enable=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=2**
```

- When you run into a setting preceded by (;) \*\*\*\*make sure to take it out, this way we’re essentially “un-commenting” it, then update accordingly.

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2016.png)

- We need to restart Apache to apply our latest changes.

```php
**sudo systemctl restart apache2**
```

- Once completed, we’ll reload our browser’s tab with the IP address from our server.

- What we should get now, is Nextcloud’s initial setup site.

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2017.png)

### We are almost there!

- We still need to create an admin account and point to the database user we created with our **GRANT** statement when setting up **MariaDB**.

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2018.png)

- Once filled in, we’ll click on **Install** and let it do its thing, it should take about a minute or so.

- Then, click on **Install recommended apps**.

- After the installation, we should be set and ready to go 🏆.

- We’ll end up with our very own open-source cloud instance with tons of plug-ins and customization options.

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2019.png)

![Untitled](Lab%20-%20Build%20a%20Nextcloud%20server%20with%20Ubuntu%2022%2004%20a80366cb91224093a93138d5874df08d/Untitled%2020.png)

## Next steps:

- Under **Administration settings → Overview**, you’ll find a dedicated section with a list of **\*\***Security & setup warnings**\*\***. It’s important we address as many of these points as we can.

- You can reference this [guide](https://docs.nextcloud.com/server/25/admin_manual/installation/harden_server.html) or my YouTube [video](https://www.youtube.com/watch?v=eARUpvD1G98) where I go through the finishing tweaks to correct **most** of these security concerns.
  **IMPORTANT:** Unless we count with a registered DNS domain, we won’t be able to correct our insecure HTTP connection issue. This is expected as we’d need to acquire a TLS certificate. If you do have one, you can refer to this setup [guide](https://certbot.eff.org/instructions?ws=apache&os=ubuntufocal) by Certbot.

## Further learning:

Just in case you want to dive a little deeper into these topics:

- [https://docs.nextcloud.com/server/25/admin_manual/installation/source_installation.html](https://docs.nextcloud.com/server/25/admin_manual/installation/source_installation.html)
- [https://mariadb.com/kb/en/documentation/](https://mariadb.com/kb/en/documentation/)
- [https://mariadb.com/kb/en/mysql_secure_installation/](https://mariadb.com/kb/en/mysql_secure_installation/)
- [https://httpd.apache.org/docs/current/getting-started.html](https://httpd.apache.org/docs/current/getting-started.html)
- [https://certbot.eff.org/pages/about](https://certbot.eff.org/pages/about)
- [https://certbot.eff.org/instructions?ws=apache&os=ubuntufocal](https://certbot.eff.org/instructions?ws=apache&os=ubuntufocal)
