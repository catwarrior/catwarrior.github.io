title: play with elgg
date: 2016-03-29 13:28:35
categories:
- coding
tags:
- php
- elgg
- social
- web
---

## Installing Elgg on Ubuntu Linux
Install the dependencies:
``` bash
sudo apt-get install apache2
sudo apt-get install mysql-server
sudo apt-get install php5 libapache2-mod-php5 php5-mysqlnd
sudo apt-get install phpmyadmin
sudo a2enmod rewrite
```
Edit /etc/apache2/sites_available/default to enable .htaccess processing (set AllowOverride to All)
Restart Apache: sudo /etc/init.d/apache2 restart

[Follow the standard installation instructions](http://learn.elgg.org/en/1.x/intro/install/ubuntu.html)
[Elgg](http://learn.elgg.org/en/1.x/intro/install.html)
[tutum lamp guide](https://hub.docker.com/r/tutum/lamp/)

## setup lamp with Docker
`boot2docker` for windows and mac osx
To enable access the container from outside, remember the `port forward` setting on the VM(on windows, via virtual box).
Use `tutum/lamp` as the image base.
Use `composer` to install elgg.

## elgg install
### port forward
40080 -> 80
43306 -> 3306
### get db user password.
``` bash
docker logs %containerid%
```
### change db password
login admin account
``` bash
SET PASSWORD = PASSWORD('newpassword');
```
### create database via phpadmin
``` bash
cd /path/to/wwwroot/
sudo ln -s /usr/share/phpmyadmin phpmyadmin
```
http://localhost:40080/phpmyadmin

### create the website from elgg
``` bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

``` bash
cd /path/to/wwwroot/
composer self-update
composer global require "fxp/composer-asset-plugin:~1.1.1"
composer create-project elgg/starter-project:dev-master .
# rename folder to elgg
mv starter-project elgg 

### additional settings
#### enable mod-rewrite
``` bash
vi /etc/apache2/sites-available/000-default.conf
``` conf
...
<Directory /var/www/elgg>
    AllowOverride All
...
```
#### update elgg settings
.htaccess
settings.php
``` bash
cd elgg
cp ../vendor/elgg/elgg/elgg-config/settings.example.php settings.php 
# update the database user password etc.
vi settings.php 
cp ../vendor/elgg/elgg/install/config/htaccess.dist .htaccess 
# update ??
vi .htaccess
```

### launch the elgg web installer
http://localhost:40080

## using docker with multi-container
### mysql container
`fan6/mysql`
Dockerfile
``` bash
FROM mysql:5.6

# Write Permission
RUN usermod -u 1000 mysql && chown mysql.mysql /var/run/mysqld/

EXPOSE 3306
VOLUME ["/opt"]
```
create & start the container with local path `~/root/db` mounted, so db data is persisted.
and with given mysql password.
``` bash
docker run -p 3306:3306 -v ~/root/db:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -it fan6/mysql
```

## Dockerize elgg application

Elgg docker image: https://hub.docker.com/r/keviocastro/elgg-docker/

Composer    image: https://hub.docker.com/r/composer/composer/


## Uderstanding ELGG
[Data model](http://learn.elgg.org/en/2.0/design/database.html)

[Web services](http://learn.elgg.org/en/2.0/guides/web-services.html)

[Helpers](http://learn.elgg.org/en/1.12/guides/helpers.html)

[Events](http://learn.elgg.org/en/1.12/guides/events-list.html)

## Plugins
https://packagist.org/packages/hypejunction/

