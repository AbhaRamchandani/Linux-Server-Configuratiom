Completed: 2-Jan-2018

# Linux-Server-Configuratiom
Host Item Catalog Web App on Lightsail Ubuntu instance

Project Info

IP address: 34.213.204.151

Accessible SSH port: 2200.

Application URL: http://ec2-34-213-204-151.us-west-2.compute.amazonaws.com/

# 1 - Create a new user named grader and grant this user sudo permissions.
  1. Log into the remote VM as root user through ssh: $ ssh root@34.213.204.151
  2. Add a new user called grader: $ sudo adduser grader
      - with password: grader and Full name: Udacity Grader
  3. Create a new file under the suoders directory: $ sudo nano /etc/sudoers.d/grader
  4. Fill that newly created file with the following line of text: “grader ALL=(ALL:ALL) ALL”, without quotes
     then save it.
  5. In order to prevent the “sudo: unable to resolve host” error, edit the hosts: $ sudo nano /etc/hosts
      - Add the host: 127.0.1.1 ip-10-20-37-65

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
      - $ sudo chmod 700 /home/grader/.ssh.
      - $ sudo chmod 644 /home/grader/.ssh/authorized_keys
      - Finally change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh
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
      - catalog
         - catalog
            - static
            - templates
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

	activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	from catalog import app as application
	application.secret_key = 'super_secret_key'
  3. Now your directory structure should look like this:
      - catalog
         - catalog
            - static
            - templates
	    .
  	    .
	    .
	    
            - local
	     - catalog.wsgi
	   - html
	
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
  
# 19 - I specifically had to perform the following steps to make this project work
 1. Updated catalog.conf to remove ServerName - 	
 	<VirtualHost *:80>
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
 
 Note: In the Virtual Host file, the ServerName directive is not needed in for this project and if included here should not be set to the IP address as is currently used, e.g. ServerName 34.213.204.151
 
 2. Updated catalog.wsgi as follows - 
 #!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py’
execfile(activate_this, dict(file=activate_this))

from catalog import app as application
application.secret_key = ‘super_secret_key’

Note: Running "sudo python project.py is OK to test your app temporarily on a port like 5000, but it will not involve the Apache web server running on port 80. To run the app on port 5000, you will need to make sure that traffic on that port is allowed through the external Lightsail firewall and the internal UFW firewall.
What happens is Apache calls the WSGI file which then imports the Flask app object as the variable application. Requests are passed to your Flask app, processed, and the responses are served to the user by Apache.
If you want to use the venv virtual environment from the command line, you need to enter it by running “source venv/bin/activate”. To get Apache to use the virtual environment, you need to add a couple of lines to the WSGI file. Details here:
 
 3. To stop the default Apache page from being displayed when loading the domain name of the server, disable the default Apache configuration - sudo a2dissite 000-default.conf
 
 4. Your app will be run by Apache and your app should be structured as a Python Package, which requires that there is a file named __init__.py in the directory structure. So, re-named project.py file to __init__.py
 
 5. Updated __init__.py file to include full path of the client_secrets.json file
 
 Note: Think about how files are referenced in the Virtual Host file, is it only a file name or the full path to the file?
 
 6. Downloaded fresh client_secrets.json file from Google Developers Console and moved it to catalog project folder on the instance
 
 Note: For the "client_secrets.json" file, it works best to make the changes in the Google console and then re-download the file since oftentimes editing the file by hand introduces some kind of minor error that is hard to detect by that causes errors in the app. It is a few more clicks to generate the file from the download, but well worth it.
 
 7. At regular intervals, to resolve errors, checked error.log file - sudo tail /var/log/apache2/error.log
Also, checked what sites are now enabled using: ls -alh /etc/apache2/sites-enabled/
 
# 20 - Restart Apache to launch the app
  Restarted the instance from lightsail
  $ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@34.213.204.151
  $ cd /var/www/catalog/catalog
  $ source venv/bin/activate
  $ sudo service apache2 restart
  
# You can now access the app @ - 
http://ec2-34-213-204-151.us-west-2.compute.amazonaws.com/
OR
http://34.213.204.151/

# References
1. https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379
2. https://discussions.udacity.com/t/lightsail-project-internal-server-error/502337/12
3. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-four-%E2%80%93-configure-and-enable-a-new-virtual-host
4. https://discussions.udacity.com/t/problems-with-the-digital-ocean-tutorial/336376 
5. http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/#working-with-virtual-environments
6. https://manpages.debian.org/jessie/apache2/a2ensite.8.en.html
7. http://docs.python-guide.org/en/latest/dev/virtualenvs/
8. http://angus.readthedocs.io/en/2014/amazon/transfer-files-between-instance.html
9. Google Searches
10. I would like to thank Udacity Mentors @trish, @greg and @swooding
11. I referred many other projects on github. This one had the most clear instructions, although not everything worked for me and I did lot of Google Searches and asked questions on forums: https://github.com/iliketomatoes/linux_server_configuration
