= ownCloud Quickstart Guide
Author <bob.roldan@gmail.com>
v0.1, 2019-01-03

ifdef::env-github[]
:imagesdir: images/
:toc:
:toc-placement!:
:tip-caption: :bulb:
:note-caption: :information_source:
:important-caption: :heavy_exclamation_mark:
:caution-caption: :fire:
:warning-caption: :warning:
endif::[]

This document provides guidance for administrators who want to deploy a locally hosted,
single-machine ownCloud Community Edition server and make it available to desktop and mobile clients.
You can use several methods to install OwnCloud. These instructions focus on using Linux package manager
for the installation.

[IMPORTANT]
=====================================================

These instructions are intended to help you to quickly
install a test deployment of ownCloud on a local computer.
The resulting server is not secured for use in a production
setting.

=====================================================

For information about other methods and for detailed instructions about deploying ownCloud,
refer to the https://doc.owncloud.org/server/10.0/admin_manual/installation/system_requirements.html[ownCloud Server Administration Manual].

toc::[]

== Introduction
ownCloud is a secure, open source, private cloud file storage system that automatically synchs files across multiple
clients.
Users can access files stored on ownCloud from remote desktop or mobile devices.

== Installing and configuring ownCloud Community Edition
=== System requirements
You can install the ownCloud server on a range of Linux operating systems. Be sure that computer where
you install ownCloud meets the minimum requirements. For more information about system requirements,
see https://doc.owncloud.org/server/10.0/admin_manual/installation/system_requirements.html[System requirements].

[NOTE]
=====================================================

You can install ownCloud on a Windows computer by installing Linux within a VirtualBox environment.

=====================================================



=== Before you begin
Administrators who install ownCloud must have command line or `cron` access.

==== Installation prerequisites

Be sure that you have installed a LAMP stack (Linux OS, Apache HTTP server, mySQL relational database,
and PHP programming language).
For example, on Ubuntu, run the following commands to install the prerequisites:

----

sudo apt-get update
sudo apt-get install lamp-server^

----

===== Create the database
// See https://www.techrepublic.com/article/how-to-install-owncloud-on-ubuntu-18-04/


You can find a number of excellent video tutorials that run through the steps
for installing prerequisite software.

=== Downloading the server ownCloud server

You can install ownCloud from the package manager of your Linux distribution, or
install the software manually, using the command line. The following instructions
guide you through the process for using the package manager to complete installation.

. Open a browser to http://download.owncloud.org/download/repositories/10.0/owncloud/
. Downloads > Linux Distribution Packages > Go to packages.
. From the *Install package owncloud-files* page, click the operating system on which
 you want to install ownCloud.

//Grab the release key
. From a terminal, run the commands to add a repository and install the server package for
  the OS version that runs on your computer.
. After the package manager finishes adding the ownCloud files to your computer, run the Installation
Wizard to complete the installation.



==== Post-installation steps

Create the file `/etc/apache2/sites-available/owncloud.conf` and set the Alias
directive for your ownCloud directory to the Web server root:

----
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
----

.Create a symbolic link to the owncloud configuration

From a command shell, type the following command to create a symbolic link
to `/etc/apache2/sites-enabled`:

----
ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf
----
== Changing the default address of the ownCloud server
//Enable connections via IP address and port 8080
//https://doc.owncloud.org/server/10.0/admin_manual/installation/changing_the_web_route.html[Changing your ownCloud URL]
//ownCloud is served by your webserver so you need to configure the used port in the configuration of your webserver.
//https://stackoverflow.com/questions/3940909/configure-apache-to-listen-on-port-other-than-80[Configure apache to listen on port other than 80]


====Restart the HTTP server and verify that it is working

. Start the Apache2 server.
//`service apache2 restart`
//`systemctl status apache2`

. From a command line, type 'ipconfig' to obtain the IP address of the computer where you installed
ownCloud.

. From a browser, type the server IP address  in the address bar.
 The default welcome page for the HTTP server
displays in the browser.

// === Create the database  <- This actually happens pre-install
//But I had to re-do setting permissions for my database user before I could
//complete the web UI step of creating my db admin

==== Access the ownCloud server

. From a browser, open the ownCloud Web UI

 type the URL for the ownCloud server: `http://<ip_address>:8080`

For example:

----
 http://192.168.1.123:8080
----

Create an administrator account by typing a name and password in the *Create an admin account* field.

Click *Finish setup*.


== Adding user accounts
Add a local user. Sign in as an adminstrator and then  Open the *User management* page of your ownCloud Web UI.
Optionally, you can add a user to a group.

== Connecting clients to an ownCloud server
To provide automatic synchronization of files across multiple device, you can
install ownCloud Synchronization Clients. Desktop clients are available for Linux,
macOS, and Microsoft Windows. Mobile clients are available for the Android and
Apple iOS operating systems.

[NOTE]
========================================================
Desktop devices can also access ownCloud from a browser.
========================================================

You can download desktop clients from the https://owncloud.com/download/#desktop-clients[ownCloud download page]
Mobile clients are available on Google Play and on the Apple App Store.

Open the https://owncloud.org/download/#install-clients[ownCloud Client download page],
click the operating system for which you want to install a client, and then follow
the instructions to add the repository and complete the installation.

Run the installation wizard to complete the ownCloud client setup.

On Windows 10, click *Yes* to allow the program to make changes to your computer,
Enter your administrator password if prompted.

You'll need to restart your computer to enable Windows Explorer integration, but
if you want to check out the client right away, it's ok to wait until later to
restart. Just realize that you won't be able to use ownCloud with Explorer just now.
If you decide not to restart now, select the checkbox on the final screen of the
Setup wizard to open the ownCloud client now.

When prompted, either click *Yes* to restart the computer now and enable Explorer
integration, or click *No* to restart later.

== Connecting the ownCloud Sychronization client to the server

After the client starts, you're ready to configure a connection to the server.

If you haven't started it, go ahead and start the server, , and then type the
 address and port of the ownCloud server, using
the format `http://xxx.xxx.xxx.xxx:<port_number>` and click *Next*. By default, the client
automatically tries to connect using HTTPS. If you installed the server
according to the instructions in this Quick Start guide, you did not secure the
HTTP connection with a certificate. So, be sure to connect with plain HTTP, or
the connection attempt fails.

Type the credentials of the user that you added from the Admin UI and then click
*Next*.

Set up the local folder options to specify how you want to use synchronization,
and then click *Connect...*.

 Click *Add Folder Sync Connection* to synchronize a new folder, and then
 specify that path to the folder and click *Next*.

 Choose the folder on the ownDrive server where you want to save the synchronized
 folder content and click *Next*.

 Deselect any folders on the server that you don't want the client to synchronize,
 and then click *Add Sync Connection*.

== Mobile  
