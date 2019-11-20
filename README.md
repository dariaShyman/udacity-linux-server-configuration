
# Linux Server Configuration: Udacity Full Stack Nanodegree Project

This is the final project toward Udacity Full Stack Web Developer Nanodegree program. The project is focused on hosting the web application developed in the previous project by a Ubuntu Linux server on an Amazon Lightsail instance: [Catalog Web Application](https://github.com/dariaShyman/item-catalog).

  

## About

  

This tutorial describes the necessary steps for installation of a Linux server and hosting the web application. It shows the ways how to secure the server, to install and configure a database server, and deploy one of the existing Flask-based applications.

  
  
## Technical Information About the Project

  

-  **Server IP Address:** 18.194.105.187

-  **SSH server access port:** 2200

-  **SSH login username:** grader

-  **Application URL:** http://18.194.105.187.xip.io

  
  
## Set up a server instance
### Start a new Ubuntu Linux Server instance on Amazon Lightsail

1. Create an AWS account

2. Click **Create instance** button on the home page

3. Select **Linux/Unix** platform

4. Select **OS Only** and **Ubuntu** as blueprint

5. Select an instance plan

6. Name your instance

7. Click **Create** button

  

### SSH into your Server

1. Download the private key from the **SSH keys** section in the **Account** section on Amazon Lightsail.

2. Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your local machine

3. Copy and paste content from downloaded private key file to **lightsail_key.rsa**

4. Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`

5. SSH into the server: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.194.105.187`

  

## Securing the system

  

### Change the SSH port from 22 to 2200

1. Run `$ sudo nano /etc/ssh/sshd_config` to open up the configuration file

2. Change the port number from **22** to **2200** and save the file

3. Restart SSH: `$ sudo service ssh restart`

  

### Configure the firewall

1. Check firewall status: `$ sudo ufw status`

2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`

3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`

4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`

5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow www`

6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow 123/udp`

7. Close port 22: `$ sudo ufw deny 22`

8. Enable firewall: `$ sudo ufw enable`

9. Check out current firewall status: `$ sudo ufw status`

10. Update the firewall configuration on Amazon Lightsail website under **Networking**. Delete default SSH port 22 and add  ports ** 80, 123, 2200**

11. Ssh via the port 2200: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.194.105.187 -p 2200`

  
 ## Give grader access

### Create a new user account **grader** and give **grader** sudo access

1. Create a new user account **grader**:`$ sudo adduser grader`

2. Create a file named grader: `$ sudo touch /etc/sudoers.d/grader`

3. Edit : `$ sudo nano /etc/sudoers.d/grader`, add code `grader ALL=(ALL:ALL) NOPASSWD:ALL`. 

  

### Set SSH login using keys

1. Create an SSH key pair for **grader** using the `ssh-keygen` tool on your local machine. Save it in `~/.ssh` path

2. Deploy public key on development environment

On your local machine, read the generated public key: `cat ~/.ssh/grader_key.pub`

On your virtual machine run: 

```

$ mkdir /home/grader/.ssh

$ touch /home/grader/.ssh/authorized_keys

$ nano /home/grader/.ssh/authorized_keys

```

3. Change file permission:  `chmod 700 /home/grader/.ssh` and `chmod 644 /home/grader/.ssh/authorized_keys` 

4. Restart SSH: `$ sudo service ssh restart`

5. Login in as grader: `$ ssh -i ~/.ssh/grader_key -p 2200 grader@18.194.105.187`

  
  

## Project deployment


### Configure the local timezone to UTC

1. Run `$ sudo dpkg-reconfigure tzdata`

2. Choose **None of the above** to set timezone to UTC

  ### Install PostgreSQL

1. Run `$ sudo apt-get install postgresql`

3. Open file: `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`

4. The file should look like this in order to not allow remote connections:

```

# Database administrative login by Unix domain socket

local all postgres peer

  

# TYPE DATABASE USER ADDRESS METHOD

  

# "local" is for Unix domain socket connections only

local all all peer

# IPv4 local connections:

host all all 127.0.0.1/32 md5

# IPv6 local connections:

host all all ::1/128 md5

```

### Create new PostgreSQL user called **catalog**

1. Switch to PostgreSQL default user **postgres**: `$ sudo su - postgres`

2. Connect to PostgreSQL: `$ psql`

3. Create user **catalog** with LOGIN role: `# CREATE ROLE catalog WITH PASSWORD 'catalog';`

4. Allow user to create database tables: `# ALTER USER catalog CREATEDB;`

5. Create database: `# CREATE DATABASE catalog WITH OWNER catalog;`

6. Connect to database **catalog**: `# \c catalog`

7. Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`

8. Grant access to **catalog**: `# GRANT ALL ON SCHEMA public TO catalog;`

9. Exit psql: `\q`

10.Exit user **postgres**: `exit`

  
 

### Install and configure Apache

1. Install **Apache**: `$ sudo apt-get install apache2`

2. Go to http://18.194.105.187/ to check if Apache is working correctly  (the default Apache2 page will be displayed)

  

### Install and configure Python mod_wsgi

1. Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`

2. Enable **mod_wsgi**: `$ sudo a2enmod wsgi`

3. Restart **Apache**: `$ sudo service apache2 restart`

  


### Install git and clone item-catalog application from github

1. Run `$ sudo apt-get install git`

2. Create dictionary: `$ mkdir /var/www/catalog`

3. CD to this directory: `$ cd /var/www/catalog`

4. Clone the item catalog application: `$ git clone https://github.com/dariaShyman/item-catalog catalog`

7. Change file **application.py** to **__init__.py**: `$ mv application.py __init__.py`

8. Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in **__init__.py** file

  

### Update the absolute path of client secrets files

1. Update the path to: `/var/www/catalog/catalog/client_secrets.json`

  

### Update all currently installed packages

1. Run `sudo apt-get update` to update packages

2. Run `sudo apt-get upgrade` to install latest versions of packages

3. Set for future updates: `sudo apt-get dist-upgrade`

  

### Setup for deploying a Flask App on Ubuntu VPS

1. Install pip: `$ sudo apt-get install python-pip`

2. Install packages:

```

$ sudo pip install httplib2

$ sudo pip install requests

$ sudo pip install oauth2client

$ sudo pip install sqlalchemy

$ sudo pip install flask

$ sudo apt-get install libpq-dev

$ sudo pip install psycopg2

```

  

### Setup and enable a virtual host

1. Create file: `$ sudo touch /etc/apache2/sites-available/catalog.conf`

2. Add the following to the file:

```

<VirtualHost *:80>

ServerName 18.194.105.187

ServerAlias ec2-18-194-105-187.compute-1.amazonaws.com

ServerAdmin grader@18.194.105.187

WSGIScriptAlias / /var/www/catalog/catalog.wsgi

<Directory /var/www/catalog/catalog/>

Order allow,deny

Allow from all

Options -Indexes

</Directory>

Alias /static /var/www/catalog/catalog/static

<Directory /var/www/catalog/catalog/static/>

Order allow,deny

Allow from all

Options -Indexes

</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log

LogLevel warn

CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```

3. Create a **.htaccess** file in the **.git** folder and put the following in this file: `RedirectMatch 404 /\.git` to make .git directory not publicly accessible via a browser
4. Enable the virtual host `$ sudo a2ensite catalog` 

5. Restart Apache: `$ sudo service apache2 reload`


  

### Configure .wsgi file

1. Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`

2. Add content below to this file and save:

```

#!/usr/bin/python

import sys

import logging

logging.basicConfig(stream=sys.stderr)

sys.path.insert(0,"/var/www/catalog/")

  

from catalog import app as application

application.secret_key = 'super_secret_key'

```

3. Restart **Apache**: `$ sudo service apache2 reload`

  

### Edit the database path

1. The connection string has the format: postgresql://username:password@host:port/database
2. Replace lines in `__init__.py`, `database_setup.py`, and `demodata.py` with `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

  

### Set up database schema

1. Run `$ sudo python database_setup.py`

2. Run `$ sudo python demodata.py`

3. Restart **Apache**: `$ sudo service apache2 reload`

4. Application should be running: http://18.194.105.187.xip.io
5. Errors can be checked here:  [Apache error file](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)

  

## Sources

1.  [Amazon Lightsail Website](https://aws.amazon.com/lightsail/?p=tile)

3.  [Udacity](https://www.udacity.com)

4.  [Apache](https://httpd.apache.org/docs/2.2/configuring.html)
