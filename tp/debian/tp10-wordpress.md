[Retour au sommaire](../../README.md)

# TP 10 : WordPress

Objectifs :

- créer un nouveau virtual host ;
- installer dans ce nouveau virtual host un blog utilisant le CMS WordPress.

## Introduction

Il existe de nombreux systèmes de gestion de contenu, et surtout écrits en
langage PHP et utilisant une base de données MySQL. Certains sont spécifiques à
un usage (Magento ou Prestashop pour créer une boutique en ligne, Dotclear ou
Pluxml pour créer un blog), d'autres sont plus génériques (Drupal, Joomla).

WordPress est le CMS le plus utilisé à ce jour. Initialement prévu pour un
blog, celui-ci est néanmoins utilisable pour des portfolios, sites
institutionnels, et même des boutiques en ligne, grâce à l'ajout de
fonctionnalités permise par ses plugins.

Ce TP fait appel aux notions vues précédemment, les commandes ne seront donc
pas détaillées. 

## Etape 0 : virtual host

Sur le serveur server11, créer un nouveau virtual host Apache, dont les
paramètres seront :

- nom du serveur : `blog.example.com` ;
- alias de serveur : `blog` ;
- emplacement des fichiers : `/srv/www/blog.example.com/public` ;
- emplacement du log d'erreur : `/srv/www/blog.example.com/log/error.log` ;
- emplacement du log d'accès : `/srv/www/blog.example.com/log/access.log` ;
- format du log d'accès : `vhost_combined`.

Créer les répertoires nécessaires sur server11. Ne pas oublier d'activer le
virtual host. Créer une page `index.html` au contenu minimal et la placer dans
l'emplacement des fichiers.

Ajouter sur la machine hôte l'entrée "blog.example.com" dans `/etc/hosts`.
Vérifier que le site est bien accessible et que les logs sont écrits.

## Etape 1 : téléchargement de WordPress

Télécharger la dernière version de WordPress depuis cette adresse :
https://wordpress.org/latest.tar.gz .

Copier l'archive sur server11, dans `/srv/www/blog.example.com/`. Décompresser
l'archive dans `/srv/www/blog.example.com/public/`. Modifier récursivement le
propriétaire des répertoires et fichiers, ils doivent appartenir à
l'utilisateur `www-data` et au groupe `www-data`.

## Etape 3 : installation de WordPress

Se rendre sur http://blog.example.com, et effectuer l'installation. WordPress
demandera à se connecter à la base de données située sur server12.

Toutefois, l'installation n'est pas optimale. Dans le menu "Tools", regarder la
rubrique "Site Health". Il est indiqué que plusieurs modules pour PHP sont
manquants. Trouver les paquets Debian correspondant et les installer.

Profiter du blog pour y écrire un contenu d'exemple, et surtout, profiter des
compétences apprises jusqu'alors. Bravo !

