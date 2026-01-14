[Retour au sommaire](../../README.md)

# TP 12 : Nginx en tant que reverse-proxy

Objectifs :

- installer le serveur HTTP Nginx sur server13;
- l'utiliser en tant que mandataire inverse (reverse proxy) pour des conteneurs
  Docker.

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

Se connecter en ssh sur server13, puis installer Nginx comme vu au TP 5.
Se rendre via le navigateur de l'hôte sur http://10.13.37.13 et vérfier que Nginx est bien lancé.

## Etape 1 : virtual hosts

Sur sa machine locale, ouvrir un terminal, et passer root. Editer le fichier de
configuration `/etc/hosts` et ajouter la ligne suivante :

```
10.13.37.13 server13 server13.example.com www13.example.com www13
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
répertoire `/etc/nginx/sites-enabled/` de server13.

Vérifier ensuite que http://server13.example.com fonctionne bien et renvoie
vers notre virtual host, et que les bons fichiers de log sont écrits.

## Etape 2 : reverse proxy basique

Toujours sur server13, en tant que root, ajouter la configuration suivante dans
le fichier de virtual host de server13.example.com, dans le bloc de
configuration `location /` :

```
proxy_pass http://10.13.37.11;
```

Mettre en commentaire la directive `try_files`, relancer Nginx puis accéder à
http://server13.example.com .

Questions : 
- quel est le site web affiché ?
- regarder les logs d'accès sur server11, quelle est la machine qui a accédé à
  son site web ?

Sur server13; éditer le fichier de configuration `/etc/hosts/` et ajouter la ligne suivante :

```
10.13.37.11 server11 server11.example.com www11.example.com
```

Puis, modifier la directive `proxy_pass` pour tenter d'accéder en particulier à
server11.example.com puis à www11.example.com. Ne pas oublier de relancer Nginx
à chaque modification du fichier de configuration.

## Etape 3 : reverse proxy avancé

Grâce aux commandes apprises lors de l'étape 2 du TP sur Docker, lancer un
conteneur docker [createleafcloud/echoip](https://github.com/leafcloudhq/echoip) et rendre
ce conteneur accessible sur le port 8080. Vérifier que le service est
disponible à l'URL http://10.13.37.13:8080 .

Changer ensuite la directive `proxy_pass` de Nginx sur le site
http://server13.example.com pour le faire pointer sur http://10.13.37.13:8080 .
Tester, puis ensuite faire pointer le reverse proxy sur http://127.0.0.1:8080 .

Questions :
- quelle est l'IP affichée via http://10.13.37.13:8080 ?
- quelle est l'IP affichée via http://server13.example.com quand il pointe sur http://10.13.37.13:8080 ?
- quelle est l'IP affichée via http://server13.example.com quand il pointe sur http://127.0.0.1:8080 ?

Ajouter ensuite les directives suivantes sous la directive `proxy_pass` :

```
proxy_set_header Host $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

Relancer Nginx et visiter de nouveau http://server13.example.com .

Questions :
- quel est maintenant le nom du site web ?
- quelle est l'IP affichée ?

Maintenant, stopper le conteneur docker et le relancer via la commande suivante
: `docker run --rm --name echoip -d -p 8080:8080 --entrypoint
"/opt/echoip/echoip" createleafcloud/echoip -p -H X-Forwarded-For`. On notera
l'ajout d'options pour :
- supprimer le conteneur lors de son arrêt ;
- modifier la commande lancée dans le conteneur (l'entrypoint) pour prendre en
  charge l'en-tête HTTP `X-Forwarded-For`.

Questions :
- quel est maintenant le nom du site web ?
- quelle est l'IP affichée ?
