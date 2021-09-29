# TP2 pt. 1 : Gestion de service

## I. Un premier serveur web

### 1. Installation

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | `80`           | ?             |

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

`[paul@web httpd]$ [paul@web httpd]$ sudo ps -ef`

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
 
 `[paul@web www]$ sudo useradd toto`
 
* Pour les options de création, inspirez-vous de l'utilisateur Apache existant
 
` [paul@web ~]$ sudo cat /etc/passwd`
 
 `[paul@web ~]$ sudo nano /etc/passwd`
* Modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur

`sudo nano /etc/httpd/conf/httpd.conf`

`User : polo`
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
