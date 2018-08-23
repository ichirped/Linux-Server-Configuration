# Linux Server Configuration

Prepare a Linux server to host web applications. Secure the server from a number of attack vectors, install and configure a database server, and deploy Item Catalog App onto it.

## Server
 - IP: 34.220.221.97
 - SSH Port: 2200
 - URL: http://34.220.221.97.nip.io/

## 1. Update and upgrade installed packages

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

## 2. Install software

```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi-py3
$ sudo apt-get install postgresql postgresql-contrib
$ sudo apt-get install git
$ sudo apt-get install python python-pip
$ sudo apt-get install python-psycopg2 python-flask
$ sudo apt-get install python-sqlalchemy python-pip
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install httplib2
```

### 3. Configure Firewall

Modify `/etc/ssh/sshd_config`:

`Port 22` to `Port 2200`

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw enable
```

## 4. Give grader Access

### 4.1 Add new user `grader` and give `sudo` permission

```
$ ssh -i LightsailDefaultPrivateKey-us-west-2.pem -p 2200 ubuntu@34.220.221.97
$ sudo useradd -d /home/grader grader
$ sudo usermod -aG sudo grader
```

### 4.2 Configure key based authentication for `grader`

#### Modify  `/etc/ssh/sshd_config`:
```
PermitRootLogin no
PasswordAuthentication yes
AllowUsers grader ubuntu
```

#### Restart ssh service:
```
$ sudo service ssh restart
```

#### Generate keys:
```
$ ssh-keygen
```

#### Copy the content of `id_rsa.pub` to `/home/grader/ssh/authorized_keys`
```
$ sudo cp /home/ubuntu/.ssh/id_rsa.pub /home/grader/.ssh/authorized_keys
```

#### Setup key file permissions:

```
$ chown -R grader:grader /home/grader/
$ sudo chown root:root /home/grader/
$ sudo chmod 700 /home/grader/.ssh
$ sudo chmod 644 /home/grader/.ssh/authorized_keys
```

#### Modify `/etc/ssh/sshd_config` to force key-based auth:
```
PasswordAuthentication no
```

#### Restart ssh service:
```
$ sudo service ssh restart
```

## 5. Deploy Catalog App

### 5.1 Configure Local time to UTC:
```
$ sudo timedatectl set-timezone UTC
```

### 5.2 Clone repo from git into `/var/www/catalog/`
```
$ sudo git init
$ sudo git clone https://github.com/ichirped/Item-Catalog-App.git
```

### 5.3 Configure Apache2

#### Modify /etc/apache2/sites-available/000-default.conf:

```python
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        #ServerAdmin webmaster@localhost
        #DocumentRoot /var/www/html

        ServerName 34.220.221.97
        ServerAdmin bhoomipvyas@gmail.com
        DocumentRoot /var/www/catalog
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <directory /var/www/catalog>
                WSGIApplicationGroup %{GLOBAL}
                Order allow,deny
                Allow from all
        </directory>
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
</VirtualHost>
```
#### Modify `/var/www/catalog/catalog.wsgi`:
```python
#! /usr/bin/python

import sys
sys.path.insert(0,"/var/www/catalog/")
from application import app as application
```

#### Enable the virtual host: 
```
$ sudo a2ensite 000-default.conf
```
#### Reload Apache: 
```
$ sudo service apache2 reload
```
## 6. References

https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604
https://www.digitalocean.com/community/questions/ubuntu-16-04-creating-new-user-and-adding-ssh-keys
https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/
https://github.com/GrahamDumpleton/mod_wsgi/issues/224
