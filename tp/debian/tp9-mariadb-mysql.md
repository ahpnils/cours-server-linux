# TP 9 : MariaDB

Objectifs :

- installer MariaDB ;
- effectuer une configuration initiale "sécurisée";
- créer des utilisateurs ;
- créer ne
- installer une application PHP.

## Introduction

MariaDB est un système de gestion de base de données édité sous licence GPL.
Il s'agit d'un fork de MySQL, ce dernier appartenant à la société Oracle.

Du fait de son origine, de nombreuses commandes dans MariaDB portent le nom
`mysql`.

## Etape 0 : installation

Sur server11, installer le paquet `mariadb-server`. Pour la configuration
initiale, lancer la commande `mysql_secure_installation `, puis répondre de la
façon suivante aux différentes questions posées :

```
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

À l'issue de cette première configuration, MariaDB n'est accessible que depuis
la machine locale. Vérifier cela grâce à `lsof -i`.

Se connecter à MariaDB via la commande `mysql` ou `mariadb`. Remarquer que le
prompt a changé. Taper la commande `status` pour voir quelques informations sur
le serveur, dont la version et des statistiques. Quitter la session via la
commande `quit` ou `exit`.

## Etape 1 : démarrage et arrêt

Comme beaucoup de services, MariaDB doit être démarré pour fonctionner. De
façon assez ordinaire, il est possible d'utiliser systemd comme cela fut le cas
pour Apache :

* `systemctl status mariadb.service` indique l'état du service `mariadb` ;
* `systemctl stop mariadb.service` arrête le service ;
* `systemctl start mariadb.service` démarre le service ;
* `systemctl restart mariadb.service` redémarre le service.

Note : il est possible d'omettre `.service` dans les commandes.

Utiliser les commandes ci-dessus pour, successivement, arrêter, démarrer, puis
redémarrer MariaDB. Utiliser la commande de status ainsi que `lsof -i` pour
vérifier l'état entre chaque action.

## Etape 2 :

.

## Etape 3 :

.

