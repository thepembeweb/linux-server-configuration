# Linux Server Configuration

> This project takes a baseline installation of a Linux server and prepares it to host a web application. The project also shows how to secure a server from a number of attack vectors, how to install and configure a database server, and how to deploy an existing web application onto it.

![](https://upload.wikimedia.org/wikipedia/commons/f/f8/Python_logo_and_wordmark.svg)
![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Postgresql_elephant.svg/200px-Postgresql_elephant.svg.png)
![](https://upload.wikimedia.org/wikipedia/commons/9/9d/Ubuntu_logo.svg)

![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)

The Linux Server Configuration project will address the following:

1. How to secure a server from a number of attack vectors. 

2. How to install and configure a database server.

3. How to deploy an existing web application onto the server.

## Project Configuration

The project has been configured with the below settings:

* **Linux Server Config:** Amazon Lightsail Ubuntu 18.04 server image
* **Website URL:** http://ec2-35-177-16-5.eu-west-2.compute.amazonaws.com/
* **Server public ip address:** 35.177.16.5
* **SSH port:** 2200
* **Enabled ports in firewall:** 2200, 80 and 123
* **Linux user:** grader

## Project Walkthrough

### Amazon Lightsail Setup

* Log into AWS Console [RegisteredUsers]('https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fnc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fhomepage&forceMobileApp=0'). If not please register to [NewRegistration]('https://portal.aws.amazon.com/billing/signup#/start')
* Click on **Create Instance**
* Select **OS Only**
* Select **Ubuntu 16.04 LTS**
* Select base instance 
* Change the name if you would like but is not mandatory
* Click on **Create Instance**
* Go to **Account**
* Go to SSH Keys and download the key (which is a .pem file) onto your local machine.
* Within the instance you just created, you can connect to it by clicking "Connect using SSH". Alternatively, take the following steps to connect via your own SSH client:
  1. Move the downloaded LightsailDefaultKey.pem public key file into the local folder ~/.ssh and rename it lightsail.pem
  2. In your terminal, type: chmod 600 ~/.ssh/lightsail.pem . This secures the public key while also making it accessible.
  3. To connect to the instance via the terminal: $ ssh -i ~/.ssh/lightsail.pem ubuntu@35.177.16.5, where 35.177.16.5 is the public IP address of the instance.

### Create user account

* Log into the remote VM as root user through ssh: `$ ssh root@35.177.16.5` where 35.177.16.5 is the public IP address of the instance.
* Create new user 'grader' `$ sudo adduser grader`
* Create a new file in the sudoers directory. `$ sudo nano /etc/sudoers.d/grader` and give 'grader' super permissions `grader ALL=(ALL:ALL) ALL`.
* To prevent the "sudo: unable to resolve host(none)" error, edit the hosts file `$ sudo nano /etc/hosts`. Under 127.0.0.1:localhost, add `127.0.0.1 YOUR-IP-ADDRESS`.

### Update packages
Run the following commands to update all packages and set for future updates:

* `$ sudo apt-get update`
* `$ sudo apt-get upgrade`
* `$ sudo apt-get dist-upgrade`

### Change the SSH port to access your instance

* Open up the configuration file: `$ sudo nano /etc/ssh/sshd_config`
* Look for port number (on line 5) and change this from `22` to `2200`
* Save and exit using `CTRL+X`, confirm with `Y`
* Restart SSH `$ sudo service ssh restart`

### Configure the Uncomplicated Firewall (UFW)

* Check the current firewall status using `$ sudo ufw status`
* Deny all incoming requests using `$ sudo ufw default deny incoming`
* Allow all outgoings using `$ sudo ufw default allow outgoing`
* Allow incoming TCP packets on port 2200 to allow SSH `$ sudo ufw allow 2200/tcp`
* Allow incoming TCP packets on port 80 to allow www using `$ sudo ufw allow www`
* Allow incoming UDP packets on port 123 to allow NTP using `$ sudo ufw allow 123/udp`
* Close access through port 22 `$ sudo ufw deny 22`
* Enable firewall using `$ sudo ufw enable`
* Check the current firewall status using `$ sudo ufw status`. The output should look like this:
```
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
```
* Update the firewall configuration in your Amazon Lightsail instance by going to the Networking tab.
* Delete default SSH port 22 and add ports 123 (UDP) and 2200(TCP) in the 'Networking' tab on Lightsail. Your settings should look like the following:
```
HTTP      TCP     80
Custom    UDP     123
Custom    TCP     2200
```
* Exit the SSH connection: `$ exit`

### Create SSH login for grader user

Open a new(second) Terminal window (Command+N) to generate a public-private key pair.

* Input `$ ssh-keygen -f ~/.ssh/grader.rsa`
* Input `$ cat ~/.ssh/grader.rsa.pub` to read the public key. Copy its contents.

Return to your original (first) terminal window logged into Amazon Lightsail as the root user.

* Move to grader's folder `$ cd /home/grader`
* In the grader folder, create an authorized_keys file
   * `$ mkdir .ssh`
   * `$ touch .ssh/authorized_keys`
   * `$ nano .ssh/authorized_keys` and paste the public key you have copied earlier (from the second terminal window). Save.
* Change the ownership and permissions of the .ssh folder to the grader user: `$ chown -R grader.grader /home/grader/.ssh`.
* Secure your authorized_keys `$ sudo chmod 700 /home/grader/.ssh` and `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
* Restart the SSH service `$ sudo service ssh restart`
* You should now be able to login as grader user with port 2200 and the generated key pair with the following command: `$ ssh -i ~/.ssh/grader.rsa -p 2200 grader@35.177.16.5`.
* You will be asked for grader's password. To disable it, `$ sudo nano /etc/ssh/sshd_config`. Find the line `PasswordAuthentication` and change text to no. After this, restart ssh again: `$ sudo service ssh restart`

### Configure the timezone to UTC
* Open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`
* Select `None of the above` to change the timezone to UTC.
* Install ntp daemon ntpd for a better synchronization of the server's time over the network connection: `$ sudo apt-get install ntp`.

### Disable SSH for root user
This prevents attackers from attempting with root:
* `$ sudo nano /etc/ssh/sshd_config`
* Find the `PermitRootLogin` line and edit to `no`
* Restart ssh `$ sudo service ssh restart`

### Install Apache and mod_wsgi

* `$ sudo apt-get install apache2`.
* Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install *mod_wsgi* with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
* Enable *mod_wsgi*: `$ sudo a2enmod wsgi`.
* `$ sudo service apache2 start`.
* In your browser go to http://35.177.16.5. You should see a Apache2 Ubuntu Default Page if Apache has been correctly configured.

### Install Git

* `$ sudo apt-get install git`.
* Configure your username: `$ git config --global user.name <username>`.
* Configure your email: `$ git config --global user.email <email>`.

### Configure Apache to Serve Flask Application

* `cd /var/www`
* `sudo mkdir catalog` - Create catalog directory.
* `sudo chown -R grader:grader Catalog` - Change owner.
* `cd catalog`
* `sudo git clone https://github.com/thepembeweb/item-catalog.git catalog` - Clone git repository and  name it **catalog**.
* `cd catalog`
* `mv app.py __init__.py` - Rename app.py file.

**Note:** _Below steps are to make git directory not accessible._

* `cd .git`
* `sudo nano .htaccess` - Create **.htaccess** file.
* Enter text `RedirectMatch 404 /\.git` and save the file.

**Note:** _Below steps are to create virtual environment for our Item Catalog App_

* `cd /var/www/catalog/catalog` - cd to catalog project directory which was cloned.
* `sudo apt-get install python-pip` - Install pip.
* `sudo pip install virtualenv` - Install virtualenv.
* `sudo virtualenv venv` - Create virtualenv.
* `sudo chmod -R 777 venv` - Change virtualenv permission.
* `source venv/bin/activate` - Activate venv.
* `sudo pip install Flask` - Install Flask.
* `sudo pip install requests`
* `sudo pip install sqlalchemy`
* `sudo pip install oauth2client`
* `sudo install psycopg2` - install postgres
* `deactivate` -  deactivate the virtual environment i.e. **venv**.
* `sudo nano /etc/apache2/sites-available/catalog.conf` - Configure new Virtual Host.
* Paste below code into the file:

```xml
<VirtualHost *:80>
    ServerName 35.177.16.5
    ServerAlias ec2-35-177-16-5.eu-west-2.compute.amazonaws.com
    ServerAdmin admin@35.177.16.5
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

**Note:** _Below steps are to create catalog.wsgi and update the file to enable apache serve the Flask App_

* `cd /var/www/catalog`
* `sudo nano catalog.wsgi` - Create **catalog.wsgi** file.
* Add below code to the file and save.

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
```

* `sudo chown -R grader:grader catalog.wsgi` - Change the owner to **grader**.

### Install and Setup Postgres Database

**Note:** _Please make sure you are logged into the instance as grader user_

* `sudo apt-get install postgres` - Install postgres.
* `sudo su - postgres` - login as a postgres user.
* `psql` - Connect to shell.
* `CREATE USER catalog WITH PASSWORD 'catalog';` - Create user catalog.
* `ALTER USER catalog CREATEDB;` - Change catalog user role to creating database.
* `CREATE DATABASE catalog with OWNER catalog;` - Create database catalog.
* `\c catalog` - connect to catalog database.
* `REVOKE ALL ON SCHEMA public FROM public;` - Revoke rights to all users.
* `GRANT ALL ON SCHEMA public TO catalog;` - Grant rights to user **catalog** only.
* `\q` - exit the datatabase.
* `exit` - logout from postgres user account.
* `cd /var/www/catalog/catalog/`
* `sudo nano setup_database.py`
* Change `create_engine('sqlite:///moviezone.db')` to `create_engine('postgresql://catalog:catalog@localhost/catalog')` and save the file.
* `sudo nano __init__.py`
* Change `create_engine('sqlite:///moviezone.db')` to `create_engine('postgresql://catalog:catalog@localhost/catalog')` and save the file.

### Authenticate login through Google
* Go to [Google Cloud Platform](https://console.cloud.google.com)
* On the left menu, hover on `APIs & Services` and click on `Credentials`
* Create an OAuth Client ID (under the Credentials tab), and add http://35.177.16.5 and http://ec2-35-177-16-5.eu-west-2.compute.amazonaws.com as authorized JavaScript origins
* Add http://ec2-35-177-16-5.eu-west-2.compute.amazonaws.com/oauth2callback as authorized redirect URI
* Download the corresponding JSON file, open it and copy its contents
* `$ nano /var/www/catalog/catalog/client_secret.json` and replace the copied contents into this file
* Update all references of `client_secrets.json` to `/var/www/catalog/catalog/client_secrets.json` in the project 

### Running the app live
* Generate the database files by running `sudo python setup_database.py`
* Restart Apache using `$ sudo service apache2 reload`
* Disable the default Apache page. Run `$ sudo a2dissite 000-default.conf`
* Restart Apache using `$ sudo service apache2 reload`
* The application should be live at http://35.177.16.5 or http://ec2-35-177-16-5.eu-west-2.compute.amazonaws.com. 

### Checking error logs
* If there are internal errors returned, check the Apache error logs by running `$ sudo tail -100 /var/log/apache2/error.log` and resolve the traceback call error(s) it displays.

## Built With

* [Amazon Lightsail](https://lightsail.aws.amazon.com/) - The web hosting service used
* [Python](https://www.python.org/) - The framework used
* [PostgreSQL](https://www.postgresql.org/) - The database used

## Authors

* **[Pemberai Sweto](https://github.com/thepembeweb)** - *Initial work* - [Linux Server Configuration](https://github.com/thepembeweb/linux-server-configuration)

## License

[![License](http://img.shields.io/:license-mit-green.svg?style=flat-square)](http://badges.mit-license.org)

- This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
- Copyright 2019 © [Pemberai Sweto](https://github.com/thepembeweb).

