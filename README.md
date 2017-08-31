# udacity-linux-server-configuration

###  Description
 Linux server configuration

- IP address: 35.158.152.130

- Accessible SSH port: 2200

- Visit site at [http://35.158.152.130](http://35.158.152.130)

### Steps

1. Create new user named grader and give it the permission to sudo
  - Run ` sudo adduser grader` to create a new user grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - paste the following line `grader ALL=(ALL:ALL) ALL`
   
2. Update all currently installed packages
  - `sudo apt-get update`
  - `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  
4. Configure firewall for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw default deny incoming`
  - `sudo ufw default allow outcoming`
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable` you should see "Firewall is active and enabled on system startup" 
  
5. Configure timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC
 
6. Configure key-based authentication for grader user
  - Run this command in your local machine `ssh-keygen`
  - In your machine server : 
      - Run this command  `mkdir .ssh`
      - Run this command  `touch .ssh/authorized_keys`
      - Run this command to paste the generated key from your local machine to server  `nano .ssh/authorized_keys`

7. Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restart ssh  `sudo service ssh restart`
  - Now you are only able to login using `ssh grader@35.158.152.130 -p 2200  -i ~/.ssh/linuxconfig`
 
 
8. Install Apache and mod_wsgi
  - Install apache2 `sudo apt-get install apache2`
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - run  `sudo a2enmod wsgi`
  - start apache2 server `sudo service apache2 start`
  
9. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - `cd catalog`
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
  - Clone item catalog from github `git clone https://github.com/raymondjo/Item-Catalog-Application catalog`
  - Rename application.py to __init__.py `mv application.py __init__.py`
  
10. Install virtual environment,Flask and other dependencies :
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`
  - Install Flask `pip install requests`

11. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
  
12. Configure and enable a new virtual host :
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  <VirtualHost *:80>
    ServerName 35.158.152.130
    ServerAdmin ubuntu@35.158.152.130
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
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
  ```
  - Enable the virtual host `sudo a2ensite catalog`

13. Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - Change create engine line in your `__init__.py` and `database_setup.py` to: 
    `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - `python /var/www/catalog/catalog/database_setup.py`
 
14. Restart Apache 
  - `sudo service apache2 restart`
