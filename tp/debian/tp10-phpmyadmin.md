# TP 10 : PHPMyAdmin

Objectifs :

- installer et découvrir PHPMyAdmin ;
- créer un utilisateur et une base via l'interface web de PHPMyAdmin ;
- effectuer des modifications de privilèges ;
- effectuer un export de base de données.

## Introduction

En plus de l'interface CLI, il est possible d'utiliser des logiciels de
gestion de base de données graphiques pour se connecter à MariaDB ou MySQL. Il
en existe trois particulièrement utilisés :

- PHPMyAdmin est une application web PHP assez complète permettant d'effectuer
  de nombreuses opérations ;
- adminer est aussi une application web PHP, mais qui se différencie par sa
  légèreté et sa simplicité d'installation (un seul fichier à déposer dans un
  emplacement web) ;
- MySQL Workbench est le client officiel d'Oracle pour MySQL, disponible pour
  Windows, macOS et différentes distributions Linux.

## Etape 0 : prérequis

Notre serveur MariaDB n'autorise une connexion de l'administrateur (root) que
depuis *localhost*. Or, l'étape suivante installera PHPMyAdmin sur server11,
qui n'est donc pas la machine locale. Il faut donc créer un utilisateur
administrateur pouvant se connecter depuis server11.

Se connecter sur server12 en tant que root, puis lancer un shell MariaDB.
Lancer les commandes suivantes :

```
CREATE USER 'root'@'192.168.122.11' IDENTIFIED BY 'remotepass';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.122.11' IDENTIFIED BY 'remotepass' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

## Etape 1 : installation

Sur server11, installer le paquet `phpmyadmin`. Ensuite, activer la
configuration via la commande `a2enconf phpmyadmin`. Vérifier la présence d'un
lien symbolique `phpmyadmin.conf` dans `/etc/apache2/conf-enabled`, et
recharger Apache via `systemctl reload apache2`. 

Accéder depuis la machine hôte aux URLs suivantes :
- http://server11.example.com/phpmyadmin/ ;
- http://www11.example.com/phpmyadmin/ ;
- http://192.168.122.11/phpmyadmin/.

Consulter le fichier `/etc/apache2/conf-enabled/phpmyadmin.conf`.

Questions :
- où est installé PHPMyAdmin ?
- pourquoi les URLs ci-dessus fonctionnent ?

Il faut encore paramétrer PHPMyAdmin. Sous Debian, il est possible de faire ce
paramétrage dans le fichier `/etc/phpmyadmin/config-db.php`. Editer ce fichier
et paramétrer la variable `$dbserver` à l'adresse IP de server12.

## Etape 2 : connexion à PHPMyAdmin, modification de privilèges

Depuis la machine hôte, se rendre sur http://server11.example.com/phpmyadmin/,
puis utiliser le login `newdb` et le mot de passe `password2`. Examiner sur la
gauche ce qui est accessible comme données, ainsi que les onglets du haut.

Se déconnecter de PHPMyAdmin puis s'y connecter à l'aide du login `root` et du
mot de passe `remotepass`.

Question : est-ce que root a accès à plus de choses ? Quoi ?

Se rendre dans l'onglet "User accounts", retrouver l'utilisateur `newdb`
utilisé plus haut, éditer ses privilèges pour la base `newdb` et lui attribuer
tous les privilèges "Data" et "Structure". Se déconnecter de la session root et
ouvrir une nouvelle session en tant qu'utilisateur `newdb`.

Questions :

- est-ce que `newdb` peut accéder à la base éponyme ?
- est-ce qu'un rechargement des privilèges a été nécessaire ?

## Etape 3 : export de base de données

Depuis la session `newdb`, se rendre dans l'onglet "export" et effectuer un
export de la base de données `newdb`, au format SQL. Un fichier sera donné à
télécharger.

Question : que contient ce fichier ? (l'ouvrir avec un éditeur de texte)

## Etape 4 : création d'un utilisateur et d'une base de données

Ouvrir une session root dans PHPMyAdmin, et se rendre dans l'onglet "user
accounts". Créer un nouvel utilisateur nommé `blog`, pouvant se connecter
depuis n'importe où, et ayant pour mot de passe `myblog`. Avant la création, ne
pas oublier de cocher la case "Create database with same name and grant all
privileges".

Vérifier en se déconnectant puis en se connectant à PHPMyAdmin que
l'utilisateur et la base sont fonctionnels.
