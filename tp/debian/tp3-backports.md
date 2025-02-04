[Retour au sommaire](../../README.md)

# TP 3 : Les Backports dans Debian

Objectif : installer des logiciels en provenance du dépôt Backports.

# Introduction

Debian, dans sa version courante, est dite "stable". Cette stabilité a un prix :
les logiciels qui la comportent sont relativement anciens. Toutefois, il arrive
qu'on ait besoin de logiciels plus récents. Il est possible de compiler
soi-même les logiciels nécessaires, voire même de les empaqueter, mais la
gestion d'une plateforme de maintenance de logiciels peut s'avérer complexe.

Le projet Debian, conscient de cette difficulté, a créé un dépôt contenant de
nombreux logiciels provenant de la branche "testing". Ce dépôt est appelé
"Backports" (rétroportages en français).

# Etape 0 : installation et utilisation du dépôt Backports

Tout d'abord, recherchons la présence dans les dépôts disponibles du paquet
`linux-image-cloud-amd64`.

Question : quelle est la version du paquet `linux-image-cloud-amd64` dans les
dépôts ?

Le nouveau dépôt est disponible à l'URL suivante :

```
deb http://deb.debian.org/debian bookworm-backports main
```

Vérifier que le fichier `/etc/apt/sources.list` contient bien l'URL
ci-dessus, puis mettre à jour le cache local des paquets disponibles.

Recherche la présence du paquet `linux-image-cloud-amd64` avec la commande 
`apt search -t bookworm-backports linux-image-cloud-amd64`.

Question : quelle est la version du paquet `linux-image-cloud-amd64` dans le
dépôt backports ?

Grâce à la commande précédente, installer le paquet `linux-image-cloud-amd64`
depuis les backports, puis redémarrer le système et vérifier quel est le noyau
en cours de fonctionnement via la commande `uname -a`. 

Question : quelle est la version du noyau courant ? À quel paquet
correspond-elle ?
