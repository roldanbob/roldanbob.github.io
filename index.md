---
layout: default
---

# ownCloud Quickstart Guide
{:.no_toc}

* TOC
{:toc max_level=3 }

## Introduction
ownCloud is a secure, open source, private cloud file storage system that automatically synchs files across multiple clients.
Users can access files stored on ownCloud from remote desktop or mobile devices.

## About this guide
This Quickstart guide has two parts. The first part guides administrators through the process of deploying a locally hosted, single-machine ownCloud Community Edition server. The second part provides guidance for users who want to install client applications that connect to the ownCloud server.

## Installing and configuring ownCloud Community Edition

You can install the ownCloud server on a range of Linux operating systems, using several different methods. These instructions focus on using the Ubuntu `apt`
command-line tool to install the ownCloud software package.


_______
**Note**
These instructions are intended to help you to quickly
install a test deployment of ownCloud on a local computer.
The resulting server is not secured for use in a production
setting. For detailed information about how to install ownCloud on the full range of supported server and client platforms, see the [ownCloud Server Administration Manual](https://doc.owncloud.org/server/10.0/admin_manual/installation/index.html).

_______


## Before you begin
You should be familiar with using the Linux command-line to install software. Administrators who install ownCloud must have sudo access to the command-line.

## System requirements
To verify that your computer meets the minimum system requirements for installing
the server, see [System requirements](https://doc.owncloud.org/server/10.0/admin_manual/installation/system_requirements.html).


----
**Note**
You can install ownCloud on a Linux guest OS that hosted is in a Windows virtual machine.

----

### Installing dependencies

The host operating system must have a LAMP stack installed (Linux OS, Apache HTTP server, mySQL relational database, and PHP programming language). On Ubuntu, you can run the following commands to install the prerequisite stack:

 ````
 sudo apt-get update
 sudo apt-get install lamp-server^
   ````
While this is a quick way to install most of the deployment you need, the default LAMP stack is not  pre-installed with all of the PHP modules you need. On Ubuntu, run the following commands to get the PHP modules that you need:

   ```
   sudo add-apt-repository ppa:ondrej/php
   sudo apt-get update

   sudo apt-get install -y libapache2-mod-php7.2 \
    openssl php-imagick php7.2-common php7.2-curl php7.2-gd \
    php7.2-imap php7.2-intl php7.2-json php7.2-ldap php7.2-mbstring \
    php7.2-mysql php7.2-pgsql php-smbclient php-ssh2 \
    php7.2-sqlite3 php7.2-xml php7.2-zip
   ```
For more information, and for instructions about  installing dependencies on other Linux distributions, see [Install the required packages](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#install-the-required-packages) in the Administration Manual.


### Testing the HTTP server

Later, you'll be modifying the default HTTP configuration, so now that you have the LAMP server installed, it's a good idea to test that the HTTP server is working is working under its default configuration.

1. On Ubuntu, you can start the Apache2 server by running the following command:

  ```
  sudo systemctl start apache2.service
  ```
1. Confirm the service status by typing the following command:

    ```
    systemctl status apache2
    ```

### Testing browser access
After you confirm that the service is running use a browser to test whether you can access the default web page. First, you need to determine the IP address of the server.

1. From a command line, type `ifconfig` to obtain the IP address of the computer where you installed ownCloud. If `net-tools` is not installed, you're prompted to run `sudo apt install net-tools` to install the package.

1. Type the IP address that `ifconfig` returns in the address bar of a browser. The default welcome page for the HTTP server displays in the browser.

### Create the database
Create a database user and the database itself by using the MySQL command line interface. After you install ownCloud, the database tables are created automatically
when you first time log in as administrator.

1. Open a command shell and run the following command:
```
       sudo mysql -u root -p
```
1. When prompted, type your sudo password.

1. From the `mysql>` prompt, type the following command to Create the database:
```
       CREATE DATABASE owncloud;
```
1. Type the following command to create the database user:
```
       CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'PASSWORD';
```
  Substitute the username and password that you want to ownCloud to use to access the database. Be sure to save this name and password for later reference.

1. Type the following command to  assign user permissions:
```
       GRANT ALL ON owncloud.* TO 'ownclouduser'@'localhost' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
```

1. Flush the privileges and then exit from the MySQL command shell:
```
       FLUSH PRIVILEGES;
       EXIT;
```

Flushing privileges reloads the grant tables in the database so that you don't have to restart the mysql service for the permission changes to take effect.

<!-- See https://www.techrepublic.com/article/how-to-install-owncloud-on-ubuntu-18-04/
You can find a number of excellent video tutorials that run through the steps
for installing prerequisite software. -->

_______
Note:
For our purposes, we won't secure the database, but you would do that in any production environment.

## installing the ownCloud server package

You can install ownCloud from the package manager of your Linux distribution, or
you can install the software manually, using the command line. The following instructions guide you through the process for using the command line to complete the installation.

1. Open a browser to the [ownCloud Downloads page](http://owncloud.org/download), scroll to **Linux distribution packages**, and then click **Go to packages**.

1. From the **Install package owncloud-files** page, click the name of the operating system on which you're installing ownCloud. The installation instructions for your operating system are displayed.

1. From a terminal, run the commands to download the release key.

    ```
    wget -nv https://download.owncloud.org/download/repositories/production/Ubuntu_18.04/Release.key -O Release.key

    sudo apt-key add - < Release.key
    ```

1. From a terminal window, open a root shell:

   ```
   sudo su
   ```
   Type your root password when prompted and then type the following commands to add the ownCloud repository and install the ownCloud server files:

   ```
   echo 'deb http://download.owncloud.org/download/repositories/production/Ubuntu_18.04/ /' > /etc/apt/sources.list.d/owncloud.list

   sudo apt-get update
   sudo apt-get install owncloud-files
   ```


## Post-installation

You're now ready to configure the server. Your Apache2 HTTP server runs under its default configuration. After you verify that everything works, you can modify the default port and set ownCloud to open from the root of the web server root URL.

1. Use a text editor such as nano or vi to create a file named `owncloud.conf` in `/etc/apache2/sites-available/`. Use `sudo` permissions when you create the new file. Add the following content to the file to set the Alias directive for your ownCloud directory to become the default ownCloud web server root:

    ````
   Alias / "/var/www/owncloud/"

   <Directory /var/www/owncloud/>
     Options +FollowSymlinks
     AllowOverride All

    <IfModule mod_dav.c>
     Dav off
    </IfModule>

    SetEnv HOME /var/www/owncloud
    SetEnv HTTP_HOME /var/www/owncloud

   </Directory>
    ````

2. Create a symbolic link to enable the ownCloud  site configuration. From a terminal, type the following command to create a symbolic link to `/etc/apache2/sites-enabled`:

    ````
    sudo ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf
    ````
### Enabling the site and the necessary modules
Run the `a2ensite` and `a2enmod` commands to execute scripts that enable the ownCloud site configuration and specified modules in the apache2 configuration. The scripts create symbolic links within `/etc/apache2/sites-enabled` and `/etc/apache2/mods-enabled`.

1. From a terminal, run the following commands:

   ```
   sudo a2ensite owncloud.conf
   sudo a2enmod rewrite
   sudo a2enmod headers
   sudo a2enmod env
   sudo a2enmod dir
   sudo a2enmod mime
   ```
  It's possible that some of the modules are already enabled, but there is no harm in re-enabling them.

1. Run the following command to restart the Apache2 HTTP server and refresh the configuration:

   ```
   sudo systemctl restart apache2
   ```
  You should now be able to bring up the ownCloud sign-in page in a browser by specifying the server's IP address.

  !["Sample ownCloud log-in page"](/images/oc_login_dflt_port.PNG)

## Changing the default address of the ownCloud server

Now you're ready to configure a custom port for serving up your ownCloud server. We'll set the port to 8080. After you make the change, the server accepts connections only from browsers that specify the new port in the URL.

ownCloud is served by your web server so you need to configure the port that you want to use in the configuration of your web server.

See [Changing your ownCloud URL](https://doc.owncloud.org/server/10.0/admin_manual/installation/changing_the_web_route.html).



## Configuring the HTTP server to listen on port 8080

https://stackoverflow.com/questions/3940909/configure-apache-to-listen-on-port-other-than-80[Configure the Apache to listen on port other than 80]


### Verify that the HTTP server runs

   <!--
1. After the package manager finishes adding the ownCloud files to your computer, run the Installation
Wizard to complete the installation.
-->

1. Type the following commands to start the Apache2 server and verify its status:

  ````
    service apache2 restart

    systemctl status apache2
 ````
1. From a command line, type `ifconfig` to obtain the IP address of the computer where you installed ownCloud. If net-tools is not installed, you're prompted to run `sudo apt install net-tools`


1. Type the IP address in the address bar of a browser. The default welcome page for the HTTP server displays in the browser.

<!-- ### Create the database  <- This actually happens pre-install
But I had to re-do setting permissions for my database user before I could complete the web UI step of creating my db admin -->

### Sign in to the ownCloud server as administrator
1. From a browser, open the ownCloud Web UI type the URL for the ownCloud server: `http://<ip_address>:8080`

   For example:

    http://192.168.1.123:8080

2. Create an administrator account by typing a name and password in the **Create an admin account** field.

3. Click *Finish setup*.


## Adding user accounts
After you log in as administrator, you can add users to the ownCloud local user registry.

For more information, see [Creating a New User](https://doc.owncloud.org/server/10.0/admin_manual/configuration/user/user_configuration.html#creating-a-new-user) in the ownCloud
Server Adminstration Manual.

From the *User management* page of the ownCloud Web UI, click

![Screenshot showing how to open the User management page from the ownCloud Web UI](/images/admin_menu.png)

Optionally, you can add a user to a group.

## Setting up your ownCloud synchronization client
To provide automatic synchronization of files across multiple device, you can install ownCloud Synchronization Clients. Desktop clients are available for Linux, macOS, and Microsoft Windows. Mobile clients are available for the Android and
Apple iOS operating systems.

****
**Note**
Desktop devices can also access ownCloud from a browser.
****
You can download desktop clients from the [ownCloud download page](https://owncloud.com/download/#desktop-clients).

1. Open the [ownCloud Client download page](https://owncloud.org/download/#install-clients),click the operating system for which you want to install a client, and then follow
the instructions to add the repository and complete the installation.

1. Run the installation wizard to complete the ownCloud client setup.

 On Windows 10, click **Yes** to allow the program to make changes to your computer. Enter your administrator password if prompted.

You'll need to restart your computer to enable Windows Explorer integration, but if you want to check out the client right away, it's ok to wait until later to restart. Just realize that you won't be able to use ownCloud with Explorer just now.
If you decide not to restart now, select the checkbox on the final screen of the Setup wizard to open the ownCloud client now.

When prompted, either click *Yes* to restart the computer now and enable Explorer integration, or click *No* to restart later.

## Connecting to the server

After the client starts, you're ready to configure a connection to the server.

1. If you haven't started it, go ahead and start the server, , and then type the address and port of the ownCloud server, using
the format `http://xxx.xxx.xxx.xxx:<port_number>`, and click **Next**. By default, the client
automatically tries to connect using HTTPS. If you installed the server according to the instructions in this Quick Start guide, you did not secure the HTTP connection with a certificate. So, be sure to connect with plain HTTP, or the connection attempt fails.

1. Type the credentials of the user that you added from the Admin UI and then click
**Next**.

1. Set up the local folder options to specify how you want to use synchronization,
and then click **Connect...**.

1. Click **Add Folder Sync Connection** to synchronize a new folder, and then
 specify that path to the folder and click **Next**.

 1. Choose the folder on the ownDrive server where you want to save the synchronized
 folder content and click **Next**.

 1. Deselect any folders on the server that you don't want the client to synchronize,
 and then click **Add Sync Connection**.

## Mobile clients  
Mobile clients are available for purchase on the Google Play Store and on the Apple App Store.
