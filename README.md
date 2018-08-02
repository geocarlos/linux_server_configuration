# Linux Server Configuration
## Sixth and last project for the Udacity Full Stack Web Developer Nanodegree

The server is on a Lightsail VPS, provided by Amazon Web Services - AWS.

IP Address of the server: 35.183.96.119
SSH port: 2200

## Configuration steps

### Get VPS and set access from local computer

 - Get a Lightsail instance
 - Generate a SSH keys in my local computer, with ssh-keygen
 - Login to AWS, then enter the server, copy the public key from my computer and add it to the 'authorized_keys' file.
 - Login from my local computer

### Update all currently installed packages
The packages were updated with the commands `sudo apt-get update` and `sudo apt-get upgrade`

### Configure SSH and UFW
- Change SSH port to 2200 in file `/etc/ssh/sshd_config`
- Check UFW by running `sudo ufw status`
- Keep UFW inactive and deny everything in, by running `sudo ufw default deny incoming`
- Allow everything out by running `sudo ufw default allow outgoing`
- Allow ssh: `sudo ufw allow ssh`
- Allow port 2200: `sudo ufw allow 2200/tcp`
- Allow port 80: `sudo ufw allow www`
- Allow port NTP (port 123): `sudo ufw allow ntp`
- Allow port 443, for SSL-secured connection: `sudo ufw allow 443/tcp`

IMPORTANT: On Lightsail, ports 2200 and 443 need to be enabled on Lightsail panel.

### Create new user: grader
- Command: `sudo adduser grader` and follow the prompts.
- Create an SSH key for grader user with ssh-keygen
- Add public key to .ssh folder in grader home directory
- Add grader to sudoers: `sudo adduser grader sudo`
- Configure grader in sudoers.d directory: `grader ALL=(ALL) NOPASSWD:ALL`

### Configure the local timezone to UTC

- Set with command `timedatectl set-timezone UTC`
- Check with command `timedatectl status`

### Apache and WSGI

For this I followed this [tutorial](https://devops.profitbricks.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/) for the initial steps.
- Install required components (notice that Apache is included): `sudo apt-get install apache2 apache2-utils libexpat1 ssl-cert python`
- Install mod_wsgi: `sudo apt-get install libapache2-mod-wsgi`

I also followed the other steps for the test proposed, before I deployed the Catalog App.

### Postgresql

- Install Postgresql: `sudo apt-get install postgresql postgresql-contrib`
- Change postgres Linux user's password: `sudo passwd postgres`
- Switch to postgres user: `su - postgres`
- As postgres user, create new password for the database: `psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'newpassword';"`

### Git
- Usually, Git is already installed, but in case it is not, `sudo apt-get install git` will do it.

### Catalog App

I cloned the Catalog App into the /var/www/catalogapp.
- WSGI file has this contents:

``import sys <br>  
sys.path.insert(0, '/var/www/catalogapp/catalog/') <br>
from app import app as application``

Since I had developed the Catalog project with SQLite, I had to adapt it to Postgresql, by installing the module psycopg2: `$ pip install psycopg2`

#### Configure a Virtual Host for the Catalog app:

- Create the file /etc/apache2/sites-available/catalogapp.conf:

<VirtualHost \*:80>
	ServerName catalogapp.online

        ServerAdmin webmaster@localhost

        WSGIDaemonProcess catalog user=www-data group=www-data threads=5
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}

        WSGIScriptAlias / /var/www/catalogapp/catalog/catalogapp.wsgi

        <Directory /var/www/catalogapp/catalog/>
                Require all granted
        </Directory>

        Alias /static /var/www/catalogapp/catalog/static

        <Directory  /var/www/catalogapp/catalog/static/>
                Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
RewriteEngine on
RewriteCond %{SERVER_NAME} =catalogapp.online
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

The last lines were added by certbot, as I set up SSL with Let's Encrypt. I chose not to allow HTTP, so port 80 always redirects to port 443.

The ServerName, if lacking a domain, can be the IP Address of the server.

### Third party resources
- finger: an application that helps handle users
- Let's Encrypt: provides SSL certificate, required for Facebook OAuth.
- SQLAlchemy: Python framework for database connection
- psycopg2: Python module to connect to a Postgresql database

### Another application

I have started developing a simple React application, with back-end in Python (Flask), called "Magic E". It is intended to help kids understand the use of the silent E, or "Magic E" in many English one-syllable words.

This Flask application serves the React files generated with `npm run build` for production and fetch and manipulate data from [Oxford Dictionaries API](https://developer.oxforddictionaries.com/).
