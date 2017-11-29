# Project Overview

You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

# Create New User Name : GRADER
Log-in to server as ```ssh ubuntu@54.189.193.41 -i ~/.ssh/myudacitykey.pem ```

1. ```$ sudo adduser grader```

2. Give User permission to sudo: ```$ sudo nano /etc/sudoers.d/grader ```

	Add: grader ALL=(ALL) NOPASSWD:ALL
  
# Update all currently installed packages
1. ```$ sudo apt-get update ```

2. ```$ sudo apt-get upgrade```

# Change the SSH port from 22 to 2200
1. ```$ sudo nano /etc/ssh/sshd_config```. Edit Port Line from 22 to 2200.

2. Disable root login. Change to ‘no’ for PermitRootLogin

3. ```$ sudo service ssh restart.```

# Generate SSH-KEYGEN
1. To generate key pair, locally (not connect with server ssh yet) : 

	```ssh-keygen -f ~/.ssh/graderkey ```

2. Log in as Grader :```$ sudo su - grader```

3. Create directory: ```mkdir .ssh```

4. Create new file within above directory to store public keys : ```touch .ssh/authorized_keys```

5. Get key from local: ```cat .ssh/graderkey.pub```. Copy

6. Open file and paste key into:  ```nano .ssh/authorized_keys```

7. Set permission : ```chmod 700 .ssh```

8. And then ```chmod 644 .ssh/authorized_keys```

# Force all user log in using key pair:
1. Back to grader server: ```$ sudo nano /etc/ssh/sshd_config```

2. Look for this line : PasswordAuthentication. Change ‘yes’ to ‘no’. SAVE

3. Restart to run as new configuration:```$ sudo service ssh restart```

grader user now can log in as ```ssh grader@54.189.193.41 -p 2200 -i ~/.ssh/graderkey```

# Configure the Uncomplicated Firewall (UFW)
Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
1. $ sudo ufw default deny incoming

2. $ sudo ufw default allow outgoing

3. $ sudo ufw allow 2200/tcp

4. $ sudo ufw allow 80/tcp

5. $ sudo ufw allow 123/udp

6. $ sudo ufw enable. 
```

# Configure the local timezone to UTC

Run ```$ sudo dpkg-reconfigure tzdata``` and then choose  ```None of the above``` and choose ```UTC```.

# Install Apache:
```
1. $ sudo apt-get install apache2

2. $ sudo apt-get install libapache2-mod-wsgi python-dev

3. sudo a2enmod wsgi
```


# Install PostgreSQL

1. ```$ sudo apt-get install libpq-dev python-dev```

2. ```$ sudo apt-get install postgresql postgresql-contrib```

3. Login as user "postgres" ```$ sudo su - postgres```

4. Get into postgreSQL shell: ``psql``

5. ```CREATE USER catalog WITH PASSWORD 'udacity';```

6. ```ALTER USER catalog CREATEDB;```

7. ```CREATE DATABASE catalog WITH OWNER catalog;```

8. To connect to db:``` \c catalog```

9. ```REVOKE ALL ON SCHEMA public FROM public;```

10. ```GRANT ALL ON SCHEMA public TO catalog;```

11. To quit ```\q``` and ```CTRL + D```

# Clone the Catalog app
1. ```$ sudo apt-get install git```

2. ```cd /var/www```

3. ```$ sudo mkdir catalog```

4. ```$ sudo chown -R grader:grader catalog ```to change owner 

6. ```Cd catalog```

7. Clone project into above direction:
   ```git clone https://github.com/Rachelspdo/Build-Catalog.git catalog```

8. ```cd catalog```

9. Rename application.py to ```__init__.py``` using ```$ sudo mv application.py __init__.py```

10. Edit ```database_setup.py, lotsofmenus.py, __init__.py``` by ```$ sudo nano database_setup/lotsofmenus.py/__init__.py``` and change engine = create_engine('sqlite:///restaurantmenuwithusers.db') to engine = create_engine('postgresql://catalog:udacity@localhost/catalog')

11. ```python lotsofmenus.py``` to import sample data to site

# Setting up a virtual environment

1. ```$ sudo apt-get install python-pip```

2. Install the virtual environment: ```$ sudo pip install virtualenv```

3. Create a new virtual environment: ```$ sudo virtualenv venv```

4. To activate the virutal environment: ```source venv/bin/activate```

5. Change permissions to the virtual environment folder: ```$ sudo chmod -R 777 venv```

6. Install Flask: ```$ sudo pip install Flask```

7. Install other dependencies:
```
	sudo apt-get install python-psycopg2 python-flask
	
	sudo apt-get install python-sqlalchemy python-pip
	
	sudo pip install oauth2client
	
	sudo pip install requests
	
	sudo pip install httplib2
  ```
8. Create database: ```$ sudo python database_setup.py```

# Create .wsgi:
1. ```cd /var/www/catalog```

2. ```$ sudo nano catalog.wsgi```

	Add this to file:
  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")
  from catalog import app as application
  
  application.secret_key = "supersecretkey"
  ```
  
  # Configure and enable a new virtual host
  
1. Create a virtual host conifg file: ```$ sudo nano /etc/apache2/sites-available/catalog.conf```

2. Add to file: 
```
<VirtualHost *:80>
    ServerName 54.189.193.41
    ServerAlias ec2-54-189-193-41.us-west-2.compute.amazonaws.com
    ServerAdmin admin@54.189.193.41
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
3. Enable the virtual host with the following command:

  ```$ sudo a2ensite catalog```

# Update Google authentication

1. ```$ sudo nano client_secrets.json```

2. Add to Authorized Javascript Origin:

	```http://54.189.193.41```
	
	```http://ec2-189-193-41.us-west-2.compute.amazonaws.com```
	
3. Add Authorized Redirect URL

	```http://ec2-189-193-41.us-west-2.compute.amazonaws.com```
	
4. Go to ```console.developers.google.com``` and also add the above addresses to the OAuth Credencials

# Restart Apache

Restart Apache with the following command to apply the changes:

```$ sudo service apache2 restart```

# Disable default site:

```$ sudo a2dissite 000-default.conf ```

## Catalog Application will be avalable at:

http://54.189.193.41

http://ec2-189-193-41.us-west-2.compute.amazonaws.com



