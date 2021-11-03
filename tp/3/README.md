# TP3 : Your own shiet

## 1. Installation

🖥️ **VM web.webradio**

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.webradio` | `192.168.56.41` | Serveur Web             | 80        | ?             |

### **Installer le serveur NGINX**

***Mise à jour de la machine***
`paul@web:~$ sudo apt update`

***Installation de NGINX***
`sudo apt install nginx`

***Ajouts du port 80***
```
paul@web:~$ sudo firewall-cmd --add-port=80/tcp --permanent
success
```

***Test du fonctionnement de nginx***
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

```
paul@web:~$ curl 192.168.56.41
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
