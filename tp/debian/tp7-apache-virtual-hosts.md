[Retour au sommaire](../../README.md)

# TP 7 : les virtual hosts dans Apache

Objectifs :

- héberger plusieurs sites web sur un même server avec une même adresse IP ;
- diviser cela en plusieurs fichiers de configuration ;
- séparer les logs par sites web.

## Introduction

Il est possible d'héberger plusieurs sites sur le même serveur ! Le serveur
peut avoir plusieurs adresses IP, mais il peut aussi avoir plusieurs noms
associés à la même adresse IP. Dans ce cas, on peut parler de "name-based
virtual hosting".

## Etape 0 : ajout de noms

Sur sa machine locale, ouvrir un terminal, et passer root. Editer le fichier de
configuration `/etc/hosts` et ajouter la ligne suivante :

```
192.168.122.11 server11 server11.example.com www11.example.com
```

Une fois le fichier sauvé, s'assurer que la machine server11 est bien
accessible via les noms ajoutés ci-dessus, par exemple avec la commande `ping`.
Maintenant, vérifier qu'Apache répond lors de requête vers les noms ajoutés.

Question : qu'affiche Apache lors des requêtes ?

## Etape 1 : création d'un virtual host

Se connecter sur server11 et passer root. Créer l'arborescence suivante :

```
/srv/
└── www
    └── server11.example.com
        ├── log
        └── public
```

Ajouter un fichier `index.html` au contenu libre dans le répertoire `public`.

Transférer le fichier `apache/server11.example.com.conf` de ce dépôt dans le
répertoire `/etc/apache2/sites-available/` de server11.

À l'aide d'un navigateur, visiter les URLs suivantes :
- http://server11
- http://server11.example.com
- http://www11.example.com

Question : quel est le chemin du fichier HTML servi ? De quel fichier de
configuration vient-il ?

Prendre note du contenu de `/etc/apache2/sites-enabled` et
`/etc/apache2/sites-available`.
Conformément à la documentation du fichier par défaut, utiliser la commande
`a2ensite server11.example.com` pour activer le fichier de virtual host ajouté.
Visiter de nouveau les URL sus-mentionnées. Consulter le fichier activé et lire
le contenu des fichiers de log paramétrés.

Question : quel est le chemin du fichier HTML servi ? De quel fichier de
configuration vient-il ? 

En utilisant le fichier de virtual host fourni comme modèle, créer un fichier
`www11.example.com.conf` ainsi qu'une arborescence et une page HTML répondant
au site *www11.example.com*.

## Etape 2 : le format des logs

L'oeil averti aura remarqué que les contenus de `/var/log/apache2/access.log`
et `/srv/www/server11.example.com/log/access.log` n'ont pas tout à fait le même
format.

Rechercher dans les fichiers `/etc/apache2/apache2.conf` et
`/etc/apache2/sites-available/server11.example.com.conf` les lignes commençant
par *CustomLog* et remarquer que le format défini est différent. Retrouver les
deux définitions de format dans `/etc/apache2/apache2.conf`.

Question : en s'aidant de la [documentation
officielle](https://httpd.apache.org/docs/2.4/fr/mod/mod_log_config.html#formats),
à quoi correspondent les champs des deux formats définis ? Quelles sont les
différences entre ces deux formats ?

