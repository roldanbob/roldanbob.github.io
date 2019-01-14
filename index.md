---
layout: default
---

# ownCloud Quickstart Guide
{:.no_toc}

* TOC
{:toc max_level=3 }

## Introduction
ownCloud is a secure, open source, private cloud for storing files. You can access and synchronize your ownCloud files from a desktop computer, a mobile phone, or a tablet.


## Audience
The following sections include information for ownCloud administrators and client users.

* _Installing and configuring ownCloud Community Edition_ is for administrators who wants to deploy an ownCloud server for testing.

* _Setting up your ownCloud synchronization client_ is for users who want to install and set up desktop or mobile clients that connect to an ownCloud server.

## Installing and configuring ownCloud Community Edition

The ownCloud server runs on any Linux&reg; operating system. Several installation methods are available. These instructions focus on using the `apt` command-line tool to install the ownCloud software package on Ubuntu.


_______
**Note**

After you complete the steps in the Quickstart guide, the ownCloud Community Edition server that is installed will be suitable as a test system for exploring the features of the product.
If you want to install a secure, production instance of ownCloud, see the [ownCloud Server Administration Manual](https://doc.owncloud.org/server/10.0/admin_manual/installation/index.html).

_______


### Before you begin
 You need sudo access to the command line to complete the installation steps.

 Familiarity with the Ubuntu command line isn't strictly required to complete these installation steps, but it can't hurt.

### System requirements
Before you install ownCloud, make sure that your computer meets the minimum system requirements. For more information, see [System requirements](https://doc.owncloud.org/server/10.0/admin_manual/installation/system_requirements.html) in the ownCloud Server Administration Manual.


____
**Note**

Run Windows&reg;, but want to install ownCloud? Don't despair. You can install ownCloud on a Linux guest OS that is hosted within a virtual machine such as VirtualBox.

____

### Installing dependencies

ownCloud depends on packages that are part of the standard LAMP stack (Linux OS, Apache HTTP server, mySQL relational database, and PHP programming language). Ubuntu meets some of the LAMP dependencies by default, but not all of them. So it's best to install the packages you need now. On Ubuntu, there's an easy way to get all of the packages that you need in one go (well, almost. Hold that thought for a moment).

To install a LAMP stack, run the following commands:

 ````
 sudo apt-get update
 sudo apt-get install lamp-server^
   ````
Ok, full disclosure. It was a quick way to install an entire LAMP server, but something's still missing. Unfortunately, the default LAMP stack doesn't have the full set of PHP modules that ownCloud needs. But no worries, on Ubuntu, you can run the following commands to get any missing PHP modules:

   ```
   sudo add-apt-repository ppa:ondrej/php
   sudo apt-get update

   sudo apt-get install -y libapache2-mod-php7.2 \
    openssl php-imagick php7.2-common php7.2-curl php7.2-gd \
    php7.2-imap php7.2-intl php7.2-json php7.2-ldap php7.2-mbstring \
    php7.2-mysql php7.2-pgsql php-smbclient php-ssh2 \
    php7.2-sqlite3 php7.2-xml php7.2-zip
   ```
For more information about installing dependencies on Ubuntu and other Linux distributions, see [Install the required packages](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#install-the-required-packages) in the Administration Manual.


### Testing the HTTP server

So the LAMP server is installed, which means that you have an Apache2 HTTP server. In a later step, you modify the default HTTP configuration. Before you change the defaults, let's test whether you can connect to the HTTP server. Run the following commands to start the server and confirm its status:

1. On Ubuntu, open a terminal type the following commands to start the Apache2 server:

  ```
  sudo systemctl start apache2.service
  ```
1. Confirm the service status by typing the following command:

    ```
    systemctl status apache2
    ```
1. After you confirm that the service is running, use a browser to test whether you can access the default web page. First, determine the IP address of the server. From a terminal, type `ifconfig` to get the computer's IP address.

   The command fails if `net-tools` aren't installed. Follow the instructions in the Ubuntu error message to install the tools and then try again.

1. Copy the IP address that `ifconfig` returns and paste it into the address bar of a browser. The default welcome page for the HTTP server is shown in the browser.

### Creating the database
Create a database user and the database itself from the MySQL command-line interface. After you install ownCloud, the database tables are created automatically
the first time that you log in as administrator.

1. Open a terminal and run the following command:
```
       sudo mysql -u root -p
```
1. When prompted, type your sudo password.

1. From the `mysql>` prompt, type the following command to create the database:
```
       CREATE DATABASE owncloud;
```
1. Type the following command to create the database user:
```
       CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'PASSWORD';
```
  Instead of 'ownclouduser' and 'PASSWORD', type the user name and password that you want ownCloud to use when it connects to the database. Be sure to save this name and password for later reference.

1. Type the following command to assign user permissions:
```
       GRANT ALL ON owncloud.* TO 'ownclouduser'@'localhost' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
```

1. Flush the privileges and then exit from the MySQL command shell:
```
       FLUSH PRIVILEGES;
       EXIT;
```

   Flushing privileges reloads the grant tables in the database so that you don't have to restart the mysql service to put permission changes into effect.


_______
**Note**

For our purposes, we won't secure the database, but you would do that in any production environment. For information about recommended practices for securing your database, see the documentation for your database type.

_______


<!--Provide link to info
-->


### Installing the ownCloud server package

You can install ownCloud from the package manager of your Linux distribution, or you can install the software manually from the command line. We'll use the command line to complete the installation.

1. Open a browser to the [ownCloud Downloads page](http://owncloud.org/download), scroll to **Linux distribution packages**, and then click **Go to packages**.

1. From the **Install package owncloud-files** page, click the name of your operating system to view the installation instructions. The next steps show the instructions to follow on Ubuntu.

1. First download the release key. Run the following commands from a terminal:

    ```
    wget -nv https://download.owncloud.org/download/repositories/production/Ubuntu_18.04/Release.key -O Release.key

    sudo apt-key add - < Release.key
    ```

1. Now open a root shell:

   ```
   sudo su
   ```
   Type your root password when prompted.

   1. In the root shell, type the following commands to add the ownCloud repository and install the ownCloud server files:

   ```
   echo 'deb http://download.owncloud.org/download/repositories/production/Ubuntu_18.04/ /' > /etc/apt/sources.list.d/owncloud.list

   sudo apt-get update
   sudo apt-get install owncloud-files
   ```
The ownCloud packages are now installed on your computer.

## Configuring the server

For the moment, your Apache2 HTTP server runs under its default configuration. After you verify that everything works, you can modify the default port and set ownCloud to open from the root of the web server root URL.

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

<!-- 2. Create a symbolic link to enable the ownCloud site configuration. From a terminal, type the following command to create a symbolic link to `/etc/apache2/sites-enabled`:

  ````
  sudo ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf
  ````

This duplicates the function of  the a2ensite command, which should achieve the same effect.
-->


### Enabling HTTP server modules
Run the `a2ensite` and `a2enmod`commands to complete the configuration. `a2ensite` triggers a script that enables the ownCloud site configuration. The `a2enmod` commands trigger scripts to enable other modules in the apache2 configuration. The scripts create symbolic links within `/etc/apache2/sites-enabled` and `/etc/apache2/mods-enabled`.

1. From a terminal, run the following commands:

   ```
   sudo a2ensite owncloud.conf
   sudo a2enmod rewrite
   sudo a2enmod headers
   sudo a2enmod env
   sudo a2enmod dir
   sudo a2enmod mime
   ```
  It's possible that some of the modules are already enabled, but there's no harm in reenabling them.

1. Run the following command to restart the Apache2 HTTP server and refresh the configuration:

   ```
   sudo systemctl restart apache2
   ```
 1. From your browser, navigate to the IP address of the server. For example, For example, http://192.168.1.123

    The ownCloud sign-in page is shown in the browser.

<br>

  !["Sample ownCloud log-in page"](/images/oc_login_dflt_port.PNG)

### Changing the default port of the ownCloud server

Now you're ready to configure a custom port for serving up your ownCloud server. We'll set the port to 8080. After you make the change, clients that want to connect to the server must specify the new port number in the server URL.

To modify the port number, you edit two Apache2 configuration files. You must open the files with sudo privileges.  

1. Open the file `/etc/apache2/ports.conf` in a text editor. For example:

   ```
   sudo vi /etc/apache2/ports.conf
   ```

1. In the first line of the file, change the port number from 80 to 8080 and then save and close the file. For example,

    ```
    Listen 8080
   ```

1. Now, open the file `/etc/apache2/sites-enabled/000-default.conf` in a text editor. For example,

   ```
   sudo vi /etc/apache2/sites-enabled/000-default.conf
   ```

1. Make a similar change to the first line of this file, by changing the port number from 80 to 8080. For example,  

   ```
   <VirtualHost *: 8080>
   ```

### Verify that the HTTP server port is now set to 8080

Restart the HTTP server to put the change into effect and then point your browser to the ownCloud server at the new port.

1. From a terminal, type the commands to restart the service and check the status. For example,

   ````
    service apache2 restart

    systemctl status apache2
  ````

1. In the address bar of a browser, type the IP address of the server and append the new port. For example, `http://192.168.1.123:8080`

    The default welcome page for the HTTP server is shown in the browser.


### Running the ownCloud installation wizard

To complete the ownCloud installation, run the installation wizard from a browser.

1. From a browser, open the ownCloud web UI, as described in the previous step.

2. Create an administrator account by typing a name and password in the **Create an admin account** field.

3. Click **Finish setup**.

   And that's it. Add some users and you're ready to start using your ownCloud server.   

_____
**Note**

As mentioned earlier, the server that is installed in these instructions is not secured for production use. For more information about hardening the ownCloud server, see [Post-installation steps](https://doc.owncloud.org/server/10.0/admin_manual/installation/installation_wizard.html#post-installation-steps) in the ownCloud Server Administration Manual.

_______

## Adding user accounts
After you log in as administrator, you can add users to the ownCloud local user registry.

From the *User management* page of the ownCloud web UI, type a user name in the **Username** text box, and an email address in the **E-mail** text box, and then click **Create**.

![Screenshot showing how to open the User management page from the ownCloud Web UI](/images/admin_menu.png)

For more information, see [Creating a New User](https://doc.owncloud.org/server/10.0/admin_manual/configuration/user/user_configuration.html#creating-a-new-user) in the ownCloud Server Adminstration Manual.


_____
**Note**

 Although you must add an email address to create a user, the ownCloud server can't send emails unless you complete further configuration. For more information, see [Email Configuration](https://doc.owncloud.org/server/10.0/admin_manual/configuration/server/email_configuration.html) in the ownCloud Server Administration Manual.


_______

## ownCloud synchronization clients
To provide automatic synchronization of files across multiple device, you can install ownCloud Synchronization Clients. Desktop Synchronization Clients are available for Linux, macOS, and Microsoft Windows.

Mobile clients are available for the Android and
Apple iOS operating systems. Users are presented with download links the first time that they log in.

### Installing the ownCloud desktop client
You can download desktop clients from the [ownCloud download page](https://owncloud.com/download/#desktop-clients).

1. Open the [ownCloud Client download page](https://owncloud.org/download/#install-clients),click the operating system for which you want to install a client, and then follow
the instructions to add the repository and complete the installation.

1. Run the installation wizard to complete the ownCloud client setup.

 On Windows 10, click **Yes** to allow the program to make changes to your computer. Enter your administrator password if prompted.

You'll need to restart your computer to enable Windows Explorer integration, but if you want to check out the client right away, it's ok to wait until later to restart. Just realize that you won't be able to use ownCloud with Explorer just now.
If you decide not to restart your computer now, select the checkbox on the final screen of the Setup wizard to open the ownCloud client now.

When prompted, either click *Yes* to restart the computer now and enable Explorer integration, or click *No* to restart later.

### Connecting clients to the server

After the client starts, you're ready to configure a connection to the server.

1. If you haven't started it, go ahead and start the server, and then type the address and port of the ownCloud server, using
the format `http://xxx.xxx.xxx.xxx:<port_number>`, and click **Next**.

   If you don't type the `http` prefix when you supply the server address, the client automatically tries to connect using HTTPS. Because we didn't install a certificate to secure incoming connections, the connection will fail. For information about using certificates to secure connections, see [How to Add Certificates](https://doc.owncloud.org/server/10.0/admin_manual/appliance/certificates.html?highlight=certificate) in the ownCloud Server Administrator Manual.

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

### Mobile clients  
Mobile clients are available for purchase on the Google Play Store and on the Apple App Store.

For more information about the ownCloud Android app, see [Using the ownCloud Android App](https://doc.owncloud.com/android/index.html)

For more information about using the ownCloud iOS app, see [Using the ownCloud iOS App](https://doc.owncloud.com/ios/index.html)
