Compte-rendu TP docker

Première partie:
Installer Docker en suivant la doc
-Utilisation de diverses commandes afin de faire une installation "stable" et durable.
Nous n'avons pas réalisés l'étape qui est optionnelle.

Installer Docker:
sudo yum install docker
Et voilà, Docker est installé.
Il faut maintenant le lancer grâce à la commande de la doc:
- sudo systemctl start docker
Et pour vérifier que cela marche:
- sudo docker run hello-world
Et on voit ainsi le message suivant: "Hello from Docker".

Création d'un groupe Docker:
Il faut créer ce groupe dans le but de pouvoir utiliser Docker sans être root.
-groupadd docker, pour créer le groupe Docker qu'on utilisera plus tard.
-usermod -aG docker centos, pour ajouter un utilisateur dans ce groupe. Ici, le nom de notre utilisateur sera centos.
Il se trouve que nous n'avions pas créer d'utilisateur, donc pour en créer un, nous avons utilisés les commandes suivantes:
-useradd centos
-passwd centos, pour l'ajout d'un mot de passe.
Puis retaper la commande "usermod -aG docker centos" pour l'ajouter au groupe précédemment créé.

---------------------------------

Utilisation élémentaire:
1)Télécharger l'image ubuntu: docker pull ubuntu
2)Lancer l'image ubuntu: docker run ubuntu
3) Pour que le conteneur reste en vie: docker run -d -it ubuntu
Pour rentrer à l'intérieur du conteneur: docker exec -it "nom" bash

Pour arrêter un conteneur: docker stop "nom"
Pour que le conteneur apparaisse en faisant docker ps: docker run -d -it alpine
Pour changer le nom du conteneur: docker run --name "nom" -d -it ubuntu

1)Télécharger l'image ubuntu: docker pull alpine
2)Lancer l'image ubuntu: docker run alpine
3) Pour que le conteneur reste en vie: docker run -d -it alpine
Pour rentrer à l'intérieur du conteneur: docker exec -it "nom" sh

1)Lancer un processus: docker run -d ubuntu sleep 99999
2)Processus lancé dans le conteneur: ps -ef
3)Mettre en évidence des namespaces: ls /proc/3613/ns
4)Mettre en évidence les cgroups: systemd-cgls ou systemd-cgtop pour le live

System-nspawn:
1)Pour créer le dossier: mkdir alpine
2)Export des dossiers alpine vers le dossier fraichement créer + décompression: docker export $(docker create alpine) | tar -C alpine -xvf -
3)Lancer un conteneur en utilisant la commande systemd: systemd-nspawn -D ./alpine
4) Créer un répertoire temporaire avec tmpfs: systemd-nspawn --tmpfs=/home/test -D alpine
5)Isoler du réseau: systemd-nspawn --private-network -D alpine

---------------------------------

Avec runc:
1)Installer runc: yum install runc
2)Démarrer le service systemctl start docker
3)Créer un repertoire roots: Dans le /home, mkdir fs_alpine
4)On se rend dans le dossier précédemment créé: cd fs_alpine
5)Export des dossiers alpine vers le dossier fraichement créer + décompression: docker export $(docker create alpine) | tar -C rootfs -xvf -
6)Générer le fichier JSON: runc spec
Le fichier JSON est un fichier dans lequel on peut passer des paramètres tel que, sleep dans la partie "args" qui affecteront le conteneur auquel il est associé.

---------------------------------

Avec rkt:
1)Installer rkt: rpm -ivh https://github.com/rkt/rkt/releases/download/v1.30.0/rkt-1.30.0-1.x86_64.rpm
2)Lancer l'image alpine: rkt --stage1-name=coreos.com/rkt/stage1-fly:1.29.0 --insecure-options=image run docker://alpine, qui télécharge l'image

3)Expliquer : systemd-run rkt --insecure-options=image --stage1-name=coreos.com/rkt/stage1-fly:1.29.0  run docker://alpine --exec /bin/sleep -- 9999
Cette ligne de commande commence par son lancement grâce à rkt, run docker correspond à la machine que l'on veut monter, --exec /bin/sleep -- 9999 est là pour faire en sorte que la machine reste en état de "sleep" pour une durée de 9999 secondes.


4)Utiliser systemctl pour voir les services lancés: systemctl list-units --type=service

----------------------------------

Basic docker run:
1a)Changer le nom du conteneur: docker run --name "nom" -d -it alpine
1b)Changer le hostname: docker run --rm -h "example.com" -it ubuntu bash
2)Changer l'utilisateur qui lance un processus: docker run -it -d --user $(id -u)

4)Monter le répertoire /home de l'hôte dans le conteneur: docker run -it -v $(pwd):/ouais alpine sh, mais je n'ai pas les droits sur le dossier que l'on vient de créer dans le conteneur
NGINX:
1a)Récupérer l'image nginx: docker pull nginx
1b)Afficher la page d'accueil de niginx sur le port 8888: docker run --name nginx -d -p 8888:80 nginx

----------------------------------------

DOCKERFILE:
1)Une base debian pour le Dockerfile: FROM debian:jessie
2)Créer un dossier à la racine de notre conteneur:  COPY . /web
3)Créer un répertoire de travail dans le Dockerfile: WORKDIR /web
4)Déposer un fichier HTML sur le conteneur: on créé un fichier.html sur notre VM puis on le balance sur le conteneur via le Dockerfile, COPY . /web
5)Pouvoir changer le sleep de base: ENTRYPOINT ["bin/sleep"] et CMD 60
