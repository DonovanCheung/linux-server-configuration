# Linux Server Configuration
This is the final project for Udacity's Full Stack Web Developer Nanodegree.

This page explains how to set up a Linux Ubuntu distribution on a virtual machine for the purposes of creating a server to host a web application.

- The Linux distribution is Ubuntu 16.04 LTS.
- The virtual private server is Amazon Lighsail.
- The web application is my Electronics Catalog project created earlier for Part 4 of this Nanodegree program.
- The database server is PostgreSQL.

You can visit http://3.218.247.29 or http://ec2-3-218-247-29.compute-1.amazonaws.com/ anytime to view the fully deployed website.

## Step 1: Create a Ubuntu server with Amazon Lightsail
1. Create an Amazon Web Services account and log into [Amazon Lightsail](https://lightsail.aws.amazon.com/).
2. After logging in, click the `Create instance` button.
3. Select `Linux/Unix` for platform and `OS Only` & `Ubuntu 16.04 LTS` for blueprint.
4. Select the $3.50 instance plan as it is free to run your instance for the first month.
5. Name your instance however you'd like.
6. Click the `Create instance` button.
7. Wait for the instance to start up.

## Step 2: SSH into the server
1. Go into the `Account page` on Amazon Lightsail, click on the `SSH keys` tab and download the Default Key.
2. Rename the file to `lightsail_key.rsa` and move it into the local `~/.ssh` directory.
3. Open the terminal and run: `chmod 600 ~/.ssh/lightsail_key.rsa`.
4. Now you can connect to the instance via the terminal with the command: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@3.218.247.29`, where `3.218.247.29` is the public IP address of the instance.

## Step 3: Configure the server
1. Update and upgrade the installed packages with the command:
```
sudo apt-get update
sudo apt-get upgrade
```
2. Edit the `/etc/ssh/sshd_config` file to change the SSH port from 22 to 2200: `sudo vim /etc/ssh/sshd_config`.
   - Press the `I` button on the keyboard to start insert mode.
3. On line 5, change the port number from `22` to `2200`.
4. Press the `ESC` button and type `:wq` and press the `Enter` key to save and exit vim.
5. Restart SSH: `sudo service ssh restart`.

## Step 4: Configure the Uncomplicated Firewall (UFW)
1. Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw status                  # The UFW should be inactive.
sudo ufw default deny incoming   # Deny any incoming traffic.
sudo ufw default allow outgoing  # Enable outgoing traffic.
sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
sudo ufw allow www               # Allow HTTP traffic in.
sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
```
2. Turn UFW on: `sudo ufw enable` and confirm with `y`. The output should look like this:
```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```
3. Check the status of UFW: `sudo ufw status`. The output should look like this:
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
4. Exit the SSH connection: `exit`.
5. Click on the `Networking` tab on the Amazon Lightsail Instance and change the `Firewall` configuration to match the internal firewall settings above. 
6. Remove the default SSH port 22 and add:
   - HTTP (TCP) Port 80
   - Custom (UDP) Port 123
   - Custom (TCP) Port 2200
   - These three should be the only allowed connection in the `Firewall` configuration.
7. In your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@3.218.247.29`, where `3.218.247.29` is the public IP address of the instance.

## Step 5: Add more security measures
1. Install Fail2Ban: `sudo apt-get install fail2ban`.
2. Install sendmail for email notice: `sudo apt-get install sendmail iptables-persistent`.
3. Create a copy of a file: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
4. Change the settings in `/etc/fail2ban/jail.local` file:
   - If it isn't alreadey, set `bantime = 600`
   - Set `destemail = ` to your email address
   - Set `action = %(action_mwl)s`
   - Under `[sshd]`, change `port = ssh` to `port = 2200`.
5. Restart the service: `sudo service fail2ban restart`.
6. Enable automatic (security) updates: `sudo apt-get install unattended-upgrades`.
7. Edit `/etc/apt/apt.conf.d/50unattended-upgrades`, uncomment the line `${distro_id}:${distro_codename}-updates` and save.
8. Modify `/etc/apt/apt.conf.d/20auto-upgrades` file so that the upgrades are downloaded and installed every day:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```
9. Enable it: `sudo dpkg-reconfigure --priority=low unattended-upgrades`.
10. Reupdate the installed packages:
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo shutdown -r now
```
   - Logged back in, and I now see this message:
```
Alains-MBP:udacity-linux-server-configuration dc355h$ ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@3.218.247.29
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-1039-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.
```

# Step 6: Create `grader` with the proper permissions
1. While ssh-ed in as `ubuntu`, add a new user `grader`: `sudo adduser grader`.
2. Enter a password (twice) and fill out information for this new user.
3. Edit the sudoers file: `sudo visudo`.
   - Search for the line that looks like this:
```
root    ALL=(ALL:ALL) ALL
```
   - Below this line, add a new line to give sudo privileges to grader user.
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```
   - Save and exit.
4. Verify that grader has sudo permissions.
   - Run `su - grader` and enter the password.
   - Run `sudo -l` and enter the password again. 
   - The output should look like this:
```
Matching Defaults entries for grader on ip-172-26-13-170.us-east-2.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
    (ALL : ALL) ALL
```
5. On a separate terminal connected to the local machine:
   - Run `ssh-keygen`.
   - Enter the file location in which to save the key (I gave it the name `grader_key`) in the local `~/.ssh` directory.
   - Enter in a passphrase twice. Two files will be generated (`~/.ssh/grader_key` and `~/.ssh/grader_key.pub`).
   - Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file.
   - Log back into to the grader's virtual machine.
6. On the grader's virtual machine:
   - Create a new directory called `~/.ssh`: `mkdir ~/.ssh`.
   - Run `sudo vim ~/.ssh/authorized_keys` and paste the content into this file, save and exit.
   - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`.
   - Edit the `/etc/ssh/sshd_config` file so that:
     -`PermitRootLogin prohibit-password` is set to `PermitRootLogin yes`. 
     -`PasswordAuthentication no` is set to `PasswordAuthentication yes`.
   - Restart SSH: `sudo service ssh restart`
7. On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@3.218.247.29`.

## Step 7: Installing Apache and PostgreSQL
1. While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata`.
2. Install Apache: `sudo apt-get install apache2`.
3. Enter public IP of the Amazon Lightsail instance into browser (`3.218.247.29`). If Apache is working, you should be able to see the Apache2 Ubuntu Default Page.
4. My project is built with Python3 so I needed to install the Python3 mod_wsgi package: `sudo apt-get install libapache2-mod-wsgi-py3`.
   - Enable mod_wsgi using: `sudo a2enmod wsgi`.
5. Install PostgreSQL: `sudo apt-get install postgresql`.
6. PostgreSQL should not allow remote connections so if you `sudo cat /etc/postgresql/9.5/main/pg_hba.conf`, you should see:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
7. Switch to the `postgres` user: `sudo su - postgres`.
8. Open PostgreSQL interactive terminal: `psql`.
9. Create the `catalog` user with a password and give them the ability to create databases:
```
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER ROLE catalog CREATEDB;
```
10. List the existing roles: `\du`. The output should look like this:
```
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```
   - Exit psql: `\q`.
11. Switch back to the `grader` user: `exit`.
12. Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
13. Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
    - Search for the lines that looks like this:
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```

   - Below this line, add a new line to give sudo privileges to `catalog` user.

```
catalog  ALL=(ALL:ALL) ALL
```
   - Save and exit using CTRL+X and confirm with Y.
14. Verify that `catalog` has sudo permissions. Run `su - catalog`, enter the password, run `sudo -l` and enter the password again. The output should look like this:
```
Matching Defaults entries for catalog on ip-172-26-13-170.us-east-2.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User catalog may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
    (ALL : ALL) ALL
```
15. While logged in as `catalog`, create a database: `createdb catalog`.
16. Run `psql` and then run `\l` to see that the new database has been created. The output should look like this:
```
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```
17. Exit psql: `\q`.
18. Switch back to the `grader` user: `exit`.

## Step 8: Deploying the Catalog Project
1. While logged in as `grader`, install `git`: `sudo apt-get install git`.
2. Create the project directory: `mkdir /var/www/catalog/`.
3. Change to that directory and clone the catalog project: `sudo git clone https://github.com/DonovanCheung/fullstack-nanodegree-vm.git catalog`.
4. From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
5. Change to the `/var/www/catalog/catalog` directory.
6. Rename the `project.py` file to `__init__.py` using: `mv project.py __init__.py`.
   - In `__init__.py`, replace line 27:
```
# app.run(host="0.0.0.0", port=8000, debug=True)
app.run()
```
   - In `database_setup.py`, replace line 9:
```
# engine = create_engine("sqlite:///catalog.db")
engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
```
where `PASSWORD` is the password for the `catalog` user
7. Go to the Google [Cloud Platform](https://console.cloud.google.com/).
8. Click `APIs & services` on left menu.
9. Click `Credentials`.
10. Create an OAuth Client ID (under the Credentials tab), and add http://100.26.251.188 and http://ec2-100-26-251-188.us-east-2.compute.amazonaws.com as authorized JavaScript origins.
Add http://ec2-100-26-251-188.us-east-2.compute.amazonaws.com/oauth2callback as authorized redirect URI.
11. Download the corresponding JSON file onto the local machine, open it and copy the contents.
12. Open `/var/www/catalog/catalog/client_secret.json` and paste the previous contents into this file.

## Step 9: Set up the virtual environment and host
1. While logged in as `grader`, install `pip`: `sudo apt-get install python3-pip`.
2. Install the virtual environment: `sudo apt-get install python-virtualenv`
3. Change to the `/var/www/catalog/catalog/` directory.
4. Create the virtual environment: `sudo virtualenv -p python3 venv3`.
5. Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.
6. Activate the new environment: `. venv3/bin/activate`.
7. Install the following dependencies:
```
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2
```
8. Run `python3 __init__.py` and you should see:
```
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
9. Deactivate the virtual environment: `deactivate`.
10. To use Python3, add the following line in the `/etc/apache2/mods-enabled/wsgi.conf` file below the `#WSGIPythonPath directory|directory-1:directory-2:...` line:
```
WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
```
11. Create `/etc/apache2/sites-available/catalog.conf` and add the following lines to configure the virtual host:
```
<VirtualHost *:80>
    ServerName 100.26.251.188
  ServerAlias ec2-100-26-251-188.us-west-2.compute.amazonaws.com
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
12. Enable virtual host: `sudo a2ensite catalog`.
13. Reload Apache: `sudo service apache2 reload`.
14. Create `/var/www/catalog/catalog.wsgi` file add the following lines:
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
application.secret_key = "..."
```
15. Restart Apache: `sudo service apache2 restart`.

## Step 10: Set up the database
1. Edit `/var/www/catalog/catalog/database_setup.py`.
2. Add this at the beginning of the file after `import sys`:
```
sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.5/site-packages") 
```
3. From `/var/www/catalog/catalog/` directory, activate the virtual environment: `. venv3/bin/activate`.
4. Run: `python database_setup.py' and then 'python lotsoftech.py`.
5. Deactivate the virtual environment: `deactivate`.

## Step 11: Launch the web application
1. Disable the default Apache site: `sudo a2dissite 000-default.conf`. 
2. Reload Apache: `sudo service apache2 reload`.
3. Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
4. Restart Apache again: `sudo service apache2 restart`.
5. Open your browser to http://3.218.247.29 or http://ec2-3-218-247-29.compute-1.amazonaws.com/.

# References
This project could not have been completed without the help of a fellow Udacitian. Their project github which this server was heavily inspired by can be found here: https://github.com/boisalai/udacity-linux-server-configuration
