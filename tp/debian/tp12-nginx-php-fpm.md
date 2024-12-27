[Retour au sommaire](../../README.md)

# TP 12 : Nginx, et PHP-FPM

Objectif :

- faire fonctionner PHP sur notre serveur web Nginx, grâce à FPM ;
- configurer un virtual host de façon à faire fonctionner un ensemble de
  processus PHP avec un utilisateur donné.

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

Se connecter sur server13 et passer root. Installer le paquet `php-fpm`. Un
nouveau service `php8.2-fpm` existe, vérifier qu'il est bien lancé.

Dans le virtual host `server13.example.com`, ajouter dans le bloc de
configuration `location /` les directives suivantes :

```
location ~ \.php$ {
              include snippets/fastcgi-php.conf;
              fastcgi_pass unix:/run/php/php8.2-fpm.sock;
       }
```

Une fois cette modification effectuée, relancer le service `nginx`. De la même
façon que dans le TP 7, étape 1, créer un fichier `info.php` dans
`/srv/www/server13.example.com/public` utilisant la fonction `phpinfo()`. Se
rendre ensuite à l'aide d'un navigateur sur
http://server13.example.com/info.php et vérifier que la page "PHPinfo"
s'affiche.

Questions :
- dans la page "PHPinfo", rechercher le tableau ayant pour titre "Environment",
  sous quel utilisateur est lancé PHP ? Sous quel utilisateur est lancé Nginx ?
- regarder la page "PHPinfo" de server11 et retrouver le même tableau ; a-t'il
  les mêmes variables ?
- toujours en comparant les deux pages "PHPinfo", quel est le fichier de
  configuration principal activé (`Loaded Configuration File `) ?

## Etape 1 : configuration personnalisée de PHP-FPM

Toujours sur server13, créer un utilisateur `server13` dont les
caractéristiques sont les suivantes :

- son répertoire de départ est `/srv/www/server13.example.com/` ;
- son UID est 2013 ;
- il faut créer un groupe de même nom ;
- son shell est `/sbin/nologin`.

Ensuite, transférer le fichier `fpm/server13.conf` dans le répertoire
`/etc/php/8.2/fpm/pool.d`. Avec la commande `usermod`, ajouter l'utilisateur `www-data` (utilisé par nginx)
au groupe `server13` afin d'autoriser l'accès en lecture et écriture au fichier
`server13.sock`. Relancer le service `php8.2-fpm`, puis vérifier que
celui-ci est bien relancé, via systemd.

Questions :
- via systemd, combien de processus php sont lancés ?
- dans quels pools sont-ils ?

Editer le fichier `/etc/php/8.2/fpm/pool.d/server13.conf` à la ligne 103, et
remplacer `pm = dynamic` par `pm = static`. Relancer le service `php8.2-fpm`,
et répondre de nouveau aux deux questions précédentes.

Editer le fichier `/etc/nginx/sites-enabled/server13.example.com.conf`, et
remplacer la ligne `fastcgi_pass unix:/run/php/php8.2-fpm.sock;` par `fastcgi_pass unix:/run/php/server13.sock;`. Relancer Nginx, et regarder la page "PHPinfo". Remettre la valeur initiale de la directive, relancer Nginx et rafraîchir la page. 

Question : quels sont les changements entre les deux pools ?

## Etape 2 : installation d'une application PHP

À partir des modèles vus plus haut, créer un pool "www13" et permettre au
virtual host "www13.example.com" d'exécuter ce pool PHP. S'il reste du temps,
installer PluxML dans ce pool.
