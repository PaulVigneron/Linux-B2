# TP2 pt. 1 : Gestion de service

## I. Un premier serveur web

### 1. Installation

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | `80`          | `10.102.1.12`             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données | `3306`           | `10.102.1.11`             |

**🌞 Installer le serveur Apache**

```
[paul@web ~]$ sudo dnf install httpd
Last metadata expiration check: 0:07:47 ago on Wed 29 Sep 2021 03:42:05 PM CEST.
Package httpd-2.4.37-39.module+el8.4.0+571+fd70afb1.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

**🌞 Démarrer le service Apache**


* Démarrez le
 
`[paul@web ~]$ sudo systemctl start httpd`


* Faites en sorte qu'Apache démarre automatique au démarrage de la machine
```
[paul@web ~]$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

* Ouvrez le port firewall nécessaire



```
[paul@web ~]$ sudo ss -alnpt
State           Recv-Q          Send-Q                   Local Address:Port                   Peer Address:Port         Process
LISTEN          0               128                                  *:80                                *:*             users:(("httpd",pid=2014,fd=4),("httpd",pid=2013,fd=4),("httpd",pid=2012,fd=4),("httpd",pid=2010,fd=4))
 [...]
```

`sudo firewall-cmd --add-port=80/tcp`

🌞 TEST

* Vérifier que le service est démarré

```
[paul@web ~]$ sudo service httpd status
Redirecting to /bin/systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-09-29 15:52:27 CEST; 16min ago
     Docs: man:httpd.service(8)
 Main PID: 2010 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 4947)
   Memory: 25.6M
   CGroup: /system.slice/httpd.service
           ├─2010 /usr/sbin/httpd -DFOREGROUND
           ├─2011 /usr/sbin/httpd -DFOREGROUND
           ├─2012 /usr/sbin/httpd -DFOREGROUND
           ├─2013 /usr/sbin/httpd -DFOREGROUND
           └─2014 /usr/sbin/httpd -DFOREGROUND

Sep 29 15:52:27 web.tp2.linux systemd[1]: Starting The Apache HTTP Server...
Sep 29 15:52:27 web.tp2.linux systemd[1]: Started The Apache HTTP Server.
Sep 29 15:52:27 web.tp2.linux httpd[2010]: Server configured, listening on: port 80
```

* Vérifier qu'il est configuré pour démarrer automatiquement

```
[paul@web ~]$ sudo systemctl is-enabled httpd
enabled
```

* Vérifier avec une commande curl localhost que vous joignez votre serveur web localement

```
[paul@web ~]$ curl localhost
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;

      }
      
[...]
```

* Vérifier avec votre navigateur (sur votre PC) que vous accéder à votre serveur web

```
PS C:\Users\Paul> curl 10.102.1.11
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
Just visiting?
This website you are visiting is either experiencing problems or could be going through maintenance.
If you would like the let the administrators of this website know that you've seen this page instead of the page
you've expected, you should send them an email. In general, mail sent to the name "webmaster" and directed to the
website's domain should reach the appropriate person.
The most common email address to send to is: "webmaster@example.com"
[...]
```

### 2. Avancer vers la maîtrise du service

**🌞 Le service Apache...**

* Donnez la commande qui permet d'activer le démarrage automatique d'Apache quand la machine s'allume

`sudo systemctl enable httpd`

* Prouvez avec une commande qu'actuellement, le service est paramétré pour démarré quand la machine s'allume

`sudo systemctl is-enabled httpd`

* Affichez le contenu du fichier httpd.service qui contient la définition du service Apache

```
# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

#       [Service]
#       Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

**🌞 Déterminer sous quel utilisateur tourne le processus Apache**

* Mettez en évidence la ligne dans le fichier de conf qui définit quel user est utilisé
 
`sudo cat /etc/httpd/conf/httpd.conf`
 
```
[...] 
User apache
Group apache
[...]
```
 
* Utilisez la commande ps -ef pour visualiser les processus en cours d'exécution et confirmer que apache tourne bien sous l'utilisateur mentionné dans le fichier de conf

`[paul@web httpd]$ sudo ps -ef`

```
apache      2011    2010  0 15:52 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2012    2010  0 15:52 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2013    2010  0 15:52 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2014    2010  0 15:52 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```

* Vérifiez avec un ls -al le dossier du site (dans /var/www/...)

```
[paul@web www]$ ls -al
total 4
drwxr-xr-x.  4 root root   33 Sep 29 15:42 .
drwxr-xr-x. 22 root root 4096 Sep 29 15:42 ..
drwxr-xr-x.  2 root root    6 Jun 11 17:35 cgi-bin
drwxr-xr-x.  2 root root    6 Jun 11 17:35 html
```


**🌞 Changer l'utilisateur utilisé par Apache**

* Créez le nouvel utilisateur
 
 `[paul@web ~]$ sudo useradd toto`
 
* Pour les options de création, inspirez-vous de l'utilisateur Apache existant
 
` [paul@web ~]$ sudo cat /etc/passwd`
 
 `[paul@web ~]$ sudo nano /etc/passwd`
* Modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur

`sudo nano /etc/httpd/conf/httpd.conf`

`User : toto`
* Redémarrez Apache

`systemctl restart httpd`

```
[paul@web ~]$ sudo service httpd status
Redirecting to /bin/systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-09-29 17:10:24 CEST; 30s ago
     Docs: man:httpd.service(8)
 Main PID: 2801 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 213 (limit: 4947)
   Memory: 24.3M
   CGroup: /system.slice/httpd.service
           ├─2801 /usr/sbin/httpd -DFOREGROUND
           ├─2803 /usr/sbin/httpd -DFOREGROUND
           ├─2804 /usr/sbin/httpd -DFOREGROUND
           ├─2805 /usr/sbin/httpd -DFOREGROUND
           └─2806 /usr/sbin/httpd -DFOREGROUND

Sep 29 17:10:24 web.tp2.linux systemd[1]: httpd.service: Succeeded.
Sep 29 17:10:24 web.tp2.linux systemd[1]: Stopped The Apache HTTP Server.
Sep 29 17:10:24 web.tp2.linux systemd[1]: Starting The Apache HTTP Server...
Sep 29 17:10:24 web.tp2.linux systemd[1]: Started The Apache HTTP Server.
Sep 29 17:10:24 web.tp2.linux httpd[2801]: Server configured, listening on: port 80
```

* Utilisez une commande ps pour vérifier que le changement a pris effet

```
[paul@web ~]$ ps -ef | grep polo
polo        2803    2801  0 17:10 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
polo        2804    2801  0 17:10 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
polo        2805    2801  0 17:10 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
polo        2806    2801  0 17:10 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
paul        3039    2686  0 17:12 pts/0    00:00:00 grep --color=auto polo
```

**🌞 Faites en sorte que Apache tourne sur un autre port**

* Modifiez la configuration d'Apache pour lui demande d'écouter sur un autre port

[paul@web ~]$ sudo nano /etc/httpd/conf/httpd.conf

`Listen 08`



* Ouvrez un nouveau port firewall, et fermez l'ancien

`sudo firewall-cmd --add-port=08/tcp`

```
[paul@web ~]$ [paul@web ~]$  sudo firewall-cmd --remove-port=80/tcp --permanent
success
```

* Redémarrez Apache

```
[paul@web ~]$ systemctl restart httpd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to restart 'httpd.service'.
Authenticating as: paul
Password:
==== AUTHENTICATION COMPLETE ====
```

* Prouvez avec une commande ss que Apache tourne bien sur le nouveau port choisi

```
[paul@web ~]$ sudo ss -alnpt

[...]
LISTEN     0          128                        *:8                       *:*        users:(("httpd",pid=3313,fd=4),("httpd",pid=3312,fd=4),("httpd",pid=3311,fd=4),("httpd",pid=3308,fd=4))
[...]
```

* Vérifiez avec curl en local que vous pouvez joindre Apache sur le nouveau port

```
[paul@web ~]$ curl localhost:08
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(20,72,50,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;

      }
[...]
```

* Vérifiez avec votre navigateur que vous pouvez joindre le serveur sur le nouveau port
 
```
PS C:\Users\Paul> curl 10.102.1.11:8
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
Just visiting?
This website you are visiting is either experiencing problems or could be going through maintenance.
If you would like the let the administrators of this website know that you've seen this page instead of the page
you've expected, you should send them an email. In general, mail sent to the name "webmaster" and directed to the
website's domain should reach the appropriate person.
The most common email address to send to is: "webmaster@example.com"
```

📁 Fichier /etc/httpd/conf/httpd.conf

## II. Une stack web plus avancée

### 2. Setup

**A. Serveur Web et NextCloud**


**🌞 Install du serveur Web et de NextCloud sur web.tp2.linux**

```

11  dnf install epel-release
12  sudo dnf install epel-release
15  sudo dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
16  dnf module list php
18  sudo dnf -y module enable php:remi-7.4
20  sudo dnf -y install httpd mariadb-server vim wget zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp
21  systemctl enable httpd
22  sudo mkdir /etc/httpd/sites-available
23  sudo mkdir /etc/httpd/sites-enabled
24  sudo nano /etc/httpd/sites-available/web.tp2.linux
26  sudo ln -s /etc/httpd/sites-available/web.tp2.linux /etc/httpd/sites-enabled/
28  sudo mkdir -p /var/www/sub-domains/web.tp2.linux/html
29  cd /usr/share/zoneinfo
30  vi /etc/opt/remi/php74/php.ini
31  sudo vi /etc/opt/remi/php74/php.ini
33  cd
34  sudo wget https://download.nextcloud.com/server/releases/nextcloud-21.0.1.zip
35  sudo vim /etc/httpd/conf/httpd.conf
36  unzip nextcloud-21.0.1.zip
37  cd nextcloud/
38  cp -Rf * /var/www/sub-domains/web.tp2.linux/html/
39  sudo cp -Rf * /var/www/sub-domains/web.tp2.linux/html/
40  chown -Rf apache.apache /var/www/sub-domains/web.tp2.linux/html
41  mv /var/www/sub-domains/web.tp2.linux/html/data /var/www/sub-domains/web.tp2.linux/
42  sudo mv /var/www/sub-domains/web.tp2.linux/html/data /var/www/sub-domains/web.tp2.linux/
44  cd
46  sudo systemctl restart httpd
47  sudo systemctl restart mariadb
48  sudo firewall-cmd --add-port=3306/tcp
49  sudo firewall-cmd --add-port=3306/tcp --permanent
50  sudo firewall-cmd --reload
52  sestatus
53  systemctl status httpd
54  cd /etc/https
55  cd /etc/httpd
56  ls
57  ls -al sites*
58  sudo vim conf/httpd.conf
59  sudo systemctl status httpd
60  sudo systemctl restart httpd
61  sudo ls -al /var/log/httpd/
62  cat sites-enabled/
63  cat sites-enabled/web.tp2.linux
64  ls -al /var/www/sub-domains/web.tp2.linux/html/
65  chown -Rf apache.apache /var/www/sub-domains/web.tp2.linux/html
68  chown -Rf apache.apache /var/www/sub-domains/web.tp2.linux/html
69  ls -al chown -Rf apache.apache /var/www/sub-domains/com.yourdomain.nextcloud/html
70  ls -al /var/www/sub-domains/web.tp2.linux/html
71  chown -R apache.apache /var/www/sub-domains/web.tp2.linux/html
72  sudo chown -R apache.apache /var/www/sub-domains/web.tp2.linux/html
73  ls -al /var/www/sub-domains/web.tp2.linux/html
74  clear
75  cd
76  history
```

📁 Fichier /etc/httpd/conf/httpd.conf /etc/httpd/sites-available/web.tp2.linux

**B. Base de données**

**🌞 Install de MariaDB sur db.tp2.linux**



```
     sudo dnf install mariadb-server
     sudo systemctl enable mariadb
     systemctl start mariadb
     mysql_secure_installation
```

* Vous repérerez le port utilisé par MariaDB avec une commande ss exécutée sur db.tp2.linux

```
[paul@db ~]$ sudo ss -alnpt
[sudo] password for paul:
State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process
[...]
LISTEN     0          80                         *:3306                    *:*        users:(("mysqld",pid=4700,fd=21))
```

**🌞 Préparation de la base pour NextCloud**

* Connectez-vous à la base de données à l'aide de la commande sudo mysql -u root

```
[paul@db ~]$ sudo mysql -u root -p
[sudo] password for paul:
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.3.28-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'paul';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)
```

**🌞 Exploration de la base de données**
```

[paul@web ~]$ mysql -u nextcloud -h 10.102.1.12 -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 17
Server version: 10.3.28-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```

```

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nextcloud          |
+--------------------+
2 rows in set (0.001 sec)

MariaDB [(none)]> USE nextcloud;
Database changed
MariaDB [nextcloud]> SHOW TABLES;
Empty set (0.001 sec)

```

* Trouver une commande qui permet de lister tous les utilisateurs de la base de données

```
MariaDB [(none)]> select user,host from mysql.user;
+-----------+-------------+
| user      | host        |
+-----------+-------------+
| nextcloud | 10.102.1.11 |
| root      | 127.0.0.1   |
| root      | ::1         |
| root      | localhost   |
+-----------+-------------+
4 rows in set (0.000 sec)
```

### C. Finaliser l'installation de NextCloud

**🌞 sur votre PC**
```

PS C:\WINDOWS\system32> C:\Windows\System32\drivers\etc\hosts
```

```
#
127.0.0.1 localhost
::1 localhost
10.102.1.11 web.tp2.linux
```

**🌞 Exploration de la base de données**

```
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| nextcloud          |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec)
```
