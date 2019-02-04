# Project: Linux Server Configuration

## The task:

'Student is expected to set up a Linux server with good security settings, a running web server, and deployment of their Item Catalog project from earlier in the ND.

Student instructions are as follows —

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.'


## Set-Up:

- I've created a static IP address for the server which is: 52.57.204.122
  You will use it to ssh into the server from your command line very soon.
  I've set up port 2200 for us to reach the server. Passing the following promt to the
  command line will log you in as the user "ubuntu":

        $ ssh -i LightsailDefaultKey-eu-central-1.pem ubuntu@52.57.204.122 -p 2200

    Make sure you have saved the private key (Lightsail... .pem) to the directory
    you are executing the command from and changed the permissions to this file by executing the following command:

        $ sudo chmod 600 LightsailDefaultKey-eu-central-1.pem

- The URL for our catalog web application is http://52.57.204.122.xip.io/
  Make sure you don't omit the xip.io, else the google authentication won't work.
  Extending the URL to xip.io is a workaround for the Authorized Redirects in the Credentials for our OAuth. Since it does not accept IP Adresses, adding xip.io basically creates a URL that renders to http://52.57.204.122 when executed.

- Grader access is granted using the second private key file provided in this project      "pynoor-linux-server-key", which is available to you (if you are a grader) in the        "Notes to Reviewer" section.
  If you have the private key, save it to your directory of choice, cd into it and execute the following command:

        $ ssh -i <name_of_file_you_saved_the_key_in> grader-2@52.57.204.122 -p 2200

  You should now be logged in as grader-2 in my ubuntu machine. (Don't worry about the "-2", I had a few issues with the first user called grader I created.)


## About the Server:

  - At first I created a LightSail Ubuntu 16.04 instance on AWS.

  ![](https://raw.githubusercontent.com/pynoor/linux-server/master/images/Screen%20Shot%202019-02-04%20at%2017.19.49.png)

  - I then went on and created a static IP address for it, which is now 52.57.204.122
  - Next up was the firewall: I wanted to be able to access the server on port 2200, so I  manually added it to the firewall settings on the aws interface...

  ![](https://raw.githubusercontent.com/pynoor/linux-server/master/images/Screen%20Shot%202019-02-04%20at%2017.58.09.png)

  ... and edited the sshd config file.

  - Once I had access from the terminal to the server, I got to work:

    -> udating the program source list

        $ sudo apt-get update


    -> upgrading existing packages

        $ sudo apt-get upgrade


    -> creating a ssh key on my *local* machine and on the server side copying the private key to the server, changing the permissions to the key file, making a .ssh folder and copying the public key into the authorized_keys file. (more on that in the first ressource)


    -> adding a new user "grader-2" and configuring sudo access:

        $ sudo adduser grader-2
        $ sudo visudo
        $ root ALL=(ALL:ALL) ALL
        $ grader ALL=(ALL:ALL) ALL


    -> configuring the ubuntu firewall (ufw)

        $ sudo ufw allow ...

    and finally,

        $ sudo ufw enable


    -> configuring the time zone


    -> installing various packages using the -H flag (!)

        $ sudo apt-get install [git, postgresql, pip3, pip, apache2, libapache2-mod-wsgi-py3, python-dev]

        $ sudo -H pip3 install [sqlalchemy, flask, flask-bootstrap, psycopg2, requests, oauth2, httplib2]

    -> cloning my remote catalog repository to /var/www/


    -> creating a wsgi file (project.wsgi) in the /var/www/catalog folder and setting up a virtual host folder in /etc/apache2/sites-available/catalog.conf

    To see the content of these filese execute:

        $ sudo cat /var/www/catalog/project.wsgi
        $ sudo cat /etc/apache2/sites-available/catalog.conf


    -> creating a catalog database with user "catalog":

        postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
        postgres=# ALTER USER catalog CREATEDB;
        postgres=# CREATE DATABASE catalog WITH OWNER catalog;
        postgres=# \c catalog
        catalog=# REVOKE ALL ON SCHEMA public FROM public;
        catalog=# GRANT ALL ON SCHEMA public TO catalog;


    -> editing relevant files of my web app to connect to the right database ('postgresql://catalog:catalog@localhost/catalog')
    and adjusting the path to the client_secret in the project.py file


    -> updating the oath credentials as such:
    ![](https://raw.githubusercontent.com/pynoor/linux-server/master/images/Screen%20Shot%202019-02-04%20at%2020.12.13.png)


    -> downloading the new client_secret to the server and replacing it with the old one


    -> following the steps of installation of my catalog web app


    -> visit http://52.57.204.122.xip.io/


### Third party resources:

https://github.com/harushimo/linux-server-configuration

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps


https://stackoverflow.com/questions/14238665/can-a-public-ip-address-be-used-as-google-oauth-redirect-uri

https://stackoverflow.com/questions/36020374/google-permission-denied-to-generate-login-hint-for-target-domain-not-on-localh
