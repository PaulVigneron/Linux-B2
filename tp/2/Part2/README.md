# TP2 pt. 2 : Maintien en condition opérationnelle

## I. Monitoring

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | `80`          | `10.102.1.12`             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données | `3306`           | `10.102.1.11`             |

### 2. Setup

**🌞 Setup Netdata**

`[root@db ~]# bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)`

```
  ^
  |.-.   .-.   .-.   .-.   .-.   .  netdata              .-.   .-.   .-.   .-
  |   '-'   '-'   '-'   '-'   '-'   is installed now!  -'   '-'   '-'   '-'
  +----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+--->
```

**🌞 Manipulation du service Netdata**

* Déterminer s'il est actif, et s'il est paramétré pour démarrer au boot de la machine

```
[paul@db ~]$ sudo systemctl status netdata
● netdata.service - Real time performance monitoring
   Loaded: loaded (/usr/lib/systemd/system/netdata.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2021-10-12 14:32:01 CEST; 10min ago
  Process: 2076 ExecStartPre=/bin/chown -R netdata:netdata /opt/netdata/var/run/netdata (code=exited, status=0/SUCCESS)
  Process: 2074 ExecStartPre=/bin/mkdir -p /opt/netdata/var/run/netdata (code=exited, status=0/SUCCESS)
  Process: 2072 ExecStartPre=/bin/chown -R netdata:netdata /opt/netdata/var/cache/netdata (code=exited, status=0/SUCCESS)
  Process: 2071 ExecStartPre=/bin/mkdir -p /opt/netdata/var/cache/netdata (code=exited, status=0/SUCCESS)
 [...]
```

`[paul@db ~]$ sudo systemctl enable netdata`

* Déterminer à l'aide d'une commande ss sur quel(s) port(s) Netdata écoute

```
[paul@db ~]$ sudo ss -alnpt
State       Recv-Q      Send-Q           Local Address:Port            Peer Address:Port     Process
LISTEN      0           128                  127.0.0.1:8125                 0.0.0.0:*         users:(("netdata",pid=2078,fd=33))
LISTEN      0           128                    0.0.0.0:19999                0.0.0.0:*         users:(("netdata",pid=2078,fd=5))         users:(("sshd",pid=857,fd=7))
LISTEN      0           128                      [::1]:8125                    [::]:*         users:(("netdata",pid=2078,fd=32))
LISTEN      0           128                       [::]:19999                   [::]:*         users:(("netdata",pid=2078,fd=6))
```

```
[paul@db ~]$ curl 10.102.1.11:19999
<!doctype html><html lang="en"><head><title>netdata dashboard</title><meta name="application-name" content="netdata"><meta http-equiv="Content-Type" content="text/html; charset=utf-8"/><meta charset="utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"><meta name="viewport"
[...]
``` 

```
[paul@web ~]$ curl 10.102.1.12:19999
<!doctype html><html lang="en"><head><title>netdata dashboard</title><meta name="application-name" content="netdata"><meta http-equiv="Content-Type" content="text/html; charset=utf-8"/><meta charset="utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"><meta name="viewport" content="width=device-width,initial-scale=1"><meta name="apple-mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-status-bar-style" content="black
[...]
```

**🌞 Setup Alerting**

* Ajustez la conf de Netdata pour mettre en place des alertes Discord

```
# discord (discordapp.com) global notification options

# multiple recipients can be given like this:
#                  "CHANNEL1 CHANNEL2 ..."

# enable/disable sending discord notifications
SEND_DISCORD="YES"

# Create a webhook by following the official documentation -
# https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/897470829867175967/YPHs59qW11Ft_6qRZg3uyUyNblW3F0ssMs6wU-Z_PqLL3Xcztc9W8wWT8LRzHSxWwXxO"

# if a role's recipients are not configured, a notification will be send to
# this discord channel (empty = do not send a notification for unconfigured
# roles):
DEFAULT_RECIPIENT_DISCORD="alert-paul"
```

* Vérifiez le bon fonctionnement de l'alerting sur Discord

```
[paul@web ~]$ sudo /opt/netdata/usr/libexec/netdata/plugins.d/alarm-notify.sh test


# SENDING TEST WARNING ALARM TO ROLE: sysadmin
2021-10-12 15:36:42: alarm-notify.sh: INFO: sent discord notification for: web.tp2.linux test.chart.test_alarm is WARNING to 'alert-paul'
# OK

# SENDING TEST CRITICAL ALARM TO ROLE: sysadmin
2021-10-12 15:36:43: alarm-notify.sh: INFO: sent discord notification for: web.tp2.linux test.chart.test_alarm is CRITICAL to 'alert-paul'
# OK

# SENDING TEST CLEAR ALARM TO ROLE: sysadmin
2021-10-12 15:36:44: alarm-notify.sh: INFO: sent discord notification for: web.tp2.linux test.chart.test_alarm is CLEAR to 'alert-paul'
# OK
```

**🌞 Config alerting**

* Créez une nouvelle alerte pour recevoir une alerte à 50% de remplissage de la RAM

`sudo /opt/netdata/etc/netdata/edit-config health.d/ram.conf`

```
     warn: $this > (($status >= $WARNING)  ? (50) : (90))
     crit: $this > (($status == $CRITICAL) ? (90) : (98))
```

* Testez que votre alerte fonctionne

`[paul@web ~]$ sudo stress --vm 4 --vm-bytes 1024M`

## II. Backup

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | ?           | ?             |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?             |

### 2. Partage NFS

**🌞 Setup environnement**

```
sudo mkdir /srv/backup/
cd /srv/backup/
sudo mkdir web.tp2.linux
```

**🌞 Setup partage NFS**



```
sudo dnf -y install nfs-utils
   62  sudo vi /etc/idmapd.conf
   63  /etc/idmapd.conf
   64  sudo /etc/idmapd.conf
   65  sudo nano /etc/idmapd.conf
   66  sudo vi /etc/exports
   67  systemctl enable --now rpcbind nfs-server
   68  firewall-cmd --add-service=nfs
   69  sudo firewall-cmd --add-service=nfs
   70  firewall-cmd --runtime-to-permanent
   71  sudo firewall-cmd --runtime-to-permanent
   72  sudo firewall-cmd --reload
```

**🌞 Setup points de montage sur web.tp2.linux**

* Monter le dossier /srv/backups/web.tp2.linux du serveur NFS dans le dossier /srv/backup/ du serveur Web

`sudo mount -t nfs backup.tp2.linux:/srv/backup/web.tp2.linux /srv/backup`

* Avec une commande mount que la partition est bien montée

```
[paul@web ~]$ mount | grep backup
backup.tp2.linux:/srv/backup/web.tp2.linux on /srv/backup type nfs4 (rw,relatime,vers=4.2,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.102.1.11,local_lock=none,addr=10.102.1.13)
```

* Avec une commande df -h qu'il reste de la place

```
[paul@web ~]$ df -h
Filesystem                                  Size  Used Avail Use% Mounted on
[...]
backup.tp2.linux:/srv/backup/web.tp2.linux  6.2G  2.3G  4.0G  36% /srv/backup
```

* Avec une commande touch que vous avez le droit d'écrire dans cette partition

`[paul@web ~]$ sudo touch /srv/backup/toto`

```
[paul@web backup]$ ls -all
total 0
[...]
-rw-r--r--. 1 root root  0 Oct 12 17:22 toto
```

* Faites en sorte que cette partition se monte automatiquement grâce au fichier /etc/fstab

`sudo vim /etc/fstab`

`backup.tp2.linux:/srv/backup/web.tp2.linux /srv/backup  nfs     defaults        0 0`

**🌟 BONUS : partitionnement avec LVM**

* Ajoutez un disque à la VM backup.tp2.linux

`On créer le disque sur la VM`

```
[paul@backup ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
[...]
sdb           8:16   0    5G  0 
[...]
```

```
[paul@backup ~]$ sudo pvcreate /dev/sdb
[sudo] password for paul:
  Physical volume "/dev/sdb" successfully created.
```

```
[paul@backup ~]$ sudo cat /etc/fstab
[...]
/srv/backup/    /dev/sdb/       nfs     defaults        0 0
```

### 3. Backup de fichiers

**🌞 Rédiger le script de backup /srv/tp2_backup.sh**


📁 Fichier /srv/tp2_backup.sh


**🌞 Tester le bon fonctionnement**

```
[paul@backup backup]$ sudo ./tp2_linux.sh /srv/backup/web.tp2.linux/ /home/paul/test/
[OK] Archive /srv/backup/tp2_backup_211013_190036.tar.gz created.
[OK] Archive /srv/backup/tp2_backup_211013_190036.tar.gz synchronized to /srv/backup/web.tp2.linux/.
[OK] Directory /srv/backup/web.tp2.linux/ cleaned to keep only the 5 most recent backups.
[paul@backup backup]$ cd web.tp2.linux/
```

```
[paul@backup web.tp2.linux]$ ls
tp2_backup_211013_190036.tar.gz
```

```
[paul@backup web.tp2.linux]$ sudo tar -xvzf tp2_backup_211013_190036.tar.gz
[sudo] password for paul:
home/paul/test/
home/paul/test/toto.txt
```

### 4. Unité de service

#### A. Unité de service


**🌞 Créer une unité de service pour notre backup**

```
[paul@backup system]$ cat tp2_backup.service
[Unit]
Description=Script Backup et compression

[Service]
ExecStart=/srv/backup/tp2_backup.sh /srv/backup/web.tp2.linux/ /home/paul/test/
Type=oneshot
RemainAfterExit=no

[Install]
WantedBy=multi-user.target
```

**🌞 Tester le bon fonctionnement**

```
[paul@backup system]$ sudo systemctl status tp2_backup.service
[...]
Oct 13 19:35:39 backup.tp2.linux tp2_backup.sh[3050]: [OK] Archive //tp2_backup_211013_193539.tar.gz created.
Oct 13 19:35:39 backup.tp2.linux tp2_backup.sh[3050]: [OK] Archive //tp2_backup_211013_193539.tar.gz synchronized to /srv/backup/web.tp>
Oct 13 19:35:39 backup.tp2.linux tp2_backup.sh[3050]: [OK] Directory /srv/backup/web.tp2.linux/ cleaned to keep only the 5 most recent >
Oct 13 19:35:39 backup.tp2.linux systemd[1]: tp2_backup.service: Succeeded.
Oct 13 19:35:39 backup.tp2.linux systemd[1]: Started Script Backup et compression.
```

```
[paul@backup system]$ sudo systemctl daemon-reload
[paul@backup system]$ sudo systemctl start tp2_backup.service
```


```
[pauln@backup web.tp2.linux]$ ls
tp2_backup_211013_190036.tar.gz  tp2_backup_211013_193539.tar.gz
```

B. Timer

**🌞 Créer le timer associé à notre tp2_backup.service**


`sudo touch tp2_backup.timer`

```
[paul@backup system]$ sudo systemctl start tp2_backup.timer
[paul@backup system]$ sudo systemctl enable tp2_backup.timer
Created symlink /etc/systemd/system/timers.target.wants/tp2_backup.timer → /etc/systemd/system/tp2_backup.timer.
```

```
[paulbackup system]$ sudo systemctl status tp2_backup.timer
● tp2_backup.timer - Periodically run our TP2 backup script
   Loaded: loaded (/etc/systemd/system/tp2_backup.timer; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-10-13 19:45:38 CEST; 1min 2s ago
  Trigger: n/a

Oct 13 19:45:38 backup.tp2.linux systemd[1]: Started Periodically run our TP2 backup script.
```


**🌞 Tests !**

```
[paul@backup web.tp2.linux]$ ls
tp2_backup_211013_193539.tar.gz  tp2_backup_211013_194641.tar.gz  tp2_backup_211013_194843.tar.gz
tp2_backup_211013_194538.tar.gz  tp2_backup_211013_194743.tar.g
```

#### C. Contexte


**🌞 Faites en sorte que...**

```
[paul@web system]$ sudo systemctl list-timers
NEXT                          LEFT     LAST                          PASSED  UNIT                         ACTIVATES
Fri 2021-10-15 03:15:00 CEST  13h left n/a                           n/a     tp2_backup.timer             tp2_backup.service
```




