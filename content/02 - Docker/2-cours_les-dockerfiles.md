---
title: Conteneurs Docker
---

# Conteneurs Docker

## _Modularisez et maîtrisez vos applications_

---

# Images et conteneurs

---

# Créer une image en utilisant un Dockerfile

<!-- A faire avec CodeWave ? -->

- ## Jusqu'ici nous avons utilisé des images toutes prêtes.

- ## Une des fonctionnalités principales de Docker est de pouvoir facilement construire des images à partir d'un simple fichier texte : **le Dockerfile**.

- Le **Dockerfile** est un fichier procédural qui permet de décrire l'installation d'un logiciel (la configuration d'un container) en enchaînant des instructions Dockerfile (en MAJUSCULE).

- Exemple:

```Dockerfile
FROM debian:stretch
RUN apt-get update && apt-get install -y cowsay fortune
ENTRYPOINT["/usr/games/cowsay"]
```

---

## Instruction `FROM`

- L'image de base à partir de laquelle est construite l'image actuelle.

## Instruction `RUN`

- Permet de lancer une commande shell (installation, configuration).

## Instruction `ADD`

- Permet d'ajouter des fichier depuis le contexte de build à l'intérieur du conteneur.
- Généralement utilisé pour ajouter le code du logiciel en cours de développement et sa configuration au conteneur.

---

## Instruction CMD

- Généralement à la fin du `Dockerfile` : elle permet de préciser la commande par défaut lancée à la création d'une instance du conteneur avec `docker run`. on l'utilise avec une liste de paramètres

```Dockerfile
CMD ["echo 'Conteneur démarré'"]
```

--

## Instruction ENTRYPOINT

- Précise le programme de base avec lequel sera lancé la commande

```Dockerfile
ENTRYPOINT ["/usr/bin/python3"]
```

---

## Instruction ENV

- Une façon recommandée de configurer vos applications Docker est d'utiliser les variables d'environnement UNIX, ce qui permet une configuration "au _runtime_".

## Documentation

- Il existe de nombreuses autres instructions possibles très clairement décrites dans la documentation officielle : [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)

---

# Lancer la construction

- La commande pour lancer la construction d'une image est :

```bash
docker build [-t <tag:version>] [-f <chemin_du_dockerfile>] <contexte_de_construction>
```

--

- ## Lors de la construction, Docker télécharge l'image de base. On constate plusieurs téléchargements en parallèle.

- ## Il lance ensuite la séquence des instructions du Dockerfile.

- Observez l'historique de construction de l'image avec `docker image history <image>`

- Il lance ensuite la série d'instructions du Dockerfile et indique un hash pour chaque étape. Pourquoi ?

---

# Les layers et la mise en cache

- ## Docker construit les images comme une série de "couches" de fichiers successives.

- On parle d'**Union Filesystem** car chaque couche (de fichiers) écrase la précédente.

--

<!-- In order to understand the relationship between images and containers, we need to explain a key piece of technology that enables Docker—the UFS (sometimes simply called a union mount). Union file systems allow multiple file systems to be overlaid, appearing to the user as a single filesytem. Folders may contain files from multiple filesystems, but if two files have the exact same path, the last mounted file will hide any previous files. Docker supports several different UFS implentations, including AUFS, Overlay, devicemapper, BTRFS, and ZFS. Which implementation is used is system dependent and can be checked by running docker info where it is listed under “Storage Driver.” It is possible to change the filesystem, but this is only recom‐ mended if you know what you are doing and are aware of the advantages and disad‐ vantages.
Docker images are made up of multiple layers. Each of these layers is a read-only fil‐ eystem. A layer is created for each instruction in a Dockerfile and sits on top of the previous layers. When an image is turned into a container (from a docker run or docker create command), the Docker engine takes the image and adds a read-write filesystem on top (as well as initializing various settings such as the IP address, name, ID, and resource limits). -->

- ## Chaque couche correspond à une instruction du Dockerfile.

- ## `docker image history <conteneur>` permet d'afficher les layers, leur date de construction et taille respectives.

- ## Ce principe est au coeur de l'**immutabilité** des images Docker.

- ## Au lancement d'un container, le Docker Engine rajoute une nouvelle couche de filesystem "normal" read/write par dessus la pile des couches de l'image.

- `docker diff <container>` permet d'observer les changements apportés au conteneur depuis le lancement.

---

# Optimiser la création d'images

- Les images Docker ont souvent une taille de plusieurs centaines de **mégaoctets** voire parfois **gigaoctets**. `docker image ls` permet de voir la taille des images.
- Or, on construit souvent plusieurs dizaines de versions d'une application par jour (souvent automatiquement sur les serveurs d'intégration continue).

  - L'espace disque devient alors un sérieux problème.

- Le principe de Docker est justement d'avoir des images légères car on va créer beaucoup de conteneurs (un par instance d'application/service).

- De plus on télécharge souvent les images depuis un registry, ce qui consomme de la bande passante.

> La principale **bonne pratique** dans la construction d'images est de **limiter leur taille au maximum**.

---

# Limiter la taille d'une image

- Choisir une image Linux de base **minimale**:

  - Une image `ubuntu` complète pèse déjà presque une soixantaine de mégaoctets.
  - mais une image trop rudimentaire (`busybox`) est difficile à débugger et peu bloquer pour certaines tâches à cause de binaires ou de bibliothèques logicielles qui manquent (compilation par exemple).
  - Souvent on utilise des images de base construites à partir de `alpine` qui est un bon compromis (6 mégaoctets seulement et un gestionnaire de paquets `apk`).
  - ## Par exemple `python3` est fourni en version `python:alpine` (99 Mo), `python:3-slim` (179 Mo) et `python:latest` (918 Mo).

- Limiter le nombre de commandes de modification du conteneur :
  - `RUN`, `ADD` et toute commande impliquant une modification du système de fichier du conteneur va créer un nouveau layer dans l'image.
    - Souvent on enchaîne les commandes en une seule pour économiser des couches.
    - Il existe une limite du nombre de couches maximum par image (42 layers).

---

# Publier des images vers un registry privé

- Généralement les images spécifiques produites par une entreprise n'ont pas vocation à finir dans un dépôt public.

- On peut installer des **registry privés**.

- On utilise alors `docker login <adresse_repo>` pour se logger au registry et le nom du registry dans les `tags` de l'image.

- Plusieurs options:
  - **Gitlab** fournit un registry très intéressant car intégré dans leur workflow DevOps.
  - **Docker Trusted Registry (DTR)** fait partie de **Docker Enterprise** et pratique des tests de sécurité sur les images.

---

# Créer des conteneurs personnalisés

- ## Il n'est pas nécessaire de partir d'une image Linux vierge pour construire un conteneur.

- ## On peut utiliser la directive `FROM` avec n'importe quelle image.

- De nombreuses applications peuvent être configurées en étendant une image officielle
- ## _Exemple : une image Wordpress déjà adaptée à des besoins spécifiques._

- L'intérêt ensuite est que l'image est disponible préconfigurée pour construire ou mettre à jour une infrastructure, ou lancer plusieurs instances (plusieurs containers) à partir de cette image.

--

- C'est grâce à cette fonctionnalité que Docker peu être considéré comme un outil d'_infrastructure as code_.

--

- On peut également prendre une sorte de snapshot du conteneur (de son système de fichiers, pas des processus en train de tourner) sous forme d'image avec `docker commit <image>` et `docker push`.

---