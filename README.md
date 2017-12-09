### Udacity FSND Project 7 - Linux Server Configuration 

### Description

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, 
to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

### Server information
- Amazon Lightsail - Ubuntu 16.04 LTS
- IP address: 34.216.126.206
- Accessible SSH port: 2200
- Application URL: http://ec2-34-216-126-206.us-west-2.compute.amazonaws.com

### Walkthrough steps

1) Create new user named grader and give permission to use sudo command
  - Using SSH connecting to server with SSH client: ssh -i ~/.ssh/udacity_key.rsa root@34.216.126.206
  - $ adduser grader with password = 'grader'
  - $ sudo nano /etc/sudoers.d/grader
  - Add the text "grader ALL=(ALL:ALL) ALL"
  - Save and exit /etc/sudoers.d/grader
   
2) Download and update all necessary Ubuntu packages 
  - $ apt-get update 
  - $ apt-get upgrade
  - $ apt-get dist-upgrade

3) Change SSH port from 22 to 2200
  - $ nano /etc/ssh/sshd_config
  - Change the port from 22 to 2200
  - Save and exit /etc/ssh/sshd_config
  - $ service ssh restart
  - Add custom port 2200 on Amazon Lightsail for server 
  - Verify port 22 no longer enabled with SSH client: ssh -i ~/.ssh/udacity_key.rsa root@34.216.126.206
  - Verify port 2200 change with SSH client: ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@34.216.126.206
  
4) Configure the Uncomplicated Firewall (UFW) for only SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - $ ufw allow 2200/tcp
  - $ ufw allow 80/tcp
  - $ ufw allow 123/udp
  - $ ufw enable
  
5) Configure the local timezone to UTC
  - $ dpkg-reconfigure tzdata 
  - Select UTC 
  - Select OK  
 
6) Configure public key authentication for grader user 
  - $ cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
  - $ chmod 700 home/grader/.ssh/
  - $ chmod 644 home/grader/.ssh/authorized_keys

7) Disable SSH access for root user
  - $ nano /etc/ssh/sshd_config
  - Change "PermitRootLogin without-password" to "PermitRootLogin no"
  - Save and exit /etc/ssh/sshd_config
  - $ service ssh restart
  - $ exit
  - Verify SSH root access diabled with SSH client: ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@34.216.126.206
  - SSH client: ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@34.216.126.206
 
8) Install Apache web server
  - $ sudo apt-get install apache2

9) Install mod-wsgi
  - $ sudo apt-get install libapache2-mod-wsgi python-dev
  - $ sudo a2enmod wsgi
  - $ sudo service apache2 start
  
10) Configure and enable a new virtual host
  - $ sudo nano /etc/apache2/sites-available/catalog.conf
  - Enter the following text block: 
    <VirtualHost *:80>
      ServerName 34.216.126.206
      ServerAlias ec2-34-216-126-206.us-west-2.compute.amazonaws.com
      ServerAdmin admin@34.216.126.206
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
  - Save and exit /etc/apache2/sites-available/catalog.conf
  - Enable the virtual host `sudo a2ensite catalog`

11) Clone the Catalog app from Github
  - $ sudo apt-get install git
  - $ cd /var/www
  - $ sudo mkdir catalog
  - $ sudo chown -R grader:grader catalog
  - $ cd /var/www/catalog
  - $ git clone https://github.com/mikefry/Udacity-Item-Catalog
  - $ cd /var/www/catalog/catalog
  - $ mv application.py __init__.py
  
12) Create catalog.wsgi file
  - $ cd /var/www/catalog/
  - $ sudo nano catalog.wsgi 
  - Enter the following text block:
  
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0, "/var/www/catalog/")
  
	from catalog import app as application
	application.secret_key = "The-secret-key"
	
  - Save and exit /etc/apache2/sites-available/catalog.conf	

13) Install virtual environment
  - $ sudo pip install virtualenv
  - $ sudo virtualenv venv
  - $ source venv/bin/activate
  - $ sudo chmod -R 777 venv

14) Install Python module dependencies
  - $ sudo apt-get install python-pip
  - $ sudo pip install psycopg2
  - $ sudo pip install flask
  - $ sudo pip install sqlalchemy 
  - $ sudo pip install httplib2
  - $ sudo pip install oauth2client

15) Update path of client_secrets.json file
  - $ sudo nano __init__.py
  - Change client_secrets.json path to "/var/www/catalog/catalog/client_secrets.json"
  
16) Install PostgreSQL and create catalog user and database
  - $ sudo apt-get install libpq-dev python-dev
  - $ sudo apt-get install postgresql postgresql-contrib
  - $ sudo su - postgres
  - $ psql
  - postgres=# CREATE USER catalog WITH PASSWORD 'password';
  - postgres=# ALTER USER catalog CREATEDB;
  - postgres=# CREATE DATABASE catalog WITH OWNER catalog;
  - postgres=# \c catalog
  - catalog=# REVOKE ALL ON SCHEMA public FROM public;
  - catalog=# GRANT ALL ON SCHEMA public TO catalog;
  - catalog=# \q
  - postgres=# exit
  
17) Change database engine in "__init__.py", "database_setup.py" and "lotsofitems.py"
  - $ sudo nano /var/www/catalog/catalog/__init__.py
  - Replace engine variable with "engine = create_engine('postgresql://catalog:password@localhost/catalog')"
  - Save and exit /var/www/catalog/catalog/__init__.py
  - $ sudo nano /var/www/catalog/catalog/database_setup.py
  - Replace engine variable with "engine = create_engine('postgresql://catalog:password@localhost/catalog')"
  - Save and exit /var/www/catalog/catalog/database_setup.py
  - $ sudo nano /var/www/catalog/catalog/lotsofitems.py
  - Replace engine variable with "engine = create_engine('postgresql://catalog:password@localhost/catalog')"
  - Save and exit /var/www/catalog/catalog/lotsofitems.py
  - $ python /var/www/catalog/catalog/database_setup.py
  - $ python /var/www/catalog/catalog/lotsofitems.py
    
18) Restart Apache 
  - $ sudo service apache2 restart
  
19) Verify any new Ubuntu packages
  - $ apt-get upgrade
  - $ apt-get dist-upgrade
  
20) Browse to project website: http://ec2-34-216-126-206.us-west-2.compute.amazonaws.com


### References

https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04
https://discussions.udacity.com/t/problems-with-the-digital-ocean-tutorial/336376/3
https://discussions.udacity.com/t/fsnd-connecting-to-aws-server-for-linux-project/387260
https://discussions.udacity.com/t/i-try-to-run-the-catalog-but-its-gave-me-error/362607/55
https://discussions.udacity.com/t/not-able-to-access-using-grader-user/478351
http://askubuntu.com/questions/323131/setting-timezone-from-terminal
http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/
https://httpd.apache.org/docs/current/mod/mod_alias.html#alias
https://manpages.debian.org/jessie/apache2/a2ensite.8.en.html
http://docs.sqlalchemy.org/en/latest/core/engines.html
