# Raspberry Pi - Server
---

## Description
This document outline a base instruction set for setting up a Raspberry Pi 2 with the following features

* Apache
* MySQL
* PHP
* Wordpress

---


again, all commands are run as sudo 

```
sudo su
```

### Install Apache


```
apt-get install apache2 -y
```

### Install PHP

```
apt-get install php5 libapache2-mod-php5 -y
```

### Install MySQL

```
apt-get install mysql-server php5-mysql -y
```
MySQL you will be asked for a root password. 

You'll need to remember this to allow your website to access the database.

### Download, Install and Run WordPress

```
cd /var/www/html/
sudo chown pi: .
sudo rm *
wget http://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz
mv wordpress/* .
rm -rf wordpress latest.tar.gz
```

#### Create a database for WordPress
```
mysql -uroot -p
```
at the mydql prompt

```
mysql > create database wordpress;
mysql > exit
```

open a browser on your pi and navigate to **http://localhost**, WordPress configuration takes over here
