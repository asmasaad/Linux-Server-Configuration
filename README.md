# Linux Server Configuration
this project is part of Full stack web devloper nano degree from udacity .we learn how to configure linux sever , secure it from number of attack , configure firewall , database server and host web Application 

**IP address**    35.156.55.253

**URL**     http://ec2-35-156-55-253.eu-central-1.compute.amazonaws.com

**SSH port**  2200


### step1
 -- create  Ubuntu Linux server instance from https://lightsail.aws.amazon.com 
 --  download private key from account page and put it on .ssh file 
-- type on terminal 
>chmod 600 mykey.pem

-- connect to the server
>  ssh -i ~/.ssh/mykey.pemubuntu@35.156.55.253

--  update packages
>sudo apt-get update

-- upgrade packages
>sudo apt-get upgrade

-- change port number from 22 to 2200 on sshd_config file 
> sudo nano /etc/ssh/sshd_config

--  Configure the Uncomplicated Firewall (UFW)

>sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable

 --go to your account and change firewall in networking page  
 -- configure the local timezone to UTC 
 > sudo dpkg-reconfigure tzdata 

--  restart service   
 > sudo service ssh restart

### step 2
--  Create new user named grader
>  sudo adduser  grader 

-- Give grader permission to sudo 
>**sudo nano /etc/sudoers.d/grader**
then add this line  and save the file 
>  **grader ALL=(ALL:ALL) NOPASSWD:ALL**

-- in the terminal of your local machine run this command to create ssh key pair 
> **ssh-keygen -f ~/.ssh/grader**
**cat ~/.ssh/grader.pub**
copy the public key in this file 

-- on your virtual machine:
>**su - grader**  
**sudo mkdir .ssh**  
**sudo touch .ssh/authorized_keys**  
-paste the public key in authorized_keys file and save it 
**sudo nano .ssh/authorized_keys**  
chane the permission of .SSH file 
**sudo chmod 700 .ssh**
**chmod 644 .ssh/authorized_keys**
restart SSH service
**service ssh restart** 

-- disable root login by changing PermitRootLogin  to **PermitRootLogin no** 
>sudo nano /etc/ssh/sshd_config


### step three 
-- install apache 
>sudo apt-get install apache2

-- configure apache mod_wsgi
>sudo apt-get install libapache2-mod-wsgi 

-- Enable mod_wsgi
> sudo a2enmod wsgi

-- install git 
>sudo apt-get install git

-- clone project from github 
> cd /var/www
sudo mkdir catalog
**cd catalog**
sudo git clone https://github.com/asmasaad/item-catalog   catalog

-- create file catalog.wsgi 
> **sudo nano /var/www/catalog/catalog.wsgi**
>and add this code 
>
``` python 
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key ='super_secret_key'
``` 
--rename project 
> sudo mv  project.py to __init__.py

-- change engine from sqlite to postegersql 
in __init.py , database_setup.py and  data.py 
```python
>engine = create_engine(**'postgresql://catalog:password@localhost/catalog'**)
``` 
-- update client_secrets.json path in __init.py 
```python 
CLIENT_ID = json.loads(
open(**'/var/www/catalog/catalog/client_secrets.json'**, **'r'**).read())[**'web'**][**'client_id'**]
``` 
--configure apache2
>sudo nano /etc/apache2/sites-available/catalog.conf


```python 
<VirtualHost *:80>
ServerName 35.156.55.253
ServerAlias 35.156.55.253.xip.io
ServerAdmin admin@35.156.55.253
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
-update main methode 
 ```python 
if __name__ == '__main__':
app.secret_key ='super_secret_key'
app.run()
```
--install pip 
>sudo apt-get install python-pip

-- install  virtual environment
>sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activat
sudo chmod -R 777 venv

--enable virtual host 
> sudo a2ensite catalog

-- Install  important packages

--setting up the database
>-  sudo apt-get install postgresql postgresql-contrib
 >- sudo -u postgres psql postgres
 >- CREATE USER catalog WITH PASSWORD 'password';
>- ALTER USER catalog CREATEDB;
>- CREATE DATABASE catalog WITH OWNER catalog;
>- \c catalog
>-   REVOKE ALL ON SCHEMA public FROM public;
>-   GRANT ALL ON SCHEMA public TO catalog;
>-   \q
>-   exit

--run database file 

>- python /var/www/catalog/catalog/database_setup.py
>- python /var/www/catalog/catalog/data.py


--restart apache
>-    sudo service apache2 restart
>-  sudo service apache2 reload

--upate and upgrade 
>- sudo apt-get update
>- sudo apt-get upgrade

### refrences 
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
