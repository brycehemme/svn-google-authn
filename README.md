svn-google-authn
================

This document is intended as a tutorial to setup SVN using two-factor authentication
with Apache and Google Authenticator. 


This document assumes install on a stock Ubuntu 12.04.3 instance with no prior installations.
It is also assumed that you're using a user with root priveleges.


Machine Configuration
=====================

Change the hostname of the server to the appropriate name so it displays as desired in Google Authenticator clients:

    $ sudo vi /etc/hosts

Add a line like this to the opened file:

    127.0.0.1 myNewMachineName
  
Save and close the file

Type the command below to change the machine name:

    $ sudo hostname myNewMachineName
    
At this point the machine name has been changed.



Install updates and C/C++ compiler and other tools:

    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo apt-get install build-essential
    


Install Subversion and Apache:
====================

    $ sudo apt-get install subversion
    $ sudo apt-get install libapache2-svn apache2
    
Enable SSL:

    $ sudo a2enmod ssl
    $ sudo vi /etc/apache2/ports.conf
    
    In the open file change "NameVirtualHost *" to "NameVirtualHost *:443"
    
    
Generate Certificate:

    $ sudo apt-get install ssl-cert
    $ sudo mkdir /etc/apache2/ssl
    $ sudo /usr/sbin/make-ssl-cert /usr/share/ssl-cert/ssleay.cnf /etc/apache2/ssl/apache.pem
    
    Accept the localhost value for the cert by hitting enter
    
Create Virtual Host:

    $ sudo cp /etc/apache2/sites-available/default /etc/apache2/sites-available/svnserver
    $ sudo vim /etc/apache2/sites-available/svnserver 
    
    In the open file change <VirtualHost *> to <VirtualHost *:443>
    Add the following below the ServerAdmin tag in the opened file and then save and close:
    
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/apache.pem
    SSLProtocol all
    SSLCipherSuite HIGH:MEDIUM
    
Enable the site:

    $ sudo a2ensite svnserver
    $ sudo vi /etc/apache2/apache2.conf
    
    Add the following line to the open file, save, and close:
    
    ServerName localhost
    
Restart Apache:

    $ sudo /etc/init.d/apache2 restart
    
Adding a repository to SVN 
===================

    $ sudo mkdir /var/svn
    $ REPOS=myFirstRepo
    $ sudo svnadmin create /var/svn/$REPOS
    $ sudo chown -R www-data:www-data /var/svn/$REPOS
    $ sudo chmod -R g+ws /var/svn/$REPOS
    
    Repeat everything but mkdir /var/svn to create additional repositories
    

Google Authenticator
=======================

Install PAM and download and build Google Authenticator:

    $ sudo apt-get install libpam0g-dev
    $ cd ~
    $ sudo wget http://google-authenticator.googlecode.com/files/libpam-google-authenticator-1.0-source.tar.bz2
    $ sudo tar -jxf libpam-google*
    $ cd libpam-google*
    $ sudo make install
    


Google Auth Apache Module
=========================

Download and install Google Auth Apache Module:

    $ wget https://google-authenticator-apache-module.googlecode.com/files/GoogleAuthApache.src.r10.bz2
    $ sudo tar -jxf GoogleAuthApache.src.r10.bz2 
    $ cd google-authenticator*
    $ sudo apt-get install apache2-prefork-dev
    $ sudo vi Makefile
    
    in the opened file, change line 1 to:
    
    APXS=apxs2
    
    then change line 7 to point to the location to install and save and close the file:
    
    install: all
         sudo cp .libs/mod_authn_google.so /usr/lib/apache2/modules/
         
         
    Replace mod_authn_google.c with the code from r21 on the project site here:
    https://code.google.com/p/google-authenticator-apache-module/source/detail?r=21
    (this is when true two-factor auth was added)

    $ sudo make install

Setup two factor auth in Apache:

    $ cd /etc/apache2/
    $ sudo mkdir two-factor 
    $ sudo vi httpd.conf
    
    Add the following line the opened file, save, and close:
    
    Loadmodule authn_google_module /usr/lib/apache2/modules/mod_authn_google.so

    $ sudo vi ports.conf
    
    In the open file change <VirtualHost *> to <VirtualHost *:443>

    $ cd /etc/apache2/mods-available
    $ sudo vi dav_svn.conf
    
    In the open file add the text below, save, and close:
    
    <Location /svn>
        DAV svn 
        SVNParentPath /var/svn
        AuthType Basic
        AuthName "Google Authenticator Code"
        AuthBasicProvider "google_authenticator"
        Require valid-user
        GoogleAuthUserPath /etc/apache2/two-factor/
        GoogleAuthCookieLife 3600
        GoogleAuthEntryWindow 2
        SSLRequireSSL
    </Location>
    
Restart Apache:

    $ sudo /etc/init.d/apache2 restart


At this point SVN, Apache, and Google Authenticator are configured. Now users can be added.


Adding Users
===============

    First, users need to be added as a system user so they show up appropriately in the Google Auth client
    $ sudo adduser firstName.lastName --force-badname
    $ sudo su - firstName.LastName
    # google-authenticator
    
    Google Authenticator will ask several quesitons. The proper response are y, y, y, n, y for security reasons. These should be reviewed based on the environment this is being deployed to.
    Copy the URL that is printed after answering the first question - this is the URL for the QR code
    
    # exit
    $ cd /etc/apache2/two-factor
    $ sudo cp /home/firstName.lastName/.google_authenticator firstName.lastName
    $ sudo chown -R :www-data /etc/apache2/two-factor/  
    $ sudo chmod g+r firstName.lastName
    $ sudo vi firstName.lastName
    
    Modify the opened file to include a line similar to the below line, save, and close:
    
    " PASSWORD=myTestPassword


Everything should now be configured. To login, browse to the URL and use the username, and password + 6 character Google Authenticator code.

    
    

    
    



