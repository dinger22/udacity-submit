# udacity-submit
URL: http://ec2-18-188-80-81.us-east-2.compute.amazonaws.com

IP: 18.188.80.81

port: 2200

Login with: ssh -i ~/.ssh/lastproject -p 2200 grader@18.188.80.81

List of 3rd-party libs:
    psycopg2,
    flask
    sqlalchemy
    oauth2client
    httplib2
    requests
    finger
    virtualenv

Step-1: Follow the instruction provided in to SSH into server
    .use chmod 600 ~/.ssh/udacity_key.pem to change the mode of key file first.
    .use ssh -i ~/.ssh/udacity_key.pem ubuntu@{publick IP} to ssh

Step-2: Secure my server.
    .use sudo apt-get update and sudo apt-get upgrade to update all the packages
    .add new rule to lightsail firewall to allow 2200/tcp 
    .use sudo nano /etc/ssh/sshd_config to change ssh port from 22 to 2200
    .use sodu ufw allow *** to Configure the Uncomplicated Firewall (UFW) 
    to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
    .reboot

Step-3:Give grader access
    .sudo adduser grader
    .check the User(grader) information :
        sudo apt-get install finger
        finger grader
    .login as grader using password.
    .mkdir .ssh
    .touch .ssh/authorized_keys
    .in local machine generate key-pair with ssh-keygen and save the file to local ~/.ssh
    .copy everything in ***.pub and past it to grader's authorized_keys and save.
    .change file permissions 
        chmod 700 .ssh 
        chmod 644 .ssh/authorized_keys
    .reboot and login with ssh -v grader@18.188.80.81 -p 2200 -i ~/.ssh/****

Step-4: Prepare to deploy your project.
    1. sudo dpkg-reconfigure tzdata to chage time zone
    2. sudo apt-get install apache2
    3. sudo apt-get install libapache2-mod-wsgi
    4. setup a template wsgi    
        sudo nano /etc/apache2/sites-enabled/000-default.conf
        add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: WSGIScriptAlias / /var/www/html/catalog.wsgi
    5. sudo apache2ctl restart 
    6. install git and config it.
        git config --global user.name "username"
        git config --global user.email "email@domain.com"
        
Step-5: Install Postgresql
    1.Install the Python PostgreSQL adapter psycopg: sudo apt-get install python-psycopg2

    3.Change to default user postgres: sudo su - postgre
    4.Create user catalog: psql CREATE USER catalog WITH PASSWORD 'pw';
    5.Allow the user to create database : ALTER USER catalog CREATEDB
    6.Create database : CREATE DATABASE catalog WITH OWNER catalog;
    7.Connect to database : \c catalog
    8.Grant the access to catalog: GRANT ALL ON SCHEMA public TO catalog;

Step-6: deploy
    1. use pip to install virtualenv and Flask. Install pip : sudo apt-get install python-pip
    2. install other packages using pip, e.g requests and httplib2
    3. set enviorment name using sudo virtualenv venv
    4. sudo nano /etc/apache2/sites-available/FlaskApp.conf
        .config the conf file 
            <VirtualHost *:80>
                ServerName [[***IP****]]
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
                <Directory /var/www/FlaskApp/FlaskApp/>
                    Order allow,deny
                    Allow from all
                </Directory>
                Alias /static /var/www/FlaskApp/FlaskApp/static
                <Directory /var/www/FlaskApp/FlaskApp/static/>
                    Order allow,deny
                    Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
    5.Enable virtual host using : udo a2ensite FlaskApp
    6. create wsgi file using sudo nano catalog.wsgi. config it as 
        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/FlaskApp/")

        from FlaskApp import app as application
        application.secret_key = 'secret_key'
    7. c;one the repository to our server, git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY
    8. move the file to /var/www/catalog
    9. update db_setup.py to use PostgreSQL
    10. update FlaskApp.conf
        WSGIScriptAlias / /var/www/FlaskApp/FlaskApp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
    11. update client_secrets.js's path to use full path.
    12. update Authorised JavaScript origins in google: http://ec2-18-188-80-81.us-east-2.compute.amazonaws.com
    13. run the application using python __init.py__. 
    14. check the hosted webapp using  http://ec2-18-188-80-81.us-east-2.compute.amazonaws.com