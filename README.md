# Linux Server Configuration for Web App
_Configured a baseline installation of Amazon Lightsail Linux Server To host Web Applications. secured the server from a number of attack, installed and configured a database server, and deploy one of my existing web applications onto it._

## SERVER INFORMATIONS

**IP ADDRESS :**  `13.232.118.216`

**SSH PORT:** `2200`

**URL :** [http://13.232.118.216.xip.io/]()

**Grader's SSH KEY LOCATION:** `/home/grader/.ssh`

**Login command for Grader :**  `ssh grader@13.232.118.216 -i ~/.ssh/graderKeyPair -p 2200`



## PROCEDURE

### _Getting Your Server________________________

#### STEP 1 : Start a new Ubuntu Linux server instance on Amazon Lightsail
* Log in to Lightsail
* Create an instance
* First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
* Choose your instance plan
* Give your instance a hostname
* Wait for it to start up
* Once your instance has started up, you can log into it with `Connect using SSh`
* `public IP` address of the instance is displayed along with its `User name`
* Download `private key` of SSH KEY PAIR.
* save this .pem file to `~/.ssh` directory

#### STEP 2 : SSH into your server
* Use these commands to SSH into your server from your console
    ```commandline
    ssh ubuntu@13.232.118.216 -i ~/.ssh/LightsailDefaultPrivateKey.pem
    ```

### _Securing Your Server________________________

#### STEP 3 : Update all currently installed packages
* Use these commands to update installed packages
    ```commandline
    sudo apt-get update
    sudo apt-get upgrade
    ```
* If *** System restart required *** is displayed at login, run:
    ```commandline
    sudo reboot
    ```

#### STEP 4 : Change the SSH port from 22 to 2200
* Firstly, Open port 2200 in `Lightsail > Networkind > Firewall` to avoid locking yourself out by adding:
    ```textmate
    Application   Protocol    Port range
    Custom        TCP	  2200
    ```
* Open `/etc/ssh/sshd_config` file using :

    ```commandline
    sudo nano /etc/ssh/sshd_config
    ```
* Change the line `Port 22` to `Port 2200`
* Then restart the SSH service:
    ```commandline
    sudo service ssh restart
    ```
* New command to login to the server:

    ```commandline
    ssh ubuntu@13.232.118.216 -i ~/.ssh/LightsailDefaultPrivateKey.pem -p 2200
    ```
    
#### STEP 5 : Configure the Uncomplicated Firewall (UFW) 
Allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
* Check UFW status
    ```commandline
    sudo ufw status
    ```
* Block all incoming connections on all ports
    ```commandline
    sudo ufw default deny incoming
    ```
* Allow outgoing connection on all ports
    ```commandline
    sudo ufw default allow outgoing
    ```
* Allow incoming connection for SSH on port 2200
    ```commandline
    sudo ufw allow 2200/tcp
    ```
* Allow incoming connections for HTTP on port 80
    ```commandline
    sudo ufw allow www
    ```
* Allow incoming connection for NTP on port 123
    ```commandline
    sudo ufw allow ntp
    ```
* check the rules that have been added before enabling the firewall
    ```commandline
    sudo ufw show added
    ```
* enable the firewall
    ```commandline
    sudo ufw enable
    ```
* Check UFW status again
    ```commandline
    sudo ufw status
    ```
    
### _GIVING GRADER SERVER ACCESS________________________
Giving `grader` access to log in to my server for reviewing my project.
#### STEP 6 : Create a new user account named `grader`
* Create user `grader`
    ```commandline
    sudo adduser grader
    ```
#### STEP 7 : Give `grader` the permission to `sudo`
* If you are signed in using a non-root user with sudo privileges, type
    ```commandline
    sudo visudo
    ```
* Search for the line that looks like this:
    ```commandline
    root    ALL=(ALL:ALL) ALL
    ```
* Below this line, copy the format you see here, changing only the word 
"root" to reference the new user that you would like to give sudo privileges to:
    ```commandline
    root    ALL=(ALL:ALL) ALL
    newuser ALL=(ALL:ALL) ALL
    ```
* Sign In to user `grader`
    ```commandline
    sudo su - grader
    ```
* Sign Out of user `grader`
    ```commandline
    exit
    ```

#### STEP 8 : Create an SSH key pair for `grader` using the `ssh-keygen` tool
Enable Key Based Authentication
* Generate SSH Key Pairs locally on your system using application `ssh-keygen`, type:
    ```commandline
    ssh-keygen
    ```
* Enter file in which to save the key (/home/user/
.ssh/id_rsa):
    ```commandline
    /home/user/.ssh/graderKeyPair
    ```
* Two Files will be created inside `~/.ssh/` i.e `graderKeyPair` `graderKeyPair.pub`

* Place the Public Key on our remote server so that SSH can use it to log in
    1. make .ssh dir inside `/home/grader/`
        ```commandline
        sudo mkdir /home/grader/.ssh
        ```
    2. create `authorized_keys` file inside `/home/grader/.ssh`
        ```commandline
        sudo touch /home/grader/.ssh/authorized_keys
        ```
        this is a special file will store all the public keys this account 
        is allowed to use for authentication.
    3. Copy the contents of `/home/user/.ssh/graderKeyPair.pub` from the local machine 
    
    4. open `authorized_keys` file inside `/home/grader/.ssh`
        ```commandline
        sudo nano /home/grader/.ssh/authorized_keys
        ```
    
    5. paste Copied Content into `/home/grader/.ssh/authorized_keys` file
        

* Set permission of `.ssh` & `authorized_keys` so that other user can not gain access to your account
    ```commmandline
    sudo chmod 700 /home/grader/.ssh
    sudo chmod 644 /home/grader/.ssh/authorized_keys
    ```
* Changer ownership of `.ssh` & `authorized_keys` grader so that grader can gain access to these file
    ```commmandline
    sudo chmod 700 /home/grader/.ssh
    sudo chmod 644 /home/grader/.ssh/authorized_keys
    ```

### _PREPARE TO DEPLOY YOUR PROJECT________________________
#### STEP 9 : Configure the local timezone to UTC
* Check the timezone
    ```commandline
    date
    ```
* If it's not UTC change it to UTC using: 
    ```commandline
    sudo timedatectl set-timezone UTC
    ```

#### STEP 10 : Install and configure Apache to serve a Python mod_wsgi application
* Install Apache:
    ```commandline
    sudo apt-get install apache2
    ```
    * Confirm Apache is working by replacing `public_ip` with your public IP and visiting : 
        ```commandline
        http://public_ip:80
        ```
    * You should see the following page: 
        ![apacheConfig](img/apache.png)
        Apache, by default, serves its files from the `/var/www/html`. Apache just returns a file requested or the `index.html` file if no file is defined

* Install the `libapache2-mod-wsgi` package:
    ```commandline
    sudo apt-get install libapache2-mod-wsgi
    ```
    It configure Apache to hand-off certain requests to an application handler - mod_wsgi
    * If project is built using `Python 3` use this instead:
        ```commandline
        sudo apt-get install libapache2-mod-wsgi-py3
        ```
* configure Apache to handle requests using the WSGI module by editing `/etc/apache2/sites-enabled/000-default.conf` file.
    1. open `/etc/apache2/sites-enabled/000-default.conf` file
        ```commandline
        sudo nano /etc/apache2/sites-enabled/000-default.conf
        ```
    1. add the following line at the end of the `<VirtualHost *:80>` block, right before the closing `</VirtualHost>` line: 
        ```commandline
        WSGIScriptAlias / /var/www/html/myapp.py
        ```
    1. Finally, restart Apache 
        ```commandline
        sudo apache2ctl restart
        ```
    * To test if you have your Apache configuration correct you can write a very basic WSGI application :
        
        _WSGI is a specification that describes how a web server communicates with web applications. Most Python web frameworks are WSGI compliant including Flask and Django. Despite having the extension .wsgi, these are just Python applications_
        
        1. defined the name of the file you need to write within your Apache configuration by using the `WSGIScriptAlias` directive
            ```commandline
            WSGIScriptAlias /test_wsgi /var/www/html/test_wsgi.py
            ```
            which is already done in above step
        1. Create the /var/www/html/myapp.wsgi file using the command : 
            ```commandline
            sudo nano /var/www/html/test_wsgi.py
            ```
        1. Within this file, write the following application: 
            ```python
               def application(environ, start_response):
                   status = '200 OK'
                   output = 'Hello World From WSGI!'
    
                   response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
                   start_response(status, response_headers)
    
                   return [output]
            ```
        1. Finally, restart Apache 
            ```commandline
            sudo apache2ctl restart
            ```
        1. If everything goes as expected, open your favorite web browser and type the URL `http://your-server-ip/test_wsgi` and hit `Enter`, You will get the newly created application:
            ![wsgiConfig](img/wsgi.png)


#### STEP 11 : Install and configure PostgreSQL
* Install PostgreSQL with:
    ```commandline
    sudo apt-get install postgresql postgresql-contrib
    ```
    * Do not allow remote connections
        
        _A simple way to remove a potential attack vector is to not allow remote connections to the database. This is the current default when installing PostgreSQL from the Ubuntu repositories._ 
        
        1. Open the postgres config file
            ```commandline
            sudo nano /etc/postgresql/9.5/main/pg_hba.conf
            ```
        1. double check that no remote connections are allowed by looking in the host based authentication file:
            ![wsgiConfig](img/psqlDenyRemote.png)
        
        1. As you can see, the first two security lines specify "local" as the scope that they apply to. This means they are using Unix/Linux domain sockets.
        
        1. The second two declarations are remote, but if we look at the hosts that they apply to (127.0.0.1/32 and ::1/128), we see that these are interfaces that specify the local machine.
                
    * Create a new database user named `catalog` that has limited permissions to your catalog application database.
        1. Create a linux user named `catalog`
            ```commandline
            sudo adduser catalog
            ```
        
        1. Create a PostgreSQL user(role) called `catalog` with:
            ```commandline
            sudo -u postgres createuser -P catalog
            ```
            you are `Prompted` for a password(-P). This creates a normal user that can't create databases, roles (users).
        1. create the database `catalog` with `catalog` as owner.
            ```commandline
            sudo -u postgres createdb -O catalog catalog
            ```
    * To Check If Databse and User Created Successfully:
        
        * Log into PostgreSQL using:
            ```
            sudo su - postgres
            psql
            ```
        * List all the current **Owners**(Role) and their attributes by typing:
            ```
            \du
            ```
        * Login to **Owner**(Role) `catalog` by typing UNIX command:
            ```
            psql -h localhost -U user_name -p <port>
            ```
        * If you don't know the port, you can always get it by running the following, as the postgres user,
            ```
            SHOW port;
            ```
        * Show all **databases** having `catalog` as owner:
            ```
            \l
            ```
        * Select **Database** `catalog`:
            ```
            \c catalog
            ```
        * Show all **tables** within `catalog` database:
            ```
            \c dt
            ```
        * Log out PostgreSQL to follow along with this section:
            ```
            sudo su - postgres
            psql
            ```

#### STEP 12 : Install git
* Install git using command:
    ```commandline
    sudo apt-get install git
    ```

### _Deploy the Item Catalog project________________________

#### STEP 13 : Clone and setup your Catalog project from the Github repository you created earlier in this Nanodegree program
* Clone the Repository
    1. Go to the www dirrectory and create a directory catalog:
        ```
        cd /var/www/
        ```
        ```
        sudo mkdir catalog
        ```
        ```
        cd catalog
        ```
    2. use git clone to download Catalog Project & rename it to catalog
        ```
        sudo git clone https://github.com/shubhamPrakashJha/Cricket-Player-Info.git
        ```
        ```
        sudo mkdir catalog
        ```
        ```
        sudo mv Cricket-Player-Info/* catalog
        ```
        ```
        sudo rm -r Cricket-Player-Info/
        ```
        ```
        cd catalog/
        ```
* Setup the Repository
    1. change `application.py` to `init.py`
        ```
        sudo mv application.py __init__.py
        ```
    2. Edit engine in `init.py`, `database_setup.py` and `lotsofplayerswithuser.py`
        * use `$ sudo nano` on each of the mentioned files, and find the line.
            ```
            engine = create_engine('sqlite:///teamplayerwithuser.db')
            ```
        * and replace them with
            ```
            engine = create_engine('postgresql://catalog:grader@localhost/catalog')
            ```
        `password` is not shown for security reasons.
    3. Modify init.py so Google+ login works.
        * Open `__init__.py`
            ```
            sudo nano /var/www/catalog/catalog/__init__.py
            ```
        * find any reference to `client_secrets.json` and replace it with its full path name
            ```
            /var/www/catalog/catalog/client_secrets.json
            ```
        * find the line `app.debug = True` and delete it.
    3. Update the Google OAuth `client_secrets` file
        * go to your [Google Console](https://console.developers.google.com)
        * select your project
        * Select `API & Services` > `credentials`
        * Select Your project Client ID
        * In `Authorized JavaScript origins` add following `url`
            ```
            http://13.232.118.216
            http://13.232.118.216.xip.io
            ```
        * In `Authorized redirect URIs` add:
            ```
            http://13.232.118.216.xip.io/login
            http://13.232.118.216.xip.io/gconnect
            ```
        * Click `Save`
        * Click on `DOWNLOAD JSON` to download updated `client_secret` file
        * Replace the content of old `client_secret` file with the content of downloaded `client_secret` file.
    
#### STEP 14 : Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your `.git` directory is not publicly accessible via a browser!
* Install packages: Flask and SQLAlchemy using following command:
     ```
     sudo apt-get install python-psycopg2 python-flask
     sudo apt-get install python-sqlalchemy python-pip
     sudo pip install oauth2client
     sudo pip install requests
     sudo pip install httplib2
     sudo pip install flask-seasurf
     ```
* create your database
     ```
     python database_setup.py
     python lotsofplayerswithuser.py
     ```
* Create .wsgi file
     ```
     cd /var/www/catalog
     sudo nano catalog.wsgi
     ```
     and paste the following
     ```
     #!/usr/bin/python
     import sys
     import logging
     logging.basicConfig(stream=sys.stderr)
     sys.path.insert(0,"/var/www/catalog/")
     
     from catalog import app as application
     application.secret_key = 'some_secret'
     ```
* Create a Virtual Host

    To serve the catalog app using the Apache2 web server, you need to create a virtual host configuration file.
     ```
     sudo nano /etc/apache2/sites-available/catalog.conf
     ```
     and paste the following
     ```
     <VirtualHost *:80>
          ServerName 13.232.118.216
          ServerAdmin admin@13.232.118.216
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
* Disable the default virtual host
    ```
    sudo a2dissite 000-default.conf
    ```
* Enable the virtual host just created
    ```
    sudo a2ensite catalog.conf
    ```
* To make these changes live restart Apache2
    ```
    sudo service apache2 restart
    ```
* Make .git file inaccessable
    ```
    sudo nano .htaccess
    ```
    add line
    ```
    RedirectMatch 404 /\.git
    ```
* To Disable root login
    1. oprn `/etc/ssh/sshd_config`:
    ```
    sudo nano /etc/ssh/sshd_config
    ```
    2. Replace `PermitRootLogin without-password` to :
    ```
    PermitRootLogin no
    ```
    2. uncomment the following line
    ```
    PasswordAuthentication no
    ```
    2. Restart SSH Service
    ```
    sudo service ssh restart
    ```

## Resources Used:

* [Connect to Your Amazon EC2 Instance](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-2-connect-to-instance.html)

* [How To Edit the Sudoers File on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)
* [How To Grant a User Sudo Privileges](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
* [A Step by Step Guide to Install LAMP](https://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)
* [Configure and Enable a New Virtual Host](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Do Not Allow Remote Connections](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [Managing PSQL users and rights](https://help.ubuntu.com/community/PostgreSQL)
* [Make .git directory web inaccessible](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)
* [Default public network ports open for specific instance images](https://lightsail.aws.amazon.com/ls/docs/en/articles/understanding-firewall-and-port-mappings-in-amazon-lightsail)
* [How To Create, Remove, & Manage Tables in PostgreSQL on a Cloud Server](https://www.digitalocean.com/community/tutorials/how-to-create-remove-manage-tables-in-postgresql-on-a-cloud-server)
* [PostgreSQL Tutorial](http://www.postgresqltutorial.com/)
* [ Vagrant configuration that starts up a PostgreSQL database in a virtual machine for local application development](https://github.com/jackdb/pg-app-dev-vm)
* [PostgreSQL_For_Development_With_Vagrant](https://wiki.postgresql.org/wiki/PostgreSQL_For_Development_With_Vagrant)
* [How to install PostgreSQL on Ubuntu 14.04](https://www.godaddy.com/garage/how-to-install-postgresql-on-ubuntu-14-04/)
* [aviaryan](https://github.com/aviaryan/flask-postgres-apache-server)
* [psql: FATAL: database “<user>” does not exist](https://stackoverflow.com/questions/17633422/psql-fatal-database-user-does-not-exist)
* [How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04)
* [How To Install Linux, Nginx, MySQL, PHP (LEMP) stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04)
* [How to change PostgreSQL user password?](https://stackoverflow.com/questions/12720967/how-to-change-postgresql-user-password)
* [login to psql user with a password over TCP](https://stackoverflow.com/questions/2172569/how-do-i-login-and-authenticate-to-postgresql-after-a-fresh-install)
* [SteveWooding](https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config)
* [Postgres login commands](https://alvinalexander.com/blog/post/postgresql/log-in-postgresql-database)
* [How To Use Roles and Manage Grant Permissions in PostgreSQL on a VPS](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)
* [Remove a directory with files and subdirectories](https://thishosting.rocks/remove-directory-linux/)
* [Remove a Linux user](https://in.godaddy.com/help/remove-a-linux-user-19158)
* [WSGIScriptAlias](https://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIScriptAlias.html)
* [Install and Configure mod_wsgi on Ubuntu 16.04](https://devops.profitbricks.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/)
* [Udacity Linux Server Configuration Project](https://stevenwooding.com/linux-server/)
* [How to change PostgreSQL user password?](https://stackoverflow.com/questions/12720967/how-to-change-postgresql-user-password)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Check your servers error log](https://stackoverflow.com/questions/6438475/the-server-encountered-an-internal-error-or-misconfiguration-and-was-unable-to-c)
* [Sean-Holcomb](https://github.com/Sean-Holcomb/Linux-Server-Configuration#install-git-and-protect-git)
* [Unable to find a source package for psycopg2](https://stackoverflow.com/questions/14391804/e-unable-to-find-a-source-package-for-psycopg2-or-use-pip-with-virtualenv)
* [ unable to locate source package](https://askubuntu.com/questions/826890/apt-build-dep-fails-unable-to-locate-source-package-despite-deb-src-lines-pres)
* [Tarja M](https://github.com/otsop110/fullstack-nanodegree-linux-server-configuration)
* [ImportError: No module named psycopg2](https://stackoverflow.com/questions/12906351/importerror-no-module-named-psycopg2)
* [Apache http server log files](https://blog.codeasite.com/how-do-i-find-apache-http-server-log-files/)
* [WSGI script mysql query](https://stackoverflow.com/questions/23047644/how-to-place-wsgi-script-mysql-query-into-a-function)
* [xip.io](http://xip.io/)