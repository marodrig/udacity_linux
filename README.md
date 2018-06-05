# Ubuntu web server setup.
This is the setup, and deployment of an ubuntu web server for the nano-degree program at Udacity.

## Server info:
The server can be accessed by the following methods: 

public ip: 18.188.16.13

SSH port: 2200

URL: http://ec2-18-188-16-13.us-east-2.compute.amazonaws.com/

## Software:
  - Python 2.7.x
  - Apache2 webserver
  - POSTGRESQL Relational database
  - Ubuntu as the OS.
  - AWS lightsail
  - Flask, as the web framework.
  - sqlalchemy, ORM.
  - Foundation CSS, frontend.
  - git for source control.
  - pip for installing Python packages.
  - glances for monitoring the server.
  
### Apache configuration:
First install Apache using apt-get:
```
sudo apt-get install apache2 apache2-doc
```
We need to configure our site, Apache has a default page currently displayed, but we want to create our own. Use your text editor of choice, mine is vi to create a new configuration file for Apache:
```
vi /etc/apache2/sites-available/FlaskApp.conf 
```
And we want the file to look like this:
```
<VirtualHost *:80>
	ServerName 18.188.16.13
	ServerAlias ec2-18-188-16-13.us-east-2.compute.amazonaws.com
	ServerAdmin grader@18.188.16.13
	WSGIDaemonProcess catalog threads=5 python-path=/var/www/FlaskApp/catalog_app:/var/www/FlaskApp/catalog_app/venv/lib/python2.7/site-packages
	WSGIProcessGroup catalog
	WSGIScriptAlias / /var/www/FlaskApp/myapp.wsgi
	<Directory /var/www/FlaskApp/catalog_app/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/catalog_app/static
	<Directory /var/www/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Apache will now be ready to serve our WSGI but we need to do some configuration before we are ready.

### WSGI configuration:
Use apt-get to install apache WSGI module:
```
sudo apt-get install libapache2-mod-wsgi
```
As I configure Apache to look for my WSGI in the following location: /var/www/FlaskApp/myapp.wsgi I will use vi to once again create myapp.wsgi.

```
sudo vi /var/www/FlaskApp/myapp.wsgi
```
And edit the file with the following content:
```#!/usr/bin/python
import sys
import os
import logging
import hashlib
logging.basicConfig(stream=sys.stderr)
sys.path.append("/var/www/FlaskApp/catalog_app/")

from views import app as application
from views import generate_csrf_token
application.jinja_env.globals['csrf_token'] = generate_csrf_token
application.secret_key = hashlib.sha256(os.urandom(1024)).hexdigest()
```
Mostly the portion of my views.py that were in the 'main'. This is the file that will be run by python.
First enable our new site:
```
sudo a2ensite FlaskApp
```
Remove Apache's default site:
```
sudo a2dissite 000-default
```
Restart apache2 with our configuration:
```
sudo service apache2 reload
```

### POSTGRE database configuration:
Use apt-get to install postgresql:
```
sudo apt-get install postgresql
```
We need to run psql as user postgres:
```
sudo su - postgres
psql
```
We are now in the psql prompt and we can create our user:
```
create user catalog_user;
```
And out database:
```
CREATE DATABASE catalog;
```
Then assign a password to our user:
```
ALTER ROLE catalog_user WITH PASSWORD 'not_the_actual_password_because_security';
```
And now let's grant the user permissions to our catalog database:
```
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog_user;
```
Now we need to changes our python code to connect as our catalog_user to our catalog database instead of sqlite.

And we created a 'www-data' user in order to play nice with the Apache2 default configuration:
```
create user "www-data" with login;
```

No remote connections are allowed as per the file located at /etc/postgresql/9.5/main/pg_hba.conf 

Restart the service:
```
sudo service postgresql restart
```

### Additional steps:
  First we need to enable oauth from our google console to the new site. Replace our localhost with our public ip and url and create a new secret key.
  You can use the nslookup command from ubuntu to find the URL of your instance:
  ```
  nslookup {private-ip}
  ```
  We installed glances: http://glances.readthedocs.io/en/latest/index.html using pip.  We used the default configuration.
  
#### Blocking access to .git:
It's a bad idea to expose your .git directory, as it allows someone to download your source code.
Our /etc/apache2/conf-enabled/security.conf was edited and this portion added and edited as needed.
  ```
  <DirectoryMatch "/\.git">
    Require all denied
  </DirectoryMatch>
  ```
  
  More info: https://davidegan.me/hide-git-repos-on-public-sites/
  

#### Third-party references/resources:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

https://github.com/kongling893/Linux-Server-Configuration-UDACITY

https://stackoverflow.com/

https://forums.aws.amazon.com/thread.jspa?threadID=160352

