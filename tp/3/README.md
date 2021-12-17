# TP3 : Your own shiet

## Prérequis

### ➜ Checklist

1. IP locale, statique
1.  hostname défini
1.  firewall actif, qui ne laisse passer que le strict nécessaire
1.  SSH fonctionnel avec un échange de clé
1.  accès Internet (une route par défaut, une carte NAT c'est très bien)
1.  résolution de nom



### **➜ Système :**

```
- Ubuntu 20.04 LTS (Recommendé)
- Ubuntu 18.04 LTS
- Ubuntu 16.04 LTS (Compatible, mise à jour recommandé)
- Debian 10 "Buster"
- Debian 9 "Stretch"
```



### **➜ Configuration requise :**

```
2 Go de RAM
20 Go ou plus d'espace disque dur
sudo,curl et git d'installer
```

## 1. Installation

🖥️ **VM web.webradio**

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.azura.linux` | `10.3.0.45` | Serveur Web             | 80        | ?             |

### **Installer d'Azura avec Docker !**

- Mise à jour de la machine
`paul@web:~$ sudo apt update`

- Création du fichier AzuraCast

`mkdir -p /var/azuracast`

`cd /var/azuracast`

- Téléchargement du Docket Utility Script.

sudo/!\ Avant tout toute l'installation devra ce faire sous root /!\

`su -`

`curl -fsSL https://raw.githubusercontent.com/AzuraCast/AzuraCast/main/docker.sh > docker.sh`

``` chmod a+x docker.sh```

```yes '' | ./docker.sh install```


**TEST**

- vérifier que le service est démarré

```
paul@web:~$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-11-08 12:12:32 UTC; 3min 47s ago
       Docs: man:nginx(8)
   Main PID: 1575 (nginx)
      Tasks: 3 (limit: 2279)
     Memory: 7.5M
     CGroup: /system.slice/nginx.service
             ├─1575 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─1576 nginx: worker process
             └─1577 nginx: worker process
```

- vérifier qu'il est configuré pour démarrer automatiquement

```
paul@web:~$ sudo systemctl enable nginx
Synchronizing state of nginx.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable nginx
```


- vérifier avec une commande `curl localhost` que vous joignez votre serveur web localement

```
paul@web:~$ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


- vérifier avec votre navigateur (sur votre PC) que vous accéder à votre serveur web

```
C:\Users\Paul>curl 10.3.0.45
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


![](./image/spongebob-too-easy.gif)

## Installation de MariaDB

**🌞 Install de MariaDB sur web.webradio**

`sudo apt install mariadb-server`

`paul@web:~$ sudo systemctl enable mariadb`

```
paul@web:~$ sudo systemctl status mariadb
● mariadb.service - MariaDB 10.3.31 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-11-08 12:35:18 UTC; 42s ago
       Docs: man:mysqld(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 2820 (mysqld)
     Status: "Taking your SQL requests now..."
      Tasks: 31 (limit: 2279)
     Memory: 65.0M
     CGroup: /system.slice/mariadb.service
             └─2820 /usr/sbin/mysqld
```

`sudo mysql_secure_installation`

```
paul@web:~$ sudo mysql -u root -p

MariaDB [(none)]> CREATE USER 'azura'@'10.3.0.45' IDENTIFIED BY 'paul';                                            
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS azuracloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.001 sec)


MariaDB [(none)]> GRANT ALL PRIVILEGES ON azuracloud.* TO 'azura'@'10.3.0.45';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> exit
Bye
```

**🌞 Préparation de la base pour Azura**

```
paul@web:~$ mysql -u azura -h 10.3.0.45 -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 44
Server version: 10.3.31-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

```
MariaDB [azuracloud]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| azuracloud         |
| information_schema |
+--------------------+
2 rows in set (0.001 sec)
```

```
MariaDB [(none)]> USE azuracloud;
Database changed
```

```
MariaDB [azuracloud]> SHOW TABLES;
Empty set (0.000 sec)
```

## Renforcement de la sécurité :

**1 – Root c’est fini, bonjour admin**

`su -`

`Nous allons créer et renforcer un user j'ai donc généré un mot de passe assez compliquer`

```
root@web:~# adduser pv
Adding user `pv' ...
Adding new group `pv' (1001) ...
Adding new user `pv' (1001) with group `pv' ...
Creating home directory `/home/pv' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for pv
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []:
```

```
On ajoute l’utilisateur aux groupes lui permettant d’avoir les privilèges root (j’ai rajouté le groupe adm pour pouvoir consulter les logs sans devoir utiliser sudo) :

root@web:~# usermod -aG root,sudo,adm pv
```

( pas fonctionnel a modifier ) On modifie maintenant la configuration SSH pour interdire la connexion en tant que root dans /etc/ssh/sshd_config.

```
...
# Interdire à root de se connecter en SSH
PermitRootLogin no 
# Facultatif : Seul admin peux se connecter en SSH
AllowUsers pv
# Facultatif : Seul les utilisateurs du groupe sshusers peuvent se connecter en SSH
AllowGroups adm
...       
```

**2 - Maîtrise des portes d’entrées / sorties**

`sudo apt install iptables`

**Définissions des règles en créant un script**

📁 Fichier `/etc/init.d/firewall`

sudo chmod +x /etc/init.d/firewall

```
pv@web:~$ sudo /etc/init.d/firewall start
[sudo] password for pv: 
firewall started [OK]
```

exécuté au démarrage pour que jamais le firewall ne soit corrompu 

` sudo update-rc.d firewall defaults`

# 3 - Fail2Ban

`sudo apt install fail2ban`

`systemctl start fail2ban`

```
pv@web:/etc/fail2ban$ sudo systemctl status fail2ban
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-11-30 10:37:22 UTC; 1min 31s ago
       Docs: man:fail2ban(1)
    Process: 12880 ExecStartPre=/bin/mkdir -p /run/fail2ban (code=exited, status=0/SUCCESS)
   Main PID: 12887 (f2b/server)
      Tasks: 5 (limit: 2279)
     Memory: 11.7M
     CGroup: /system.slice/fail2ban.service
             └─12887 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
```

`systemctl enable fail2ban`

### **Créer le fichier dans /etc/fail2ban/jail.d/custom.conf**

```
pv@web:/etc/fail2ban$ sudo cat /etc/fail2ban/jail.d/custom.conf
[DEFAULT]
ignoreip = 10.3.0.45 77.196.149.138 10.33.16.255
findtime = 10m
bantime = 24h
maxretry = 3
```

* ignoreip ⇒ mon IP en plus de l'interface de bouclage ;
* findtime = 10m (10 minutes), 3600 secondes (une heure)
* bantime = 24h (24 heures) ou 86400 secondes
* maxretry = 3 (une IP sera bannie après 3 tentatives de connexion avortées).

### **Configurer fail2ban pour les services actifs**

*Dans le fichier /etc/fail2ban/jail.conf, dans la partie jail vous trouverez des blocs du type :*

```
[sshd]

port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

* port = les ports à bloquer au moyen des règles iptables ;
* logpath = l'emplacement des fichiers de log à surveiller ;
* backend = le moteur de surveillance des logs.

*Pour activer la surveillance des connexion SSH, il suffit d'ajouter dans le fichier /etc/fail2ban/jail.d/custom.conf :*

```
[sshd]
enabled = true
```

`sudo systemctl restart fail2ban`

```
pv@web:~$ sudo fail2ban-client status
[sudo] password for pv: 
Status
|- Number of jail:      1
 - Jail list:   sshd
```

### **Vérifier le bon fonctionnement de votre configuration Fail2Ban**

```
pv@web:~$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|   - File list:        /var/log/auth.log
 - Actions
   |- Currently banned: 0
   |- Total banned:     0
    - Banned IP list:
```

### **Configuration avancée**

***Recevoir un email après chaque bannissement :***

Pour cela installer le paquet **ssmtp**

`sudo apt install ssmtp`

*Ouvrez le fichier le fichier /etc/ssmtp/ssmtp.conf et modifier les informations souhaitées*

```
root@web:~# sudo cat /etc/ssmtp/ssmtp.conf
#
# Config file for sSMTP sendmail
#
# The person who gets all mail for userids < 1000
# Make this empty to disable rewriting.
root=paulvigneron2@gmail.com

# The place where the mail goes. The actual machine name is required no
# MX records are consulted. Commonly mailhosts are named mail.domain.com
mailhub=smtp.gmail.com:587

# Where will the mail seem to come from?
rewriteDomain=

# The full hostname
hostname=paulvigneron2@gmail.com

# Are users allowed to set their own From: address?
# YES - Allow the user to specify their own From: address
# NO - Use the system generated From: address
FromLineOverride=YES

# Nom d'utilisateur du compte email avec lequel vous envoyez les courriels
AuthUser=paulvigneron2@gmail.com
AuthPass=paul

UseSTARTTLS=YES
```

**Configuration de revaliases**

*Ouvrez le fichier le fichier /etc/ssmtp/revaliases*

```
# sSMTP aliases
#
# Format:       local_account:outgoing_address:mailhub
#
root:paulvigneron2@gmail.com:smtp.gmail.com:587       
pierre:paulvigneron2@gmail.com:smtp.gmail.com:587
```

**Ouverture des ports**

Ouverture du port 25 en tcp en sortie (smtp simple) :
`sudo ufw allow out 25`

Ouverture du port 465 en tcp en sortie (smtp sécurisé) :
`sudo ufw allow out 465/tcp`

Ouverture du port 587 en tcp en sortie (smtp sécurisé avec TLS/SSL) :
`sudo ufw allow out  587/tcp`

