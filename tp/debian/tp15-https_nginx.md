[Retour au sommaire](../../README.md)

# TP 17 : HTTPS avec Nginx

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
  arrivée, chaque virtual host devait avoir une adresse IP
  propre pour pouvoir faire du HTTPS.

## Etape 0 : prérequis

Certains serveurs web "historiques" embarquent HTTPS comme module, désactivé
par défaut. Nginx embarque HTTPS nativement, et certaines versions récentes
permettent même d'obtenir un certificat très simplement (d'autres serveurs web
modernes le permettent aussi). Mais nous allons faire les choses "à l'ancienne"
dans ce TP.

Bien souvent, en environnement de test, générer un certificat peut s'avérer
fastidieux. C'est pourquoi il existe un paquet Debian nommé `ssl-cert` qui
comporte, entre autres un certificat auto-signé. Ce TP a néanmoins pour
objectif de générer soi-même un tel certificat, nous n'utiliserons donc pas ce
raccourci.

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
mkdir -vp /etc/nginx/selfcerts
openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout /etc/nginx/selfcerts/www13.example.com.key -out /etc/nginx/selfcerts/www13.example.com.crt -subj '/C=FR/ST=IdF/L=Paris/O=Example Corp/OU=Dev/CN=example/CN=www13.example.com'
```

Décortiquons la dernière commande : elle a créé et signé un certificat d'une
durée de vie de 365 jours, la clé privée se trouve dans
`/etc/nginx/selfcerts/www13.example.com.key` et le certificat se trouve dans
`/etc/nginx/selfcerts/www13.example.com.crt`. Ce certificat est valable pour
le nom `www13.example.com`.

## Etape 1 : configuration virtual host HTTPS

La simple création d'un certificat ne suffit pas à avoir un site HTTPS. Il nous
faut aussi créer un virtual host. Transférer le fichier [`ssl_www13.example.com.conf`](../../nginx/ssl_www13.example.com.conf) sur `server13` dans l'emplacement `/etc/nginx/sites-enabled`. Redémarrer Nginx.

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

Note : si la partie reverse-proxy est toujours active, il sera peut-être
nécessaire d'ajouter la directive Nginx `proxy_set_header X-Forwarded-Proto
$scheme;` pour que cela fonctionne mieux.
