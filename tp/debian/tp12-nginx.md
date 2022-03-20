# TP 12 : à la découverte de Nginx

Objectifs :

- installer le serveur HTTP Nginx ;
- créer des virtual hosts ;
- l'utiliser en tant que mandataire inverse (reverse proxy).

## Introduction

Nginx est un serveur HTTP, au même titre qu'Apache. Il est un peu moins connu
mais néanmoins particulièrement populaire grâce à ses capacités de serveur
mandataire inverse (reverse proxy en anglais). Cela veut dire qu'il se place en
frontal d'un ou de plusieurs serveurs web existant, et permet de mettre en
place différentes stratégies, parmi lesquelles :

- rediriger du trafic vers un serveur ou en local ;
- mettre en cache pour servir du contenu plus rapidement ;
- effectuer de la répartition de charge entre plusieurs serveurs identiques ;
- effectuer un filtrage de sécurité.

À noter que contrairement à Apache, Nginx n'est pas développé par une fondation
mais par une entreprise. Le logiciel d'origine est libre, sous licence BSD
2-clauses, mais il existe une version commerciale, Nginx Plus.

## Etape 0 : installation

Comme bon nombre de distributions Linux et BSD, Nginx est disponible via des
paquets binaires pré-compilés pour Debian. 

Se connecter en ssh sur server13, puis passer root. Installer Nginx via la
commande `apt install nginx`. Se rendre via le navigateur de l'hôte sur
http://192.168.122.13 et vérfier que Nginx est bien lancé.

Consulter le répertoire `/etc/nginx/` et comparer par rapport à
`/etc/apache2/`, en particulier pour les répertoires des virtual hosts et des
modules.

## Etape 1 : virtual hosts

Sur sa machine locale, ouvrir un terminal, et passer root. Editer le fichier de
configuration `/etc/hosts` et ajouter la ligne suivante :

```
192.168.122.13 server13 server13.example.com www13.example.com www13
```
Une fois le fichier sauvé, s'assurer que la machine server13 est bien
accessible via les noms ajoutés ci-dessus, par exemple avec la commande `ping`.
Maintenant, vérifier que Nginx répond lors de requêtes vers les noms ajoutes.

Se connecter sur server13 et passer root. Créer l'aborescence suivante :

```
/srv/
└── www
    └── server13.example.com
        ├── log
        └── public
```

Ajouter un fichier `index.html` au contenu libre dans le répertoire `public`.

Transférer le fichier `nginx/server13.example.com.conf` de ce dépôt dans le
répertoire `/etc/nginx/sites-available/` de server13.

Nginx ne possède pas de programme d'aide comme Apache pour simplifier la
création des liens. Se rendre dans `/etc/nginx/sites-enabled/`, puis créer un
lien symbolique du fichier de configuration récemment transféré : `ln -sv
/etc/nginx/sites-available/server13.example.com.conf ./`. Nginx étant démarré
comme n'importe quel service, utiliser `systemctl restart nginx` pour le
redémarrer.

Vérifier ensuite que http://server13.example.com fonctionne bien et renvoie
vers notre virtual host, et que les bons fichiers de log sont écrits.

À partir ce modèle, créer une arborescence et un fichier de configuration pour
répondre à http://www13.example.com.


## Etape 2 : reverse proxy basique

Toujours sur server13, en tant que root, ajouter la configuration suivante dans
le fichier de virtual host de server13.example.com, dans le bloc de
configuration `location /` :

```
proxy_pass http://192.168.122.11;
```

Relancer Nginx puis accéder à http://server13.example.com .

Questions : 
- quel est le site web affiché ?
- regarder les logs d'accès sur server11, quelle est la machine qui a accédé à
  son site web ?

Sur server13; éditer le fichier de configuration `/etc/hosts/` et ajouter la ligne suivante :

```
192.168.122.11 server11 server11.example.com www11.example.com
```

Puis, modifier la directive `proxy_pass` pour tenter d'accéder en particulier à
server11.example.com puis à www11.example.com. Ne pas oublier de relancer Nginx
à chaque modification du fichier de configuration.
