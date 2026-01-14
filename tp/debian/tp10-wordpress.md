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

Sur le serveur server11, créer un nouvel utilisateur "blog" qui servira pour
PHP-FPM. Ses caractéristiques sont les suivantes :

- son répertoire de départ est `/srv/www/blog.example.com/`;
- son UID est 3011 ;
- il faut créer un groupe du même nom ;
- son shell est `/usr/sbin/nologin'.

Créer l'arborescence suivante (les répertoires appartiennent à l'utilisateur
`blog`) :

```
/srv/
└── www
    └── blog.example.com
        ├── log
        ├── public
        ├── session
        └── tmp
```

## Etape 1 : virtual host

Sur le serveur server11, créer un nouveau bloc server Nginx, dont les
paramètres seront :

- nom du serveur : `blog.example.com` ;
- alias de serveur : `blog` ;
- emplacement des fichiers : `/srv/www/blog.example.com/public` ;
- emplacement du log d'erreur : `/srv/www/blog.example.com/log/error.log` ;
- emplacement du log d'accès : `/srv/www/blog.example.com/log/access.log` ;
- format du log d'accès : comme server11.example.com.

Créer les répertoires nécessaires sur server11. Créer une page `index.html` 
au contenu minimal et la placer dans l'emplacement des fichiers. Il faudra
certainement réajuster les droits sur l'arborescence créée lors de l'étape
précédente.

Sur la machine hôte, ajouter l'entrée "blog.example.com" dans `/etc/hosts`.
Vérifier que le site est bien accessible et que les logs sont écrits.

## Etape 2 : PHP-FPM

Sur le serveur server11, créer un nouveau pool PHP-FPM, en se basant sur l'un
de ceux déjà présents dans `/etc/php/8.4/fpm/pool.d/`. Ses caractéristiques
seront les suivantes :

- nom du pool : blog ;
- user : `$pool` (astuce pour éviter de copier-coller "blog" partout)
- group : `$pool`
- listen : `/run/php/$pool.sock`
- listen.owner : `$pool`
- listen.group : `www-data`
- listen.mode : `0660`
- pm : `ondemand`
- access.log : `/srv/www/blog.example.com/log/php_access>log`

À la fin du fichier de pool PHP-FPM, ajouter les paramètres suivants :

```
env[TMP] = /srv/www/blog.example.com/tmp
env[TMPDIR] = /srv/www/blog.example.com/tmp
env[TEMP] = /srv/www/blog.example.com/tmp
php_admin_value[error_log] = /srv/www/blog.example.com/log/php_error.log
php_admin_value[open_basedir] = /srv/www/blog.example.com/public/:/tmp:/srv/www/blog.example.com/tmp
php_admin_value[session.save_path] = /srv/www/blog.example.com/session
php_admin_flag[log_errors] = on
php_admin_value[upload_tmp_dir] = /srv/www/blog.example.com/tmp
```

Rappel : pour Nginx comme pour PHP-FPM, la création ou la modification de
fichiers de configuration n'est pas prise en compte tant que le service n'a pas
redémarré.

Retour à Nginx : ajouter une configuration pour PHP dans le bloc server du site
"blog.example.com" :

```
index index.php index.html;
location ~ \.php$ {
  include snippets/fastcgi-php.conf;
  fastcgi_pass unix:/run/php/blog.sock;
}
```

Vérifier en créant un fichier `/srv/www/blog.example.com/public/info.php`
contenant un "PHPinfo" que PHP est bien relié à Nginx, et le visualiser dans un
navigateur.

## Etape 3 : création d'un utilisateur et d'une base de données

Ouvrir une session root dans PHPMyAdmin, et se rendre dans l'onglet "user
accounts". Créer un nouvel utilisateur nommé `blog`, pouvant se connecter
depuis n'importe où, et ayant pour mot de passe `myblog`. Avant la création, ne
pas oublier de cocher la case "Create database with same name and grant all
privileges".

Vérifier en se déconnectant puis en se connectant à PHPMyAdmin que
l'utilisateur et la base sont fonctionnels.

## Etape 4 : téléchargement de WordPress

Télécharger la dernière version de WordPress depuis cette adresse :
https://wordpress.org/latest.tar.gz .

Copier l'archive sur server11, dans `/srv/www/blog.example.com/`. Décompresser
l'archive dans `/srv/www/blog.example.com/public/`. Modifier récursivement le
propriétaire des répertoires et fichiers, ils doivent appartenir à
l'utilisateur `blog` et au groupe `blog`.

## Etape 5 : installation de WordPress

Se rendre sur http://blog.example.com, et effectuer l'installation. WordPress
demandera à se connecter à la base de données située sur server12.

Toutefois, l'installation n'est pas optimale. Dans le menu "Tools", regarder la
rubrique "Site Health". Il est indiqué que plusieurs modules pour PHP sont
manquants. Trouver les paquets Debian correspondant et les installer.

Profiter du blog pour y écrire un contenu d'exemple, et surtout, profiter des
compétences apprises jusqu'alors. Bravo !
