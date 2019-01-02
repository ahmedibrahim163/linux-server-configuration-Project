# udacity linux server configuration Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it. 

* IP address: 3.81.255.121

* Accessible SSH port: 2200

* Application URL: http://catalogproject.com.3.81.255.121.xip.io/

# Requirements
* A Web Browser such as Chrome or Firefox is installed.

* Access to a command line terminal such as bash or an SSH client such as PuTTY to remotely connect to      the server. 

# Usage
* SSH into the Linux server as 'grader' using the provided key.
   ` $ ssh grader@3.81.255.121 -p 2200 -i ~/.ssh/grader_rsa `
* Enter passphrase for grader_rsa key (Both the key and the passphrase are included in the "Notes to Reviewer" field).
* Access the Item Catalog being run on the server. 


# Server Configuration

# Initial server setup

 * Create instance on Amazon Lightsail.
 * Update system packages to most recent versions.
  `$ sudo apt-get update`
  `$ sudo apt-get upgrade`
  `$ sudo apt-get autoremove`

# User management
 * Disable remote login as root user.
    `$ sudo nano /etc/ssh/sshd_config`
 * Update PermitRootLogin to no and save.
    `$ PermitRootLogin no`
 * Restart SSH.
    `$ sudo service ssh restart`

 Create grader user with sudo privileges.
 * Add new user called 'grader' with password.
    `$ sudo adduser grader`
 * Grant grader sudo privileges.
    `$ sudo nano /etc/sudoers.d/grader`

   * Add the following lines and save.
      `# User rules for grader`
      `grader ALL=(ALL) NOPASSWD:ALL`
   
      `password : admin` 
 
 * Setup grader to login with an SSH key.
    Generate RSA key for grader on physical machine (not remote server).
    `$ ssh-keygen`
 * Allow grader to login with password (Update PasswordAuthentication to yes).
    `$ sudo nano /etc/ssh/sshd_config`
 * Restart SSH.
    `$ sudo service ssh restart`
 * login as grader.
    `$ ssh grader@3.81.255.121`
 * Create .ssh directory.
    `$ mkdir .ssh`
 * Copy and paste the public RSA key into authorized_keys.
    `$ nano .ssh/authorized_keys`
 * Set file permissions.
    `$ chmod 700 .ssh`
    `$ chmod 644 .ssh/authorized_keys`
 * Force key-based authentication (Update PasswordAuthentication to no).
    `$ sudo nano /etc/ssh/sshd_config`
 * Restart SSH.
    `$ sudo service ssh restart`

# Firewall Management

  # Update the Amazon Lightsail Firewall to allow SSH through Port 2200.
    Login to Amazon Lightsail account.
    Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    Under the 'Networking' tab, go to the 'Firewall' settings.
    Click '+ Add another' and 'Save' after making the following rule:
    `Custom TCP 2200`
 
   * Enable Port 2200 SSH access on the linux server.
    `$ sudo vim /etc/ssh/sshd_config`
   * Add the following line and save.
    `$ Port 2200`
   * Restart SSH.
    `$ sudo service ssh restart`

  # Disable Port 22 SSH access through the Amazon Lightsail Firewall.
    Login to Amazon Lightsail account.
    Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    Under the 'Networking' tab, go to the 'Firewall' settings.
    Click 'Edit rules', then 'X'(delete) the following SSH rule for port 22, and then save:
    SSH TCP 22

  # Enable Port 123 NTP access through the Amazon Lightsail Firewall.
    Login to Amazon Lightsail account.
    Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    Under the 'Networking' tab, go to the 'Firewall' settings.
    Click '+ Add another' and 'Save' after making the following rules:
    Custom TCP 123
    Custom UDP 123
 * Setup and enable the Uncomplicated Firewall on the linux server.
    `$ sudo ufw default deny incoming`
    `$ sudo ufw default allow outgoing`
    `$ sudo ufw allow 2200/tcp`
    `$ sudo ufw allow www`
    `$ sudo ufw allow ntp`
    `$ sudo ufw enable` 
 
 # Reboot the linux server for the Uncomplicated Firewall to update.
  * Login to Amazon Lightsail account.
  * Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
  * Click the 'Reboot' button.
 
 # Configure the local timezone to UTC 
 * Run`sudo dpkg-reconfigure tzdata` and then choose UTC
 * Install Apache
    `sudo apt-get install apache2`
 * Install mod_wsgi
    Run `sudo apt-get install libapache2-mod-wsgi python-dev`
 * Install git using: `sudo apt-get install git`
    `cd /var/www`
 * Enable mod_wsgi with `sudo a2enmod wsgi`
 * Start the web server with `sudo service apache2 start`   

# Clone the Catalog app from Github
 * `sudo mkdir catalog`
 * Change owner of the newly created catalog folder 
   `sudo chown -R grader:grader catalog`
   `cd /catalog`
 * Clone your project from github `git clone https://github.com/ahmedibrahim163/catalog-item.git catalog`

# Create a `catalog.wsgi` file, then add this inside:
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'supersecretkey'

 * Rename application.py to init.py `mv application.py __init__.py` 
 ** Install virtual environment**
 * Install the virtual environment `sudo pip install virtualenv`
 * Create a new virtual environment with `sudo virtualenv venv`
 * Activate the virutal environment source `venv/bin/activate`
 * Change permissions `sudo chmod -R 777 venv`
 * Install Flask and other dependencies
 * Install pip with `sudo apt-get install python-pip`
 * Install Flask `pip install Flask`
 * Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`
 * Update path of client_secrets.json file
    `nano __init__.py`
 * Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
 * Configure and enable a new virtual host
 * Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
     *Paste this code:*
     `<VirtualHost *:80>`
     `ServerName 3.81.255.121`
     `ServerAlias HOSTNAME catalogproject.com.3.81.255.121.xip.io/`
     `ServerAdmin ahmad7fter@gmail.com`
     `WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages`
     `WSGIProcessGroup catalog`
     `WSGIScriptAlias / /var/www/catalog/catalog.wsgi`
     `<Directory /var/www/catalog/catalog/>`
         `Order allow,deny`
         `Allow from all`
     `</Directory>`
     `Alias /static /var/www/catalog/catalog/static`
     `<Directory /var/www/catalog/catalog/static/>`
         `Order allow,deny`
         `Allow from all`
     `</Directory>`
     `ErrorLog ${APACHE_LOG_DIR}/error.log`
     `LogLevel warn`
     `CustomLog ${APACHE_LOG_DIR}/access.log combined`
    `</VirtualHost>`

 * Enable the virtual host `sudo a2ensite catalog`
# Install and configure PostgreSQL
 * `sudo apt-get install libpq-dev python-dev`
 * `sudo apt-get install postgresql postgresql-contrib`
 * `sudo su - postgres`
 * `psql`
 * `CREATE USER catalog WITH PASSWORD 'password';`
 * `ALTER USER catalog CREATEDB;`
 * `CREATE DATABASE catalog WITH OWNER catalog;`
 * `\c catalog`
 * `REVOKE ALL ON SCHEMA public FROM public;`
 * `GRANT ALL ON SCHEMA public TO catalog;`
 * `\q`
 * `exit`
 
 * Change create engine line in your `__init__.py` and database_setup.py to: `engine =     create_engine('postgresql://catalog:password@localhost/catalog')`
 * `python /var/www/catalog/catalog/database_setup.py`

# Make sure no remote connections to the database are allowed. Check if the contents of this file 
 * `sudo nano /etc/postgresql/9.3/main/pg_hba.conf looks like this:`
 `local`   `all`             `postgres`                                  `peer`
 `local`   `all`             `all`                                       `peer`
 `host`    `all`             `all`             `127.0.0.1/32`            `md5`
 `host`    `all`             `all`             `::1/128`                 `md5`

# The error log file.
  `$ sudo less /var/log/apache2/error.log`

# The access log file.
  `$ sudo less /var/log/apache2/access.log`

# Restart Apache
 * ` sudo service apache2 restart`

# Visit site at `http://3.81.255.121

# Resources:
 https://help.ubuntu.com/
 
 https://help.ubuntu.com/community/UbuntuTime
 
 https://modwsgi.readthedocs.io/en/master/user-guides/configuration-guidelines.html
 
 http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
 
 https://discussions.udacity.com/c/nd004-p7-linux-based-server-configuration
 
 https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 
 https://discussions.udacity.com/t/500-internal-server-error-and-mod-wsgi-warnings-in-error-logs/489043
 
 https://docs.sqlalchemy.org/en/latest/core/engines.html
 
 https://www.postgresql.org/docs/10/client-authentication.html

