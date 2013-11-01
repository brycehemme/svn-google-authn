svn-google-authn
================

This document assumes install on a stock Ubuntu 12.04.3 instance with no prior installations


Connect to instance through shell or Putty if using Windows

Change the hostname of the server to the appropriate name:

    $ sudo vi /etc/hosts

Add a line like this to the opened file:

    127.0.0.1 myNewMachineName
  
Save and close the file

Type the command below to change the machine name:

    $ hostname myNewMachineName
    
At this point the machine name has been changed.

-----------------------------
-----------------------------

Install updates and C/C++ compiler:

    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo apt-get install build-essential
    
-----------------------------
-----------------------------

Install SubVersion and Apache:

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
    $ sudo /etc/init.d/apache2 restart
    
Adding Repositories:



