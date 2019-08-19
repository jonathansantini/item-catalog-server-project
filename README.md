# Udacity Full Stack Linux Server Configuration Project

Repository for instructions on  setting up a flask application using a Postgres database on an ubuntu server setup with Amazon Lightsail.

-------

Above link is now unavailable because I have graduated from the nanodegree program.

* Public IP address: 18.218.86.151
* SSH port:

# Installed packages
<table>
  <tr>
    <td><strong>NAME</strong></td>
  </tr>
  <tr>
    <td>apache2</td>
  </tr>
  <tr>
    <td>libapache2-mod-wsgi</td>
  </tr>
  <tr>
    <td>finger</td>
  </tr>
  <tr>
    <td>postgresql</td>
  </tr>
  <tr>
    <td>sqlalchemy</td>
  </tr>
  <tr>
    <td>flask</td>
  </tr>
  <tr>
    <td>python-psycopg2</td>
  </tr>
  <tr>
    <td>libpq-dev</td>
  </tr>
  <tr>
    <td>python-dev</td>
  </tr>
</table>

# Create a new Ubuntu server on Amazon Lightsail

1. Create an AWS account.
2. Select the Lightsail option in the main menu.
3. Click the create instance button.
4. Select Linux/Unix under platform.
5. Select OS Only under the select a blueprint area.
6. Select Ubuntu 16.04 or 18.04, up to you.
7. Select instance plan (depends on what you need and budget)
8. Name your instance.
9. Finally, click the create instance button.

# SSH into your new server

1. Find the ssh keys for your account and download the .pem file.
2. Locate the public IP and username for your new aws server instance.
3. Open up a terminal window on your computer and enter the following command.

```
ssh <username>@<ip address> -p 22 -i ~/.ssh/<name of .pem file>
```

# Update all installed packages
1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install newest versions of packages
3. Set for future updates: `sudo apt-get dist-upgrade`

# Update default SSH port to 2200
The default SSH port for unix is 22.  In order to change that port to something like 2200, do the following.

1. Run the following command:
```sudo vi /etc/ssh/sshd_config```
2. Locate the following line:
```# Port 22```
3. Remove # and change 22 to your desired port number.
4. Restart the sshd service by running the following command:
```sudo service sshd restart```

# Setup firewall
1. Check initial firewall status: `$ sudo ufw status`
2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`
3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow 123/udp`
7. Close port 22: `$ sudo ufw deny 22`
8. Enable firewall: `$ sudo ufw enable`
9. Check out current firewall status: `$ sudo ufw status`
10. Update the firewall configuration on Lightsail website under **Networking**. Delete default SSH port 22 and add port 80, 123, 2200 using the custom option.
11. Open up a new terminal window and you can now ssh in via the new port 2200 instead of 22.

# Configure the local timezone to UTC
To set the server timezone to UTC, do the following:
1. `$ sudo dpkg-reconfigure tzdata`
2. `Select UTC`

# Create a new account user called **grader**
1. To create a new user, enter: `$ sudo adduser grader`
2. Follow the prompts and give the new user a password.
3. Copy default sudoer file and name it grader: `$ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`
4. `$ sudo vi /etc/sudoers.d/grader`
5. Change the line `ubuntu ALL=(ALL) NOPASSWD:ALL` to `grader ALL=(ALL) NOPASSWD:ALL`
6. Save and exit.

# Create separate set of SSH login keys for **grader**
1. On your local machine, use `ssh-keygen` to generate a new set of keys.
2. Follow the prompts and add a client secret just in-case someone does get a hold of these keys.
3. Go back to the server as the admin and create a `.ssh` folder for **grader** `$ sudo mkdir /home/grader/.ssh`
4. Set user and group for `.ssh` folder: `$ sudo chown grader:grader /home/grader/.ssh`
5. Set permissions for `.ssh` folder to 700. `$ sudo chmod 700 /home/grader/.ssh`
6. Create `authorized_users` file to the store the newly create key:
`$ sudo touch /home/grader/.ssh/authorized_keys`
7. Set user and group for `authorized_keys`: `$ sudo chown grader:grader /home/grader/.ssh/authorized_keys`
8. Set permissions for `authorized_keys` file to 644. `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
9. On your local machine, copy the contents of the newly created `.pub` file.
10. On the server, paste the contents into the new `authorzied_keys` file and save.
9. Now you can open a new terminal window and ssh into the server as grader.

# Install **Apache** and **mod_wsgi**
1. `sudo apt-get install apache2`
2. Go to your public ip address and you should see the default apache landing page.
3. `sudo apt-get install sudo apt-get install libapache2-mod-wsgi python-dev`
4. Enable mod_wsgi: `$ sudo a2enmod wsgi`
5. Restart Apache: `$ sudo service apache2 restart`

# Install PostgreSQL
1. Run `$ sudo apt-get install postgresql`
2. Run `$ sudo apt-get install python-psycopg2`
3. Run `$ sudo apt-get install libpq-dev`
4. Open file: `$ sudo vi /etc/postgresql/9.5/main/pg_hba.conf`
5. Make sure bottom of the config looks like this:
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5
```

# Create a new user and DATABASE
1. Log into postgres: `$ sudo -u postgres psql`
2. Create database: `postgres=# CREATE DATABASE categories;`
3. Create new user: `postgres=# CREATE USER new_username;`
4. Add new user roles: `postgres=# ALTER USER new_username SUPERUSER CREATEDB;`
5. Add new user pass: `postgres=# ALTER USER postgres PASSWORD 'my_postgres_password';`
6. Enable user to a database: `postgres=# grant all privileges on database categories to ubuntu;`
7. Check changes by listing all users: `postgres=# \du`

# Create .wsgi file
1. After installing application and modules, create a wsgi conflig file in root folder of application: `$ sudo vi <APP Name>.wsgi`
2. Make sure the file looks like the following:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/<application folder>/")

from ItemCatalog import app as application
application.secret_key = '<secret key>'
```
3. Restart Apache: `$ sudo service apache2 reload`

# Create virtual host file
1. Create file: `$ sudo touch /etc/apache2/sites-available/<app name>.conf`
2. Add the following to the file:
```
<VirtualHost *:80>
        ServerName <public ip>
        ServerAdmin <your email>
        WSGIScriptAlias / /var/www/<app directory>/<app name>.wsgi
        <Directory /var/www/<app directory>/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/<app directory>/static
        <Directory /var/www/<app directory>/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable virtual host: `$ sudo a2ensite <app name>`
4. Restart Apache: `$ sudo service apache2 reload`
5. If all went well, go to the public ip address and your landing page should be present.

# Errors
If errors are present, run the following command to display the server error log as you reload the site.
`tail -f /var/log/apache2/error.log`

# Sources
1. [Amazon Lightsail](https://aws.amazon.com/lightsail/)
2. [Google Developer Console](https://console.developers.google.com/)
2. [Gavin Marsh](http://gavinmarsh.io/blog/amazon_lightsail/)
3. [Deploying a flask app tutorial from devops.ionos.com](https://devops.ionos.com/tutorials/deploy-a-flask-application-on-ubuntu-1404/)
4. [Stackover issue on not able to connect to database](https://stackoverflow.com/questions/16973018/createuser-could-not-connect-to-database-postgres-fatal-role-tom-does-not-e)
