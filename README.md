# Project 5 - Linux Server Configuration
 by Mohammed Yousuffi<br>


## Table of Contents

1. [Description](#Description)<br>
2. [Design](#Design)<br>
3. [Prerequisites](#Prerequisites)<br>
4. [InstantiateLinuxServer](#InstantiateLinuxServer)<br>
5. [Update Server Software](#Update Server Software)<br>
6. [Set Server DateTime](#Set Server DateTime)<br>
7. [SSH over Custom Port](#SSH over Custom Port)<br>
8. [Configure UFW Firewall](#Configure UFW Firewall)<br>
9. [Add User](#Add User)<br>
10. [Create SSH Keys](#Create SSH Keys)<br>
11. [Disable Password Login](#Disable Password Login)<br>
12. [Disable Remote Root Login](#Disable Remote Root Login)<br>
13. [Install Python3.6](#Install Python3.6)<br>
14. [Install Python Packages](#Install Python Packages)<br>
15. [Install PostgresSQL](#Install PostgresSQL)<br>
16. [Install Apache2](#Install Apache2)<br>
17. [Deploy Web Application](#Deploy Web Application)<br>
18. [Configure Google OAuth](#Configure Google OAuth)<br>
19. [Verify Web Application is Running](#Verify Web Application is Running)<br>
20. [Debugging](#Debugging)<br>

## Description

This project demonstrates configuring and deploying a web application using a linux server. The web application will
be hosted on a linux Ubuntu server obtained through Amazon AWS Lightsail. The server will be secured to allow only authorized 
SSH access over a custom port and limited to SSH, HTTP, and NTP traffic.  The web application is deployed using a Flask application and an Apache HTTP server. 
Step-by-step instructions to configure the linux server and deploy the web application are provided in this document. 

**Server Info:**

Server Public IP: 54.145.30.90<br>
Web Application URL: [http://54.145.30.90](http://54.145.30.90 ) 


## Design

The project uses a linux server instantiated through Amazon Lightsail.  The server instance is created with
a baseline linux Ubuntu 16.04 OS and minimal configuration. Initial SSH access to the server is secured through
SSH key authentication over port 22.  

To deploy the web application the following will be implemented:

1. SSH access will be secured via custom port 2200
2. Remote access to the server will require SSH key authentication
3. Server will be secured to disallow remote root access
4. Server will only allow traffic from TCP port 2200 (SSH), TCP port 80 (HTTP), and UDP port 123 (NTP)
5. The server will host HTTP server using Apache2 HTTP Server
6. A catalog web application deployed using Flask App and Apache2
7. A backend database implemented using PostgresSQL
8. Remote access for user id "grader" with sudo access will be created

## Prerequisites

To deploy this web application the following prerequisites must be met:

1. A windows or linux host running Python3 or later
2. A google account to login and perform Create, Read, Update, Delete (CRUD) operations
3. Google Developer Client-ID for OAuth configuration


## InstantiateLinuxServer

Create a Linux server instance through [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/create/)

1. Select platform **Linux/Unix**
2. Select blueprint **Os Only**
3. Select OS **Ubuntu**
4. Select an instance plan. (1st Month on bottom tier is free)
5. Create a name for your instance and click **Create Instance**


## Update Server Software

Update the baseline linux server software.

To perform software update:

1. Update the package source list

    ```bash
    sudo apt-get update
    ```
2.  Update the software

    ```bash
    sudo apt-get upgrade
    ```

## Set Server DateTime

Set the DATETIME of the server to UTC

1. Use tzdata to configure datetime

    ```bash
    sudo dpkg-reconfigure tzdata
    ```
2.  Select **None of the Above**
3.  Select **UTC** and press ENTER

    ```bash
    Example output:
    
    Current default time zone: 'Etc/UTC'
    Local time is now:      Sun Jan  5 13:54:17 UTC 2020.
    Universal Time is now:  Sun Jan  5 13:54:17 UTC 2020.
    ```

## SSH over Custom Port

Enable custom TCP port 2200 for SSH

1. Select the server instance in the [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) GUI
2. Navigate to the **Networking** tab
3. Under **Firewall** click **+Add Another**
4. Add **Application** Custom, **Protocol** TCP and **Port Range** 2200
5. Save Firewall configuration
 
## Configure UFW Firewall

Configure Uncomplicated Firewall (UFW) to allow traffic from TCP port 2200 (SSH), TCP port 80 (HTTP), and UDP port 123 (NTP)


1. Check UFW status. Verify it is currently inactive.

    ```bash
    sudo ufw status
    
    Example:
    $ sudo ufw status
    Status: inactive   
    ```
2. Create rule to deny incoming traffic

    ```bash
    sudo ufw default deny incoming
    
    Example:
    $ sudo ufw default deny incoming
	Default incoming policy changed to 'deny'
    (be sure to update your rules accordingly) 
    ```
    
3. Create rule to allow outgoing traffic

    ```bash
    sudo ufw default allow outgoing
    
    Example:
    S$ sudo ufw default allow outgoing
	Default outgoing policy changed to 'allow'
    (be sure to update your rules accordingly)  
    ```
    
4. Create rule to allow traffic for TCP port 2200 (SSH)

    ```bash
    sudo ufw allow 2200/tcp
    
    Example:
    $ sudo ufw allow 2200/tcp
    Rules updated
    Rules updated (v6)  
    ```
4. Create rule to allow www traffic

    ```bash
    sudo ufw allow www
    
    Example:
    $ sudo ufw allow www
    Rules updated
    Rules updated (v6)
    ```

4. Create rule to allow traffic for UDP port 123 (NTP)

    ```bash
    $ sudo ufw allow 123/udp
    Rules updated
    Rules updated (v6) 
    ```

4. Enable the UFW. Note existing SSH sessions over port 22 will be dropped.

    ```bash
    sudo ufw enable
    
    Example:
    $ sudo ufw enable
	Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    Firewall is active and enabled on system startup
    ```
    
4. Verify UFW is active and rules have been created successfully

    ```bash
    sudo ufw allow 2200/tcp
    
    Example:
    $ sudo ufw status
	Status: active	
	To                         Action      From
	--                         ------      ----
	2200/tcp                   ALLOW       Anywhere                  
	80/tcp                     ALLOW       Anywhere                  
	123/udp                    ALLOW       Anywhere                  
	2200/tcp (v6)              ALLOW       Anywhere (v6)             
	80/tcp (v6)                ALLOW       Anywhere (v6)             
    123/udp (v6)               ALLOW       Anywhere (v6)  
    ```    

    

**Reference:** https://help.ubuntu.com/community/UFW

## Add User

1. With sudo access add user **grader**

    ```bash
    sudo adduser grader
    
    Example:
    $ sudo adduser grader
	Adding user `grader' ...
	Adding new group `grader' (1001) ...
	Adding new user `grader' (1001) with group `grader' ...
	Creating home directory `/home/grader' ...
	Copying files from `/etc/skel' ...
	Enter new UNIX password: 
	Retype new UNIX password: 
	passwd: password updated successfully
	Changing the user information for grader
	Enter the new value, or press ENTER for the default
	        Full Name []: Udacity grader user account
	        Room Number []: 
	        Work Phone []: 
	        Home Phone []: 
	        Other []: 
    Is the information correct? [Y/n] y
    ````

2. Give user **grader** sudo access

1. Make a copy of the initial user sudoer file
    
    ```bash
    sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/90-cloud-init-grader
    ```

2. Use nano or vi to update the sudoer file and change user name to grader

    ```bash
    sudo nano /etc/sudoers.d/90-cloud-init-grader
    
    Updated File Output Example:
    # Created by cloud-init v. 17.2 on Fri, 03 Jan 2020 21:20:10 +0000
	
	# User rules for ubuntu
    grader ALL=(ALL) NOPASSWD:ALL
    ```

3. Login as grader and verify **grader** user has sudo access

    ```bash
    sudo -l 
    
    Example:
    $ sudo -l
    Matching Defaults entries for grader on ip-172-26-13-168.ec2.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User grader may run the following commands on ip-172-26-13-168.ec2.internal:
    (ALL) NOPASSWD: ALL
    ```
## Create SSH Keys

1. From a local machine with **ssh-key** generate a SSH key pair

    ```bash
    ssh-keygen
    
    Example:
    
        $ssh-keygen
        Generating public/private rsa key pair.
        Enter file in which to save the key (C:\Users\user/.ssh/id_rsa): C:\Users\user\grader_rsa
        Enter passphrase (empty for no passphrase):
        Enter same passphrase again:
        Your identification has been saved in C:\Users\user\grader_rsa.
        Your public key has been saved in C:\Users\user\grader_rsa.pub.
        The key fingerprint is:
        SHA256:WeAaT9i37Pcb2c2iEMitbWWvQp2EBZcQ domain\host
        The key's randomart image is:
        +---[RSA 2048]----+
        |      .==        |
        |       =E.       |
        |      o+oo       |
        |      +=X=o.     |
        |      .Xoo  .    |
        |    . + X.+o     |
        |     + . +o=o..  |
        |    .  ... . . o |
        |      ...     o  |
        +----[SHA256]-----+
    ```

2. On the Remote server login or switch user to **grader**
3. Maker the **.ssh** directory in the users home directory

    ```bash
    mkdir ~/.ssh
    ```
4. Change to the **.ssh** director and create a file called **authorized_keys***

    ```bash
    touch .ssh/authorized_keys
    ```

5. Add the public key that was generated on the local machine to the authorized_keys file

    ```bash
    vi authorized_keys 
    ```
6.  From local machine SSH to remote machine using private key and verify SSH Access
   
    ```bash
    ssh -i [private_key_path] userid@remoteip -p 2200
    
    Example:
    ssh -i C:\Users\user\grader_rsa grader@54.145.30.90 -p 2200
    ```
    
**Reference:** https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604

## Disable Password Login

1. Update the /etc/ssh/sshd_config file to disable password login

    ```bash
    sudo vi /etc/ssh/sshd_config
    
    From:
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication yes
    
    To:
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication no
    
    ```

2. Restart SSH service

    ```bash
    sudo service ssh restart
    ```

## Disable Remote Root Login

1. Update the /etc/ssh/sshd_config file to disable remote root login

    ```bash
    sudo vi /etc/ssh/sshd_config
    
    From:
    PermitRootLogin yes
    
    To:
    # Change to no to disable tunnelled clear text passwords
    PermitRootLogin no
    
    ```

2. Restart SSH service

    ```bash
    sudo service ssh restart
    ```

## Install Python3.6

1. Add PPA to update package source list

    ```bash
    sudo add-apt-repository ppa:deadsnakes/ppa
    ```

2. Perform software update

    ```bash
    sudo apt-get update
    ```
    
3. Install Python3.6 using apt-get

    ```bash
    sudo apt-get install python3.6
    ```

4. Install Python package installer pip

    ```bash
    curl https://bootstrap.pypa.io/get-pip.py | sudo -H python3.6
    ``` 
    
**Reference:** https://askubuntu.com/questions/865554/how-do-i-install-python-3-6-using-apt-get

## Install Python Packages

1. Install PostgreSQL adapter dependencies

    ```bash
    sudo apt-get install libpq-dev python3-dev
    sudo apt-get install build-essential
    sudo apt-get install -y python3.6-dev
    ```

2. Install Python WSGI

    ```bash
    sudo apt-get install libapache2-mod-wsgi-py3 python3.6-dev
    sudo -H pip3.6 install mod_wsgi
    ```

3. Create a requirements.txt file with the following packages for teh WebApp and install using pip

Requirements File:

    * sqlalchemy
    * flask
    * sqlalchemy
    * oauth2client
    * httplib2
    * requests
    * pyasn1
    * psycopg2

    Install using pip:
    
    sudo -H  pip3.6 install -r requirements.txt

## Install PostgresSQL

1. Install PostgresSQL

    ```bash
    sudo apt install postgresql
    ```

2. Connect tot he default PostgreSQL template database

    ```bash
    sudo -u postgres psql template1
    ```
    
3. Configure the password for the postgres user

    ```bash
    ALTER USER postgres with encrypted password 'udacity123';
    ```

4. Edit the file /etc/postgresql/9.5/main/pg_hba.conf to use MD5 authentication with the postgres user

    ```bash
    Example updated line:
    
    local   all         postgres                          md5
    ```    
    

5. Verify login using postgres user 

    ```bash
    $psql -U postgres
    Password for user postgres: 
    psql (9.5.19)
    Type "help" for help.
    
    postgres=# 
    ```
    
6. Create a database in the PostgreSQL DB

    ```bash
    CREATE DATABASE [database_name];
    
    Example:
    postgres=# CREATE DATABASE catalog;
    CREATE DATABASE
    
     postgres=# \l
                                       List of databases
        Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
     -----------+----------+----------+-------------+-------------+-----------------------
      catalog   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
      postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
      template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                |          |          |             |             | postgres=CTc/postgres
      template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                |          |          |             |             | postgres=CTc/postgres
     (4 rows)   
    ```


**Reference:** https://help.ubuntu.com/lts/serverguide/postgresql.html<br>
**Reference:** https://www.postgresql.org/docs/9.0/sql-createdatabase.html

## Install Apache2

1. Install Apache2 using apt-get

    ```bash
    sudo apt-get install apache2
    ```
2. Install Apache2 using apt-get

    ```bash
    ALTER USER postgres with encrypted password 'udacity123';
    ```
    
3. Verify Apache is Running

    ```bash
    sudo systemctl status apache2
    ```
    
4. Verify Apache server is reachable

    a. Obtain public IP
    ```bash
    curl -4 icanhazip.com
    ```
    
 4. Verify Apache is reachable from browser - [http://54.145.30.90](http://54.145.30.90 ) 

**Reference:** https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04

## Deploy Web Application


1. Create folder in the /var/www directory called catalog

    ```bash
    sudo mkdir /var/www/catalog
    ```

2. Change ownership of /var/www/catalog to grader user

    ```bash
    sudo chown -R grader:grader catalog/
    ```

3. Change to the /var/www/catalog/ folder and clone the Web App Git repository

    ```bash
    sudo git clone https://github.com/mbasith/MyCatalog.git
    ```

4. Rename the MyCatalog git repo folder to "mycatalog"

    ```bash
    mv /var/www/catalog/MyCatalog /var/www/catalog/mycatalog 
    ```

5. Create the WSGI file named **flaskapp.wsgi** under /var/www/catalog with the following contents

    ```bash
    #!/usr/bin/python3.6
    import logging
    import sys
    
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, '/var/www/catalog/')
    from mycatalog import app as application
    application.secret_key = 'secret_key'
    ```
6. Rename the webapp in the /var/www/catalog/mycatalog directory from **"myproject.py"** to **"\_\_init\_\_.py**"

    ```bash
    mv /var/www/catalog/mycatalog/myproject.py /var/www/catalog/mycatalog/__init__.py
    ```

7. Update the SQLAlchemy engine to point to the PostgreSQL DB in the app and database files.

    ```bash
    App File (__init__.py):
    
        From:     
        # engine = create_engine('sqlite:///videogamecatalog.db',
        #                        connect_args={'check_same_thread': False})
        
        To:
        engine = create_engine('postgresql://postgres:udacity123@localhost/catalog')
    
    App File (database_setup.py):
    
        From:     
        ## engine = create_engine('sqlite:///videogamecatalog.db')
        
        To:
        engine = create_engine('postgresql://postgres:udacity123@localhost/catalog')
        ```

8. Create the Apache site conf file for the web application and update it with the following information

    ```bash
    sudo vi /etc/apache2/sites-available/catalog.conf
    
    <VirtualHost *:80>
     # Add machine's IP address (use ifconfig command)
     ServerName 54.145.30.90
     # Give an alias to to start your website url with
     WSGIScriptAlias / /var/www/catalog/flaskapp.wsgi
     <Directory /var/www/catalog/mycatalog/>
             Order allow,deny
             Allow from all
     </Directory>
     Alias /static /var/www/catalog/mycatalog/static
     <Directory /var/www/catalog/mycatalog/static/>
             Order allow,deny
             Allow from all
     </Directory>
     Alias /templates /var/www/catalog/mycatalog/templates
     <Directory /var/www/catalog/mycatalog/templates/>
             Order allow,deny
             Allow from all
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     LogLevel warn
     CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```

9. Enable the the Apache conf file

    ```bash
    sudo a2ensite catalog.conf
    
    Example:
    grader@ip-172-26-13-168:/etc/apache2/sites-available$ sudo a2ensite catalog.conf
    Enabling site catalog.
    To activate the new configuration, you need to run:
      service apache2 reload
    ```

10. Reload apachhe2 after enabling site

    ```bash
    service apache2 reload
    ```

11. Final Flask Application will have the following directory structure 

    ```bash
    /var/www/catalog
    |-- database_setup.py
    |-- flaskapp.wsgi
    `-- mycatalog
        |-- client_secrets.json
        |-- database_setup.py
        |-- __init__.py
        |   `-- database_setup.cpython-36.pyc
        |-- readme.md
        |-- static
        |   |-- css
        |   |   `-- styles.css
        |   `-- js
        |       `-- star_rating.js
        |-- templates
        |   |-- deletegame.html
        |   |-- deletegenre.html
        |   |-- editgame.html
        |   |-- editgenre.html
        |   |-- game.html
        |   |-- genres.html
        |   |-- header.html
        |   |-- login.html
        |   |-- main.html
        |   |-- newgame.html
        |   |-- newgenre.html
        |   |-- publicgame.html
        |   `-- publicgenres.html
    ```

**References:**<br>
[Minimal Apache configuration for deploying a flask app](https://www.codementor.io/@abhishake/minimal-apache-configuration-for-deploying-a-flask-app-ubuntu-18-04-phu50a7ft)<br>
[How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)<br>
[Youtube - Flask Web Development in Python - 2 - Your first Flask Web App](https://www.youtube.com/watch?v=_80H2WIuA7w&t=1233s)<br>


## Configure Google OAuth

1. Got to [Google Developer Console](https://console.developers.google.com/) and select catalog API
2. Navigate to the **Credentials** table and create or edit the existing OAuth2.0 client ID
3. Update the **Authorized JavaScript origins** with the address of the server. Ex. http://54.145.30.90 or http://domain
4. Update the **Authorized redirect URIs** using the URI for the webserver IP using xp.io or an actual domain. Ex. http://54.145.30.90.xp.io or http://domain
5. Save updates and download the JSON file
6. Update the client_json info in the Web Application

##  Verify Web Application is Running

1. Launch WebApp URL

Server Public IP: 54.145.30.90 
Web Application URL: [http://54.145.30.90](http://54.145.30.90 ) 


## Debugging

1. To monitor Apache2 logs for errors
    
    ```bash
    sudo tail -f /var/log/apache2/error.log
    ```

2. To Reload/Restart Apache Server

    ```bash
    sudo service apache2 reload
    sudo service apache2 restart    
    ```