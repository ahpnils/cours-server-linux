[Retour au sommaire](../../README.md)

# TP 5 : Installation et paramétrage basique du serveur web Nginx

Objectifs :

- installer le serveur HTTP Nginx ;
- le démarrer, l'arrêter et l'activer au démarrage du serveur ;
- déposer du contenu et le rendre accessible ;
- personnaliser l'emplacement du contenu ;
- accéder aux fichiers de logs.

## Introduction

Nginx est un serveur HTTP, devenu il y a quelques années une référence côté
performances, car il fut le premier à résoudre le [problème des dix mille
connexions simultanées](https://fr.wikipedia.org/wiki/C10k_problem).

Nginx est aussi capable d'agir comme mandataire inverse (reverse proxy en
anglais) pour HTTP(S), et pour certains protocoles de mail. Cela veut dire
qu'il se place en frontal d'un ou de plusieurs serveurs web existant (Nginx
aussi, ou d'autres comme Apache HTTP Server), et permet de mettre en place
différentes stratégies, parmi lesquelles :

- rediriger du trafic vers un serveur ou en local ;
- mettre en cache pour servir du contenu plus rapidement ;
- effectuer de la répartition de charge entre plusieurs serveurs identiques ;
- effectuer un filtrage de sécurité.

À noter que Nginx n'est pas développé par une fondation mais par une entreprise. 
Le logiciel d'origine est libre, sous licence BSD à 2 clauses, mais il existe 
une version commerciale, Nginx Plus, disposant de fonctions supplémentaire et
d'un support.

## Étape 0 : installation

Comme bon nombre de distributions Linux et BSD, Nginx est disponible via des
paquets binaires pré-compilés pour Debian. 

Se connecter sur server11, puis passer root.
Avant l'installaton, regarder quels sont les services réseau en fonctionnement
: `lsof -n -i`. Seul sshd devrait apparaître. Maintenant, installer Nginx :
`apt -y install nginx`. Relancer la commande d'avant.

Question : est-ce que l'installation de Nginx a lancé celui-ci ?

Se rendre via le navigateur de l'hôte sur http://10.13.37.11. Il est aussi
possible, depuis server11 ou depuis l'hôte, d'utiliser un requêteur HTTP en
ligne de commande comme `curl`. 

Vérifier la liste des paquets de Nginx installés via `dpkg --list | grep
nginx`.

Question : quels sont les paquets installés ?

## Étape 1 : démarrage, arrêt, et activation au démarrage

Se connecter en ssh sur server11, puis passer root.
Vérifier grâce à la commande `pgrep nginx` si Nginx est toujours en fonctionnement.

Questions : 
- comment s'appelle le programme ?
- combien de processus correspondants sont en fonctionnement ?
- à quels utilisateurs appartiennent ces processus ?

De nombreuses distributions Linux utilisent *systemd* comme système d'init.
Entre autres, *systemd* permet de gérer le lancement et l'activation au
démarrage de démons, appelés services.

Lancer la commande `systemctl stop nginx`, puis vérifier via `pgrep nginx` que
celui-ci est bien arrêté. Lancer ensuite la commande `systemctl start nginx`,
et vérifier de la même façon que Nginx fonctionne. Ouvrir un deuxième terminal,
se connecter sur server11, puis passer root. Lancer la commande `top -u
www-data`. Depuis le premier terminal, lancer la commande `systemctl restart
nginx`, puis regarder les changements au niveau de `top`.

Question : en utilisant `systemctl status nginx`, est-ce que Nginx est lancé
au démarrage du système ?

Redémarrer server11 et s'assurer que Nginx a bien démarré sans action.
Utiliser ensuite `systemctl disable nginx` puis redémarrer, s'assurer
qu'il n'est pas lancé. Ré-activer Nginx (`systemctl enable nginx`) et le
démarrer.

## Étape 2 : déposer du contenu

Se connecter sur [http://10.13.37.11](http://10.13.37.11), et remarquer la page
d'accueil par défaut. Se connecter en ssh sur server11, et passer root.
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
`/etc/nginx/sites-enabled/` et lire le fichier de configuration
`default`.

Question : quelle est la directive qui permet de paramétrer l'emplacement du
contenu ?

Créer un répertoire `/srv/www/html` puis ajouter à cet endroit un fichier
`index2.html` au contenu libre. Modifier la directive `root` pour y
utiliser `/srv/www/html`, puis redémarrer Nginx. Rafraîchir
http://10.13.37.11 .

Question : qu'observe-t'on ?

Ajouter au fichier `default`, dans la directive `index` l'option `index2.html`, avant tous les autes fichiers. Relancer Nginx.

Question : que fait la directive `index` ?

Astuce : il est possible de vérifier la validité de la syntaxe des fichiers de
configuration de Nginx avec la commande `nginx -t` en tant que root.

## Étape 4 : les fichiers de logs (journaux)

De nombreuses requêtes HTTP ont été réalisées dans les étapes précédentes,
certaines avec succès, d'autres générant une erreur. Il peut être utile de voir
ce qui se passe bien et ce qui se passe moins bien. Nginx permet entre autres
d'enregistrer des informations sur les requêtes dans des fichiers dits
journaux, ou log en anglais. Par convention, sous Unix, les logs se trouvent
dans le répertoire `/var/log`, et sous Debian, Nginx stocke ses logs dans
`/var/log/nginx` par défaut. Tout ceci est bien entendu configurable.

Repérer dans `/etc/nginx/nginx.conf` les directives
`error_log` et `access_log`, et retrouver les fichiers correspondants dans
`/var/log/nginx`. 

Questions :
- quelles informations trouve-t-on dans `/var/log/nginx/access.log` ?
- quelles informations trouve-t-on dans `/var/log/nginx/error.log` ?

Depuis un terminal, lancer la commande `tail -f /var/log/nginx/error.log` puis dans
un autre terminal, en tant que root, lancer la commande `systemctl restart
nginx`.

Question : qu'observe-t-on ? 

Couper `tail`, puis lancer la commande `tail -f /var/log/nginx/access.log` puis
utiliser son navigateur pour visiter le site web, et tenter d'accéder à des
pages inexistantes.

Question : qu'observe-t-on ?
