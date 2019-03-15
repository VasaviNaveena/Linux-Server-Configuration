#Project : Linux-Server-Configuration
This is a project for Udacity's [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)

## Project Summary
* Creation and installation of a linux server in the cloud environment.
* Setting up of this server to host a web application.
* Installation and configuration of a database server to serve the web application.
* Deployment of the web application.

## Introduction
This project requires to create and set up linux server (e.g. Ubuntu) using the cloud services (Amazon AWS). The server should be then configured to host a web application while ensuring security, availability and accessibility of the server and the app. 

. The server should be initialized and available through a cloud environment.

. The server should be made secure and remotely accessible through SSH - using the public-private key pair authentication feature instead of typical password protection.

. The server should have an account for the user `grader` with `sudo` privileges.

. The server should let the user `grader` to log in to the server remotely through SSH using a public-private key pair authentication.

. The server should not allow remote logins for the root user.

. The server should allow the incoming network traffic only for SSH on port 2200, HTTP on port 80, and NTP on port 123.

. The server should have all the packages up-to-date.

. The server should have a database server set up and running for the web application.

. The server should be set up to serve the  'catalog' Project as wsgi application.*

## Ubuntu Server as Amazon EC2 instance
Amazon EC2 provides somewhat more feature-rich and usable interface to set up, configure and run virtual machine or servers.

# I created an Ubuntu Server on EC2 using the following steps:
1. Login/signup to https://console.aws.amazon.com/ and login to default user (ubuntu)
2. On the EC2 Dashboard, access the Instances menu and click on the Instances option. 
3. On the EC2 Instance screen, click on the Launch Instance button.
4. On the list presented, locate and select the Ubuntu Server(64-bit Arm).
5. And then click on the Review and Launch button.
6. On the summary screen, click on the Launch button.
7. Select the Key pair authorized to connect to the new virtual machine and click on the Launch Instances.
8. Download the newly created Key Pair(.pem file)
9. On the EC2 Dashboard, access the Instances menu and click on the Instances option.As you can see a virtual machine was created.

 SERVER DETAILS
. IPv4 Public IP: 54.212.41.249

. Public DNS (IPv4): ec2-54-212-41-249.us-west-2.compute.amazonaws.com

. Application's URL: http://ec2-54-212-41-249.us-west-2.compute.amazonaws.com

. Accessible SSH Port: 2200

10. In launch-wizard-18 go to inbound and add the following:

Application      Protocol      Port range
SSH              TCP           22
HTTP             TCP           80
Custom           UDP           123
Custom           TCP           2200

## Linux Server Configuration:

# connecting to server via SSH
* Save the .pem file where your project folder is located.
* I used the Git bash client and logged in using following command: 

       `ssh -i Catalogkey.pem ubuntu@54.212.41.249`

# Updating packages
`sudo apt-get update`

`sudo apt-get upgrade`

Trying to run these commands wont install packages kept back,then use 

`sudo apt-get dist-upgrade`

It allows you to install new packages when needed 

# Change the SSH port from 22 to 2200
- Edit the /etc/ssh/sshd_config file: sudo vi /etc/ssh/sshd_config.
- Change the port number on line 5 from 22 to 2200.
- Save and exit using esc and confirm with :wq.
- Restart SSH: sudo service ssh restart.
- Change inbound rules in Amazon EC2 --> Type : Custom TCP Rule as 2200
-To check port 2200 weather working or not by using `ssh -i Catalogkey.pem -p 2200 ubuntu@54.212.41.249`

# Configure the Uncomplicated Firewall (UFW)
* Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

sudo ufw status                  # The UFW should be inactive.
sudo ufw default deny incoming   # Deny any incoming traffic.
sudo ufw default allow outgoing  # Enable outgoing traffic.
sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
sudo ufw allow www               # Allow HTTP traffic in.
sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
sudo ufw deny 22                 # Deny tcp and udp packets on port 53.

* Turn UFW on by using following command: 
    `sudo ufw enable` 
The output should be like this:
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

* Check the status of UFW to list current roles:
    `sudo ufw status` 
The output should be like this:
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)

# Create user grader
* Create a new user account named grader
- While logged in as ubuntu, add user: 
        `sudo adduser grader`
- Enter a password (twice) and fill out information for this new user.
* Give grader the permission to sudo
- Edits the sudoers file: sudo visudo.

- Search for the line that looks like this:

    root    ALL=(ALL:ALL) ALL
- Below this line, add a new line to give sudo privileges to grader user.

    root    ALL=(ALL:ALL) ALL
    grader  ALL=(ALL:ALL) ALL
- Save and exit using CTRL+X and confirm with Y.

- Verify that grader has sudo permissions. Run su - grader, enter the password.

# Create an SSH key pair for grader
* Configure key-based authentication for grader user

. create .ssh folder by mkdir /home/grader/.ssh
. Run this command cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
. change ownership `chown grader.grader /home/grader/.ssh`
. add 'grader' to sudo group `usermod -a G sudo grader`
. change permissions for .ssh folder `chmod 0700 /home/grader/.ssh/`, for authorized_keys: `chmod 644 authorized_keys`
. Check in vi /etc/ssh/sshd_config file if PermitRootLogin is set to no
. Restart SSH: `sudo service ssh restart`
. On the local machine, cheking if the grader account working or not by running this command :

            `ssh -i Catalogkey.pem -p 2200 grader@54.212.41.249`

# Configure local timezone to UTC(While logged in as grader)
1. Configure the timezone:  `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC

# Install and configure Apache to serve a Python mod_wsgi application(while logged in as grader)
. Install Apache  `sudo apt-get install apache2`
. Enter public IP of the Amazon EC2 instance into browser to Check Apache is working or not by executing public IP.
. My project is built with Python 3. So, I need to install the Python3 mod_wsgi package:
            `sudo apt-get install libapache2-mod-wsgi-py3`
. Enabled mod_wsgi with `sudo a2enmod wsgi`

# Install and configure PostgreSQL
. sudo apt-get install libpq-dev python-dev
. sudo apt-get install postgresql postgresql-contrib
. sudo su - postgres
. psql
. CREATE USER catalog WITH PASSWORD 'catalog';
. ALTER USER catalog CREATEDB;
. CREATE DATABASE itemcatalog WITH OWNER catalog;
. \c itemcatalog
. REVOKE ALL ON SCHEMA public FROM public;
. GRANT ALL ON SCHEMA public TO catalog;
. \q
. exit
. Switch back to the grader user: exit.

# Install git, clone and setup project(while logged in as grader)
. Install Git using: `sudo apt-get install git`
. Use cd /var/www to move to the /var/www directory
. Clone project 'catalog' from github
  `https://github.com/VasaviNaveena/catalog.git` 
. Create application directory  `sudo mkdir catalog`
. Move files in to catalog directory using: `mv !(catalog) catalog`
. Change the ownership of the catalog directory to grader using: `sudo chown -R grader:grader catalog/`
. Change to the /var/www/catalog/catalog directory.
. Rename the 'catalog.py' file to __init__.py using: `mv mainpage.py __init__.py`
. change the sqlite to postgresql create_engine in __init__.py,database_setup.py and populated_db.py.
. Search for create_engine in and keep it in comments
  '#engine = create_engine("sqlite:///itemcatalog.db")'
  And change the create_engine to the following:
 'engine = create_engine('postgresql://catalog:catalog@localhost/itemcatalog')'

# Authenticate login through Google
- In Google Developer's consoloe go to Google Cloud Platform'https://console.cloud.google.com/'.
- Click APIs & services on left menu.
- Click Credentials.
- Set the authorized JavaScript origins of the project to the IPv4 & Default DNS:
  http://54.212.41.249.xip.io & http://ec2-54-212-41-249.us-west-2.compute.amazonaws.com
- Set the redirect_uris to http://54.212.41.249.xip.io/login & http://54.212.41.249.xip.io/gconnect & 
  http://54.212.41.249.xip.io/callback
- Download the corresponding JSON file credentials, open it and copy the contents.
- updated the credentials in the project in 'G_client_secret.json' & in '/templates/login.html'

# Installation of virtual environment and dependencies(while logged in as grader)
. Install pip: `sudo apt-get install python-pip`
. Install the virtual environment: `sudo apt-get install python-virtualenv`
. Change to the /var/www/catalog/catalog/ directory.
. Create the virtual environment: `sudo virtualenv -p python3 venv3`
. Change the ownership to grader with: `sudo chown -R grader:grader venv3/`
. Activate the new environment: `. venv3/bin/activate`
. Installation of dependencies using the following commands:
    pip install httplib2
    pip install requests
    pip install --upgrade oauth2client
    pip install sqlalchemy
    pip install flask
    sudo apt-get install libpq-dev
    pip install psycopg2-binary


# Configure and Enable a Virtual Host
Creating the configuration file as: `sudo vi /etc/apache2/sites-available/catalog.conf` Add the following code to it:

	<VirtualHost *:80>
    ServerName 54.212.41.249.xip.io
    ServerAlias ec2-54-212-41-249.us-west-2.compute.amazonaws.com
    ServerAdmin ubuntu@54.212.41.249
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv3/lib/python3.6/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
 Run the following commands in order:
. To Disable default Virtual Host: `sudo a2dissite 000-default.conf`
. To Enable Virtual Host : `sudo a2ensite catalog.conf`
. To Reload Apache: `sudo service apache2 reload`

# Set up the Flask application & Adding wsgi file to the project
. Create /var/www/catalog/catalog.wsgi file add the following lines:

  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  from catalog import app as application
  application.secret_key = 'supersecretkey'

. Restart Apache: `sudo service apache2 restart`
. From the /var/www/catalog/catalog/ directory, activate the virtual environment: `. venv3/bin/activate`
. Run: python database_setup.py.
. Deactivate the virtual environment: deactivate.
. Reload Apache: `sudo service apache2 reload`
. Run: python catalog.wsgi

# Launch the Web Application
. Enable the virtual host `sudo a2ensite catalog.conf` 
. Restart Apache again `sudo service apache2 restart` 
. open browser to visit site at http://54.212.41.249 or http://ec2-54-212-41-249.us-west-2.compute.amazonaws.com .

# Useful commands
. To get log messages from Apache server: `sudo tail /var/log/apache2/error.log`
. To restart Apache: `sudo service apache2 restart`

Special Thanks to Anumsh for a very helpful README in Linux-Server-Configuration Project-Udacity.

