# login-vuejs-apiology-docker

auhor: m4r4v

---

# Description

A regular login, using apiology as the REST API that communicates with db. For the frontend is all vuejs using quasar and pinia.

The purpose is to authenticate a user and keeping the session alive during X time of inactivity or wrong token validation.

---

# Environment Setup

This settings are for a frontend and backend solution integrated in one container. In order to obtain the desired result, the following architecture is being used:

- apiology folder that container inside all the logic for the API, is located inside the `/var/www` folder, at the same level as the `html` folder that contains public data.
- the front end the `dist` folder, contains the *production* / *built* files from *vuejs*; and this folders content needs to be inside the `html` folder.


### Dev - Prod Rules

the port is the only factor being used to determine if is the dev or prod environment, so when going live into production, the port must change to 80:

- **dev port** : 8010
- **prod port** : 80

### Technical Environment

- **NodeJs**: v20.10.0
- **NPM**: v10.2.4
- **VueJS**: @vue/cli 5.0.8
- **Quasar**: @quasar/cli v2.3.0
- **PHP**: v8.2.10
- **Docker**: v24.0.6
- **Docker Image**: m4r4v/lamp-debian-bullseye:latest
- **OS**: GNU/Linux
- **Distro**: Debian Bullseye v11.8
- **DB**: MariaDB version 15.1 Distrib 10.5.21-MariaDB

---

## 1. Create container - docker run

```bash
docker run -p 8010:80 -d -v ${PWD}:/var/www -e MYSQL_ROOT_PASSWORD=123456 --name login-vue-apiology m4r4v/lamp-debian-bullseye:latest
```

**-p** : sets the docker port redirection from container to host

- web server:
    - local port: 8010
    - host port: 80
- db server:
    - local port: 3306

**-d** : dettached mode

**-v** : bind mount a volume

**${PWD}:/var/www** : defines host working directory into /var/www container working directory

**-e** : Sets MYSQL default root password
- **MYSQL_ROOT_PASSWORD=123456**

**--name** : _container name_

**m4r4v/lamp-debian-bullseye:latest** : docker hub image repository

## 2. OS setup

#### **Update and Upgrade**

```bash
apt update -y && apt upgrade -y
```

#### **Install dependencies**

```bash
apt install -y wget vim curl unzip git libfreetype6-dev libjpeg62-turbo-dev libpng-dev
```

#### **Enable apache modules**

```bash
a2enmod rewrite && a2enmod headers && a2enmod proxy && a2enmod proxy_http
```

#### **Restart Apache**

```bash
service apache2 restart
```

## Clone Apiology

Clone **apiology** repository from [apiology](https://github.com/m4r4v/apiology) into `/var/www` folder

1. move to container working folder

```bash
cd /var/www
```

2. clone repository

```bash
git clone https://github.com/m4r4v/apiology
```

## Set up Composer

1. Download installer

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```

2. Verify Integrity by checksum (check for hash in [Download Composer Page](https://getcomposer.org/download/))

```bash
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e21205b207c3ff031906575712edab6f13eb0b361f2085f1f1237b7126d785e826a450292b6cfd1d64d92e6563bbde02') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

3. Install Composer

```bash
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

4. Remove setup file

```bash
php -r "unlink('composer-setup.php');"
```

5. check composer version

```bash
composer --version
```

6. Install and Update *apiology* packages

```bash
# move to the apiology folder
cd /var/www/apiology

# install and/or update dependencies from conposer.json file settings
composer install
```

7. Permissions

give permissions to the api folder so it can log logs

```bash
# giving permission to apache2 group
chown -R root:www-data /var/www/apiology/api
```
set permissions to logs.log

```bash
chmod 0766 /var/www/apiology/api/logs/logs.log
```

## Create Subdomain

To let the frontend interact with the backend using the API, is being set as a subdomain, i.e.: `api.localhost`

1. edit hosts file

```bash
vim /etc/hosts

# add entry to the subdomain so it can point to localhost
127.0.0.1   api.localhost
```

2. create a new virtual host configuration file for the subdomain

```bash
# copy apache default .conf file
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/api.localhost.conf

# edit file
vim /etc/apache2/sites-available/api.localhost.conf

# add the following parameters
<VirtualHost *:80>
    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.
    ServerName api.localhost

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/apiology
    <Directory /var/www/apiology>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ProxyPass / http://localhost:80/
    ProxyPassReverse / http://localhost:80/

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

3. enable the new virtual host

```bash
# enale apache2 modues
a2ensite subdomain.localhost.conf

# reload apache2 service
# IMPORTANT: in some cases may be better to restart the apache2 service
# service apache2 restart
service apache2 reload
```

Test the subdomain by accessing `http://api.localhost:8880` in a web browser. It should proxy requests to your Docker container.

## Allow MariaDB socket communication

MariaDB can communicate in two ways:

1. sockets: useful for shared hosts with web services
1. bind address: useful when using different hosts and using network to access

### step 1: my.cnf

```bash
# backup the file
cp /etc/mysql/my.cnf /etc/mysql/my.cnf.backup
```

socket communication because it shares host with other services allowing socket communication 

```bash
# open in editor
vim /etc/mysql/my.cnf
```

uncomment the following line `socket = /run/mysqld/mysqld.sock`

Now, restart mysql

```bash
/etc/init.d/mysql restart
```

### step 2: execute mysql_secure_installation

```bash
mysql_secure_installation
```

Recommended to change root password and anonymous users.

### step 3: connect to mysql / mariadb

```bash
mysql -u root -p
password:
```
