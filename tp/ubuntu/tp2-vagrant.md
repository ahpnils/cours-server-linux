[Retour au sommaire](../../README.md)

# TP 2 : installation de Vagrant

Objectifs :

- installer Vagrant ;
- déployer et gérer 3 machines virtuelles Debian 12 avec Vagrant.

## Etape 0 : récupération du dépôt GitHub du projet

Se placer dans son répertoire favori de stockage de dépôts git, puis cloner le
dépôt : 

```
git clone https://github.com/ahpnils/ahp-vagrant-labs
```

## Etape 1 : installation de Vagrant et de son plugin libvirt

L'installation de Vagrant sous Ubuntu peut s'avérer complexe, en particulier le
plugin "vagrant-libvirt". Une automatisation via `make` a donc été mise en
place dans le dépôt [ahp-vagrant-labs](https://github.com/ahpnils/ahp-vagrant-labs) 
afin de faciliter cette étape. Se placer dans le répertoire
du dépôt, puis dans le répertoire `csl-libvirt-amd64`.

Installer l'outil `make` via les paquets Debian :
```
sudo apt update && sudo apt -y install make
```

Puis lancer la cible "make" pour installer et configurer Vagrant ainsi que le
plugin pour libvirt :
```
make ubuntu-deps # pour Ubuntu 24.04
make ubuntu2204-deps # pour Ubuntu 22.04
```

## Etape 2 : premiers pas avec Vagrant

Déployer les machines virtuelles s'effectue tout en restant dans le répertoire
`csl-libvirt-amd64`, via la commande suivante :
```
vagrant up
```

Cette commande va lire le fichier `Vagrantfile`, déployer les machines
virtuelles et les paramétrer au niveau système et réseau.

Une fois les machines installées, il est possible de vérifier qu'elles sont
démarrées via la commande `vagrant status`.

Question : combien de machines virtuelles sont déployées et quels sont les noms
de celles-ci ?

Il est maintenant possible de se connecter sur chaque machine via la commande
`vagrant ssh <nomduserveur>`. Tester la connexion sur toutes les VM.

Questions : quel est le nom de l'utilisateur une fois connecté ? Dispose-t'il
des droits super-utilisateurs (aide : `sudo -i` ou `sudo -l`) ?

Dans certains cas de figure, il se peut qu'il soit nécessaire de détruire une
VM. Pour cela, utiliser la commande `vagrant destroy <nomduserver>` et valider
via `vagrant status` qu'elle est détruite.

Une cible "make" permet de détruire tout le déploiement : `make destroy`.
Détruire tout le déploiement et le recréer.

## Etape 3 : communication réseau entre les VM

Se connecter sur chaque serveur, puis vérifier les éléments suivants :
- la commande `ip addr show` propose une interface en 10.13.37.X où X
  correspond au numéro du nom serveur ;
- la commande `ping -c 5 ouf.linuxfr.org` n'occasionne aucune perte de paquet ;
- la commande `cat /etc/hosts` montre bien des entrées pour les autres VM ;
- sur la première VM uniquement, lancer la commande `ping -c 5 <nomduserver>` pour
  chaque autre VM.

Sous Debian, la configuration de l'interface réseau se trouve dans le fichier
`/etc/network/interfaces`. Un répertoire `/etc/network/interfaces.d` peut 
contenir des fichiers de configuration supplémentaires.

Les commandes `ip addr show` et `ip route show` affichent les adresses IP et la
table de routage de la machine.

Questions :

- quel est le nom de l'interface réseau principale de la machine ?
- comment est configurée cette interface ?
- quelle est l'adresse IP de cette interface ?
- quelle est l'adresse IP de la passerelle par défaut ?
- comment sont configurées les autres interfaces ?

La commande `man interfaces` donne la syntaxe du fichier de configuration de
l'interface. 
