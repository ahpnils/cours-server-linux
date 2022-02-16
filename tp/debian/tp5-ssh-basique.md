# TP 5 : découverte de SSH

Objectifs :

- se connecter et lancer des commandes à distance via un shell sécurisé ;
- configurer OpenSSH pour faciliter ses connexions ;
- copier des fichiers et des arborescences entre son poste et la machine
  distante ;
- effectuer une authentification par clé au lieu du mot de passe.

## Introduction

SSH (pour *Secure SHell*) désigne à la fois un protocole et un logiciel
permettant une connexion à distance sécurisée. Son implémentation la plus
connue et la plus utilisée, *OpenSSH*, remplace avantageusement des outils et
protocoles historiques comme telnet, rcp, ou ftp.

Un outil connu utilisé conjointement avec SSH est `rsync`, qui permet de faire
de la synchronisation unidirectionnelle entre deux fichiers ou arborescences de
fichiers, locaux mais aussi distants.

## Etape 0 : premières connexions SSH

Pour se connecter à distance à la machine server11, lancer la commande
suivante : `ssh student@192.168.122.11`. Le format est donc `ssh
<utilisateur>@<adresse IP ou nom>`. À la première connexion, une empreinte de
clé est affichée et OpenSSH demande une confirmation avant de continuer la
connexion. Il s'agit du principe
[TOFU](https://en.wikipedia.org/wiki/Trust_on_first_use). Le mot de passe est
ensuite demandé (on peut retrouver ce mot de passe dans le fichier de
configuration preseed). Enfin, des informations sont affichées et le prompt de
la machine distante est disponible.

Vérifier qu'on est bien connecté sur une machine distante, grâce aux commandes
`ip addr show` et `hostname`.

Pour quitter la session SSH, il suffit de taper `exit` ou d'utiliser le
raccourci clavier `Ctrl+D`.

Relancer la même commande de connexion SSH, puis se déconnecter :
cette-fois-ci, pas de demande d'acceptation de clé.
Puis, essayer les commandes de connexion suivantes :

```
ssh 192.168.122.11
ssh server11.example.com
ssh student@server11.example.com
ssh server11
ssh student@server11
```

Question : les commandes ci-dessus ont-elles fonctionné ? Pourquoi ?

## Etape 1 : configurer son client SSH

Le client OpenSSH dispose de nombreuses fonctionnalités, l'une des plus
pratiques au quotidien est la possibilité d'utiliser un fichier de
configuration pour appliquer certaines options. Le fichier de configuration du
client OpenSSH est `/etc/ssh/ssh_config`, mais il est possible d'en ajouter un
second dans `~/.ssh/config`. Le répertoire `~/.ssh` a normalement été créé lors
de la première connexion.

Créer ou éditer le fichier `~/.ssh/config` et ajouter le contenu suivant :

```
Host server11
  Hostname 192.168.122.11
```

Une fois le fichier sauvegardé, tenter de se connecter à l'aide des commandes
suivantes :

```
ssh student@server11
ssh server11
```

Question : pourquoi la commande `ssh server11` ne fonctionne pas ?

Modifier le fichier de configuration `~/.ssh/config` pour que la partie
concernant server11 soit comme ceci :

```
Host server11
  Hostname 192.168.122.11
  User student
```

Se connecter en utilisant la commande `ssh server11`.

À partir de cet exemple, ajouter les configurations pour server12 et server13
et se connecter au deux machines. Vérifier que les connexions correspondent
bien aux machines, puis se déconnecter.

## Etape 2 : transfert de fichiers avec scp

OpenSSH permet bien plus que simplement taper des commandes en toute sécurité.
Il permet aussi de transférer des fichiers en utilisant deux protocoles : scp
et sftp. Il s'agit aussi des noms des outils. À noter que ces outils tirent
parti du fichier de configuration ssh, ainsi, en conservant le fichier de
configuration de l'étape précédente, les deux commandes `scp
student@192.168.122.11:/etc/hosts /tmp` et `scp server11:/etc/hosts /tmp` sont
équivalentes.

Se placer à la racine de ce dépôt, et copier le fichier `README.md` vers
server12 avec la commande suivante : `scp ./README.md server12:` (les deux
points à la fin sont importants). Vérifier ensuite que le fichier est bien sur
la machine distante.

Question : dans quel répertoire est le fichier `README.md` sur la machine
distante ?

Se placer dans son répertoire de départ, puis récupérer le fichier
`/etc/resolv.conf` de la machine server11, via la commande suivante : `scp
server11:/etc/resolv.conf ./`.

Question : où est arrivé le fichier `resolv.conf` de server11 ? Modifier la
commande pour que le fichier arrive dans `/tmp` sur la machine locale.

L'outil scp permet aussi de faire du transfert récursif (option `-r`) et d'être
verbeux (option `-v`). Effectuer une copie récursive et verbeuse de ce dépôt
dans le répertoire `/tmp` de server13.

Question : quelle est la commande ?

## Etape 3 : transfert de fichiers avec sftp

L'outil sftp permet, de la même façon que ftp, de se connecter et de parcourir
l'arborescence du serveur distant, tout en profitant des options de ssh. Comme
pour scp, il est donc possible de se connecter à server13

## Etape 4 : authentification par clés

En plus de l'authentification par mot de passe, OpenSSH gère un autre moyen
d'authentification, l'utilisation de clés, basé sur le concept de clé privée /
clé publique. Le client SSH a donc à sa disposition une clé privée, et le
serveur SSH doit posséder la clé publique correspondante (généralement
paramétrée au préalable). La connexion peut donc s'effectuer sans mot de passe,
et est très pratique dans le cas de connexion et d'exécution de commandes par
des robots (sauvegarde, audit, déploiements automatisés...). À noter que pour
une sécurité accrue, il est possible de paramétrer une phrase de passe pour la
clée privée : de cette façon, une fuite de ce fichier ne compromettra pas
l'accès au compte utilisateur. Enfin, tant que le mot de passe n'est pas
deviné...

La génération de clés se fait via l'outil `ssh-keygen`. Il est probable que des
clés existent déjà dans le répertoire `~/.ssh/` (des fichiers commençant par
`id_`, certains se finissant en `.pub`). Par précaution, les actions menées ici
vont créer de nouveaux fichiers afin de ne pas perturber l'existant.

Générer une première clé avec la commande suivante, sans entrer de phrase de
passe : `ssh-keygen -f ~/.ssh/student_rsa`.

Questions :
- quel est le fichier de clé privée ?
- quel est le fichier de clé publique ?
- quels sont le format et la taille de clé ?
- quels sont les droits sur les fichiers de clés ?

Installer la clé sur un des serveurs distants avec la commande suivante :
`ssh-copy-id -i ~/.ssh/student_rsa server11`. Suivre les consignes du
programme. Se connecter à server11 via la commande `ssh -i
~/.ssh/student_rsa server11`.

Questions :
- quel est le contenu du fichier `~/.ssh/authorized_keys` ?
- quels sont les droits sur ce fichier ?
- à quel fichier de clé sur le poste de travail correspond le contenu de
  `~/.ssh/authorized_keys` de server11 ?

Pour les clés de type RSA, il est possible de spécifier la taille des clés. Une
taille de clé plus importante permet, en théorie, une plus grande sécurité,
mais réclame plus de puissance de calcul. Cela se fait grâce à l'option `-b`
suivie du nombre de bits composant la taille de la clé.

À l'aide des commandes précédentes, générer une paire de clés ayant les
propriétés suivantes :
- taille de clé de 4096 bits ;
- le fichier de clé privée se trouve dans `~/.ssh/student_rsa_4k`

Ensuite, installer la clé sur server11, et tenter se connecter en utilisant la
nouvelle clé.

Question : quel est le contenu du fichier `~/.ssh/authorized_keys` ?

OpenSSH dispose aussi d'autres types de clés, correspondant à d'autres
technologies de chiffrement, comme les courbes elliptiques. Les commandes `man
ssh-keygen` et `ssh-keygen --help` permettent de voir les différents type de
chiffrement disponibles.

Rechercher comment générer une clé de type *ed25519*, et en générer une se
nommant `student_ed25519`, puis l'ajouter au fichier `authorized_keys` de
server11 et se connecter en spécifiant cette clé.

Question : en regardant `authorized_keys`, quelle est la clé publique la plus
petite, et quelle est la plus grande, en nombre de caractères ?

Noter les droits des fichiers de clés et l'authorized_keys. Ceux-ci devraient
être :

- `0600` pour `authorized_keys` ;
- `0600` pour les clés privées ;
- `0644` pour les clés publiques.

Pour les fichiers `authorized_keys` et `student_rsa`, modifier un par un les
droits en 0644 puis tenter de se connecter (avec la clé correspondante).
Recommencer en modifiant les droits en 0666, et le faire avec les fichiers de
clés publiques. Ne pas oublier de restaurer les droits initiaux après les
tentatives de connexion.

Question : est-ce qu'OpenSSH est sensible sur les droits des fichiers de clés
et d'autorisation de clés ?

Il est bien entendu possible de spécifier la clé dans le fichier
`~/.ssh/config`, via l'option `IdentityFile`. Ajouter cette option pour les
trois machines, en spécifiant la clé au format ed25519, et installer cette clé
sur les deux machines qui ne l'ont pas. Se connecter en utilisant juste la
commande `ssh server11`, et ainsi de suite pour les deux autres machines.

Pour aller plus loin : recommencer toute l'étape 4 en spécifiant une phrase de
passe à chaque clé.
