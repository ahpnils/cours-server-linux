# TP 17 : HTTPS avec Nginx

Objectifs :

- configurer un virtual host HTTPS ;
- configurer la redirection HTTP vers HTTPS.

## Introduction

Comme Apache, Nginx gère HTTPS. Mais la syntaxe des directives diffère.

## Etape 0 : prérequis

Bien souvent, en environnement de test, générer un certificat peut s'avérer
fastidieux. C'est pourquoi il existe un paquet Debian nommé `ssl-cert` qui
comporte, entre autres un certificat auto-signé.

Installer, si ce n'est pas déjà la cas, le paquet `ssl-cert` sur `server13`

## Etape 1 : configuration virtual host HTTPS

Comme pour Apache, ce dépôt contient un fichier de configuration déjà prêt.
Transférer le fichier `ssl_server13.example.com.conf` du répertoire `nginx/` de
ce dépôt sur `server13` dans le répertoire `/etc/nginx/sites-available/`.
Activer le site via un lien symbolique dans `/etc/nginx/sites-enabled/`, puis
redémarrer Nginx.

Visiter http://server13.example.com puis https://server13.example.com, en
autorisant le navigateur à s'y connecter.

Question : quelles sont les directives supplémentaires du fichier de
configuration ajouté ? À quoi correspondent-elles ?

Vérifier que les requêtes apparaissent bien dans les logs.

## Etape 2 : redirection HTTP vers HTTPS

Si un même site est disponible en HTTP et HTTPS, la redirection n'est pas
automatique. Pour éviter que des utilisateurs restent sur le site HTTP, il est
possible d'ajouter une règle de redirection. 

Ajouter dans le virtual host de `server13.example.com` la ligne suivante :
```
return 301 https://$host$request_uri;
```

Redémarrer Nginx, et se rendre sur http://server13.example.com. La
redirection devrait fonctionner.

