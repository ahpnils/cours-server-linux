# TP 3 : clonage de machine virtuelle

Objectif :

- expérimenter le clonage de machine virtuelle

## Etape 0 : clonage de la machine virtuelle

Éteindre la machine virtuelle server-11. Faire un clic droit sur celle-ci dans
le gestionnaire de machine virtuelles puis cliquer sur "Cloner..." Changer le
nom de la nouvelle machine virtuelle en "server-12".

Ouvrir les deux machines virtuelles (sans les démarrer) et vérifier là où elles
sont identiques et là où elles diffèrent. En particulier, examiner :

- le processeur ;
- la mémoire vive ;
- le fichier de disque dur ;
- la taille du disque dur ;
- la carte réseau.

Question : quels sont les points de différences entre les deux machines
virtuelles ?

### Manipulation optionnelle

Depuis la machine hôte, lancer (en tant qu'utilisateur) les commandes suivantes
:

```
virsh dumpxml server-11 | tee -a ~/server-11.xml
virsh dumpxml server-12 | tee -a ~/server-12.xml
```

Cela permet d'exporter les fichiers de définition des machines virtuelles.

Lancer la commande suivante pour comparer les deux machines virtuelles :

```
diff --color -bu ~/server-11.xml ~/server-12.xml
```

Question : voyez-vous des différences en plus ?

## Etape 1 : renommage et reconfiguration IP de la machine virtuelle

Démarrer la machine virtuelle "server-12", et s'y connecter.

Questions :

- quel est le nom de la machine virtuelle au niveau du système ?
- quelle est son adresse IP ?
- quelle aurait été la situation au niveau IP si la configuration était restée
  en DHCP ?

Nous allons renommer la machine virtuelle et modifier son adresse IP. Pour
cela, il faut savoir quels fichiers de configuration contiennent le nom et
l'adresse IP.
En tant que root, lancer la commande suivante :

```
grep --color -ir server-11 /etc/*
```

Relancer la même commande avec l'adresse IP.

Questions : 

- quels fichiers contiennent le nom de la machine ?
- quels fichiers contiennent l'adresse IP de la machine ?

Remplacer dans les fichiers concernés, à l'exception de ceux se trouvant dans
`/etc/ssh/`, le nom de la machine par son nouveau nom, et l'adresse IP par
192.168.122.12.

Pour la partie ssh, il s'agit de clés publiques, il vaut mieux supprimer ces 
fichiers et les clés privées associées, et les regénérer :
```
rm -vf /etc/ssh/ssh_host_*_key*
dpkg-reconfigure openssh-server
systemctl restart ssh
```

Redémarrer la machine virtuelle.

Reprendre les questions du début de cette étape, et relancer les grep.
Vérifier que de nouveaux fichiers "host key" sont apparus dans `/etc/ssh/`.

