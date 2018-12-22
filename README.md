# Linux Server Configuration
## Project Description
This is the final project for Udacity's Full Stack Web Developer Nanodegree.

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.


## General
- IP address: 3.17.162.132
- Accessible SSH port: 2200
- Application URL:(http://ec2-3-17-162-132.us-east-2.compute.amazonaws.com/)
- Login with: `ssh -i ~/.ssh/grader_key -p 2200 grader@3.17.162.132`

### 1- Start a new Ubuntu Linux server instance on Amazon Lightsail
Follow the steps on this link to create an instance on Amazon Lightsail:
[Get started on Lightsail](https://classroom.udacity.com/nanodegrees/nd004-connect/parts/226fb92a-d5dc-4d10-add0-c1dabff6ee69/modules/56cf3482-b006-455c-8acd-26b37b6458d2/lessons/046c35ef-5bd2-4b56-83ba-a8143876165e/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)


### 2- SSH into the server

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultKey-us-east-2.pem` into the local folder `~/.ssh` and rename it `udacity_key.rsa`.
- In your terminal, type: `chmod 600 ~/.ssh/udacity_key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/udacity_key.rsa ubuntu@3.17.162.132`,
  where `3.17.162.132` is the public IP address of the instance.


### 3- Update and upgrade installed packages

`sudo apt-get update`
`sudo apt-get upgrade`

### 4- Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number on line 5 from `22` to `2200`.
- Modify:
  ```
  PermitRootLogin no
  PasswordAuthentication no
  ```
- Save and exit using CTRL+X and confirm with Y.
- Restart SSH: `sudo service ssh restart`.

### 5- Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  
  sudo ufw default deny incoming   
  sudo ufw default allow outgoing
  sudo ufw allow 2200/tcp  
  sudo ufw allow www      
  sudo ufw allow 123/udp
  sudo ufw deny 22  
  ```

- Turn UFW on: `sudo ufw enable`.

- Check the status of UFW to list current roles: `sudo ufw status`.

- Exit the SSH connection: `exit`.

- Click on the `Manage` option of the Amazon Lightsail Instance,
then the `Networking` tab, and then change the firewall configuration to match the internal firewall settings above.

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From your local terminal, run: `ssh -i ~/.ssh/udacity_key.rsa -p 2200 ubuntu@3.17.162.132`, where `3.17.162.132` is the public IP address of the instance.



### 6- Create a new user account named `grader`

- While logged in as `ubuntu`, add user: `sudo adduser grader`.
- Enter a password (twice) and fill out information for this new user.


### 7- Give `grader` the permission to sudo

- Edits the sudoers file: `sudo visudo`.

- Add this line `grader  ALL=(ALL:ALL) ALL` to give sudo permission to `grader` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- Verify that `grader` has sudo permissions. Run `su - grader`, enter the password, run `sudo -l` and enter the password again.


### 8- Create an SSH key pair for `grader` using the `ssh-keygen`

- On the local machine:
  - Run `ssh-keygen`
  - Enter file in which to save the key (I gave the name `grader_key`) in the local directory `~/.ssh`
  - Enter in a passphrase twice. Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`.
  - Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@3.17.162.132`.


### 9- Configure the local timezone to UTC

- While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata` and then choose UTC .


### 10- Install and configure Apache to serve a Python mod_wsgi application

- While logged in as `grader`, install Apache: `sudo apt-get install apache2`.
- Enter public IP of the Amazon Lightsail instance into browser. If Apache is working, you should see:
  <img src="https://github.com/Rasha-1994/images/blob/master/apache.png" width="600px">

- My project is built with Python 3. So, I need to install the Python 3 mod_wsgi package:  
 `sudo apt-get install libapache2-mod-wsgi-py3`.
- Enable `mod_wsgi` using: `sudo a2enmod wsgi`.


### 11- Install and configure PostgreSQL

- While logged in as `grader`, install PostgreSQL:
 `sudo apt-get install postgresql`.
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

- Switch to the `postgres` user: `sudo su - postgres`.
- Open PostgreSQL interactive terminal with `psql`.
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- List the existing roles: `\du`.

- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.
- Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.

- Add this line `catalog  ALL=(ALL:ALL) ALL` to give sudo permission to `catalog` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- Verify that `catalog` has sudo permissions. Run `su - catalog`, enter the password, run `sudo -l` and enter the password again.

- While logged in as `catalog`, create a database: `createdb catalog`.
- Run `psql` and then run `\l` to see that the new database has been created.

- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.


### 12- Install git

- While logged in as `grader`, install `git`: `sudo apt-get install git`.

### 13- Clone and setup the Item Catalog project from the GitHub repository

- While logged in as `grader`, create `/var/www/catalog/` directory.
- Change to that directory and clone Catalog App project:
`sudo git clone https://github.com/Rasha-1994/Item-Catalog.git catalog`.
- From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
- Change to the `/var/www/catalog/catalog` directory.
- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.

- In `__init__.py`, replace line :
  ```
  # app.run(host="0.0.0.0", port=8000, debug=True)
  ```
  With line:
  ```
  app.run()
  ```

- In `__init__.py`,`database_setup.py` and `categoryitems.py`, replace line:
   ```
   # engine = create_engine("sqlite:///itemcatalog.db")
   ```
   With line:
   ```
   engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
   ```

### 14- Authenticate login through Google

- Go to [Google Cloud Plateform](https://console.cloud.google.com/).
- Click `APIs & services` on left menu.
- Click `Credentials`.
- Create an OAuth Client ID and add http://3.17.162.132 and
http://ec2-3-17-162-132.us-east-2.compute.amazonaws.com as authorized JavaScript origins.
- Add http://ec2-3-17-162-132.us-east-2.compute.amazonaws.com/oauth2callback , http://ec2-3-17-162-132.us-east-2.compute.amazonaws.com/login and http://ec2-3-17-162-132.us-east-2.compute.amazonaws.com/gconnect
as authorized redirect URI.
- Download the corresponding JSON file, open it and copy the contents.
- Open `/var/www/catalog/catalog/client_secrets.json` and paste the previous contents into this file.
- Edit the path of `client_secrets.json` file in `__init__.py` like this:
    ```
    APP_PATH = '/var/www/catalog/catalog/'
    CLIENT_ID = json.loads(
    open(APP_PATH + 'client_secrets.json', 'r').read())['web']['client_id']
    APPLICATION_NAME = "Item Catalog"
    ```
    And also in `gconnect` function:
    ```
    oauth_flow = flow_from_clientsecrets(APP_PATH + 'client_secrets.json', scope='')
    oauth_flow.redirect_uri = 'postmessage'
    credentials = oauth_flow.step2_exchange(code)
    ```
- Replace the client ID  of the `templates/login.html` file in the project directory.


### 15- Install the virtual environment and dependencies

- While logged in as `grader`, install pip: `sudo apt-get install python3-pip`.
- Install the virtual environment: `sudo apt-get install python-virtualenv`.
- Change to the `/var/www/catalog/catalog/` directory.
- Create the virtual environment: `sudo virtualenv -p python3 venv3`.
- Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.
- Activate the new environment: `. venv3/bin/activate`.
- Install the following dependencies:
  ```
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  pip install flask_bootstrap
  pip install okta
  pip install sqlalchemy
  pip install flask
  sudo apt-get install libpq-dev
  pip install flask_wtf
  pip install psycopg2
  pip install wtforms
  pip install Flask-SQLAlchemy
  pip install flask-oidc
  ```

- Run `python __init__.py` and you should see:
  ```
  * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
  ```

- Deactivate the virtual environment: `deactivate`.



### 16- Set up and enable a virtual host

- Add the following line in `/etc/apache2/mods-enabled/wsgi.conf` file
to use Python 3.

  ```
  #WSGIPythonPath directory|directory-1:directory-2:...
  WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.7/site-packages
  ```

- Create `/etc/apache2/sites-available/catalog.conf` and add the
following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
	    ServerName 3.17.162.132
	    ServerAdmin admin@3.17.162.132
        ServerAlias ec2-3-17-162-132.us-east-2.compute.amazonaws.com
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

- Enable virtual host: `sudo a2ensite catalog`.

- Reload Apache: `sudo service apache2 reload`.


### 17- Set up the Flask application

- Create `/var/www/catalog/catalog.wsgi` file add the following lines:

  ```
  activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog/")
  sys.path.insert(1, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = "super_secret_key"
  ```

- Restart Apache: `sudo service apache2 restart`.



### 18- Set up the database schema and populate the database
- From the `/var/www/catalog/catalog/` directory, open `database_setup.py` and change line 38:`description = Column(String(250))`
with: `description = Column(String(500))`
- From the `/var/www/catalog/catalog/` directory,
activate the virtual environment: `. venv3/bin/activate`.
- Run: `python categoryitems.py`.
- Deactivate the virtual environment: `deactivate`.

### 19- Disable the default Apache site

- Disable the default Apache site: `sudo a2dissite 000-default.conf`.

- Reload Apache: `sudo service apache2 reload`.

### 20- Launch the Web Application

- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to http://3.17.162.132 or http://ec2-3-17-162-132.us-east-2.compute.amazonaws.com .

### The Output:
  <img src="https://github.com/Rasha-1994/images/blob/master/output1.png" width="600px">
  <img src="https://github.com/Rasha-1994/images/blob/master/output2.png" width="600px">
  <img src="https://github.com/Rasha-1994/images/blob/master/output3.png" width="600px">

## Helpful Resources
- [Vagrant](https://www.vagrantup.com/downloads.html)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [SSH: How to access a remote server and edit files](https://www.youtube.com/watch?v=HcwK8IWc-a8)
- [Configuring Linux Web Servers](https://classroom.udacity.com/courses/ud299-nd)
