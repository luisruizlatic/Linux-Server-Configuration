 # Linux Server Configuration Project

Udacity Linux Server Configuration Project.

### Access Information

- **Server IP Address:** 52.91.108.97
- **SSH server access port:** 2200
- **SSH login username:** grader
- **Application Domain:** http://www.fitnesssupplements.info


## Server Setup

### 1. Create a Linux instance on Amazon Lightsail

 Create an amazon web services account <https://signin.aws.amazon.com/> and create a new Ubuntu 16.04 instance on Amazon Lightsail.

### 2. Access Linux Server

On amazon lightsail, download the ssh key server access to your computer in order to use your own terminal and manage the server using the username and public ip provided.

   ```
   Public IP: 52.91.108.97
   Username: Ubuntu
   Access:
   ssh ubuntu@52.91.108.97 -i <Local machine Key path>
   ```

### 3. Update Linux packeges

Using the default sudo user execute the following commands:

   ```
  $ sudo apt-get update
  $ sudo apt-get upgrade
   ```

### 4. Change SSH ports

Access to the SSH configuration file, change the port line from 22 to 2200 and restart the service:

   ```
  $ sudo nano /etc/ssh/sshd_config
  $ service sshd restart
   ```
##### IMPORTANT INFORMATION: You could lose the access to the server, follow step 2 adding the new port (-p 2200):

  ```
  ssh ubuntu@52.91.108.97 -p 2200 -i <Local machine Key path>
   ```

### 5. Configure Linux UFW (Uncomplicated Firewall)

Make ports 2200, 80 and 123 the only incoming connections:

   ```
  $ sudo ufw default deny incoming
  $ sudo ufw allow 2200/tcp
  $ sudo ufw allow 80/tcp
  $ sudo ufw allow 123/tcp
  $ sudo ufw allow status #Verify if the firewall ws enabled succesfully
   ```

### 5. Create a new user

Create a new user called "grader":

   ```
   $ sudo adduser grader
   ```

### 6. Give sudo privileges

Give the user "grader" sudo privileges, adding it to the sudoers.d directory

  ```
   $ sudo echo /etc/sudoers.d/grader # We create a new file for the user
   $ sudo nano /etc/sudoers.d/grader # We open file deitor
      - Add the following lines to the grader file and save it:
           # Created by YOU on DATE
           # User rules for grader
           grader ALL=(ALL) NOPASSWD:ALL
   $ sudo su grader # Login using grader and test a sudo command
   ```
For security is recommended to block the user root:

   ```
    $ sudo nano /etc/ssh/sshd_config
       - Modify the following line to "no":
           PermitRootLogin no
   ```


### 7. Create a SSH Key to access the server

From a local machine open a terminal and generate a new SSH key:

  ```
    ssh-keygen
   ```
The terminal will ask you for a passphrase which can be typed or leave empty.

Add the path where you would like to save the new key.

  ```
    Enter file in which to save the key (/some/path/): /your/path/filename
   ```

Public and private keys will be generated, the public key will have the file name and .pub as extension (filename.pub) and the private key just will be named with the specified file name without extension.  

Once the file was created we need to access our Ubuntu server to save the generated public key on the home directory:

  ```
    $ sudo mkdir /home/.ssh
    $ sudo touch /home/.ssh/authorized_keys # Create a new file to store our public key
    $ sudo nano /home/.ssh/authorized_keys # Add public key content and save
    $ sudo chmod 700 /home/.ssh # Add permissions
    $ sudo chmod 664 /home/.ssh/authorized_keys # Add permissions
   ```

Now we are able to login using the generated key

 ```
  ssh grade@52.91.108.97 -p 2200 -i <Local machine Key path>
   ```
### 8. Change server time zone

To change the server time zone to UTC we need to run the following command and use the GUI:

  ```
    $ sudo dpkg-reconfigure tzdata
   ```

### 9. Install HTTP Server

To deploy our Python project we need to install Apache 2 and mod_wsgi:

  ```
    $ sudo apt-get install apache2 # Install Apache 2
    $ sudo apt-get install libapache2-mod-wsgi-py3 # Install mod_wsgi to publish python 3 apps
    $ sudo a2enmod wsgi # Enable mod_wsgi 
   ```

Download your project from GitHub or other resource on the path /var/www/


We need to create a new .wsgi file in your project path:

  ```
    $ sudo echo /your/project/path/<ProjectMainScript>.wsgi
    $ sudo nano /your/project/path/<ProjectMainScript>.wsgi 
      -Add the following lines:
         import sys

         if sys.version_info[0]<3: #check if is run with python3
             raise Exception("Python3 is required to run this program! Current version: '%s'" % sys.version_info)

         sys.path.insert(0,'/your/project/path')#path where the project is located
         from <Python ProjectMainScript name> import app as application
   ```
We need to set up the apache configuration file to make mod_wsgi work:

  ```
    $ sudo nano /etc/apache2/sites-enabled/000-default.conf
      -Add the following lines between <VirtualHost *:80> </VirtualHost> tags:
         WSGIDaemonProcess FitnessSupplementsApp python-path=/your/project/path
         WSGIProcessGroup %{GLOBAL}
         WSGIScriptAlias / /your/project/path/ProjectMainScript.wsgi
   ```
Once everything is setup you should be able to see your project running when you access your public address <http://52.91.108.97/>.

##### IMPORTANT INFORMATION: In case your project is not loading check the apache errors.log to verify the problem:

  ```
    $ sudo cat /var/log/apache2/error.log
   ```


### 10. Postgre Database

Once your application is running we need to install postgres database:

  ```
    $ sudo apt-get install postgresql
    $ su - postgres # Log into postgres
   ```

You have to change your project database connections:



We need to create a new .wsgi file in your project path:

  ```
    app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://<DB Username>:<DB Password>@localhost/<DB Name>'    
   ```
In case you don't have a database_setup script in your project you must create the database manually:

  ```
    $ su - postgres # Log into postgres
    $ psql 
    $ CREATE DATABASE dbname;
   ```

### References

<https://www.udacity.com/>
