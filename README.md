# LINUX-PROJECT
# Create New User Name : GRADER
1. sudo adduser grader

2. Give User permission to sudo: sudo nano /etc/sudoers.d/grader 

Add: grader ALL=(ALL:ALL) ALL
  
# Update all currently installed packages
1. $ sudo apt-get update 

2. $ sudo apt-get upgrade

# Change the SSH port from 22 to 2200
1. $ sudo nano /etc/ssh/sshd_config. Edit Port Line from 22 to 2200.

2. Disable root login. Change to ‘no’ for PermitRootLogin

3. $ sudo service ssh restart.

# Generate SSH-KEYGEN
1. To generate key pair, locally (not connect with server ssh yet) : 

ssh-keygen -f ~/.ssh/graderkey 

2. Log in as Grader :sudo su - <new_user>

(ssh grader@"public_IP_address" -p 2200)

3. Create directory: mkdir .ssh

4. Create new file within above directory to store public keys : touch .ssh/authorized_keys

5. Get key from local: cat .ssh/graderkey.pub. Copy

6. Open file and paste key into:  nano .ssh/authorized_keys

7. Set permission : chmod 700 .ssh

8. And then cdmod 644 .ssh/authorized_keys

# Force all user log in using key pair:
1. Back to grader server: sudo nano /etc/ssh/sshd-config

2. Look for this line : PasswordAuthentication. Change ‘yes’ to ‘no’. SAVE

3. Restart to run as new configuration: sudo service ssh restart

# Configure the Uncomplicated Firewall (UFW)
Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. $ sudo ufw default deny incoming

2. $ sudo ufw default allow outgoing

3. $ sudo ufw allow 2200/tcp

4. $ sudo ufw allow 80/tcp

5. $ sudo ufw allow 123/udp

6. $ sudo ufw enable.

# Install Apache:
1. $ sudo apt-get install apache2

2. $ sudo apt-get install libapache2-mod-wsgi

# Install PostgreSQL
1. $ sudo apt-get install postgresql postgresql-contrib

# Install Git
1. $ sudo apt-get install git
