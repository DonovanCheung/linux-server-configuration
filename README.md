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
