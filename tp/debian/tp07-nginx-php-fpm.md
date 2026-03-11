[Retour au sommaire](../../README.md)

# TP 7 : Nginx et PHP-FPM

Objectifs :

- installer PHP ;
- faire fonctionner PHP sur notre serveur web Nginx, grâce à FPM ;
- configurer PHP depuis son fichier de configuration ;
- installer des extensions à PHP ;
- installer une application PHP.

## Introduction

PHP est un langage de script complet. Celui-ci est facile à appréhender grâce à
la simplicité d'installation de `mod_php` pour Apache, mais ce n'est pas la
seule possibilité. Il est possible d'installer et d'utiliser des scripts PHP de
la même façon que d'autres scripts (Bash ou Python par exemple), mais de le
faire communiquer avec un serveur web via un protocole standard comme CGI.

Ce moyen de communication a laissé sa place à des moyens plus rapides et plus
efficaces, comme FastCGI et FPM (pour Fast Process Manager) et dernièrement PM,
mais FPM est sans doute le plus populaire actuellement.

## Etape 0 : installation de PHP-FPM

Se connecter sur server11 et passer root. Installer le paquet `php-fpm`. Un
nouveau service `php8.4-fpm` existe, vérifier qu'il est bien lancé.

Dans le bloc server définissant `server11.example.com`, ajouter les directives
suivantes :

```
location ~ \.php$ {
  include snippets/fastcgi-php.conf;
  fastcgi_pass unix:/run/php/php8.4-fpm.sock;
}
```

Une fois cette modification effectuée, relancer le service `nginx`.

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

Se rendre ensuite à l'aide d'un navigateur sur 
http://server11.example.com/info.php et vérifier que la page "PHPinfo"
s'affiche.

Questions :
- à l'aide de la page "PHPinfo", quel est le chemin vers fichier de
  configuration (`php.ini`) utilisé par `php-fpm` ?
- à l'aide de la page "PHPinfo", quelle est l'API utilisée pour la
  communication entre Nginx et PHP ?
- à l'aide de la page "PHPinfo", quel est le fuseau horaire (timezone) par
  défaut configuré dans `php-fpm` ?
- dans la page "PHPinfo", rechercher le tableau ayant pour titre "Environment",
  sous quel utilisateur est lancé PHP ? Sous quel utilisateur est lancé Nginx ?

Regarder le contenu des fichiers `/etc/nginx/fastcgi.conf` et
`/etc/nginx/snippets/fastcgi-php.conf`. Ces fichiers permettent de faire "la
glue" entre Nginx et PHP.

## Etape 1 : configuration personnalisée de PHP-FPM

Toujours sur server11, créer un utilisateur `server11` dont les
caractéristiques sont les suivantes :

- son répertoire de départ est `/srv/www/server11.example.com/` ;
- son UID est 2011 ;
- il faut créer un groupe de même nom ;
- son shell est `/sbin/nologin`.

Ensuite, transférer le fichier [`fpm/server11.conf`](../../fpm/server11.conf) de ce dépôt dans le répertoire
`/etc/php/8.4/fpm/pool.d`. Avec la commande `usermod`, ajouter l'utilisateur `www-data` (utilisé par nginx)
au groupe `server11` afin d'autoriser l'accès en lecture et écriture au fichier
`server11.sock`. Relancer le service `php8.4-fpm`, puis vérifier que
celui-ci est bien relancé, via systemd.

Questions :
- via systemd, combien de processus php sont lancés ?
- dans quels pools sont-ils ?

Editer le fichier `/etc/php/8.4/fpm/pool.d/server11.conf` à la ligne 103, et
remplacer `pm = dynamic` par `pm = static`. Relancer le service `php8.4-fpm`,
et répondre de nouveau aux deux questions précédentes.

Editer le fichier `/etc/nginx/sites-enabled/server11.example.com.conf`, et
remplacer la ligne `fastcgi_pass unix:/run/php/php8.4-fpm.sock;` par `fastcgi_pass unix:/run/php/server11.sock;`. Relancer Nginx, et regarder la page "PHPinfo". 

Question : quelle erreur est affchée ?

Ajouter l'utilisateur `www-data` au groupe `server11` puis relancer Nginx ? La
page "PHPinfo" s'affiche-t'elle ?

Question : quels sont les changements entre les deux pools ?

Remettre la valeur initiale de la directive, relancer Nginx et rafraîchir la page. 

## Etape 2 : installation d'une application PHP

[PluXml](https://pluxml.org/) est un moteur de blog et CMS en PHP, qui a la
particularité de ne pas nécessiter de base de données pour fonctionner.

Comme pour l'étape 1, créer un utilisateur `www11` qui servira à un autre pool
FPM. Créer ensuite un pool FPM `www11` comme le pool `server11`, et s'assurer
que le pool démarre. Configurer le bloc `server` de `www11.example.com` pour
que ce site utilise le pool `www11`, en utilisant une page "phpinfo" pour le
vérifier.

Installer PluXml dans le bloc `server` de `www11.example.com` et écrire une page
personnalisée, supprimer au besoin les fichiers existant dans
`/srv/www/www11.example.com/public/`

Quelques points d'assistance :
- PluXml est distribué sous forme d'archive zip, il faudra donc installer le
  paquet unzip sur server11 ;
- il est possible de télécharger directement l'archive de PluXml depuis
  server11 grâce au logiciel wget ;
- PluXml a besoin des extensions PHP XML et GD ;
- il faut passer `cgi.fix_pathinfo` à 0 dans le fichier
  `/etc/php/8.4/fpm/php.ini` (ligne 803).

Dans le bloc server du fichier `www11.example.com.conf`, adaptez les
directives suivantes (source : [documentation officielle](https://wiki.pluxml.org/docs/install/nginx.html)):

```
index  index.php;

# client_header_buffer_size 1k;
# client_max_body_size 1m;
client_max_body_size 8m; # évite une erreur 413 si on upload un fichier

## BASE

# Règle principale
location / {
  try_files $uri $uri/ @handler;
}

# Réécriture vers l'index
location @handler {
  rewrite ^/(.*)$ /index.php?^$1 last;
}

# Parseur PHP
location ~ \.php$ {
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  # NOTE: Utilisez "cgi.fix_pathinfo = 0;" dans php.ini
  include fastcgi.conf;
  fastcgi_index index.php;
  fastcgi_pass unix:/run/php/php8.4-fpm.sock; # À remplacer par le bon pool FPM
}

## REDIRECTIONS

# Flux RSS
location /feed/ {
  rewrite ^/feed\/(.*)$ /feed.php?^$1 last;
}

# Sitemap
location = /sitemap.xml {
  rewrite .* /sitemap.php;
}

## PROTECTION REPERTOIRES

location ~ /(version|update|readme|data/configuration) {
  deny all;
}

## CACHING
# https://www.theodo.fr/blog/2016/06/improve-the-performance-of-your-webapp-configure-nginx-to-cache/
# http://www.supinfo.com/articles/single/2843-implementer-cache-navigateur-avec-nginx

# cache-control
location /data/ {
  add_header Cache-Control public;
  expires 12h;
}
location /core/ {
  add_header Cache-Control public;
  expires 12h;
}
location /plugins/ {
  add_header Cache-Control public;
  expires 12h;
}
location /themes/ {
  add_header Cache-Control public;
  expires 12h;
}
```
