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

- Téléchargement du Docket Utility Script

`curl -fsSL https://raw.githubusercontent.com/AzuraCast/AzuraCast/main/docker.sh > docker.sh`

``` chmod a+x docker.sh ```

```yes '' | ./docker.sh install```

![](./image/magique.gif)
