# Project 4: Linux Server Configuration

This is Python Flask CRUD Restaurant Menu App web runs live on Amazon Lightsail web server, Google Sign-In and Godaddy DNS.

### SSH Information and Application URL

- Server IP Address: 52.57.174.115
- SSH access port: 2200
- SSH login username: grader
- Application URL: http://www.somalishare.com




### 1. Creating RSA Key Pair

- Create lightsail virtual server instance
- Download the AWS provided default private key
- Change the private key file permission `chmod 600 LightsailDefaultKey-eu-central-1.pem`

### 2. Configuration SSH

- Logged in ssh ` ssh ubuntu@52.57.174.115 -p 22 -i C:/Users/Ismail/.ssh/LightsailDefaultKey-eu-central-1.pem`
- On your local machine, set up the public key pair

   ```Open Git Bash then type
   $ ssh-keygen
  ```
When it asks me to enter a passphrase, i entered 1433, but you can  leave it empty.


### 3. Updating the System

- sudo apt-get update
- sudo apt-get upgrade

### 4. Changing the SSH Port from 22 to 2200

- Open the `/etc/ssh/sshd_config`:
	
   ```
   # nano /etc/ssh/sshd_config
   # Port 22 change it to Port 2200
   # service ssh restart
   # ssh root@52.57.174.115 -p 2200
   ```

### 5. Configure Timezone to Use UTC

```
# sudo dpkg-reconfigure tzdata
```

### 6. Create user grader
- logged into the virtual server
- Create new user grader

	```
	# adduser grader
	```

- Adding `grader` to the Group `sudo`

	```
	# usermod -aG sudo grader
	```

### 7. Create an SSH key pair for grader

- On Git Bash, generate key pair using `ssh-keygen`

	```
    su - grader
    mkdir .ssh
    touch .ssh/authorized_keys
    nano .ssh/authorized_keys

    Copy the public key content from local machine to this file and save, change the access level
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
    ```
### 8. Setting Up the Firewall

```
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw deny 22
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status
```
### 9. Install and configure PostgreSQL

- `sudo apt-get install postgresql`
- Switching to postgres user
    
	```
    sudo su - postgres
    psql
    \q
    exit
    ```
- Create a new user named restaurant and new database named restaurant
    ```
    sudo su - postgres
    CREATE USER restaurant WITH PASSWORD '1433';
    \du
    CREATE DATABASE restaurant;
    \l
    ```

### 10. Installing Apache Web Server

```
sudo apt install apache2
```
- Then test URL http://52.26.30.44 

### 11. Cloning the project application

- Change the current working directory

   ```
   cd /var/www/
   ```
   
- Create a directory called `RestaurantApp`

   ```
   sudo mkdir RestaurantApp
   ```

- Change the working directory and clone your GitHub repository

	```
	cd RestaurantApp/
	sudo git clone https://github.com/icxuseen/RestaurantApp.git RestaurantApp
	```

### 12. Setting Up the VirtualHost Configuration

- To set up a file called `RestaurantApp.conf` to configure the virtual hosts:

   ```
   $ sudo nano /etc/apache2/sites-available/RestaurantApp.conf
   ```

- Add the following lines

   ```
        <VirtualHost *:80>
			ServerName 52.57.174.115
			ServerAlias www.somalishare.com
			ServerAdmin icxusen@gmail.com
			WSGIScriptAlias / /var/www/RestaurantApp/restaurantapp.wsgi
			<Directory /var/www/RestaurantApp/RestaurantApp/>
				Order allow,deny
				Allow from all
			</Directory>
			Alias /static /var/www/RestaurantApp/RestaurantApp/static
			<Directory /var/www/RestaurantApp/RestaurantApp/static/>
				Order allow,deny
				Allow from all
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

   ```
   
- Enable the virtual host:

   ```
   $ sudo a2ensite RestaurantApp
   ```

- Restart Apache server:

   ```
   $ sudo service apache2 restart
   ```

### 13. Create the .wsgi File

   Move to the `/var/www/RestaurantApp/` directory and create a file named `restaurantapp.wsgi` with following commands:

   ```
   $ cd /var/www/RestaurantApp/
   $ sudo nano restaurantapp.wsgi
   ```

```
#!/usr/bin/python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/RestaurantApp/")

from RestaurantApp import app as application
application.secret_key = 'super_secret_key'
```
	
- Restart Apache

	```
	sudo service apache2 restart
	```

### 14. Install required modules and run restaurantapp.py


```
sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo pip install Flask
pip install httplib2
sudo apt-get install python3-oauth2client
sudo apt-get install python3-requests
sudo apt-get install python-requests
sudo apt-get install  python3-sqlalchemy
sudo apt-get python3-psycopg2
```

### 15. Third-party resources 

- https://help.ubuntu.com
- https://www.postgresql.org/docs/current/static/sql-createuser.html</br>
- http://flask.pocoo.org/docs/0.12/

