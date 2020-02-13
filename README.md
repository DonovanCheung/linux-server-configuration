# Linux Server Configuration
This is the final project for Udacity's Full Stack Web Developer Nanodegree.

This page explains how to set up a Linux Ubuntu distribution on a virtual machine for the purposes of creating a server to host a web application.

- The Linux distribution is Ubuntu 16.04 LTS.
- The virtual private server is Amazon Lighsail.
- The web application is my Electronics Catalog project created earlier for Part 4 of this Nanodegree program.
- The database server is PostgreSQL.

You can visit http://http://100.26.251.188/ or http://ec2-100-26-251-188.compute-1.amazonaws.com/ anytime to view the fully deployed website.

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
4. Now you can connect to the instance via the terminal with the command: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@100.26.251.188`, where `100.26.251.188` is the public IP address of the instance.

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
7. In your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@100.26.251.188`, where `100.26.251.188` is the public IP address of the instance.

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
10. Restart Apache: `sudo service apache2 restart`.
11. Reupdate the installed packages:
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo shutdown -r now
```
   - Logged back in, and I now see this message:
```
Alains-MBP:udacity-linux-server-configuration boisalai$ ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@100.26.251.188
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
