# Linux-Server-Configuration


### Project Description

installation of a Linux distribution on a virtual machine and prepare it to host your web applications

- IP address: 35.184.63.42

- SSH port: 2200

- Application URL: http://catalogsalah.tk/

### Walkthrough
 Update all currently installed packages
  `sudo apt-get update`
  `sudo apt-get upgrade`

1. Create new user named grader and give it the permission to sudo
  - access the server through gcloud 
  - `' sudo adduser grade'  create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`
   


2. Change SSH port from 22 to 2200 ,Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`

3. Configure the Uncomplicated Firewall 
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
  
4. Configure the local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC

5. key generation
 - Run 'ssh-keygen' in pc terminal 
 - file .pub copy content
 
6. Configure key-based authentication for grader user
  - Run this command `sudo nano /home/grader/.ssh/authorized_keys`
  - paste the copied content 

7. Disable ssh login for root user
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh grader@35.184.63.42 -p 2200 -i id_rsa`
 
8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

  
10. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - Clone your project from github `git clone https://github.com/ahmedsaalah/Item-Catalog.git catalog`
  - Create a start.wsgi file, then add this inside:
  ```
  
import sys
sys.path.insert(0, "/var/www/catalog/catalog")
from __init__  import app as application
  ```
  - Rename application.py to __init__.py `mv project.py __init__.py`
  
  
11. Install virtual environment
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

12. Install Flask and other dependencies
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - Install other project dependencies sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf


13. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
  
14. Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  
<VirtualHost *:80>
    WSGIDaemonProcess catalog user=www-data group=www-data threads=5
    WSGIScriptAlias / /var/www/catalog/start.wsgi
    <Directory /var/www/catalog>
        Require all granted
    </Directory>
    alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Require all granted
    </Directory>
</VirtualHost>

  ```
  - Enable the virtual host `sudo a2ensite catalog`

15. Install and configure PostgreSQL
$ sudo -su postgres
$psql
>CREATE USER graderdb WITH PASSWORD 'pasword';
>CREATE DATABASE catalogdb;

>/q
$exit
  - Change create engine line in your `__init__.py` ,'database_seed.py'and `database_setup.py` to: 
 ' url = "postgresql://graderdb:pasword@localhost:5432/catalogdb" '
  `engine = create_engine(url)`
  - `python /var/www/catalog/catalog/database_setup.py`
   - `python /var/www/catalog/catalog/database_seed.py`

16. Restart Apache 
  - `sudo service apache2 restart`

17.Update the Google OAuth client secrets file ,Update the facebook OAuth client secrets file
  
18. Visit site at [http://catalogsalah.tk/]

