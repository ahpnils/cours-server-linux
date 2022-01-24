# TP 8 : découverte de mod_php

Objectifs :

- installer PHP en tant que module pour Apache ;
- configurer PHP depuis son fichier de configuration ;
- configurer PHP depuis un virtual host Apache ;
- installer des extensions à PHP ;
- installer une application PHP.

## Introduction

## Etape 0 : Apache et les modules

Apache a un fonctionnement en modules, qui étendent ses fonctionnalités.
Certains modules sont compilés de façon statique, c'est-à-dire qu'ils sont
intégrés au fichier binaire `/usr/sbin/apache2`. D'autres modules sont compilés
en tant que bibliothèque partagée.

Se connecter sur server11, puis passer root. Lancer la commande `apachectl -M`
et examiner le résultat.

Question : est-ce que les modules "proxy", "mpm_event" et "log_config" sont
chargés ? De quel type de module s'agit-il ?

Comme pour les virtual hosts, pour Debian, les modules Apache disposent d'une
organisation spécifique sur le système de fichiers. Dans la liste de la
question précédente, regarder si des fichiers correspondent à ces modules dans
`/etc/apache2/mods-available/` et dans `/etc/apache2/mods-enabled/`.

Question : pour quel module il n'y a pas de fichier correspondant dans
`/etc/apache2/mods-enabled/` ?

Lancer la commande `a2enmod <module>` en remplaçant `<module>` par le nom du
module en question, et suivre les instructions de relance d'Apache. Vérifier
ensuite via la commande `apachectl -M` dans `/etc/apache2/mods-enabled/` que le
module est bien paramétré et activé. Lancer la commande `a2dismod`
correspondante pour désactiver le module et vérifier que c'est bien le cas via
les mêmes commandes.

## Etape 1 : installation et configuration de mod_php

Lancer la commande `apachectl -M` et garder de côté le résultat.

Question : est-ce que le module "mpm_event" est toujours chargé ?

Installer le paquet `libapache2-mod-php` et observer attentivement les
informations affichées à l'écran.

Question : selon les informations affichées à l'écran, est-ce que le module
"mpm_event" est toujours chargé ? Est-ce que d'autres modules sont chargés ?

Vérifier que les informations affichées à l'écran ne sont pas mensongères à
l'aide de `apachectl -M`, puis comparer cet affichage à celui du début de
l'étape.

Vérifier maintenant que PHP est bien activé. Dans le répertoire
`/srv/www/server11.example.com/public/`, créer un fichier `info.php` dont le
contenu est le suivant :

```
<?php
    phpinfo();
?>
```

PHP est maintenant installé et fonctionnel. La page "PHPinfo" permet
d'obtenir des informations sur la configuration et les modules installés.

Questions : 

- à l'aide de la page "PHPinfo", quel est le chemin vers fichier de
  configuration (`php.ini`) utilisé par `mod_php` ?
- à l'aide de la page "PHPinfo", quel est le fuseau horaire (timezone) par
  défaut configuré dans `mod_php` ?

Editer de le fichier de configuration trouvé et trouver la directive
`date.timezone`. La décommenter, et, à l'aide de la documentation officielle
(http://php.net/date.timezone), configurer le fuseau horaire sur Paris.
Redémarrer Apache puis rafraîchir et vérifier dans la page "PHPinfo" que la
configuration a bien été prise en compte.

Il est aussi possible de modifier la configuration depuis Apache. Ajouter dans
la configuration du virtual host pour server11.example.com la directive
suivante : `php_admin_value date.timezone "Europe/Warsaw"`. Redémarrer Apache,
puis rafraîchir la page "PHPinfo".

Questions : 
- est-ce que la page "PHPinfo" mentionne la nouvelle configuration de fuseau
  horaire ?
- est-ce que la configuration "Europe/Paris" est toujours présente malgré tout
  ?

Renommer le fichier `info.php` en `index.php`.

Question : qu'est-ce que http://server11.example.com/ affiche ?

Ajouter dans la configuration du virtual host pour ce site la directive
`DirectoryIndex index.php`. Redémarrer Apache puis rafraîchir la page.

Question : qu'est-ce que http://server11.example.com/ affiche maintenant ?

Renommer `index.php` en `info.php` puis rafraîchir la page, et prendre note du
résultat. Ensuite, paramétrer la directive `DirectoryIndex` à la valeur
`index.php index.html`. Redémarrer Apache, rafraîchir la page, et prendre note
du résultat. Renommer de nouveau `info.php` en `index.php`.

Question : quel est le comportement de `DirectoryIndex` ?

## Etape 2 : ajout d'extensions

PHP, comme Apache, est conçu de façon modulaire. Il est possible de lui ajouter
des fonctionnalités par le biais d'extensions. Bon nombre d'entre elles sont
disponibles sous la forme de paquets logiciels pour Debian, et l'installation
s'en trouve facilitée. 

Nous allons installer l'extension
[GD](https://www.php.net/manual/fr/book.image.php), qui permet de faire du
traitement d'images. Nous ne l'utiliserons pas, il s'agit d'un prérequis à
l'étape suivante.

Installer le paquet `php7.4-gd` puis rafraîchir la page "PHPinfo". Vérifier si
GD est activé. Relancer Apache puis rafraîchir la page "PHPinfo". Vérifier de
nouveau si GD est activé.

## Etape 3 : installation d'une application PHP

[PluXml](https://pluxml.org/) est un moteur de blog et CMS en PHP, qui a la
particularité de ne pas nécessiter de base de données pour fonctionner.

Supprimer les fichiers `index.html` et `index.php`.

Installer PluXml dans le virtual host `server11.example.com` et écrire une page
personnalisée.
