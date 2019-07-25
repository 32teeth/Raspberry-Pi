# Raspberry Pi - Virtual Hosts
---

## Description
This document outline a base instruction set for setting up a Raspberry Pi 2 with the following features

* Virtual Hosts

again, all commands are run as sudo 

```
sudo su
```


---


In order to create a new virtual host with a specific web root create a new directory in your webroot, in this example we call it ***'example.com'***, you will need to replace the word 'example.com' with whatever your new domain is

```
cd /var/www
mkdir example.com
cd example.com
```

create a new html file
```
nano index.html
```

```
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <!-- Place this data between the <head> tags of your website -->
    <title>example.com</title>    
  </head>
  <body>
  	<h1>example.com</h1>
  </body>
</html>  
```
*ctrl+x y*

##### create a new conf file

```
cd /etc/apache2/sites-available
nano example.com.conf
```

copy paste the below virtual host conf *(again, replace 'example.com' with whatever your new domain name is)*

```
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@example.com
    DocumentRoot /var/www/example.com
    CustomLog /var/log/apache2/example.com/access.log common
    ErrorLog /var/log/apache2/example.com/error.log
</VirtualHost>
```

*ctrl+x y*

add 'example.com' to logs
```
mkdir /var/log/apache2/example.com
```

add 'example.com' to hosts file
```
nano /etc/hosts
```

add these lines to the bottom

```
# virtual hosts
127.0.0.1       example.com
```

*ctrl+x y*

##### set up for apache to run

```
cd
chown -R www-data:www-data /var/www/example.com
a2ensite example.com
apache2ctl -t
service apache2 reload
```

ping your new domain

```
ping example.com
```

the response to the above should look like this

```
PING example.com (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.277 ms
```

test it in a browser by entering 'example.com' in the location. if all went well, you will see your new domain with a simple title and header

| *browser* |
|---:|
| **example.com** |

### bonus
want to change your root host name from 'raspberry' to whatever else?

in this case we changed it to example
```
nano /etc/hostname
```

this file can only contain one (1) word, change 'raspberry' to your new root host name

set and reboot
```
/etc/init.d/hostname.sh
reboot now
```

next time you log, you will see

```
root@example:~$
```
