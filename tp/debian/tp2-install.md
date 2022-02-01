# TP 2 : installation d'un premier serveur Debian

Objectifs :

- installer une machine virtuelle Debian ;
- gérer les logiciels : installation, suppression, mise à jour, gestion des
  dépôts...
- reconfigurer le réseau ;
- gérer les utilisateurs et les groupes

## Etape 0 : création de la machine virtuelle

Se rendre sur https://www.debian.org/download/ et télécharger l'image ISO
d'installation par le réseau (netinstall) pour amd64. Placer le fichier dans
`~/libvirt/boot/`.

Créer une nouvelle machine virtuelle avec les paramètres suivants :

- utiliser l'image ISO téléchargée comme média d'installation ;
- système d'exploitation Debian 11 ;
- 1 vcpu ;
- 1024 Mo de mémoire vive ;
- un disque de 10 Gio dans le pool "student_images", nommé `server11.qcow2` ;
- nom : server11 ;
- réseau virtuel "default", en NAT.

## Etape 1 : installation de Debian

La machine virtuelle démarre sur l'installeur Debian. L'option par défaut est
"Graphical Install". Appuyer sur la touche entrée.

L'installeur fonctionne à l'aide d'un menu qui propose différentes options.
Paramétrer les options suivantes :

- language : anglais ;
- location : other > Europe > France ;
- locales : en_US.UTF-8 ;
- keyboard : selon votre système ;
- hostname : server11 ;
- domainname : example.com ;
- root password : password ;
- full name for the new user : student ;
- password for the new user : password ;
- partitioning method : Guided - use entire disk ;
- partitioning scheme : All files in one partition ;
- Write the changes to disk : yes ;
- Software selection : cocher "SSH server" et "standard system utilities",
  décocher tout le reste ;
- device for boot loader installation : `/dev/vda`.

Les options non précisées ci-dessus doivent être laissées par défaut.

Une fois l'installation terminée et la machine redémarrée, se connecter en tant
qu'utilisateur student, puis utiliser la commande `su -` pour passer root.
Attention : les mots de passe sont tapés à l'aveugle !

## Etape 2 : gestion de logiciels

Debian, comme de nombreuses distributions Linux, permet d'installer des
logiciels via des paquets, disponibles dans des dépôts. On peut assimiler
l'installation de paquets logiciels à l'installation d'applications depuis le
"store" de son smartphone. De la même façon, il est possible d'installer des
paquets directement depuis des fichiers téléchargés (comme on peut directement
installer des fichiers `.apk` sous Android, mais les bonnes pratiques
découragent cet usage).

Sous Debian, `dpkg` et `apt` permettent de gérer les paquets logiciels, au
format ".deb". Le premier est un outil basique, alors que le second est bien
plus avancé et permet de gérer les dépendances (un logiciel ayant besoin d'un
autre ou d'une bibliothèque pour fonctionner).

La commande `dpkg --list` permet d'afficher la liste des paquets installés sur
le système. Dans cette liste, les lignes comportant les logiciels installés
commencent par `ii`.

Question : combien de logiciels sont installés ?

Question : les paquets tmux, rsync et openssh-server sont-il installés ? Si
oui, en quelle version ?

Le logiciel `apt` permet les actions suivantes :

- `apt install <package>` installe le paquet "package" ;
- `apt search <package>` recherche le paquet "package" dans les dépôts ;
- `apt purge <package>` désinstalle le paquet "package" ;
- `apt update` met à jour le cache local contenant la liste des paquets
  disponibles ;
- `apt upgrade` met à jour les paquets installés.

À l'aide de ces commandes, effectuer les actions suivantes dans l'ordre :
- mettre à jour le cache local des paquets disponibles
- installer les logiciels suivants : tmux, sudo, rsync, git
- désinstaller le paquet "laptop-detect"

Attention, certaines de ces commandes nécessitent les droits root !

Question : est-ce que des mises à jour de paquets sont disponibles ?

Passons à la gestion des dépôts logiciels. Ceux-ci sont configurés à plusieurs
endroits : dans le fichier `/etc/apt/sources.list`, ou dans un fichier dont
l'extension se termine par `.list` et se trouvant dans le répertoire
`/etc/apt/sources.list.d`.

Nous allons ajouter le dépôt "backports". Il s'agit d'un dépôt officiel du
projet Debian, qui vise à fournir des versions logicielles plus récentes dans
la branche stable.

Tout d'abord, recherchons la présence dans les dépôts disponibles du paquet
`linux-image-cloud-amd64`.

Question : quelle est la version du paquet `linux-image-cloud-amd64` dans les
dépôts ?

Le nouveau dépôt est disponible à l'URL suivante :

```
deb http://deb.debian.org/debian bullseye-backports main
```

Créer un fichier `/etc/apt/sources.list.d/backports.list` et y ajouter l'URL
ci-dessus, puis mettre à jour le cache local des paquets disponibles.

Rechercher à nouveau la présence dans les dépôts disponibles du paquet
`linux-image-cloud-amd64`. Recommencer avec la commande `apt search -t
bullseye-backports linux-image-cloud-amd64`.

Question : quelle est la version du paquet `linux-image-cloud-amd64` dans le
dépôt backports ?

Grâce à la commande précédente, installer le paquet `linux-image-cloud-amd64`
depuis les backports.

## Etape 3 : configuration du réseau

Sous Debian, la configuration de l'interface réseau se trouve dans le fichier
`/etc/network/interfaces`. Comme pour la configuration des dépôts, un
répertoire `/etc/network/interfaces.d` peut contenir des fichiers de
configuration supplémentaires.

Les commandes `ip addr show` et `ip route show` affichent les adresses IP et la
table de routage de la machine.

Questions :

- quel est le nom de l'interface réseau principale de la machine ?
- comment est configurée cette interface ?
- quelle est l'adresse IP de cette interface ?
- quelle est l'adresse IP de la passerelle par défaut ?

La commande `man interfaces` donne la syntaxe du fichier de configuration de
l'interface. Grâce à cette documentation, modifier la configuration IP pour
qu'elle ait ces paramètres :

- adresse IP : 192.168.122.11/24 ;
- passerelle par défaut : 192.168.122.1.

Lancer la commande `ifdown <interface>` puis `ifup <interface>` et vérifier que
les paramètres réseau sont bien appliqués. Vérifier via les commandes `ping -c
5 192.168.122.1` et `ping -c 5 1.1.1.1` que l'accès au réseau et à Internet
sont toujours disponibles.

## Etape 4 : gestion des utilisateurs

Un utilisateur a été créé lors de l'installation, mais il est possible de créer
d'autres utilisateurs une fois le système installé.

Lancer la commande suivante, en tant que root :
```
useradd -m -U -g 2001 -u 2001 -s /bin/bash student1
```

Celle-ci crée un utilisateur, dont les propriétés sont les suivantes :

* son login est `student1` ;
* son shell (`-s`) est `/bin/bash` ;
* son UID (`-u`) est `2001` ;
* son groupe principal est aussi `student1`, et son GID `2001` (`-U`) ;
* son répertoire "home" est créé (`-m`).

Créer maintenant deux utilisateurs :

* `student2` et `student3` ;
* leur shell sera `/bin/bash` ;
* leurs UID sont respectivement `2002` et `2003` ;
* les GID et groupes principaux seront identiques aux UID et logins ;
* `student2` fera partie du groupe `staff` (option `-G`) ;
* `student3` fera partie du groupe `users` ;
* les répertoires "home" sont créés.

Vérifier que tout est bien créé dans les contenus des fichiers `/etc/passwd` et
`/etc/group`. Vérifier la présence des répertoires utilisateurs dans `/home`.

Toujours en tant que root, ajouter un mot de passe aux trois comptes créés dans
cette étape. Par exemple, la commande `passwd student1` permet de modifier le
mot de passe du compte `student1`. Se déconnecter de la session root et
vérifier que le mot de passe est bien fonctionnel sur chaque compte.

## Etape 5 : gestion des groupes

