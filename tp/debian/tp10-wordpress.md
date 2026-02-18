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

## Etape 0 : utilisateur dédié

Sur le serveur server11, créer un nouvel utilisateur "wp11" qui servira pour
PHP-FPM. Ses caractéristiques sont les suivantes :

- son répertoire de départ est `/srv/www/wp11.example.com/`;
- son UID est 3011 ;
- il faut créer un groupe du même nom ;
- son shell est `/usr/sbin/nologin'.

Créer l'arborescence suivante (les répertoires appartiennent à l'utilisateur
`blog`) :

```
/srv/
└── www
    └── wp11.example.com
        ├── log
        ├── public
        ├── session
        └── tmp
```

## Etape 1 : virtual host

Sur le serveur server11, réutiliser le bloc server Nginx pour wp11.example.com,
dont les paramètres seront :

- nom du serveur : `wp11.example.com` ;
- alias de serveur : `wp11` ;
- emplacement des fichiers : `/srv/www/wp11.example.com/public` ;
- emplacement du log d'erreur : `/srv/www/wp11.example.com/log/error.log` ;
- emplacement du log d'accès : `/srv/www/wp11.example.com/log/access.log` ;
- format du log d'accès : comme server11.example.com.

Sur la machine hôte, vérifier que l'entrée "wp11.example.com" est bien présente 
dans `/etc/hosts`. Vérifier que le site est bien accessible et que les logs sont écrits.

## Etape 2 : PHP-FPM

Sur le serveur server11, créer un nouveau pool PHP-FPM, en se basant sur l'un
de ceux déjà présents dans `/etc/php/8.4/fpm/pool.d/`. Ses caractéristiques
seront les suivantes :

- nom du pool : wp11 ;
- user : `$pool` (astuce pour éviter de copier-coller "wp11" partout)
- group : `$pool`
- listen : `/run/php/$pool.sock`
- listen.owner : `$pool`
- listen.group : `www-data`
- listen.mode : `0660`
- pm : `ondemand`
- access.log : `/srv/www/wp11.example.com/log/php_access.log`

À la fin du fichier de pool PHP-FPM, ajouter les paramètres suivants :

```
env[TMP] = /srv/www/wp11.example.com/tmp
env[TMPDIR] = /srv/www/wp11.example.com/tmp
env[TEMP] = /srv/www/wp11.example.com/tmp
php_admin_value[error_log] = /srv/www/wp11.example.com/log/php_error.log
php_admin_value[open_basedir] = /srv/www/wp11.example.com/public/:/tmp:/srv/www/wp11.example.com/tmp
php_admin_value[session.save_path] = /srv/www/wp11.example.com/session
php_admin_flag[log_errors] = on
php_admin_value[upload_tmp_dir] = /srv/www/wp11.example.com/tmp
```

Rappel : pour Nginx comme pour PHP-FPM, la création ou la modification de
fichiers de configuration n'est pas prise en compte tant que le service n'a pas
redémarré.

Retour à Nginx : ajouter une configuration pour PHP dans le bloc server du site
"wp11.example.com" :

```
index index.php index.html;
location ~ \.php$ {
  include snippets/fastcgi-php.conf;
  fastcgi_pass unix:/run/php/wp11.sock;
}
```

Vérifier en créant un fichier `/srv/www/wp11.example.com/public/info.php`
contenant un "PHPinfo" que PHP est bien relié à Nginx, et le visualiser dans un
navigateur.

## Etape 3 : création d'un utilisateur et d'une base de données

Ouvrir une session root dans PHPMyAdmin, et se rendre dans l'onglet "user
accounts". Créer un nouvel utilisateur nommé `wp11`, pouvant se connecter
depuis n'importe où, et ayant pour mot de passe `mywp11`. Avant la création, ne
pas oublier de cocher la case "Create database with same name and grant all
privileges".

Vérifier en se déconnectant puis en se connectant à PHPMyAdmin que
l'utilisateur et la base sont fonctionnels.

## Etape 4 : téléchargement de WordPress

Télécharger la dernière version de WordPress depuis cette adresse :
https://wordpress.org/latest.tar.gz .

Copier l'archive sur server11, dans `/srv/www/wp11.example.com/`. Décompresser
l'archive dans `/srv/www/wp11.example.com/public/`. Modifier récursivement le
propriétaire des répertoires et fichiers, ils doivent appartenir à
l'utilisateur `wp11` et au groupe `wp11`.

## Etape 5 : installation de WordPress

Se rendre sur http://wp11.example.com, et effectuer l'installation. WordPress
demandera à se connecter à la base de données située sur server12.

Toutefois, l'installation n'est pas optimale. Dans le menu "Tools", regarder la
rubrique "Site Health". Il est indiqué que plusieurs modules pour PHP sont
manquants. Trouver les paquets Debian correspondant et les installer.

Profiter du blog pour y écrire un contenu d'exemple, et surtout, profiter des
compétences apprises jusqu'alors. Bravo !
