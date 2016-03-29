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

## setup lamp with Docker
`boot2docker` for windows and mac osx
To enable access the container from outside, remember the `port forward` setting on the VM(on windows, via virtual box).
Use `tutum/lamp` as the image base.
Use `composer` to install elgg.
