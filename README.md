LINUX SERVER

This project is linked to the Configuring Linux Web Servers course, which teaches you to secure and set up a Linux server. By the end of this project, you will have one of your web applications running live on a secure web server.
To complete this project, you'll need a Linux server instance. We recommend using Amazon Lightsail for this. If you don't already have an Amazon Web Services account, you'll need to set one up. Once you've done that, here are the steps to complete this project.


•	Public IP Address: 18.210.244.219

•	Server Alias: 18.210.244.219.xip.io/

•	Port: 2200

Get your server.

#1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.

#2. Follow the instructions provided to SSH into your server.
Secure your server.
#3. Update all currently installed packages.
1.	$ sudo apt-get update
2.	$ sudo apt-get upgrade

#4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
1.	On Lightsail website, under network tab
a.	create custom TCP application with 2200 port number
b.	create custom UDP application with 123 port number
2.	reboot server
3.	$ sudo nano /etc/ssh/sshd_config
a.	add Port 2200 below Port 22 
b.	Change “PasswordAuthentication no” to “PasswordAuthentication yes”
c.	write file and exit (ctrl o, enter, ctrl x)
4.	reboot the server

#5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
1.	$ sudo ufw default deny incoming
2.	$ sudo ufw default allow outgoing
3.	$ sudo ufw allow ssh
4.	$ sudo ufw allow 2200/tcp
5.	$ sudo ufw allow 123/udp
6.	$ sudo ufw allow www
7.	$ sudo ufw enable
a.	$ sudo ufw status
8.	Reboot server

Give grader access.
In order for your project to be reviewed, the grader needs to be able to log in to your server.

#6. Create a new user account named grader.
1.	$ sudo apt-get install finger
2.	$ sudo adduser grader
a.	Password: grader
b.	Full Name: Linux Course Grader

#7. Give grader the permission to sudo.
1.	$ sudo nano /etc/sudoers.d/grader
a.	Type: grader ALL=(ALL) NOPASSWD:ALL
b.	Save and close

#8. Create an SSH key pair for grader using the ssh-keygen tool.
1.	In local terminal, access the server: $ ssh grader@18.210.244.219 -p 2200
2.	On the local machine: $ ssh-keygen
  a.	generate key pair
  b.	Do this on the local machine
  c.	File should be the same .ssh directory as asked about with no file extension 
  d.	Key name: graderkey
  e.	Passphrase: linuxserver
  f.	$ cat ~/.ssh/graderkey.pub
  i.	Copy the contents
3.	As the new user
  a.	$mkdir .ssh – create new directory
  b.	$ touch .ssh/authorized_keys – make this file
  c.	$ sudo nano .ssh/authorized_keys
  d.	Paste and save
  e.	Set the permissions: $ chmod 700 .ssh
  i. $ chmod 644 .ssh/authorized_keys
4.	Disable PW login
  a.	$ sudo nano /etc/ssh/sshd_config
  b.	Change “PasswordAuthentication yes” to “PasswordAuthentication no”
5.	Erase “Port 22” in the sshd_config file and $ sudo ufw deny 22
6.	Restart the service with $ sudo service ssh restart
7.	To login $ ssh grader@18.210.244.219 -p 2200 -i ~/.ssh/graderkey
Prepare to deploy your project.

#9. Configure the local timezone to UTC.
•	Confirm it is UTC using: $ date
#10. Install and configure Apache to serve a Python mod_wsgi application.

•	Followed the guide on https://umar-yusuf.blogspot.com/2018/02/deploying-python-flask-web-app-on.html and https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps 

1.	Install apache: $ sudo apt-get install apache2
2.	Go to 18.210.244.219 and ensure webpage works
3.	Install application handler: $ sudo apt-get install libapache2-mod-wsgi
4.	Install Python: $ sudo apt-get install python
5.	Check version of Python: $ python (Ctrl+D to quit)
6.	Install pip: $ sudo apt-get install python-pip
7.	Install flask: $ sudo apt-get install python-flask

#11. Install and configure PostgreSQL
•	Do not allow remote connections
•	Create a new database user named catalog that has limited permissions to your catalog application database.
1.	http://www.yolinux.com/TUTORIALS/LinuxTutorialPostgreSQL.html 

1.	$ sudo apt-get install postgresql
2.	Check that file is for local connections only
  a.	$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
3.	Create new user “postgres” and switch to that user: $ sudo su – postgres
4.	Login to postgresql: $ psql
5.	Create catalog database: # CREATE DATABASE catalog;
6.	Add catalog user: # CREATE USER catalog;
7.	Change the password: # ALTER ROLE catalog WITH PASSWORD ‘catalog’;
8.	Allow catalog user to make changes to catalog database: # GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
9.	Change password: # ALTER USER catalog PASSWORD ‘catalog’;
10.	Exit: # \q
11.	$ exit

#12. Install git.
1.	$ sudo apt-get install git

Deploy the Item Catalog project.
#13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
1.	Github repository: https://github.com/leothelion15/CatalogAppServer

2.	$ cd /var/www
3.	$ sudo chown -R grader /var/www/
4.	$ git clone https://github.com/leothelion15/CatalogAppServer
5.	$ cd CatalogAppServer/
6.	$ sudo mv catalogApp.py __init__.py
7.	In all python files, change “engine = create_engine('sqlite:///catalogdatabasewithusers.db')” to “engine = create_engine (‘postgresql://catalog:catalog@localhost/catalog ')”
8.	$ sudo pip install –upgrade pip
9.	$ sudo pip install flask_sqlalchemy
10.	$ sudo pip install sqlalchemy-utils
11.	$ sudo apt-get install python-psycopg2
12.	$ sudo pip install requests
13.	$ sudo pip install --upgrade google-api-python-client oauth2client
14.	Change the client_secrets path using $ sudo nano __init__.py
o	CLIENT_ID = json.loads(open('/var/www/CatalogAppServer/client_secrets.json', 'r').read())['web']['client_id']
15.	Load python files: $ sudo python databaseSetupCatalog.py , $ sudo python lotsofcategories.py , and $ sudo python __init__.py

#14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!
1.	$ sudo nano CatalogAppServer.wsgi
      
     
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0, "/var/www/CatalogAppServer/")
      from __init__ import app as application

      application.secret_key = 'Add your secret key'
      
      
o	save and close

2.	$ sudo chown -R grader /etc/apache2

3.	$ cd /etc/apache2/sites-available

4.	$ sudo nano CatalogAppServer.conf

      <VirtualHost *>
        ServerName 18.210.244.219
        ServerAlias 18.210.244.219.xip.io
        ServerAdmin lb527c@att.com
        WSGIScriptAlias / /var/www/CatalogAppServer/CatalogAppServer.wsgi
        <Directory /var/www/CatalogAppServer/>
                 Order allow,deny
                 Allow from all
        </Directory>
        Alias /static /var/www/CatalogAppServer/static
        <Directory /var/www/FlaskApp/CatalogAppServer/static/>
                 Order allow,deny
                 Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>
      
      
5.	$ cd /var/www/CatalogAppServer
6.	$ sudo a2dissite 000-default.conf
7.	$ sudo a2ensite CatalogAppServer.conf
8.	$ sudo service apache2 reload
