# TP 9 : MariaDB

Objectifs :

- installer MariaDB ;
- effectuer une configuration initiale "sécurisée";
- créer des utilisateurs et recharger les privilèges;
- créer une base et y insérer des données ;
- paramétrer une connexion distante.

## Introduction

MariaDB est un système de gestion de base de données édité sous licence GPL.
Il s'agit d'un fork de MySQL, ce dernier appartenant à la société Oracle.

Du fait de son origine, de nombreuses commandes dans MariaDB portent le nom
`mysql`.

## Etape 0 : installation

Sur server12, installer le paquet `mariadb-server`. Pour la configuration
initiale, lancer la commande `mysql_secure_installation `, puis répondre de la
façon suivante aux différentes questions posées :

```
Switch to unix_socket authentication [Y/n] y
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

## Etape 2 : une première base de données (et du contenu)

Toujours sur server12, se reconnecter à MariaDB localement :
- se connecter sur server12 ;
- passer root ;
- lancer la commande `mariadb`.

Vérifier d'abord si des bases de données ne sont pas déjà présentes dans
l'installation, avec la commande `SHOW DATABASES;`.

Question : quelles sont les bases de données présentes par défaut ?

Pour créer sa première base de données, on peut utiliser la commande suivante :
```
CREATE DATABASE newdb;
```

Vérifier ensuite la bonne création de celle-ci en listant les bases présentes.

Question : comment s'appelle la nouvelle base de données ?

Avant de créer une table, il faut sélectionner la base dans laquelle elle sera
créée. Pour cela, taper la commande `USE newdb;`, qui sélectionnera la base
"newdb".

Ensuite, créer une table `students`, avec la commande suivante :
```
CREATE TABLE students (id INT, name VARCHAR(20), email VARCHAR(20));
```
Cette table contient donc trois colonnes, une première "id", contenant un
nombre entier, une deuxième "name" et une troisième "email", ces dernières 
pouvant chacune contenir jusqu'à 20 caractères.

Vérifier la bonne création de la table avec la commande `SHOW TABLES;`

Insérer une première donnée dans notre table avec la commande suivante :
```
INSERT INTO students (id,name,email) VALUES(01,"student1","student1@example.com");
```

En se basant sur ce premier exemple, insérer des données pour deux autres
étudiants. Une fois que ceci est fait, vérifier que les données existent avec
la commande `SELECT * FROM students;`

## Etape 3 : utilisateurs et privilèges

Jusqu'à maintenant la connexion vers MariaDB s'est faite uniquement avec
l'utilisateur root (de MariaDB). Mais il est possible de créer plusieurs autres
utilisateurs et de leur donner des droits précis.

Créer un utilisateur ayant le même nom que la base de données, mais qui pourra
se connecter uniquement depuis la machine locale, et qui utilisera le mot de
passe "password" pour se connecter :

```
CREATE USER 'newdb'@'localhost' IDENTIFIED BY 'password';
```

Une fois l'utilisateur créé, quitter la session root de MariaDB et se connecter
en tant qu'utilisateur "newdb" avec la commande `mariadb -u newdb -p`. Une fois
connecté, tenter d'accéder à la base "newdb" avec la commande : `USE newdb;`.

Question : quel est le résultat de cette dernière commande ? Pourquoi ?

Il faut donc ajouter des privilèges à l'utilisateur. Se déconnecter de la
session MariaDB, et se reconnecter en tant que root, puis lancer la commande
suivante :

```
FLUSH PRIVILEGES;
```

Cette commande permet d'appliquer tous les changements apportés aux privilièges
utilisateurs. Se déconnecter de la session MariaDB, puis tenter de nouveau de
se connecter en tant qu'utilisateur "newdb".

## Etape 4 : connexion distante

Pour le moment la base de donnée n'est accessible qu'en local. Pour modifier
cela, il faut agir à deux emplacements :

- le paramétrage réseau de MariaDB ;
- les privilièges utilisateurs.

Pour le premier emplacement, en tant que root :

- stopper le serveur MariaDB via `systemctl stop mariadb` ;
- éditer le fichier `/etc/mysql/mariadb.conf.d/50-server.cnf` ;
- commenter la ligne contenant `bind-address            = 127.0.0.1` ;
- ajouter en-dessous :
  ```
  skip-networking=0
  skip-bind-address
  ```
- sauvegarder le fichier, puis démarrer MariaDB via `systemctl start mariadb`.

Vérifier avec `lsof -i` (toujours en tant que root) que MariaDB écoute bien sur
toutes les interfaces réseau.

Tenter de se connecter à MariaDB via la commande suivante :

```
mariadb -h 192.168.122.12 -u newdb -p
```

Question : quel est le résultat ?

Se reconnecter à MariaDB en tant que root en local, et créer un nouvel
utilisateur "newdb", comme dans l'étape précédente, mais en remplaçant
`'localhost'` par `'%'` et en paramétrant le mot de passe à "password2".
**Ne pas oublier de valider les privilèges**.

Tenter de se connecter de nouveau à MariaDB via la commande qui a échoué plus
haut.

Question : combien d'utilisateurs "newdb" sont présent ? Quels sont leurs 
paramètres et leurs privilèges ?
