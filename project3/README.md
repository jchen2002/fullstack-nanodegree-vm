
# Linux Server Configuration Project
This project install and configure a Linux Server by using Amazon Lightsail.
And deploy and configure the second web application project `catalog` in it.

##########Set up Ubuntu and SSH to Ubuntu##########
1. create Ubuntu instance from AWS lightsail and create a static ip for this instance
my ubuntu server static ip is: 100:21:99:215

http://lightsail.aws.amazon.com
create ubuntu instance
create static_ip

2. Enabled port `2200` for the access in `lightsail` (Networking -> Firewall panel -> Add New rule).

3. Go to ubuntu instance to download private key "*****.pem" under SSH for user "ubuntu", and
change the name to lightsail.pem.

4. Open a new local terminal from your machine,  make .ssh folder under current user,
and copy the "lightsail.pem" to .ssh folder and chmod.
`mkdir /Users/kaiyanchen/.ssh`
`chmod 700 .ssh`
`chmod 600 lightsail.pem`

5. SSH to the Ubuntu sever and su to root user in terminal, sudo to root user and do the rest of
tasks under root user.
   `ssh -i lightsail.pem ubuntu@100-21-99-215`
   then `sudo su - root`

6. Changed server ssh port to 2200.
   `vi /etc/ssh/sshd_config`
   Find port, change it to
   `Port 2200`

7. Allowed and enabled the ports(`80,2200,123`) by using the following command:
   `ufw allow 80`
   `ufw allow 2200`
   `ufw allow 123`
   `ufw enable`

8. Disable login as root remotely by changing PermitRootLogin property to no.
  'cd /etc/ssh/sshd_config`
  `PermitRootLogin no`

9. Rebooted the server in AWS lightsail account `lightsail`.

10. Checked if success change the ssh port to 2200 by typing in the terminal:
   `ssh -i lightsail.pem ubuntu@52.89.190.205 -p 2200`


##########create user "grader" and give user "grader" root permission##########
1. After ssh to ubuntu, create new user "grader"
`sudo su - root`
`adduser grader`

give grader root permission
`vi /etc/sudoers.d/grader`
Added below line to the file.
`grader ALL=(ALL) NOPASSWD:ALL`

2. To solve "unable to resolve host error",
edit the hosts by $ sudo nano /etc/hosts, and add the following two lines:
"100-21-99-215" is ubuntu's static ip

127.0.0.1 localhost
127.0.0.1 ip-100-21-99-215

3. Open a new terminal from your local machine, under current user default folder,
generate a public/private rsa key pair,
`ssh-keygen -f ~/.ssh/kcudacity_key.rsa`

4. Copy public key and and add to user "grader", open file
`more  ~/.ssh/kcudacity_key.rsa.pub`, copy the content

5. Add public key to user "grader",  go back to ubuntu terminal, paste the public keys
to authorized_key file under /home/grader
`sudo su - root`
using root user
`cd /home/grader`
`mkdir .ssh`
`vi authorized_keys`

6. SSH to server using "grader" account.


##########now loggin in via port 2200, and install software ##########
1. Open a new local terminal, ssh to server
`ssh -i ~/.ssh/kcudacity_key.rsa -p 2200 grader@100.21.99.215`

ssh -i ~/.ssh/kcudacity_key.rsa -p 2200 grader@100.21.99.215

ssh -i ~/.ssh/id_rsa -p 2200 grader@100.21.99.215


authorized_keys

2. Install software
update current installed software before installation
`sudo apt-get update`

install python
`sudo apt-get install python-dev`

install apache
`sudo apt-get install apache2`

Installing mod_wsgi for python3
`sudo apt-get install apache2 libapache2-mod-wsgi-py3`

install postgres database
`sudo apt-get install postgresql postgresql-contrib`

install packages and modules
`sudo apt-get install libapac`
`sudo -H pip3 install sqlalchemy`


Rebooted the ubuntu server in AWS lightsail account `lightsail`.
`sudo service ssh restart`

restart apache2
`apache2ctl restart`

to reload the webapps
`sudo service apache2 reload`

##########deploy catalog application##########
1. Upload source code
`sftp -i ~/.ssh/kcudacity_key.rsa -P 2200 grader@100.21.99.215`
`put catalog.zip /home/grader/catalog`

2. From the ssh terminal root user, copy /home
`mkdir /var/www/catalog`
`mkdir /var/www/catalog/catalog`
`cd /var/www/catalog`
`cp /home/grader/catalog/catalog.zip .`
`unzip catalog.zip`


##########setup database and user account#######
1. Login to postgres super user account
`sudo su - postgres`
postgres@ip-172-26-4-192:~$`psql`
postgres=#`CREATE USER kctest WITH PASSWORD 'kcpwd;`

create a new database, and assign it to project user
`createdb test1`
`alter database test1 owner to kctest;`

##########update client_secrets.json ##########
1. find my server domain by public ip: 100.21.99.215
https://whatismyipaddress.com/ip-hostname

2. Go to console.developers.google.com to include the server domain name
to the following two locations.

Add the following to "Authorized JavaScript origins"
http://ec2-100-21-99-215.us-west-2.compute.amazonaws.com

Add the following to "Authorized redirect URIs"
http://ec2-100-21-99-215.us-west-2.compute.amazonaws.com/gconnect

3. update new client_secrets.json and chmod
'chmod 644 /var/www/catalog/catalog/client_secrets.json'

#########update source code#########
1. SSH to server from local terminal
`ssh -i ~/.ssh/kcudacity_key.rsa -p 2200 grader@100.21.99.215`

2. Create catalog.wsgi under /var/www/catalog
'sudo su - root'
'cd /var/www/catalog'

=======catalog.wsgi=====
#!/usr/bin/python3
import os, sys

PROJECT_DIR = '/var/www/catalog'
sys.path.insert(0, PROJECT_DIR)

sys.path.append('/var/www/catalog/catalog')

from catalog import app as application
application.secret_key='super_secret_key'
========================

3. Update engine to use postgres database in catalog_dbsetup.py, loaddb.py,
engine = create_engine('postgresql://kctest:kcpwd@localhost:5432/test1');'

4. create tables and load data to db
`python3 catalog_setup.py`
`python3 loaddb.py`

5, Rename application.py to __init__.py, and update to run application at port 80
`cd /var/www/catalog/catalog`
`mv application.py __init__.py`

========code change in __init__.py
if __name__ == '__main__':
    app.secret_key = 'super_secret_key'
    app.debug = True

    #app.run(host='0.0.0.0', port=8000, threaded=False)
    app.run()

=======open __init__.py, update client_secrets.json to full location
CLIENT_ID = json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope = '')


##########Config project##########
1. Add catalog.conf to the following location:

`sudo su - root`
`cd /etc/apache2/sites-available`
`vi catalog.conf`, add the following to catalog.conf

============catalog.conf==========
<VirtualHost *:80>
		ServerName 100-21-99-215
		ServerAdmin admin@100-21-99-215
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
	  <Directory /var/www/catalog/catalog/templates>
      Order allow,deny
      Allow from all
    </Directory>
    <Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
=========================


##########Run project ##########
1. run project from browser
http://ec2-100-21-99-215.us-west-2.compute.amazonaws.com/catalog

2. view serve error log from:
/var/www/log/apache2/error.log

3. restart apache server:
`sudo apache2ctl restart`

reference:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://wixelhq.com/blog/how-to-install-postgresql-on-ubuntu-remote-access
