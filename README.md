# Linux-Server-Configuratiom
Host Item Catalog Web App on Lightsail Ubuntu instance

Project Info

IP address: 34.213.204.151
Accessible SSH port: 2200.
Application URL: http://ec2-34-213-204-151.us-west-2.compute.amazonaws.com/

# 1 - Create a new user named grader and grant this user sudo permissions.
  1. Log into the remote VM as root user through ssh: $ ssh root@34.213.204.151
  2. Add a new user called grader: $ sudo adduser grader
     with password: grader and Full name: Udacity Grader
  3. Create a new file under the suoders directory: $ sudo nano /etc/sudoers.d/grader
  4. Fill that newly created file with the following line of text: “grader ALL=(ALL:ALL) ALL”, without quotes
     then save it.
  5. In order to prevent the “sudo: unable to resolve host” error, edit the hosts: $ sudo nano /etc/hosts
     Add the host: 127.0.1.1 ip-10-20-37-65

# 2 - Update all currently installed packages
  1. $ sudo apt-get update
  2. $ sudo apt-get upgrade
  3. Install finger, a utility software to check users’ status: $ apt-get install finger

# 3 - Configure the local timezone to UTC
  1. Open time configuration dialog and set it to UTC with: $ sudo dpkg-reconfigure tzdata
  2. Install ntp daemon ntpd for a better synchronization of the server’s time over the network connection: $ sudo apt-get          install ntp

# 4 - Configure the key-based authentication for grader user
  1. Generate an encryption key on your local machine with: $ ssh-keygen -f ~/.ssh/udacity_key.rsa
  2. Log into the remote VM as root user through ssh and perform the following instructions: $ su grader -> $ cd /home/grader      -> mkdir .ssh -> cd .ssh -> sudo nano authorized_keys
  3. Copy the content of the udacity_key.pub file from your local machine to the /home/grader/.ssh/authorized_keys file you        just created on the remote VM. Then change some permissions:
     $ sudo chmod 700 /home/grader/.ssh.
     $ sudo chmod 644 /home/grader/.ssh/authorized_keys
     Finally change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh
  4. Now you are able to log into the remote VM through ssh with the following command: $ ssh -i ~/.ssh/udacity_key.rsa            grader@34.213.204.151

Note: To access public file on your machine cd .ssh -> provide your machine password

# 5 - Enforce key-based authentication
  1. $ sudo nano /etc/ssh/sshd_config. Find the PasswordAuthentication line and edit it to ‘no’. For me it was already ‘no’
  2. $ sudo service ssh restart

# 6 - Change the SSH port from 22 to 2200
  1. $ sudo nano /etc/ssh/sshd_config. Find the Port line and chat it from 22 to 2200
  2. $ sudo service ssh restart
  3. On the Lightsail account, go to your instance -> ‘Networking’ tab -> Add Custom port 2200 to the list
  4. Now you are able to log into the remote VM through ssh with the following command: $ ssh -i ~/.ssh/udacity_key.rsa -p          2200 grader@34.213.204.151

# 7 - Disable ssh login for root user
  1. $ sudo nano /etc/ssh/sshd_config. Find the PermitRootLogin line and change it from ‘prohibit-password’ to ‘no’
  2. $ sudo service ssh restart

# 8 - Configure the Uncomplicated Firewall (UFW)
  Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP           (port 123)
  1. $ sudo ufw allow 2200/tcp
  2. $ sudo ufw allow 80/tcp
  3. $ sudo ufw allow 123/udp
  4. $ sudo ufw enable

# 9 - Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers
  Install fail2ban in order to mitigate brute force attacks by users and bots alike.
  1. $ sudo apt-get update
  2. $ sudo apt-get install fail2ban
  3. We need the sendmail package to send the alerts to the admin user: $ sudo apt-get install sendmail
  4. Create a file to safely customize the fail2ban functionality: $ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
  5. Open the jail.local and edit it: $ sudo nano /etc/fail2ban/jail.local. Set the destemail field to root@localhost

Note: You can choose to skip this step

# 10 - Configure cron scripts to automatically manage package updates
1. Install unattended-upgrades if not already installed: $ sudo apt-get install unattended-upgrades
2. To enable it, do: $ sudo dpkg-reconfigure --priority=low unattended-upgrades

# 11 - Install Apache, mod_wsgi
  1. $ sudo apt-get install apache2
  2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install mod_wsgi with the              following command: $ sudo apt-get install libapache2-mod-wsgi python-dev
  3. Enable mod_wsgi: $ sudo a2enmod wsgi
  4. $ sudo service apache2 start

# 12 - Install Git
  1. $ sudo apt-get install git
  2. Configure your username: $ git config --global user.name
  3. Configure your email: $ git config --global user.email

# 13 - Clone the Catalog app from Github
  1. Use the following command to move to the /var/www directory: cd /var/www
  2. Create the application directory structure using mkdir as shown: sudo mkdir catalog
  3. Move inside this directory using the following command: cd FlaskApp
  4. Clone the catalog repository from Github: $ git clone https://github.com/AbhaRamchandani/Item-Catalog.git catalog
  5. Install tree and run it. Your folder structure looks like this -
      |Catalog
         -|Catalog
              -|static
              -|templates
￼
  6. To test everything is set up correctly, do the following. We will do the actual test in the next steps.
      - create the init.py file that will contain the flask application logic: sudo nano init.py
      - Add following logic to the file:
  	      from flask import Flask
  	      app = Flask(__name__)
  	      @app.route("/")
  	      def hello():
				      return "Hello, I love Digital Ocean!"
  	      if __name__ == "__main__":
				      app.run()

# 14 - Install virtual environment, Flask and the project’s dependencies
  1. We will use pip to install virtualenv and Flask. If pip is not installed, install it on Ubuntu through apt-get: sudo apt-      get install python-pip
  2. If virtualenv is not installed, use pip to install it using following command: sudo pip install virtualenv
  3. Give the following command (where venv is the name you would like to give your temporary environment): sudo virtualenv        venv
  4. Now, install Flask in that environment by activating the virtual environment with the following command: source                venv/bin/activate
  5. Change permissions to the virtual environment folder: $ sudo chmod -R 777 venv
  6. Give this command to install Flask inside: sudo pip install Flask
  7. Next, run the following command to test if the installation is successful and the app is running: sudo python init.py
     (This is the test we wanted to perform on setup 13, substep 6)
     You should expect to see the following message -
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
  8. Install all the other project’s dependencies: $ pip install bleach httplib2 request oauth2client sqlalchemy psycopg2          sqlalchemy_utils
  9. To deactivate the environment, give the following command:
     deactivate

# 15 - Configure and enable a new virtual host
  1. Create a virtual host conifg file: $ sudo nano /etc/apache2/sites-available/catalog.conf
  2. Paste in the following lines of code:
      <VirtualHost *:80>
          ServerName 34.213.204.151
          ServerAdmin admin@34.213.204.151
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
  3. Enable the new virtual host: $ sudo a2ensite catalog

# 16 - Create the .wsgi File
  1. Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/catalog directory and create a file named              catalog.wsgi with following commands:
     cd /var/www/catalog
     sudo nano catalog.wsgi
  2. Add the following lines of code to the catalog.wsgi file:
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/catalog/")

      from catalog import app as application

# 17 - Install and configure PostgreSQL

  1. Install some necessary Python packages for working with PostgreSQL: $ sudo apt-get install libpq-dev python-dev
  2. Install PostgreSQL: $ sudo apt-get install postgresql postgresql-contrib.\
  3. Postgres is automatically creating a new user during its installation, whose name is ‘postgres’. That is a tusted user        who can access the database software. So let’s change the user with: $ sudo su - postgres, then connect to the database        system with $ psql
  4. Create a new user called ‘catalog’ with his password: # CREATE USER catalog WITH PASSWORD ‘catalog’;.
  5. Give catalog user the CREATEDB capability: # ALTER USER catalog CREATEDB;
  6. Create the ‘catalog’ database owned by catalog user: # CREATE DATABASE catalog WITH OWNER catalog;
  7. Connect to the database: # \c catalog
  8. Revoke all rights: # REVOKE ALL ON SCHEMA public FROM public;
  9. Lock down the permissions to only let catalog role create tables: # GRANT ALL ON SCHEMA public TO catalog;
  10. Log out from PostgreSQL: # \q. Then return to the grader user: $ exit
  11. Inside the Flask application, the database connection is now performed with:
      engine = create_engine(‘postgresql://catalog:catalog@localhost/catalog’)
      So, update this line in the following files under catalog: database_setup.py, lotsofmenus.py, project.py
  12. Setup the database with: $ python /var/www/catalog/catalog/database_setup.py
  13. To prevent potential attacks from the outer world we double check that no remote connections to the database are               allowed. Open the following file: $ sudo nano /etc/postgresql/9.5/main/pg_hba.conf and edit it, if necessary, to make it       look like this:
      local all postgres peer
      local all all peer
      host all all 127.0.0.1/32 md5
      host all all ::1/128 md5

  Note: I didn’t have to make any changes here

# 18 - Add the following links to ‘Authorized JavaScript origins’ in Google Developer’s Console -
  http://ec2-34-213-204-151.us-west-2.compute.amazonaws.com
  http://34.213.204.151

# 19 - Restart Apache to launch the app
  $ sudo service apache2 restart
