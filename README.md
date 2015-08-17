#Linux-Server-Configuration

A baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.
Visit the site at http://54.149.46.103/
- **PUBLIC IP** -->  **54.149.46.103**
- **APPLICATION URL** --> **http://54.149.46.103/**

![Got-Room Web Application](/uni.jpg "http://54.149.46.103/")



###User Management: Create a new user and give user the permission to sudo
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS")

1. Create a new user:  
  `$ adduser NEWUSER (grader)`
2. Install finger , sudo apt-get install finger
  verify user has been added by **finger NEWUSER**

#### Grant NEWUSER (grader) permission to sudo
1. access sudoers.d by **cd /etc/sudoers.d**
2. create new file as grader **touch grader**
3. add the following and save --> **grader ALL=(ALL) NOPASSWD:ALL**
4. log onto the server with user grader



###Update and upgrade all currently installed packages

Reference: [Ask Ubuntu](http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade "What is the difference between apt-get update and upgrade?") 
    
1. Update the list of available packages and their versions:  
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:  
  `$ sudo sudo apt-get upgrade`


###Change the SSH port from 22 to 2200 and configure SSH access
Reference: [Ask Ubuntu](http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server "Create a new SSH user on Ubuntu Server")  

1. Change ssh config file:
  1. Open the config file:  
    `$ nano /etc/ssh/sshd_config` 
  2. Change to Port 2200 and **service ssh restart**

###Configure Uncomplicated Firewall (UFW)
Reference: [Ubuntu documentation](https://help.ubuntu.com/community/UFW "UFW - Uncomplicated Firewall") 

1. Turn UFW on with the default set of rules:  
  `$ sudo ufw enable` 
2. *Check the status of UFW:  
  `$ sudo ufw status verbose`
3. Allow incoming TCP packets on port 2200 (SSH):  
  `$ sudo ufw allow 2200/tcp` 
4. Allow incoming TCP packets on port 80 (HTTP):  
  `$ sudo ufw allow 80/tcp` 
5. Allow incoming UDP packets on port 123 (NTP):  
  `$ sudo ufw allow 123/udp` 

#### Allow grader to ssh into the server 
1. Generate new RSA key on local computer by --> **ssh-keygen**
2. save the 2 files to home folder location in a folder called .ssh (one will have .pub as extension)
3. On the Ubuntu server access grader home directory
4. create .ssh directory --> **mkdir .ssh**
5. create new file --> **touch .ssh/authorized keys**
6. copy the contents of the file with **.pub** extension from the local computer into the authorized_keys file on the 
ubuntu server and save the file 
7. set permissions as **chmod 700 .ssh** and **chmod 644 .ssh/authorized_keys**
8. Now grader can ssh as ssh -i ~/.ssh/NewRSAFile grader@54.149.46.103 -p 2200


#### Enforce ssh login only  (Forcing Key-based Authentication)
1. access **sshd_config** file --> **sudo nano /etc/ssh/sshd_config**
2. Set **PasswordAuthentication** to **no**

#### Disable Root login for secuity purposes
1. access **sshd_config** file --> **sudo nano /etc/ssh/sshd_config**
2. Set **PasswordRootLogin** to **no**


### Configure the local timezone to UTC
Reference: [Ubuntu documentation](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29 "Ubuntu Time Management")

1. Open the timezone selection dialog:  
  `$ sudo dpkg-reconfigure tzdata`
2. Then chose 'None of the above', then UTC.
3. *Setup for time synchroniztion:  
  `$ sudo apt-get install ntp`
4. *Chose closer NTP time servers:  
  1. Open the NTP configuration file:  
    `$ sudo nano /etc/ntp.conf`
  2. Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list.  


### Install and configure Apache to serve a Python mod_wsgi application
Reference: [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html "A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu")

1. Install Apache web server:  
  `$ sudo apt-get install apache2`\
2. Install **mod_wsgi** for serving Python apps from Apache and the helper package **python-setuptools**:  
  `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart the Apache server for mod_wsgi to load:  
  `$ sudo service apache2 restart`  
4. *Get rid of the message "Could not reliably determine the servers's fully qualified domain name" after restart
  Reference: [Ask Ubuntu](http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name "Could not reliably determine the server's fully qualified domain name?")
  1. Create an empty Apache config file with the hostname:  
    `$ echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/fqdn.conf`
  2. Enable the new config file:  
    `$ sudo a2enconf fqdn`

###  Install and configure git
Reference: [GitHub](https://help.github.com/articles/set-up-git/#platform-linux "Set Up Git for Linux")
        
1. Install Git:  
  `$ sudo apt-get install git`
2. Set your name, e.g. for the commits:  
  `$ git config --global user.name "YOUR NAME"`
3. Set up your email address to connect your commits to your account:  
  `$ git config --global user.email "YOUR EMAIL ADDRESS"`

### Setup for deploying a Flask Application on Ubuntu VPS
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "How To Deploy a Flask Application on an Ubuntu VPS")

1. Extend Python with additional packages that enable Apache to serve Flask applications:  
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi (if not already enabled):  
  `$ sudo a2enmod wsgi`
3. Create a Flask app:  
  1. Move to the www directory:  
    `$ cd /var/www`
  2. Setup a directory for the app, e.g. GotRoom:  
    1. `$ sudo mkdir GotRoom`  
    2. `$ cd GotRoom` and `$ sudo mkdir GotRoom`  
    3. `$ cd GotRoom` and `$ sudo mkdir static templates`  
    4. Create the file that will contain the flask application logic:  
      `$ sudo nano __init__.py`
    5. Paste in the following code:  
    ```python  
      from flask import Flask  
      app = Flask(__name__)  
      @app.route("/")  
      def hello():  
        return "It works yayness"  
      if __name__ == "__main__":  
        app.run()  
    ```  
4. Install Flask
  1. Install pip installer:  
    `$ sudo apt-get install python-pip` 
  2. Install virtualenv:  
    `$ sudo pip install virtualenv`
  3. Set virtual environment to name 'venv':  
    `$ sudo virtualenv venv`
  4. Enable all permissions for the new virtual environment (no sudo should be used within):         
    `$ sudo chmod -R 777 venv`
  5. Activate the virtual environment:  
    `$ Reference venv/bin/activate`
  6. Install Flask inside the virtual environment:  
    `$ pip install Flask`
  7. Run the app:  
    `$ python __init__.py`
  8. Deactivate the environment:  
    `$ deactivate`
5. Configure and Enable a New Virtual Host#
  1. Create a virtual host config file  
    `$ sudo nano /etc/apache2/sites-available/GotRoom.conf`
  2. Paste in the following lines of code and change names and addresses regarding your application:  
  ```
    <VirtualHost *:80>
        ServerName PUBLIC-IP-ADDRESS
        ServerAdmin admin@PUBLIC-IP-ADDRESS
        WSGIScriptAlias / /var/www/GotRoom/GotRoom.wsgi
        <Directory /var/www/GotRoom/GotRoom/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/GotRoom/GotRoom/static
        <Directory /var/www/GotRoom/GotRoom/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
  ```
  3. Enable the virtual host:  
    `$ sudo a2ensite GotRoom`
6. Create the .wsgi File and Restart Apache
  1. Create wsgi file:  
    `$ cd /var/www/GotRoom` and `$ sudo nano GotRoom.wsgi`
  2. Paste in the following lines of code:  
  ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/GotRoom/")
    
    from GotRoom import app as application
    application.secret_key = 'Add your secret key'
  ```
  7. Restart Apache:  
    `$ sudo service apache2 restart`


### Clone GitHub Application repository and make it web inaccessible

#####Clone project 3 solution repository on GitHub, the branch using PostgreSQL:

1. Clone project 3 solution repository on GitHub:  
  `$ git clone -b PostgreSQL https://github.com/CruzanCaramele/Got-Room.git`
2. Move all content of created GotRoom directory to `/var/www/GotRoom/GotRoom/`-directory and delete the leftover empty directory.
3. Make the GitHub repository inaccessible:  
  Reference: [Stackoverflow](http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible "Make .git directory web inaccessible")
  1. Create and open .htaccess file:  
    `$ cd /var/www/GotRoom/` and `$ sudo nano .htaccess` 
  2. Paste in the following:  
    `RedirectMatch 404 /\.git`

###### Install needed modules & packages
pip install -r requirements.txt

###  Install and configure PostgreSQL
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps "How To Secure PostgreSQL on an Ubuntu VPS")

1. Install PostgreSQL:  
  `$ sudo apt-get install postgresql postgresql-contrib`
2. Check that no remote connections are allowed (default):  
  `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
3. Open the database setup file:  
  `$ sudo nano database_setup.py`
4. Change the line starting with "engine" to (fill in a password):  
  ```python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')```  
5. Change the same line in application.py respectively
6. Rename application.py:  
  `$ mv application.py __init__.py`
7. Create needed linux user for psql:  
  `$ sudo adduser catalog` (choose a password)
8. Change to default user postgres:  
  `$ sudo su - postgre`
9. Connect to the system:  
  `$ psql`
10. Add postgre user with password:  
  References: [Trackets Blog](http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html "PostgreSQL Basics by Example")
  1. Create user with LOGIN role and set a password:  
    `# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB';` (# stands for the command prompt in psql)
  2. Allow the user to create database tables:  
    `# ALTER USER catalog CREATEDB;`
  3. *List current roles and their attributes:
    `# \du`
11. Create database:  
  `# CREATE DATABASE catalog WITH OWNER catalog;`
12. Connect to the database catalog
  `# \c catalog` 
13. Revoke all rights:  
  `# REVOKE ALL ON SCHEMA public FROM public;`
14. Grant only access to the catalog role:  
  `# GRANT ALL ON SCHEMA public TO catalog;`
15. Exit out of PostgreSQl and the postgres user:  
  `# \q`, then `$ exit` 
16. Create postgreSQL database schema:  
  $ python database_setup.py

### Run application

1. Restart Apache:  
  `$ sudo service apache2 restart`
2. Open a browser and put in your public ip-address as url, e.g. http://54.149.46.103/ - if everything works, the application should come up
3. *If getting an internal server error, check the Apache error files:  
   
  1. View the last 30 lines in the error log: 
    `$ sudo tail -30 /var/log/apache2/error.log`
  2. *If a file like 'g_client_secrets.json' couldn't been found:  
    Reference: [Stackoverflow](http://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory "Python: No such file or directory")  


## Exceeds Specification Requirements Below:

### Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04 "How To Install and Use Fail2ban on Ubuntu 14.04")

1. Install Fail2ban:  
  `$ sudo apt-get install fail2ban`
2. Copy the default config file:  
  `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
3. Check and change the default parameters:  
    1. Open the local config file:  
      `$ sudo nano /etc/fail2ban/jail.local`
    2. Set the following Parameters:  
    ```  
      set bantime  = 2000  
      destemail = YOURNAME@DOMAIN  
      action = %(action_mwl)s  
      under [ssh] change port = 2220  
    ```    
4. *Check the current firewall rules:  
  `$ sudo iptables -S`
5.   `$ sudo apt-get install sendmail iptables-persistent`  and Stop the service:  
  `$ sudo service fail2ban stop`
6. Start it again:  
  `$ sudo service fail2ban start`


### Install Monitor application Glances

References: [Web Host Bug](http://www.webhostbug.com/install-use-glances-ubuntudebian/ "How to install and use Glances on Ubuntu/Debian")

1. `$ sudo apt-get install python-pip build-essential python-dev`
2. `$ sudo pip install Glances`
3. `$ sudo pip install PySensors`

**Incase of error in installing PySensors** :
- install **lm-sensors** first by **sudo apt-get install lm-sensors**, then install PySensors


![Glances of the Server](/glances.jpg)


### Include  automatic manage package updates
Reference: [Ubuntu documentation](https://help.ubuntu.com/14.04/serverguide/automatic-updates.html) 

1. Install the unattended-upgrades package:  
  `$ sudo apt-get install unattended-upgrades`
2. Enable the unattended-upgrades package:  
  `$ sudo dpkg-reconfigure -plow unattended-upgrades`
3. email an administrator information about any packages on the system that have updates available:  
  `$ sudo apt-get install apticron`
4. Change Email in /etc/apticron/apticron.conf to :  
  `EMAIL="root@example.com`
