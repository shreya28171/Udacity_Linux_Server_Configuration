# Udacity - Linux Server Configuration Project

An Udacity Full Stack Web Developer II Nanodegree Project.

## About

This tutorial will guide you through the steps to take a baseline installation of a Linux server and prepare it to host your Web applications. You will then secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing Flask-based Web applications onto it.

In this project, I have set up an Ubuntu 18.04 image on a DigitalOcean droplet. The technical details of the server as well as the steps that have been taken to set it up can be found in the succeeding sections.

### Technical Information About the Project

- **Server IP Address:** 138.197.214.32
- **SSH server access port:** 2200
- **SSH login username:** grader
- **Application URL:** http://138.197.214.32.xip.io

## Steps to Set up the Server

### 1. Creating the RSA Key Pair

On your local machine, you will first have to set up the public and private key pair. This key pair will be used to authenticate yourself while securely logging in to the server via SSH. The private key will be kept with you in your local machine, and the public key will be stored in the server.

To generate a key pair, run the following command:

   ```console
   $ ssh-keygen
   ```

When it asks to enter a passphrase, you may either leave it empty or enter some passphrase. A passphrase adds an additional layer of security to prevent unauthorized users from logging in.

The whole process would look like this:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/shreya/.ssh/id_rsa): /home/shreya/.ssh/udacity_project
Created directory '/home/shreya/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/shreya/.ssh/udacity_project.
Your public key has been saved in /home/shreya/.ssh/udacity_project.pub.
The key fingerprint is:
SHA256:JnpV8vpIZqstoTm8aaDAqVmGSF/tGhtDfXAfL2fs/U8 shreya@shreya-VirtualBox
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|       . . .     |
|      o + o +    |
| .   o o = o =   |
|+.o o o S . = .  |
|+o+. =.= .   . . |
|o= o.oB.=       E|
|+   *=.= +     ..|
|   .oo.o+ .     o|
+----[SHA256]-----+
```

You now have a public and private key that you can use to authenticate. The public key is called `udacity_project.pub` and the corresponding private key is called `udacity_project`. The key pair is stored inside the `~/.ssh/` directory.

### 2. Setting Up a DigitalOcean Droplet

1. Log in or create an account on [DigtalOcean](https://cloud.digitalocean.com/login).

2. Go to the Dashboard, and click **Create Droplet**.

3. Choose **Ubuntu 18.04 x64** image from the list of given images.

4. Choose a preferred size. In this project, I have chosen the **1GB/1 vCPU/25GB** configuration.

5. In the section **Add Your SSH Keys**, paste the content of your public key, `udacity_project.pub`:

6. Click **Create** to create the droplet. This will take some time to complete. After the droplet has been created successfully, a public IP address will be assigned. In this project, the public IPv4 address that I have been assigned is 138.197.214.32.

### 3. Logging In as `root` via SSH and Updating the System

#### 3.1. Logging in as `root` via SSH

As the droplet has been successfully created, you can now log into the server as `root` user by running the following command in your host machine:

```
  $ ssh root@138.197.214.32
```

This will look for the private key in your local machine and log you in automatically if the corresponding public key is found on your server. 

#### 3.2. Updating the System

Run the following command to update the virtual server:

```
 # apt update && apt upgrade
```

This will update all the packages. If the available update is a kernel update, you might need to reboot the server by running the following command:

```
# reboot
```

### 4. Changing the SSH Port from 22 to 2200

1. Open the `/etc/ssh/sshd_config` file with `nano` or any other text editor of your choice:

   ```
   # nano /etc/ssh/sshd_config
   ```

2. Find the line `#Port 22` (would be located around line 13) and change it to `Port 2200`, and save the file.

3. Restart the SSH server to reflect those changes:
   ```
   # service ssh restart
   ```

4. To confirm whether the changes have come into effect or not, run:
   ```
   # exit
   ```

   This will take you back to your host machine. After you are back to your local machine, run:

   ```
   $ ssh root@138.197.214.32 -p 2200
   ```
   
   You should now be able to log in to the server as `root` on port 2200. The `-p` option explicitly tells at what port the SSH server operates on. It now no more operates on port number 22. 

### 5. Configure Timezone to Use UTC

To configure the timezone to use UTC, run the following command:

```
# sudo dpkg-reconfigure tzdata
```

It then shows you a list. Choose ``None of the Above`` and press enter. In the next step, choose ``UTC`` and press enter.

You should now see an output like this:

```
Current default time zone: 'Etc/UTC'
Local time is now:      Thu May 24 11:04:59 UTC 2018.
Universal Time is now:  Thu May 24 11:04:59 UTC 2018.
```

### 6. Setting Up the Firewall

Now we would configure the firewall to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):

```
# ufw allow 2200/tcp
# ufw allow 80/tcp
# ufw allow 123/udp
```

To enable the above firewall rules, run:

```
# ufw enable
```

To confirm whether the above rules have been successfully applied or not, run:

```
# ufw status
```

You should see something like this:

```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```

### 7. Creating the User `grader` and Adding it to the `sudo` Group

#### 7.1. Creating the User `grader`

While being logged into the virtual server, run the following command and proceed:

```
  # adduser grader
```

The output would look like this:

```
  Adding user `grader' ...
  Adding new group `grader' (1000) ...
  Adding new user `grader' (1000) with group `grader' ...
  Creating home directory `/home/grader' ...
  Copying files from `/etc/skel' ...
  Enter new UNIX password:
  Retype new UNIX password:
  passwd: password updated successfully
  Changing the user information for grader
  Enter the new value, or press ENTER for the default
	  Full Name []: Grader
	  Room Number []:
	  Work Phone []:
	  Home Phone []:
	  Other []:
  Is the information correct? [Y/n]
```

**Note**: Above, the UNIX password I have entered for the user `grader` is, `root`. 

#### 7.2. Adding `grader` to the Group `sudo`

Run the following command to add the user `grader` to the `sudo` group to grant it administrative access:

```
  # usermod -aG sudo grader
```

### 8. Adding SSH Access to the user `grader`

To allow SSH access to the user `grader`, first log into the account of the user `grader` from your virtual server:

```
# su - grader
```

You should see a prompt like this:

```console
grader@ubuntu-s-1vcpu-1gb-sgp1-01:~$
```

Now enter the following commands to allow SSH access to the user `grader`:

```
$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh/
$ touch authorized_keys
$ chmod 644 authorized_keys
```

After you have run all the above commands, go back to your local machine and copy the content of the public key file `~/.ssh/udacity_project.pub`. Paste the public key to the server's `authorized_keys` file using `nano` or any other text editor, and save.

After that, run `exit`. You would now be back to your local machine. To confirm that it worked, run the following command in your local machine:

ssh grader@138.197.214.32 -p 2200


You should now be able to log in as `grader` and would get a prompt to enter commands.

Next, run `exit` to go back to the host machine and proceed to the following step to disable `root` login.

### 9. Disabling Root Login

1. Run the following command on your local machine to log in as `root` in the server:
   ```
   $ ssh root@138.197.214.32 -p 2200
   ```

2. After you are logged in, open the file `/etc/ssh/sshd_config` with `nano`:
   ```
   # nano /etc/ssh/sshd_config
   ```

3. Find the line `PermitRootLogin yes` and change it to `PermitRootLogin no`.

4. Restart the SSH server:
   ```
   # service ssh restart
   ```

5. Terminate the connection:
   ```
   # exit
   ```

6. After you are back to the host machine, when you try to log in as `root`, you should get an error:

### 10. Installing Apache Web Server to serve a Python mod_wsgi application

To install the Apache Web Server, run the following command after logging in as the `grader` user via SSH:

```
$ sudo apt update
$ sudo apt install apache2
```

To confirm whether it successfully installed or not, enter the URL `http://138.197.214.32` in your Web browser:

If the installation has succeeded, you should see the Apache default Webpage.
```
Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
`sudo service apache2 restart`

```

### 11. Installing and Configuring PostgreSQL

#### 11.1. Installing PostgreSQL

 Install PostgreSQL:

   ```
   $ sudo apt install postgresql-10
   ```

#### 11.2. Configuring PostgreSQL

1. Log in as the user `postgres` that was automatically created during the installation of PostgreSQL Server:

   ```
   $ sudo su - postgres
   ```

2. Open the `psql` shell:

   ```
   $ psql
   ```

3. This will open the `psql` shell. Now type the following commands one-by-one:

   ```sql
   postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER catalog;
   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   ```

   Then exit from the terminal by running `\q` followed by `exit`.

### 12 Install git, clone and setup your Catalog App project

 `sudo apt-get install git`
- to install git
1. Move into cd /var/www/
2. Create a directory called `FlaskApp` and change the working directory to it:

   ```
   $ sudo mkdir FlaskApp
   $ cd FlaskApp/
   ```

3. Clone this repository directory `FlaskApp`:

   ```
   $ sudo git clone https://github.com/shreya28171/Udacity-Item-Catalog-Project.git FlaskApp
   ```

4. Move inside the newly created directory:

   ```
   $ cd FlaskApp/
   ```

5. Checkout to the `development` branch:

   ```
   $ sudo git checkout development
   ```

6. Install pip 
```
sudo apt-get install python-pip
``` 

7. Install required packages:

   ```
   $ sudo pip install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary
   ```

### 13. Setting Up Virtual Hosts

1. Run the following command in terminal to set up a file called `FlaskApp.conf` to configure the virtual hosts:

   ```
   $ sudo nano /etc/apache2/sites-available/FlaskApp.conf
   ```

2. Add the following lines to it:

   ```

   <VirtualHost *:80>
      ServerName 138.197.214.32
      ServerAlias 178.128.66.43.xip.io
      ServerAdmin shreya28171@gmail.com
      WSGIDaemonProcess FlaskApp python-path=/var/www \
        python-home=/var/www/FlaskApp/venv
      WSGIProcessGroup FlaskApp
      WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
      <Directory /var/www/FlaskApp/FlaskApp/>
          Require all granted
      </Directory>
      Alias /static /var/www/FlaskApp/FlaskApp/static
      <Directory /var/www/FlaskApp/FlaskApp/static/>
          Require all granted
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

   ```

3. Enable the virtual host:

   ```
   $ sudo a2ensite FlaskApp
   ```

4. Restart Apache server:

   ```
   $ sudo service apache2 restart
   ```

5. Creating the .wsgi File

   Apache uses the `.wsgi` file to serve the Flask app. Move to the `/var/www/FlaskApp/` directory and create a file named `flaskapp.wsgi` with following commands:

   ```
   $ cd /var/www/FlaskApp/
   $ sudo nano flaskapp.wsgi
   ```

   Add the following lines to the `flaskapp.wsgi` file:

   ```python
   #!/var/www/FlaskApp/venv/bin/python3
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/FlaskApp/")

   from FlaskApp import app as application
   application.secret_key = 'Add your secret key'
   ```

6. Restart Apache server:

   ```
   $ sudo service apache2 restart
   ```
   
   Now you should be able to run the application at <http://138.197.214.32.xip.io>.
   
   **Note**: You might still see the default Apache page despite setting everything up correctly. To resolve it and see your Flask app running, run the following commands in order:
   
   ```
   $ sudo a2dissite 000-default.conf
   $ sudo service apache2 restart
   ```

## Debugging

If you are getting an _Internal Server Error_ or any other error(s), make sure to check out Apache's error log for debugging:

```
$ sudo cat /var/log/apache2/error.log
```
There might be some errors due to different version of python or flask as well. Look for the same version. Install the same version as used while making Item Catalog Project. 

Do check that all the libraries used within your project are installed as well. Even if one is missing it would lead to error. 
## References

[1] <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04>

[2] <http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu>

[3] <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

[4] <https://github.com/kongling893/Linux-Server-Configuration-UDACITY>
