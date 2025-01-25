[Retour au sommaire](../../README.md)

# TP 14 : sudo

Objectifs :

- configurer sudo finement

## Introduction

L'utilitaire sudo a déjà été abordé dans [le chapitre
16](https://github.com/ahpnils/cours-linux-shell/blob/main/ch/ch16_root_sudo.md)
du [cours Linux
Shell](https://github.com/ahpnils/cours-linux-shell/), mais de façon assez
brève. Voyons ici quelques configurations plus fines.

Attention : une mauvaise configuration peut donner trop de droits à un ou
plusieurs utilisateurs. Cela peut faciliter la compromission d'un serveur. Pour
limiter cela, la lecture du document ["Recommandations de configuration d’un
système
GNU/Linux"](https://www.ssi.gouv.fr/particulier/guide/recommandations-de-securite-relatives-a-un-systeme-gnulinux/) 
de l'ANSSI est fortement encouragé, et contient quelques paragraphes sur sudo.

## Etape 0 : installation de sudo

Comme beaucoup de logiciels, le paquet à installer correspond au nom de
celui-ci. Le paquet "sudo" est normalement déjà installé.

En tant que root, sur `server11`, vérifier que le paquet "sudo" est installé,
et que les fichiers suivants sont présents :
- `/usr/bin/sudo`
- `/usr/sbin/visudo`
- `/etc/sudoers`

## Etape 1 : premières utilisations et configurations

Par commodité, lorsque sudo demande un mot de passe, et que l'authentification
est réussie, celle-ci considérée valide pendant un certain temps (15 ou 5
minutes selon les versions). Cela peut perturber le TP et fausser des
résultats, nous allons donc désactiver ce "temps de grâce". Se connecter sur
`server11`, et passer root. Lancer la commande `visudo`. Celle-ci lance un
éditeur de texte. Dans le texte, repérer les lignes commençant par "Defaults"
et ajouter sous celles-ci la ligne suivante :
```
Defaults    timestamp_timeout=0
```
Sauver le fichier, et quitter l'éditeur.

Se connecter sur `server11` en tant qu'utilisateur student (créé au TP4) et lancer la
commande `sudo -l`. Celle-ci permet de lister les droits sudo disponibles à
l'utilisateur l'appelant. Elle devrait demander un mot de passe.

Question : de quels droits sudo dispose l'utilisateur student ? Même question
pour l'utilisateur root.

En tant que root, ajouter l'utilisateur student au groupe sudo. Déconnecter la
session de l'utilisateur student, puis s'y reconnecter. Lister de nouveau les
droits sudo depuis l'utilisateur student.

Question : de quels droits sudo dispose maintenant l'utilisateur student ?

Toujours en tant qu'utilisateur student, lancer la commande `sudo whoami`.

Question : quel est le résultat de la commande `sudo whoami` ?

## Etape 2 : sudo sans mot de passe

Il est possible de lancer des commandes avec les droits root sans mot de passe.
Profitons-en pour découvrir une autre fonctionnalité de sudo, l'inclusion de
fichiers de configuration complémentaire.

Toujours en tant que root sur `server11`, créer un nouveau fichier de
configuration sudo via la commande `visudo -f /etc/sudoers.d/student`, et y
insérer le contenu suivant :
```
student		ALL=(ALL)	NOPASSWD: ALL
```

Se reconnecter en tant que student, et lancer les commandes `sudo -l` ainsi que
`sudo whoami`, constater qu'il n'y a pas besoin de mot de passe.

Comparer la syntaxe de la déclaration ci-dessus avec celle du fichier
`/etc/sudoers` qui donne l'accès à sudo au groupe `sudo`.

Questions :
- à quel endroit définit-on l'utilisateur ou le groupe ?
- à quel endroit définit-on qu'on ne souhaite pas de mot de passe ?
- à quel endroit définit-on l'inclusion de fichiers supplémentaires ?

## Etape 3 : les alias

En plus d'utiliser les utilisateurs et groupes du système, sudo est capable de
créer ses propres listes d'utilisateurs ou de commandes autorisées. En tant que
root, sur `server11`, modifier la configuration sudo créée à l'étape précédente
comme suit :

```
Cmnd_Alias SERVICES = /usr/bin/systemctl status, /usr/bin/top
student ALL=(ALL) NOPASSWD:SERVICES
``` 

Questions : 
- quelles commandes sudo peut lancer student sans mot de passe ?
- Est-ce qu'il y a une limite sur les arguments ?
- Que se passe-t'il pour les autres commandes ? Pourquoi ?

## Etape 4 : là où on va il n'y a pas besoin de root

En tant que root, sur server11, créer un utilisateur nommé student1.

Depuis la session de l'utilisateur student, lancer la commande suivante :
```
sudo -u student1 whoami
```

En tant que root, ajouter la configuration suivante à sudo :
```
Cmnd_Alias WHO = /usr/bin/whoami
student ALL=(student1) NOPASSWD:WHO
```

Relancer la commande sudo précédente et analyser la différence.

## Etape 5 : parce qu'il faut bien s'amuser !

En tant que root sur `server11`, ajouter à la configuration de sudo la ligne
suivante sous les directives `Defaults` :

```
Defaults insults
```

En tant qu'utilisateur student, lancer une commande via sudo et taper plusieurs
fois le mauvais mot de passe ;-)
