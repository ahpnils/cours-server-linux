[Retour au sommaire](../../README.md)

# TP 16 : HTTPS avec Apache

Objectifs :

- générer un certificat x509 autosigné ;
- configurer un virtual host HTTPS ;
- configurer la redirection HTTP vers HTTPS.

## Introduction

L'HyperText Transfer Protocol Secure (HTTPS, littéralement « protocole de
transfert hypertextuel sécurisé ») est la combinaison du HTTP avec une couche
de chiffrement comme SSL ou TLS. (page [Wikipédia](https://fr.wikipedia.org/wiki/HyperText_Transfer_Protocol_Secure)).

Quelques points-clés sur HTTPS :
- il est accessible par défaut sur le port 443/tcp ;
- il faut impérativement une clé privée et un certificat ;
- grâce à SNI (Server-Name Indicator), il est possible d'héberger plusieurs
  sites web (chacun ayant son certificat) sur une même adresse IP ; avant son
  arrivée, chaque virtual host dans Apache devait avoir une adresse IP
  propre pour pouvoir faire du HTTPS.

## Etape 0 : prérequis

HTTPS est disponible dans Apache grâce au module `ssl_module`. Ce module est fort
heureusement disponible dans Debian avec la paquet `apache2`. Dans notre cas,
il est donc déjà installé sur `server11`.

Question : le module `ssl_module` est-il activé sur `server11` ?

Se connecter sur `server11`, activer le module avec la commande `a2enmod
ssl`, puis redémarrer Apache. Vérifier que le module est bien activé.

## Etape 1 : générer un certificat autosigné

Comme pour SSH, HTTPS fonctionne sur un principe de clés privées et clés
publiques (les certificats). Ceux-ci utilisent un format nommé x509, mais par
abus de langage, on parle de certificat SSL. À noter que le protocole de
chiffrement SSL est aujourd'hui considéré obsolète et non sécurisé, et a fait
place à TLS.

L'obtention d'un certificat nécessite quatres étapes :
- la création d'une demande de signature de certificat (CSR, Certificate
  Signing Request), à l'occasion de laquelle sont générés la clé privée et la
  clé publique (le certificat) ;
- la communication de ce CSR à une autorité de certification (CA, Certificate
  Authority) ;
- la signature par cette CA de la CSR, résultant en un certificat signé ;
- la communication de ce certificat signé.

Il existe plusieurs CA, certaines publiques, certaines internes à des
organisations. Dans notre cas, nous n'utiliserons aucune d'entre elles, pas
même Let's Encrypt, pour une raison simple : nos serveurs ne sont pas
joignables depuis Internet. Nous allons donc créer une CSR et la signer
nous-même. Cela entraînera un message d'erreur, mais dans ce cas (et dans ce
cas uniquement), celui-ci n'est pas dangereux.

Lancer (en tant que root sur `server11`) les commandes suivantes :

```
apt -y install openssl
mkdir -vp /etc/apache2/selfcerts
openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout /etc/apache2/selfcerts/www11.example.com.key -out /etc/apache2/selfcerts/www11.example.com.crt -subj '/C=FR/ST=IdF/L=Paris/O=Example Corp/OU=Dev/CN=example/CN=www11.example.com'
```

Décortiquons la dernière commande : elle a créé et signé un certificat d'une
durée de vie de 365 jours, la clé privée se trouve dans
`/etc/apache2/selfcerts/www11.example.com.key` et le certificat se trouve dans
`/etc/apache2/selfcerts/www11.example.com.crt`. Ce certificat est valable pour
le nom `www11.example.com`.

## Etape 1 : configuration initiale HTTPS

La simple création d'un certificat ne suffit pas à avoir un site HTTPS. Il nous
faut aussi créer un virtual host. Dans le répertoire `apache` de ce dépôt se
trouve le fichier `ssl_www11.example.com.conf`. Le transférer sur `server11`
dans l'emplacement `/etc/apache2/sites-available`. Puis, activer la virtual
host via la commande `a2ensite ssl_www11.example.com` (en tant que root ou via
`sudo`).

Visiter http://www11.example.com puis https://www11.example.com, en autorisant
le navigateur à s'y connecter. Puis regarder les deux fichiers de configuration
des virtual hosts.

Question : quelles sont les directives supplémentaires dans
`ssl_www11.example.com.conf` ? Quel est leur usage ?

Vérifier que les requêtes apparaissent bien dans les logs.

## Etape 2 : redirection HTTP vers HTTPS

Si un même site est disponible en HTTP et HTTPS, la redirection n'est pas
automatique. Pour éviter que des utilisateurs restent sur le site HTTP, il est
possible d'ajouter une règle de redirection. Plusieurs manières de faire cette
redirection existent. L'une d'entre elles utilise un module nommé
`mod_rewrite`.

Comme pour le module `ssl`, activer le module `rewrite` : `a2enmod rewrite`.
Redémarrer Apache. Ajouter ensuite dans la configuration du virtual host
`www11.example.com` les directives suivantes :

```
<IfModule mod_rewrite.c>
   RewriteEngine On
   RewriteCond %{HTTPS} !=on
   RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
</IfModule>
```

Redémarrer de nouveau Apache, et se rendre sur http://www11.example.com. La
redirection devrait fonctionner.

