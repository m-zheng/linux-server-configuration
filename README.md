# About

This project is a part of Full Stack Developer Nanodegree from [Udacity](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

A Linux distribution on a virtual machine provided by [Amazon Lightsail](https://lightsail.aws.amazon.com/) is used to host the web application. This project also includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The public IP address is [http://34.201.49.136/](http://34.201.49.136/) and the URL for the web application is [http://ec2-34-201-49-136.compute-1.amazonaws.com/](http://ec2-34-201-49-136.compute-1.amazonaws.com/).


## 1. Get the server.
  - Login to [Amazon Lightsail](https://lightsail.aws.amazon.com/).
  - Choose Ubuntu instance.
  - After the instance is set up, visit [account page](https://lightsail.aws.amazon.com/ls/webapp/account) to download ssh private key.
  - Convert AWS Private Key Using [PuTTYgen](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html).
  - Use putty to access to the server. The deault username for the server is ubuntu and the deault ssh port is 20.

## 2. Secure the server.
### 2.1 Update all currently installed packages.
  - Update all packages
    ```
    sudo apt-get update
    sudo apt-get upgrade
    ```

### 2.2 Change the SSH port from 22 to 2200.
  - Change the SSH port from 22 to 2200.
    ```
    sudo nano /etc/ssh/sshd_config
    ```
  - Locate the following line:
     ```
    # Port 22
    ```   
  - Remove # and change 22 to 2200.
  - Save and exit this file.
  - Configure the firewall to allow SSH connection for port 2200 and 20:
     ```
    sudo ufw allow 2200
    sudo ufw allow 20
    sudo ufw enable
    ```  
  - Restart the sshd service by running the following command:
     ```
    sudo service sshd restart
    ```   

### 2.3 Disable remote login of the root user.
  - Make some changes in ssh configure file
    ```
    sudo nano /etc/ssh/sshd_config
    ```
  - Locate the following line:
     ```
    # Authentication:
    LoginGraceTime 120
    PermitRootLogin yes
    StrictModes yes
    ```   
  - Change PermitrootLogin yes to PermitRootLogin no.
     ```
    # Authentication:
    LoginGraceTime 120
    PermitRootLogin no
    StrictModes yes
    ```
  - Locate the following line:    
     ```
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication yes
    ```  
  - Change PasswordAuthentication yes to PasswordAuthentication no.
     ```
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication no
    ```
  - Save and exit this file.
  - Restart the sshd service by running the following command:
     ```
    sudo service sshd restart
    ```  

### 2.4 Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  - Add some new rules to the firewall
    ```
    sudo ufw allow 2200
    sudo ufw allow 80
    sudo ufw allow 123
    ```
  - Enable the firewall
    ```
    sudo ufw enable
    ```  
  - Check the firewall status
    ```
    sudo ufw status numbered
    ```
  - Delete all rules relating to port 20.
    ```
    sudo ufw delete (rule number)
    ```
  - If the status shows as the following then the firewall is configured correctly
    ```
    Status: active
    
         To                         Action      From
         --                         ------      ----
    [ 1] 2200                       ALLOW IN    Anywhere
    [ 2] 80                         ALLOW IN    Anywhere
    [ 3] 123                        ALLOW IN    Anywhere
    [ 4] 2200 (v6)                  ALLOW IN    Anywhere (v6)
    [ 5] 80 (v6)                    ALLOW IN    Anywhere (v6)
    [ 6] 123 (v6)                   ALLOW IN    Anywhere (v6)

    ```
    
## 3. Give grader access.
### 3.1 Create a new user account named grader.
  - Add a new user:
    ```
    sudo adduser grader
    ```

### 3.1 Give grader the permission to sudo.
  - Create grader file
    ```
    sudo touch /etc/sudoers.d/grader
    ```  
  - Edit grader file
    ```
    sudo nano /etc/sudoers.d/grader
    ```  
  - Add a new rule to grader file
    ```
    # User rules for grader
    grader ALL=(ALL) NOPASSWD:ALL
    ```  
   - Save and exit the nano editor

### 3.2 Create an SSH key pair for grader using the ssh-keygen tool.
  - Change directory to grader's home folder.
    ```
    cd /home/grader
    ```  
  - Create .ssh folder
    ```
    sudo mkdir .ssh
    ``` 
  - Create a public key in .ssh folder
    ```
    sudo touch /home/grader/.ssh/authorized_keys
    ```    
  - Generate an SSH key pair using ssh-keygen tool.
  - Copy and paste the generated public key to authorized_keys.
    ```
    sudo nano /home/grader/.ssh/authorized_keys
    ```
  - Change file permissions.
     ```
    sudo chmod 700 /home/grader/.ssh
    sudo chown grader /home/grader/.ssh
    sudo chgrp grader /home/grader/.ssh
    sudo chmod 400 /home/grader/.ssh/authorized_keys   
    sudo chown grader /home/grader/.ssh/authorized_keys
    sudo chgrp grader /home/grader/..sh/authorized_keys
    ```
  - Now the grader can login as grader by using terminal (private_key_file_path is the file path in the local machine).
     ```
    ssh grader@34.201.49.136 -p 2200 -i private_key_file_path
    ```


## 4. Prepare to deploy your project.
### 4.1 Configure the local timezone to UTC.
  - Change timezone:
    ```
    sudo timedatectl set-timezone Etc/UTC
    ```
  - check timezone:
    ```
    date
    ```

### 4.2 Install and configure Apache to serve a Python mod_wsgi application.
  - Install apache2 and libapache-mod-wsgi:
    ```
    sudo apt-get update
    sudo apt-get install apache2 libapache2-mod-wsgi
    ```
  - If the apache was installed corrected, the default page will be shown when visiting [34.201.49.136](34.201.49.136)  
  - configure apache to handle requests using Python mod_wsgi module.
    - Edit 000-default.conf
      ```
      sudo nano /etc/apache2/sites-enabled/000-default.conf
      ```
    - Add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost>.
      ```
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
      ```
    - Restart apache server:
      ```
      sudo service apache2 restart
      ```

### 4.3 Install and configure PostgreSQL
#### 4.3.1 Install postgresql and postgresql-contrib
  - Install postgresql and postgresql-contrib:
    ```
    sudo apt-get update
    sudo apt-get install postgresql postgresql-contrib
    ```

#### 4.3.2 [Configure PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
  - Disable Remote Connections:
  
    ```
    sudo nano /etc/postgresql/9.5/main/pg_hba.conf 
    ```
  - At the end of this file, make sure it looks like the belowing code.
    ```
    # DO NOT DISABLE!
    # If you change this first entry you will need to make sure that the
    # database superuser can access the database using some other method.
    # Noninteractive access to all databases is required during automatic
    # maintenance (custom daily cronjobs, replication, and similar tasks).
    #
    # Database administrative login by Unix domain socket
    local   all             postgres                                peer
    
    # TYPE  DATABASE        USER            ADDRESS                 METHOD
    
    # "local" is for Unix domain socket connections only
    local   all             all                                     peer
    # IPv4 local connections:
    host    all             all             127.0.0.1/32            md5
    # IPv6 local connections:
    host    all             all             ::1/128                 md5
    # Allow replication connections from localhost, by a user with the
    # replication privilege.
    #local   replication     postgres                                peer
    #host    replication     postgres        127.0.0.1/32            md5
    #host    replication     postgres        ::1/128                 md5
    ```

#### 4.3.2 Create a new database and setup permissions for web applications.
  - Log into PostgreSQL to follow along with this section:
    ```
    sudo su - postgres
    psql
    ```
  - Create a new user. The username is catalog and the password is 1234567890.
    ```
    CREATE USER catalog WITH PASSWORD '1234567890';
    ```
  - Confirm the user was created correctly:
    ```
    \du
    ```
  - Change user's permission:
    ```
    ALTER USER catalog CREATEDB;
    ```
  - Create a new database:
    ```
    CREATE DATABASE catalog WITH OWNER catalog;
    ```
  - Connect to the catalog database:
    ```
    \c catalog
    ```
  - Lock down the permissions to only let "catalog" create tables:
    ```
    REVOKE ALL ON SCHEMA public FROM public;
    GRANT ALL ON SCHEMA public TO access_role;
    ```
  - Exit postgresql:
    ```
    \q
    exit
    ```    

### 4.4 Install git
  - Install git with apt
    ```
    sudo apt-get update
    sudo apt-get install git  
    ```

## 5. Deploy the Item Catalog project.
### 5.1 Download the repo from github.
  - Change directory
    ```
    cd /var/www
    ```
  - Git clone the repo
    ```
    sudo git clone https://github.com/minghua1991/item-catalog.git
    ```
  - Rename the project from item-catalog to catalog
    ```
    sudo mv /var/www/item-catalog /var/www/catalog
    sudo mv /var/www/catalog/item-catalog /var/www/catalog/catalog
    ```

### 5.2 Configure apache to serve Item Catalog
  - Change apache configure file
    ```
    sudo nano /etc/apache2/sites-available/000-default.conf
    ```
  - Make sure 000-default.conf file looks like the following code:
    ```
    <VirtualHost *:80>
            # The ServerName directive sets the request scheme, hostname and port that
            # the server uses to identify itself. This is used when creating
            # redirection URLs. In the context of virtual hosts, the ServerName
            # specifies what hostname must appear in the request's Host: header to
            # match this virtual host. For the default virtual host (this file) this
            # value is not decisive as it is used as a last resort host regardless.
            # However, you must set it for any further virtual host explicitly.
            #ServerName www.example.com
    
            ServerAdmin webmaster@localhost
            #DocumentRoot /var/www/html
    
            # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
            # error, crit, alert, emerg.
            # It is also possible to configure the loglevel for particular
            # modules, e.g.
            #LogLevel info ssl:warn
    
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    
            # For most configuration files from conf-available/, which are
            # enabled or disabled at a global level, it is possible to
            # include a line for only one particular virtual host. For example the
            # following line enables the CGI configuration for this host only
            # after it has been globally disabled with "a2disconf".
    
            #Include conf-available/serve-cgi-bin.conf
    
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
     <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
    </Directory>
    
    </VirtualHost>
    
    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
    ``` 
  - Create catalog.wsgi file in catalog directory
    ```
    sudo touch /var/www/catalog/catalog.wsgi
    ```
  - Edit catalog.wsgi
    ```
    sudo nano /var/www/catalog/catalog.wsgi
    ```
  - Make sure it looks like the following code:
    ```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")
    
    from catalog import app as application
    application.secret_key = 'Add your secret key'
    ```
  - Save and exit catalog.wsgi file
  - Rename project.py
    ```
    sudo mv /var/www/catalog/catalog/project.py /var/www/catalog/catalog/__init__.py
    ```

### 5.3 Configure PostgreSQL to serve Item Catalog
  - Edit database_setup.py
    ```
    sudo nano /var/www/catalog/catalog/database_setup.py
    ```
  - Locate to the following line:
    ```
    engine = create_engine("sqlite:///itemCatalog.db")
    ```
  - Change it to:
    ```
    engine = create_engine("postgresql://catalog:1234567890@localhost/catalog")
    ```
  - Run the database_setup.py
    ```
    cd /var/www/catalog/catalog/
    python database_setup.py
    ```
  - Edit init file
    ```
    sudo nano /var/www/catalog/catalog/__init__.py
    ```
  - Locate to the following line:
    ```
    engine = create_engine("sqlite:///itemCatalog.db")
    ```
  - Change it to:
    ```
    engine = create_engine("postgresql://catalog:1234567890@localhost/catalog")
    ```
  - Locate to the following line:
    ```
    # Get client id from the json file provided by google.
    CLIENT_ID = json.loads(open("client_secrets.json",
                                "r").read())["web"]["client_id"]
    ```
  - Change it to:
    ```
    # Get client id from the json file provided by google.
    CLIENT_ID = json.loads(open("/var/www/catalog/catalog/client_secrets.json",
                                "r").read())["web"]["client_id"]
    ```
  - Locate to the following line:
    ```
    app_id = json.loads(open("fb_client_secrets.json",
                             "r").read())["web"]["app_id"]

    app_secret = json.loads(open("fb_client_secrets.json",
                                 "r").read())["web"]["app_secret"]    
    ```
  - Change it to:
    ```
    app_id = json.loads(open("/var/www/catalog/catalog/fb_client_secrets.json",
                             "r").read())["web"]["app_id"]

    app_secret = json.loads(open("/var/www/catalog/catalog/fb_client_secrets.json",
                                 "r").read())["web"]["app_secret"]
    ```
  - Locate to the following line:
    ```
    if __name__ == "__main__":
        app.secret_key = "super_secret_key"
        app.debug = True
        app.run(host="0.0.0.0", port=8080)
    ```
  - Change it to:
    ```
    if __name__ == "__main__":
        app.secret_key = "super_secret_key"
        app.debug = True
        app.run()    
    ```

### 5.4 Update Facebook Api
  - Login to [Facebook Api Concole](https://developers.facebook.com)
  - In then app console, click facebook login -> settings. Add the following new urls to the Valid OAuth redirect URIs
    ```
    http://ec2-34-201-49-136.compute-1.amazonaws.com/
    http://34.201.49.136/
    ```
  - Save Changes

### 5.5 Update Google Api
  - Login to [Google Api Console](https://console.developers.google.com/).
  - In the app console, click Cridentials, then edit the oauth client.
  - Add the following urls to Authorised JavaScript origins:
    ```
    http://ec2-34-201-49-136.compute-1.amazonaws.com/
    http://34.201.49.136
    ```
  - Add the following urls to Authorised redirect URIs:
    ```
    http://ec2-34-201-49-136.compute-1.amazonaws.com/
    http://ec2-34-201-49-136.compute-1.amazonaws.com/gconnect
    http://ec2-34-201-49-136.compute-1.amazonaws.com/login
    ```
  - Click Save then download JSON.
  - Open the JSON file, copy and paste the content to client_secrets.json.
    ```
    sudo nano /var/www/catalog/catalog/client_secrets.json
    ```
  - Save all changes.

### 5.6 Make .git directory not publicly accessible via a browser
  - Change directory to catalog root folder:
    ```
    cd /var/www/catalog
    ```
  - Create a hidden file called .htaccess
    ```
    sudo touch /var/www/catalog/.htaccess
    ```
  - Edit the hidden file.
    ```
    sudo nano /var/www/catalog/.htaccess
    ```    
  - Add the following code to .htaccess file.
    ```
    RedirectMatch 404 /\.git
    ```      
  - Save and exit.


## 6. Live Demo
  - Navigate to [http://ec2-34-201-49-136.compute-1.amazonaws.com/](http://ec2-34-201-49-136.compute-1.amazonaws.com/) in web browser. 
