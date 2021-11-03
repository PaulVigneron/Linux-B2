# TP3 : Your own shiet

## 1. Installation

🖥️ **VM web.webradio**

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.webradio` | `192.168.56.41` | Serveur Web             | 80        | ?             |

### **Installer le serveur NGINX**

- paquet `nginx`

***Mise à jour de la machine***
`paul@web:~$ sudo apt update`

***Installation de NGINX***
`sudo apt install nginx`

***Ajouts du port 80***
```
paul@web:~$ sudo firewall-cmd --add-port=80/tcp --permanent
success
```

**Démarrer le service Nginx**

`paul@web:~$ sudo systemctl start nginx`

**TEST**

- vérifier que le service est démarré

```
paul@web:~$ sudo systemctl status nginx
sudo: impossible de résoudre l'hôte web.webradio: Nom ou service inconnu
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-11-03 13:15:19 CET; 10min ago
       Docs: man:nginx(8)
   Main PID: 2435 (nginx)
      Tasks: 3 (limit: 4679)
     Memory: 8.5M
        CPU: 19ms
     CGroup: /system.slice/nginx.service
             ├─2435 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─2437 nginx: worker process
             └─2438 nginx: worker process
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
C:\Users\GeekE>curl 192.168.56.41
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

Configuration des blocs de serveur

### **Configuration des blocs du serveur**

`sudo mkdir -p /var/www/web.webradio.linux/html`

`sudo chown -R $USER:$USER /var/www/web.webradio.linux/html`

```
paul@web:/var/www$ cat /var/www/web.webradio.linux/html/index.html
<html>
    <head>
        <title>Welcome to web.webradio.linux</title>
    </head>
    <body>
        <h1>Success! Your Nginx server is successfully configured for <em>web.webradio.linux</em>. </h1>
<p>This is a sample page.</p>
    </body>
</html>
```

```
paul@web:/var/www$ cat /etc/nginx/sites-available/web.webradio.linux
server {
        listen 80;
        listen [::]:80;

        root /var/www/web.webradio.linux/html;
        index index.html index.htm index.nginx-debian.html;

        server_name web.webradio.linux www.web.webradio.linux;

        location / {
                try_files $uri $uri/ =404;
        }
```

`sudo ln -s /etc/nginx/sites-available/web.webradio.linux /etc/nginx/sites-enabled/`

```
paul@web:/var/www$ cat /etc/nginx/nginx.conf
[...]
        server_names_hash_bucket_size 64;
[...]
```

`sudo nginx -t`

`sudo systemctl restart nginx`

```
paul@web:/var/www$ curl web.webradio.linux
<html>
    <head>
        <title>Welcome to web.webradio.linux</title>
    </head>
    <body>
        <h1>Success! Your Nginx server is successfully configured for <em>web.webradio.linux</em>. </h1>
<p>This is a sample page.</p>
    </body>
</html>
```

## Installation AzuraCast

