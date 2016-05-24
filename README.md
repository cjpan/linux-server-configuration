# Linux-Server-Configuration
Udacity FSND Project5

## Description
> You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application deployed is the **Catalog**, previously developed for [Project 3](https://github.com/cjpan/fullstack-nanodegree-vm/tree/master/vagrant/catalog).

## Overview info

IP address: 52.38.19.193  

Accessible SSH port: 2200  

Application URL: [http://ec2-52-38-19-193.us-west-2.compute.amazonaws.com/](http://ec2-52-38-19-193.us-west-2.compute.amazonaws.com/)  

## Configuration  

### 1 - SSH access to the Udacity Development Environment  

Launch personal Virtual Machine with personal Udacity Account  
1. Download private key
2. Move the private key file into the folder ~/.ssh (where ~ is the environment's home directory)

        mv ~/Downloads/udacity_key.rsa ~/.ssh/

3. Open terminal and change private key file's permission, only for the owner read and write

        chmod 600 ~/.ssh/udacity_key.rsa

4. Log into the Development environment

        ssh -i ~/.ssh/udacity_key.rsa root@52.38.19.193

#### Reference:
[Udacity](https://www.udacity.com/account#!/development_environment)  

### 2 - User Management & Security  
1. Create a new user name **grader**  
  ```
    adduser grader
  ```  
2. Add **grader** to sudoers  
Create a new file called 'grader' in /etc/sudoers.d/  
```
    vim /etc/sudoers.d/grader
```  
In the file put in: `grader ALL=(ALL) NOPASSWD:ALL` Save and quit with `wq`.  
3. Configure to use ssh key for grader to log in.  
  In the Develop Environment, create a new folder `.ssh` in `/home/grader/`  
```
    mkdir /home/grader/.ssh
```  
Create a new file `authorized_keys` to store public key  
Create a ssh key for grader to log in own computer terminal, generate key pairs locally  
```
    ssh-keygen
```  
Save the key in `uda_id_rsa` file on local computer.  
Copy the content of public key file `uda_id_rsa.pub` to `/home/grader/.ssh/authorized_keys` file.  
Change permision of the directory/file  
```
    chmod 700 /home/grader/.ssh
    chmod 644 /home/grader/.ssh/authorized_keys
```
Change owner of the directory/file to `grader`  
```
    chown grader /home/grader/.ssh
    chown grader /home/grader/.ssh/authorized_keys
```  
Change group of the directory/file to `grader`  
```
    chgrp grader /home/grader/.ssh
    chgrp grader /home/grader/.ssh/authorized_keys
```  
4. Change the SSH port from 22 to 2200  
Edit configue file  
```
		vim /etc/ssh/sshd-config
```  
In hte file, change the `Port 22` to  `Port 2200`  
Restart ssh service  
```
		service sshd restart
```  
5. log in with `grader`and ssh method  
```
    ssh -i ~/.ssh/uda_id_rsa grader@52.38.19.193 -p 2200
```  
#### Reference:
[Udacity Class](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089477)  

### 3 - Update all currently installed packages  
1. Update package source list:  
```
    sudo apt-get update
```  

2. Upgrade all the application:  
```
    sudo apt-get upgrade
```  

### 4 - Configure the local timezone to UTC  
Open time configuration dialog and set it to UTC  
```
    sudo dpkg-reconfigure tzdata
```  

#### Reference:
 [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)  

### 5 - Configure the Uncomplicated Firewall(UFW)  
1. Disable all incoming connections and allow all outgoing connections  
```
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
```  

2. Allow incoming connections for SSH (port 2200)  
```
		sudo ufw allow 2200/tcp
```  

3. Allow incoming connections for HTTP (port 80)  
```
		sudo ufw allow www
```  

4. Allow incoming connections for NTP (port 123)  
```
		sudo ufw allow ntp
```  

5. Eable the ufw  
```
		sudo ufw enable
```  

#### Reference:  
[Udacity Class](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089477)  

### 6 - Install Apache, mod_wsgi application, Git  
1. Install  Apache  
```
		sudo apt-get install apache2
```  

2. Install mod_wsgi  
```
		sudo apt-get install libapache2-mod-wsgi
```  

3. Install git  
```
		sudo apt-get install git
```  

### 7 - Configure Apache to serve a Python mod_wsgi application  
1. Clone **Catalog** repo from Github in home direcory:  
```
		git clone https://github.com/cjpan/fullstack-nanodegree-vm.git
```  

link the repo in `/var/www/`  
```
    sudo ln -s ~/fullstack-nanodegree-vm /var/www/CatalogApp
```  

2. To make .git directory is not publicly accessible via a browser. Create a .htaccess file in the .git folder and put the following in this file:  
```
		RedirectMatch 404 /\.git
```  

3. Make the `project.py` file name to `__init__.py`    

4. Install pip , virtualenv (in /var/www/Catalog)  
```
		sudo apt-get install python-pip
```  

using pip install virtualenv, and enable virtualenv  
```
		sudo pip install virtualenv   
		sudo virtualenv venv  
		source venv/bin/activate  
```  

5. Using pip install following nessary packages  
```  
		sudo pip install Flask  
		sudo pip install oauth2client  
		sudo pip install requests  
		sudo pip install httplib2  
    sudo pip install flask-httpauth  
```  

6. Configure and Enable a New Virtual Host  
```  
		sudo vim /etc/apache2/sites-available/Catalog.conf  
```  

Add these content:  
```  
	<VirtualHost *:80>  
        ServerName 52.38.19.193  
        ServerAdmin webmaster@localhost  
        ServerAlias ec2-52-38-19-193.us-west-2.compute.amazonaws.com  
        ErrorLog ${APACHE_LOG_DIR}/error.log  
        CustomLog ${APACHE_LOG_DIR}/access.log combined  
        #Include conf-available/serve-cgi-bin.conf  
        WSGIScriptAlias / /var/www/CatalogApp/vagrant/catalog.wsgi  
        <Directory /var/www/CatalogApp/vagrant/Catalog/>  
            Order allow,deny  
            Allow from all  
        </Directory>  
        Alias /static /var/www/CatalogApp/vagrant/Catalog/static  
        <Directory /var/www/CatalogApp/vagrant/Catalog/static/>  
            Order allow,deny  
            Allow from all  
        </Directory>  
		</VirtualHost>  
```  

Enble  virtual host:  
```  
		sudo a2ensite Catalog  
```  

7. Create the .wsgi File  
Create a file named `catalog.wsgi` with following commands:  
```  
		cd /var/www/CatalogApp/vagrant/  
		sudo vim catalog.wsgi  
```  

Add these content:  
```  
		#!/usr/bin/python  
		import sys  
		import logging  
		logging.basicConfig(stream=sys.stderr)  
		sys.path.insert(0,"/var/www/CatalogApp/vagrant/")  

		from Catalog import app as application  
		application.secret_key = 'super_secret_key'  
    application.debut = True  
```  

Right now my directory tree(In the /var/www/):  
```  
		CatalogApp  
		--vagrant  
		----catalog.wsgi  
		----Catalog  
		------static  
		------__init__.py  
```  

8. Some changes in python file:  
	- In the `__init__.py` change the client_secrets.json to absolute path  

  - In Google developer console, add javascript origins  `http://ec2-52-38-19-193.us-west-2.compute.amazonaws.com/` and `http://52.38.19.193`  

  - In Google developer console, add oauth2callback `http://ec2-52-38-19-193.us-west-2.compute.amazonaws.com/oauth2callback`  

#### Reference:  
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)   
[Udacity discussion](https://discussions.udacity.com/t/cant-find-image/43142/2)   

### 8 - Install and configure PostgreSQL  
1. Install PostgreSQL  
```  
		sudo apt-get install postgresql postgresql-contrib  
```  

2. Do not allow remote connections  
	in the `/etc/postgresql/9.1/main/pg_hba.conf` configure file, the default setting has already disabled remote connections  

3. Add a new user named catalog  
```  
		sudo adduser catalog  
```  

4. Log into PostgreSQL  
```  
		sudo su - postgres  
		psql  
```  

5. Create a new role and grant the role to create database  
```  
		CREATE ROLE catalog WITH CREATEDB;  
		ALTER USER catalog WITH PASSWORD pass;  
    ALTER USER catalog CREATEDB;  
```  

using `\du` can see the roles list  

6. Create database  
```  
		CREATE DATABASE catalog WITH OWNER catalog;  
```  

7.  Connect to the database and lock down the permissions to only let "access_role" create tables  
```  
		\c catalog  
		REVOKE ALL ON SCHEMA public FROM public;  
		GRANT ALL ON SCHEMA public TO access_role;  
```  

Finally press `Control`+`D` to quit  

#### Reference:  
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)   
[Udacity discussion](https://discussions.udacity.com/t/using-postgresql-instead-of-sqlite/13575/11)  

### 9 - Make the app run  
1. Go to the Catalog app directory  
```  
		cd /var/www/CatalogApp/vagrant/  
```  

2. Edit the `__init__.py`, `lotsofitems.py` and `database_setup.py` file:  
	Change `engine = create_engine('sqlite:///categoryitems.db')` to `engine = create_engine('postgresql://catalog:pass@localhost/catalog')`  

3. Install python-psycopg2  
```  
    sudo apt-get install psysopg2  
```  

4. Restart Apache  
```  
		sudo service apache2 restart  
```  

Now can visit the page thourgh <http://ec2-52-38-19-193.us-west-2.compute.amazonaws.com/>.  

Refer to error log (`/var/log/apache2/`) for any error in running.  
