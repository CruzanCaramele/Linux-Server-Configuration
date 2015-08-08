#Linux-Server-Configuration

A baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.

##User Management: Create a new user and give user the permission to sudo
Source: DigitalOcean

Create a new user:
$ adduser NEWUSER
Give new user the permission to sudo
Open the sudo configuration:
$ visudo
Add the following line below root ALL...:
NEWUSER ALL=(ALL:ALL) ALL
*List all users (Source: Ask Ubuntu):
$ cut -d: -f1 /etc/passwd

##Update and upgrade all currently installed packages

Source: Ask Ubuntu

Update the list of available packages and their versions:
$ sudo apt-get update
Install newer vesions of packages you have:
$ sudo sudo apt-get upgrade

##Change the SSH port from 22 to 2200 and configure SSH access
Source: Ask Ubuntu

Change ssh config file:
Open the config file:
$ vim /etc/ssh/sshd_config
Change to Port 2200.
Change PermitRootLogin from without-password to no.
* To get more detailed logging messasges, open /var/log/auth.log and change LogLevel from INFO to VERBOSE.
Temporalily change PasswordAuthentication from no to yes.
Append UseDNS no.
Append AllowUsers NEWUSER.
Note: All options on UNIXhelp

##Configure Uncomplicated Firewall (UFW)
Source: Ubuntu documentation

Turn UFW on with the default set of rules:
$ sudo ufw enable
*Check the status of UFW:
$ sudo ufw status verbose
Allow incoming TCP packets on port 2200 (SSH):
$ sudo ufw allow 2200/tcp
Allow incoming TCP packets on port 80 (HTTP):
$ sudo ufw allow 80/tcp
Allow incoming UDP packets on port 123 (NTP):
$ sudo ufw allow 123/udp


## Configure the local timezone to UTC
Source: Ubuntu documentation

Open the timezone selection dialog:
$ sudo dpkg-reconfigure tzdata
Then chose 'None of the above', then UTC.
*Setup the ntp daemon ntpd for regular and improving time sync:
$ sudo apt-get install ntp
*Chose closer NTP time servers:
Open the NTP configuration file:
$ sudo vim /etc/ntp.conf
Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list.


## Install and configure Apache to serve a Python mod_wsgi application
Source: Udacity

Install Apache web server:
$ sudo apt-get install apache2
Open a browser and open your public ip address, e.g. http://52.25.0.41/ - It should say 'It works!' on the top of the page.
Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
Restart the Apache server for mod_wsgi to load:
$ sudo service apache2 restart
*Get rid of the message "Could not reliably determine the servers's fully qualified domain name" after restart Source: Ask Ubuntu
Create an empty Apache config file with the hostname:
$ echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf
Enable the new config file:
$ sudo a2enconf fqdn

##  Install and configure git
Source: GitHub

Install Git:
$ sudo apt-get install git
Set your name, e.g. for the commits:
$ git config --global user.name "YOUR NAME"
Set up your email address to connect your commits to your account:
$ git config --global user.email "YOUR EMAIL ADDRESS"

## Setup for deploying a Flask Application on Ubuntu VPS
Extend Python with additional packages that enable Apache to serve Flask applications:
$ sudo apt-get install libapache2-mod-wsgi python-dev
Enable mod_wsgi (if not already enabled):
$ sudo a2enmod wsgi
Create a Flask app:
Move to the www directory:
$ cd /var/www
Setup a directory for the app, e.g. catalog:
$ sudo mkdir catalog
$ cd catalog and $ sudo mkdir catalog
$ cd catalog and $ sudo mkdir static templates
Create the file that will contain the flask application logic:
$ sudo nano __init__.py
Paste in the following code:
  from flask import Flask  
  app = Flask(__name__)  
  @app.route("/")  
  def hello():  
    return "Veni vidi vici!!"  
  if __name__ == "__main__":  
    app.run()  
Install Flask
Install pip installer:
$ sudo apt-get install python-pip
Install virtualenv:
$ sudo pip install virtualenv
Set virtual environment to name 'venv':
$ sudo virtualenv venv
Enable all permissions for the new virtual environment (no sudo should be used within):
Source: Stackoverflow
$ sudo chmod -R 777 venv
Activate the virtual environment:
$ source venv/bin/activate
Install Flask inside the virtual environment:
$ pip install Flask
Run the app:
$ python __init__.py
Deactivate the environment:
$ deactivate
Configure and Enable a New Virtual Host#
Create a virtual host config file
$ sudo nano /etc/apache2/sites-available/catalog.conf
Paste in the following lines of code and change names and addresses regarding your application:
  <VirtualHost *:80>
      ServerName PUBLIC-IP-ADDRESS
      ServerAdmin admin@PUBLIC-IP-ADDRESS
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
Enable the virtual host:
$ sudo a2ensite catalog
Create the .wsgi File and Restart Apache

Create wsgi file:
$ cd /var/www/catalog and $ sudo vim catalog.wsgi
Paste in the following lines of code:
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'Add your secret key'
Restart Apache:
$ sudo service apache2 restart


## Clone GitHub Application repository and make it web inaccessible

Clone project 3 solution repository on GitHub:
$ git clone https://github.com/CruzanCaramele/Got-Room.git
Move all content of created Got Room  directory to /var/www/catalog/catalog/-directory and delete the leftover empty directory.
Make the GitHub repository inaccessible:
Source: Stackoverflow
Create and open .htaccess file:
$ cd /var/www/catalog/ and $ sudo vim .htaccess
Paste in the following:
RedirectMatch 404 /\.git

##  Install and configure PostgreSQL
Source: DigitalOcean (alternatively, nice short guide on Kill The Yak as well)

Install PostgreSQL:
$ sudo apt-get install postgresql postgresql-contrib
Check that no remote connections are allowed (default):
$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
Open the database setup file:
$ sudo vim database_setup.py
Change the line starting with "engine" to (fill in a password):
python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')
Change the same line in application.py respectively
Rename application.py:
$ mv application.py __init__.py
Create needed linux user for psql:
$ sudo adduser catalog (choose a password)
Change to default user postgres:
$ sudo su - postgre
Connect to the system:
$ psql
Add postgre user with password:
Sources: Trackets Blog and Super User
Create user with LOGIN role and set a password:
# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB'; (# stands for the command prompt in psql)
Allow the user to create database tables:
# ALTER USER catalog CREATEDB;
*List current roles and their attributes: # \du
Create database:
# CREATE DATABASE catalog WITH OWNER catalog;
Connect to the database catalog # \c catalog
Revoke all rights:
# REVOKE ALL ON SCHEMA public FROM public;
Grant only access to the catalog role:
# GRANT ALL ON SCHEMA public TO catalog;
Exit out of PostgreSQl and the postgres user:
# \q, then $ exit
Create postgreSQL database schema:
$ python database_setup.py

## Run application

Restart Apache:
$ sudo service apache2 restart
Open a browser and put in your public ip-address as url, e.g. http://54.149.38.242 - if everything works, the application should come up


## Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers
Source: DigitalOcean

Install Fail2ban:
$ sudo apt-get install fail2ban
Copy the default config file:
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
Check and change the default parameters:

Open the local config file:
$ sudo vim /etc/fail2ban/jail.local
Set the following Parameters:
  set bantime  = 1800  
  destemail = YOURNAME@DOMAIN  
  action = %(action_mwl)s  
  under [ssh] change port = 2220 
.

Install needed software for our configuration:
$ sudo apt-get install sendmail iptables-persistent

*Check the current firewall rules:
$ sudo iptables -S
Stop the service:
$ sudo service fail2ban stop
Start it again:
$ sudo service fail2ban start


## Install Monitor application Glances

Sources: Glances and Web Host Bug

$ sudo apt-get install python-pip build-essential python-dev
$ sudo pip install Glances
$ sudo pip install PySensors

**Incase of error in installing PySensors** :
- install **lm-sensors** first by **sudo apt-get install lm-sensors**, then install PySensors
