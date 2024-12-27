[Retour au sommaire](../../README.md)

# TP 5 : Installation et paramétrage basique du serveur web Apache

Objectifs :

- installer le serveur HTTP Apache ;
- le démarrer, l'arrêter et l'activer au démarrage du serveur ;
- déposer du contenu et le rendre accessible ;
- personnaliser l'emplacement du contenu ;
- accéder aux fichiers de logs.

## Introduction

Apache HTTP Server (Apache) est un serveur HTTP, probablement le plus connu au
monde. Le nom complet est bien "Apache HTTP Server", mais souvent par abus de
langage, on le nomme en raccourci "Apache". Cela peut parfois porter à
confusion avec d'autres logiciels de la fondation éponyme, qui gère le
développement de ce serveur web mais aussi, entre autres, d'Open Office (suite
bureautique), Tomcat (serveur d'applications) et de nombreux logiciels utilisés
en environnement Java (comme log4j).

## Étape 0 : installation

Comme bon nombre de distributions Linux et BSD, Apache est disponible via des
paquets binaires pré-compilés pour Debian. Le nom du paquet varie selon les
distributions, ici il se nomme `apache2`.

Se connecter sur server11, puis passer root.
Avant l'installaton, regarder quels sont les services réseau en fonctionnement
: `lsof -n -i`. Seul sshd devrait apparaître. Maintenant, installer Apache :
`apt -y install apache2`. Relancer la commande d'avant.

Question : est-ce que l'installation d'Apache a lancé celui-ci ?

Se rendre via le navigateur de l'hôte sur http://10.13.37.11. Il est aussi
possible, depuis server11 ou depuis l'hôte, d'utiliser un requêteur HTTP en
ligne de commande comme `curl`. La lecture de la page affichée est recommandée,
et contient des informations intéressantes pour les prochaines étapes et
prochains TP.

## Étape 1 : démarrage, arrêt, et activation au démarrage

Se connecter en ssh sur server11, puis passer root.
Vérifier grâce à l'outil `ps` si Apache est toujours en fonctionnement.

Questions : 
- comment s'appelle le programme ?
- combien de processus correspondants sont en fonctionnement ?
- à quels utilisateurs appartiennent ces processus ?

Apache dispose d'un petit programme permettant de le contrôler. Sous Debian, il
est disponible via la commande `apachectl` ou `apache2ctl`. Il est possible
d'utiliser ce programme pour démarrer, arrêter et relancer Apache, entre
autres. 

Lancer la commande `apachectl stop`, puis vérifier via `ps` que celui-ci est
bien arrêté. Lancer ensuite la commande `apachectl start`, et vérifier de la
même façon qu'Apache fonctionne. Ouvrir un deuxième terminal, se connecter en
ssh sur server11, puis passer root. Lancer la commande `top -u www-data`.
Depuis le premier terminal, lancer la commande `apachectl restart`, puis
regarder les changements au niveau de `top`.

Question : qu'a fait `apachectl restart` ?

De nombreuses distributions Linux utilisent *systemd* comme système d'init.
Entre autres, *systemd* permet de gérer le lancement et l'activation au
démarrage de démons, appelés services. Refaire les actions précédentes en
utilisant `systemctl start|stop|restart apache2`. En plus de `top`, utiliser la
commande `systemctl status apache2` pour voir l'état de lancement d'Apache.

Question : en utilisant `systemctl status apache2`, est-ce qu'Apache est lancé
au démarrage du système ?

Redémarrer server11 et s'assurer qu'Apache a bien démarré sans action.
Utiliser ensuite `systemctl disable apache2` puis redémarrer, s'assurer
qu'Apache n'est pas lancé. Ré-activer Apache (`systemctl enable apache2`) et le
démarrer.

## Étape 2 : déposer du contenu

Si ce n'est pas fait, lire la documentation de la page par défaut disponible à
l'URL http://10.13.37.11. Se connecter en ssh sur server11, et passer root.
se rendre dans `/var/www/html/`, et y créer un fichier `page2.html`. Insérer du
texte ou du code HTML dans ce fichier, peu importe tant que c'est visible.
Visiter l'URL http://10.13.37.11/page2.html et admirer le résultat.

Question : à quel utilisateur appartient `page2.html`, et quels sont les droits
dessus ?

Changer les droits sur `page2.html` de la façon suivante : `chmod 000
page2.html`. Tenter à nouveau d'accéder à la page. Rechercher ensuite les
droits minimum pour que la page soit de nouveau accessible.

Question : quels sont les droits minimum pour accéder à `page2.html` depuis le
serveur web ? En owner root ainsi qu'en owner www-data.

## Étape 3 : personnaliser l'emplacement du contenu

Par défaut, le contenu se trouve donc dans `/var/www/html`. Mais il est
possible de modifier cet emplacement. Se rendre dans
`/etc/apache2/sites-enabled/` et lire le fichier de configuration
`000-default.conf`.

Question : quelle est la directive qui permet de paramétrer l'emplacement du
contenu ?

Créer un répertoire `/var/www/html2` puis ajouter à cet endroit un fichier
`index2.html` au contenu libre. Modifier la directive `DocumentRoot` pour y
utiliser `/var/www/html2`, puis redémarrer Apache. Rafraîchir
http://10.13.37.11 .

Question : qu'observe-t'on ?

Ajouter au fichier `000-default.conf` sous la directive `DocumentRoot` la
directive `DirectoryIndex index.html index2.html`. Relancer Apache.

Question : que fait la directive `DirectoryIndex` ?

Créer un répertoire `/srv/www/html` et y créer un fichier `index.html` au
contenu libre, puis modifier à nouveau la directive `DocumentRoot` pour y
insérer ce nouveau répertoire. Relancer Apache et rafraîchir la page dans le
navigateur.

Question : quel est le résultat ?

Éditer le fichier de configuration `/etc/apache2/apache2.conf` et retrouver la
directive ouvrante `<Directory /var/www>`. En dessous se trouve une directive
similaire pour `/srv/`. Décommenter les directives et relancer Apache.

## Étape 4 : les fichiers de logs (journaux)

De nombreuses requêtes HTTP ont été réalisées dans les étapes précédentes,
certaines avec succès, d'autres générant une erreur. Il peut être utile de voir
ce qui se passe bien et ce qui se passe moins bien. Apache2 permet entre autres
d'enregistrer des informations sur les requêtes dans des fichiers dits
journaux, ou log en anglais. Par convention, sous Unix, les logs se trouvent
dans le répertoire `/var/log`, et sous Debian, Apache stocke ses logs dans
`/var/log/apache2` par défaut. Tout ceci est bien entendu configurable.

Repérer dans `/etc/apache2/apache2.conf` la directive `ErrorLog`. Retrouver le
fichier correspondant dans `/var/log/apache2`.

Repérer dans `/etc/apache2/sites-enabled/000-default.conf` les directives
`ErrorLog` et `CustomLog`, et retrouver les fichiers correspondants dans
`/var/log/apache2`. 

Questions :
- quelles informations trouve-t-on dans `/var/log/apache2/access.log` ?
- quelles informations trouve-t-on dans `/var/log/apache2/error.log` ?

Depuis un terminal, lancer la commande `tail -f /var/log/apache2/error.log` puis dans
un autre terminal, en tant que root, lancer la commande `apachectl restart`.

Question : qu'observe-t-on ? 

Couper `tail`, puis lancer la commande `tail -f /var/log/apache2/access.log` puis
utiliser son navigateur pour visiter le site web, et tenter d'accéder à des
pages inexistantes.

Question : qu'observe-t-on ?
