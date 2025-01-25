[Retour au sommaire](../../README.md)

# TP 15 : ssh avancé

Objectifs :

- effectuer une connexion SSH par rebond ;
- effectuer une redirection de port distant vers sa machine, via un tunnel.

## Introduction

OpenSSH est capable de bien plus que juste lancer des commandes au travers d'un
canal sécurisé. Ce TP montre deux usages pratiques.

## Etape 0 : préparatifs

Créer un utilisateur student sur les machines server12 et server13, et
configurer une authentification ssh par clé. Si besoin, se référer au TP 4.

S'assurer aussi que la configuration suivante est présente dans `~/.ssh/config`
sur le poste local : 
```
Host server11
  Hostname 10.13.37.11
  User student
Host server12
  Hostname 10.13.37.12
  User student
Host server13
  Hostname 10.13.37.13
  User student
```

## Etape 1 : connexion par rebond

Pour plus de sécurité, certains accès administratifs sont parfois restreints et
il n'est pas possible d'accéder directement à certaines machines. Supposons que
ce soit le cas pour `server11` et `server12`, et que seul `server13` soit
autorisé à s'y connecter. La première solution consiste alors à se connecter
sur `server13`, puis à se connecter sur la machine de destination...

Depuis le poste local, lancer la commande suivante : `ssh server11 -J
server13`. Se déconnecter, puis se reconnecter en utilisant en plus l'option
`-v`, et observer les différentes phases de connexion.

Questions : 
- sur quelle machine est-on connecté ?
- en utilisant la commande `who`, depuis quelle machine est-on connecté ?

L'option `-J` dispose d'un équivalent en fichier de configuration. Ajouter dans
le bloc de définition de `server12` la ligne suivante :
```
ProxyJump server13
```

Se connecter ensuite sur `server12` depuis le poste local de façon classique
`ssh server12` et vérifier avec la commande `who` d'où vient la connexion sur
`server12`.

Créer un utilisateur student1 sur `server13`, puis remplacer la ligne "ProxyJump"
par la ligne suivante :
```
ProxyJump student1@server13
```

Se connecter de nouveau à `server12`.

## Etape 2 : redirection de port via tunnel

De la même façon que dans l'étape 1, les accès administratifs peuvent aussi
être des vues de statistiques de serveur web. Un module pour Apache, nommé
`mod_status` permet de voir en direct le nombre de connexion vers le célèbre
serveur web, et cela tombe bien, il est activé sur `server11`. Or, comme
l'atteste le fichier `/etc/apache2/mods-enabled/status.conf`, l'accès à ces
statistiques n'est accessible que depuis la machine locale. Il est possible
d'utiliser un navigateur en mode texte, ou de récupérer la page à l'aide de
`curl` ou de `wget`, mais ceci est peu confortable...

Depuis le poste local, se connecter à `server11` via la commande suivante : 
```
ssh -L 8080:127.0.0.1:80 server11
```

Dans un autre terminal, toujours sur le poste local, vérifier que le port 8080
est bien en écoute sur la machine, via la commande `sudo lsof -i -n | grep
ssh`.

Se connecter ensuite depuis le poste local sur http://10.13.37.11/server-status, puis se connecter sur http://127.0.0.1:8080/server-status .

Cette connexion peut aussi se faire au niveau du fichier de configuration. Se
déconnecter de la session ssh vers `server11`, et ajouter dans le bloc de
configuration la directive suivante :
```
LocalForward 8080 127.0.0.0.1:80
```

Vérifier que http://127.0.0.1:8080/server-status fonctionne toujours.

