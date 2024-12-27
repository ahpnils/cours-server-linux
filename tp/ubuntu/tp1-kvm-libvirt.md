# TP 1 : paramètres supplémentaires de libvirt

Objectifs :

- découvrir les paramètres réseau de la machine virtuelle ;
- configurer le stockage des machines virtuelles en mode graphique.

## Etape 0 : paramètres réseau

Démarrer la machine virtuelle créée lors du TP 0. Attention, le clavier est
toujours configuré en QWERTY. Lancer la commande `setup-interfaces` (spécifique
à la distribution Alpine Linux), laisser les choix par défaut puis lancer la
commande `ifup eth0`. Enfin, lancer les commandes `ip addr show` et `ip route
show`. Noter l'adresse IP allouée à la machine virtuelle, ainsi que celle de la
passerelle par défaut.

Dans le gestionnaire de machines virtuelles, dans le menu "Edition", cliquer
sur "Détails de la connexion" puis dans l'onglet "Réseaux virtuels".

Depuis un terminal de l'hôte, lancer la commande `ping -c 5 <adresse ip de la
machine virtuelle>`

Questions : 

- quels sont les paramètres réseau appliqués à la machine virtuelle ?
- l'hôte et la machine virtuelle peuvent-il communiquer ?

# Etape 1 : stockage

Dans le gestionnaire de machines virtuelles, dans le menu "Edition", cliquer
sur "Détails de la connexion" puis dans l'onglet "stockage". 

Question : quels sont les pools de stockage disponibles ?

Créer les répertoires `~/libvirt/images` et `~/libvirt/boot`, et à partir du
menu de stockage dans le gestionnaire de machines virtuelle, créer deux pools
de stockage :
- un premier nommé `student_images` dont l'emplacement est le répertoire
  `libvirt/images` créé dans son répertoire personnel ;
- un deuxième nommé `student_boot` dont l'emplacement est le répertoire
  `libvirt/boot` créé dans son répertoire personnel.

Les deux pools de stockage doivent doivent être démarrés au démarrage de
l'ordinateur.

De retour dans la machine virtuelle, lancer la commande `fdisk -l`. Noter le
nom du disque et sa capacité. Eteindre la machine virtuelle, puis se rendre
dans les détails de celle-ci. Cliquer sur "Ajouter un matériel", sélectionner
"stockage", puis cliquer sur "Sélectionner un stockage personnalisé". Cliquer
sur "Gérer..." puis sélectionner le pool "student_images". Utiliser le bouton
"+" vert pour créer un nouveau volume de 2 Gio. Terminer, puis redémarrer la
machine virtuelle.

Lancer de nouveau la commande `fdisk -l`. 

Question : quels sont les disques présents et quelle est leur capacité ?

Eteindre la machine virtuelle et la supprimer. S'assurer que les deux disques
durs sont bien effacés.
