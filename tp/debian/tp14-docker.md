# TP 14 : les conteneurs et Docker

Objectifs :

- installer Docker ;
- lancer un premier conteneur, et contrôler son exécution ;
- lancer un conteneur disposant d'un programme installé ;
- créer son conteneur via un Dockerfile.

## Introduction

Le principe des conteneur est de fournir une couche d'isolation plus légère que
dans le cas des machines virtuelles. Cela s'obtient en limitant le nombre de
couches entre le noyau de l'hôte et l'application cible en espace utilisateur.

Pour aller plus loin : https://fr.wikipedia.org/wiki/Virtualisation#Isolateur

Il existe plusieurs technologies de conteneurs, généralement attachées à un
système d'exploitation donné (les zones pour Solaris, les jails pour FreeBSD).
Sous Linux, la solution la plus populaire est
[Docker](https://www.docker.com/). Il en exsite d'autres, comme
[LXC](https://linuxcontainers.org/lxc/) ou [podman](https://podman.io/).

## Etape 0 : installation de Docker

Se connecter sur server13 et passer root. Installer le paquet `docker.io`.
Vérifier que l'installation s'est bien déroulée en utilisant la commande
`docker version`.

Docker fonctionne à l'aide d'un service. Grâce à `systemctl`, vérifier que le
service `docker` est bien lancé et activé par défaut.

## Etape 1 : un premier conteneur, et contrôles

Pour démarrer un premier conteneur, il faut d'abord avoir une image contenant
le système. Comme les machines virtuelles sont sous Debian, télécharger une
image basique de conteneur Debian avec la commande suivante (toujours en tant
que root) : `docker pull debian `

Une fois l'image téléchargée, lancer un premier conteneur de type "Hello World"
:
```
docker run debian /bin/echo "Hello World"
```

Une fois l'action exécutée, le conteneur s'arrête. Vérifier qu'aucun conteneur
n'est en cours d'exécution via la commande `docker ps`, et voir le statut de
l'ancien conteneur en ajoutant l'option `-a` à cette commande.

Lancer la commande suivante :
```
docker run -it debian /bin/bash
```
On remarque alors que le prompt a changé. En effet, les options `-it`
permettent de passer en mode interactif.

Questions :
- quel est maintenant le nom d'hôte ?
- est-ce que nginx est installé ?
- quelle est la version installée du noyau ?
- quelle place reste-t'il sur le disque dur ?

Vérifier aussi la version du noyau ainsi que l'espace disque de server13, en
lançant un autre terminal et en effectuant une deuxième connexion ssh.
Vérifier aussi que le conteneur est toujours en exécution via `docker ps` (en 
tant que root).

Quitter le conteneur via la commande `exit`.

Relancer le conteneur, en mode interactif, puis vérifier depuis un autre
terminal que celui-ci est en cours d'exécution. De retour sur le shell du
conteneur, utiliser les raccourcis clavier Ctrl+p puis Ctrl+q.

Question : est-ce que le conteneur est toujours en cours d'exécution ?

Se reconnecter au conteneur en utilisant la commande `docker attach <id>`, où
`<id>` est à remplacer par l'ID, donné par le champ "CONTAINER ID" de la
commande `docker ps`. Cet ID est aussi ce qui remplace le nom de la machine
dans le prompt.

De retour sur l'autre terminal, on peut maintenant "éteindre" le conteneur via
la commande `docker kill <id>`

## Etape 2 : un service réseau dans le conteneur

Docker permet aussi de disposer de conteneur contenant des services prêt à
déployer, comme par exemple Nginx. Télécharger l'image officielle Nginx via la
commande `docker pull nginx` (en tant que root). On peut vérifier la listes des
images installées sur notre système via la commande `docker image ls`.

Exécuter ensuite le conteneur via la commande `docker run -it -p 8080:80
nginx`. Depuis l'hôte, accéder à l'URL http://192.168.122.13:8080/ . Quitter le
conteneur via `Ctrl-C`, puis le relancer via `docker run -d -p 8080:80 nginx`.
Pour accéder au contenu du conteneur alors qu'il est en exécution détachée,
utiliser la commande `docker exec -it <id> /bin/bash`.

Une fois dans le conteneur, vérifier sur quel port écoute Nginx (aide :
regarder le contenu de `/etc/nginx/conf.d/default.conf`). Les plus avancés
pourront remarquer que la configuration Nginx ne prévoit pas de journalisation
des requêtes.

Eteindre le conteneur ainsi lancé.

## Etape 3 : un conteneur contenant un programme de son choix

Lancer un conteneur via la commande suivante : 
```
docker run debian /bin/bash -c "apt-get update; apt-get -y install nginx"
```

Une fois que le conteneur a terminé son exécution, retrouver son ID dans
l'historique des conteneurs exécutés (`docker ps -a`). Créer une image
personnalisée via la commande suivante :
```
docker commit <id> cours-linux/debian-nginx
```

Vérifier que l'image existe via `docker image ls`. Lancer un nouveau conteneur
grâce à cette image :
```
docker run -it -p 8080:80 cours-linux/debian-nginx /bin/bash
```

Puis, lancer Nginx manuellement, depuis le conteneur : 
```
/usr/sbin/nginx
```

Vérifier que http://192.168.122.13:8080/ répond bien. Eteindre le conteneur.

## Etape 4 : conteneur personnalisé avec un Dockerfile

Transférer sur server13 le fichier `docker/Dockerfile` de ce dépôt. Depuis le
répertoire où se trouve le fichier `Dockerfile`, lancer la commande suivante
(en tant que root):
```
docker build -t cours-linux/debian-apache2:latest ./
```

Ensuite, lancer un conteneur utilisant l'image personnalisée ainsi crée :
```
docker run -d -p 8080:80 cours-linux/debian-apache2
```

Depuis l'hôte, se rendre sur http://192.168.122.13:8080/

Pour aller plus loin : à partir du Dockerfile fourni, récupérer le contenu [de
ce dépôt](https://github.com/ahpnils/anotherhomepage.org/tree/main/content) et
le servir via Docker. En bonus ++, paramétrer Nginx sur server13 en tant que
reverse-proxy sur http://127.0.0.1:8080.
