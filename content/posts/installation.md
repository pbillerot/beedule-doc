---
title: "Installation de Victor"
date: 2021-01-30
draft: false
#categories:
tags:
- technique
cover: "/media/parametrage.jpg"
style: bee-doc
#menu:
#  page:
#    parent: technique
#    weight: 10
---
*Guide pour l'administrateur technique*
<!--more-->
{{< toc >}}
{{< diaporama >}}

Je vous propose d'installer une plateforme complète pour héberger notre application **Victor**.

Nous utiliserons une VM (Machine Virtuelle) [DEBIAN 10](https://fr.wikipedia.org/wiki/Debian) avec le gestionnaire de conteneur [Docker](https://fr.wikipedia.org/wiki/Docker_(logiciel)) installé.

Ce document ne décrit pas l'installation d'une VM Debian et de Docker.  
Pour ma part je loue une **VPS Debian 10 Docker** chez l'hébergeur [OVH](https://www.ovhcloud.com/fr/vps/) (1 vCore 2 Go 20 Go 1 domaine.eu pour 46.16 €/an)

## Prérequis du système hôte

Système : `Debian Buster`
```shell
>uname -a
Linux vps-7d2d773f 4.19.0-16-cloud-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64 GNU/Linux
```
Gestionnaire de conteneur Docker :
```shell
>docker version
Client: Docker Engine - Community
 Version:           20.10.5
 API version:       1.41
 Go version:        go1.13.15
 ...
 
Server: Docker Engine - Community
 Engine:
  Version:          20.10.5
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  ...
```

```shell
>docker-compose version
docker-compose version 1.21.0, build unknown
docker-py version: 3.4.1
CPython version: 3.7.3
OpenSSL version: OpenSSL 1.1.1d  10 Sep 2019
```


## La plateforme Docker

{{< image image="/media/docker.png" >}}

Notre plateforme sera composée de 4 containers :

- [Caddy Server](https://caddyserver.com/docs/) le frontal web, c'est l'élément le plus important. Il sera chargé :  
   - de contrôler le trafic http (:80) et https (:443)
   - de renouveller le certificat lié au nom de domaine
   - de gérer les authentifications pour certaines [URI](https://fr.wikipedia.org/wiki/Uniform_Resource_Identifier)
   - de rediriger les flux vers les autres containers en fonction des URI
   - de journaliser les accès et les erreurs
- **Victor** le container qui va administrer et servir les web-applications **Hugo**

Pour plus de confort, j'utilise  
- [Portainer](https://korben.info/portainer-io-un-outil-graphique-pour-gerer-vos-environnements-docker-en-toute-securite.html) pour gérer graphiquement l'environnement Docker  
- [Filebrowser](https://filebrowser.org/features) pour manipuler les fichiers du répertoire partagé (volshare)

Les 4 containers ont accès à la même ressource de fichiers `volshare` et les échanges entre **Caddy Server** et les autres containers se font à travers le réseau privé `web`. Ces containers ne sont pas  accessibles de l'extérieur.

La configuration de **Docker** se fait à travers le fichier `/volshare/docker/docker-compose.yaml`, 
**Caddy Server** via `/volshare/docker/caddy/caddyfile.conf`

Nous allons détailler tout cela ci-aprés.

## Volume partagé /volshare

`/volshare` est le répertoire partagé entre tous les containers.

Il aura la structure suivante :
```
/volshare
  /logs
    access.log access.0.log ... access.9.log
  /etc
    (les certificats du domaine)
  /bivouac
    (le site web Hugo administré par Victor)
  /filebrowser
    database.db
  /data (le répertoire des données à sauvegarder)
    /store
      (le répertoire des fichiers statiques servi par Caddy)
    /bivouac
      bivouac_content --> ../../bivouac/content (lien symbolique)
  /docker (les fichiers de configuration des containers)
    docker-compose.yaml
    /caddy
      caddyfile.conf
    /filebrowser
      filebrowser.conf
    /victor
      dockerfile
      hugo.yaml
    /bivouac
      deploy.sh (script de déploiement sur un autre site)
```

## Container Filebrowser

{{< image image="/media/filebrowser.gif" taille="m" position="droite" >}}

[Filebrowser](https://filebrowser.org/features) est un explorateur de fichiers en ligne.

Nous allons le régler pour explorer le répertoire `/volshare` du système hôte via l'url `/fb`

### /volshare/docker/docker-compose.yaml

```yaml
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    restart: unless-stopped
    volumes:
    - /volshare:/srv
    - /volshare/filebrowser/database.db:/database.db
    - ./filebrowser/filebrowser.json:/.filebrowser.json    
    networks:
    - web
```

### /volshare/docker/filebrowser/filebrowser.json

```json
{
  "port": 80,
  "baseURL": "/fb",
  "address": "",
  "log": "stdout",
  "database": "/database.db",
  "root": "/srv",
}
```

### /volshare/docker/caddy/caddyfile.conf

```shell
# filebrowser /fb
redir /fb /fb/
reverse_proxy /fb/* filebrowser:80
```

## Container Portainer

{{< image image="/media/portainer.png" taille="m" position="droite" >}}

[Portainer](https://korben.info/portainer-io-un-outil-graphique-pour-gerer-vos-environnements-docker-en-toute-securite.html) via son interface web permet de visualiser les ressources de la plateforme Docker, les containers, les images, les volumes, le réseau.

### /volshare/docker/docker-compose.yaml

```yaml
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    command: -H unix:///var/run/docker.sock
    restart: unless-stopped
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    networks:
    - web
```

### /volshare/docker/caddy/caddyfile.conf

```shell
# portainer 
# on supprime le préfix /portainer après le routage
redir /portainer /portainer/
route /portainer/* {
    uri strip_prefix /portainer
    reverse_proxy portainer:9000
}
```

## Container Victor

{{< image image="/media/page-site.png" position="droite" taille="m" >}}

Ce container va héberger l'application **Victor** libre d'utilisation sur [Github](https://github.com/pbillerot/victor).

**Victor** a été écrit dans le langage [Golang](https://fr.wikipedia.org/wiki/Go_(langage)).

_À noter que les 4 containers et Docker lui-même ont été développés en Golang_

### /volshare/docker/docker-compose.yaml

```yaml
  victor:
    build:
      context: victor
    image: victor
    container_name: victor
    restart: unless-stopped
    user: 1000:1000
    volumes:
    - /volshare:/volshare
    - ./victor/hugo.yaml:/src/victor/conf/hugo.yaml
    networks:
    - web
```

### /volshare/docker/victor/hugo.yaml

```yaml
# Liste des webapp victor à gérer
hugoapp:
  - name: bivouac
    title: "Bivouac Admin"
    folder: "/volshare/bivouac"
    baseurl: "/bivouac/"
    theme: "beedream"
    themehelp: "https://pbillerot.freeboxos.fr/beedream/"
    deploy: "/volshare/docker/bivouac/deploy.sh
```

### /volshare/docker/caddy/caddyfile.conf

```shell
# victor
# qui va servir les webapp hugo décrites dans hugo.yaml
reverse_proxy /* victor:8080

# PROTECTION par mot de passe pour les uri /victor /hugo
# user: admin password: admin (qui a été haché par la commande ci-dessous)
# docker container exec -it caddy /bin/sh
# puis
# caddy hash-password [--plaintext <password>]

basicauth /victor/* {
    admin JDJhJDE0JGRKNmt6a3g5L1BlSXRmbmVWV2RXeU9Lc2NlZzFGUnV6eHFyYlVYOUFGc3FmL3NsSS5zRXdt
}
basicauth /hugo/* {
    admin JDJhJDE0JGRKNmt6a3g5L1BlSXRmbmVWV2RXeU9Lc2NlZzFGUnV6eHFyYlVYOUFGc3FmL3NsSS5zRXdt
}

```

## Image Victor

Ci-après le script qui permet de construire l'image Victor.

_À noter que ce container intègre le moteur **Hugo** et des utilitaires **Git** **SSH** et **Ftp** pour déployer les sites sur des serveurs externes_

### /volshare/docker/victor/dockerfile

```dockerfile
# IMAGE VICTOR

# ETAPE COMPILATION
# Le GOPATH par défaut de cette image est /go.
FROM golang:alpine as goalpine

# Installation de GCC et GIT
RUN apk add build-base git

# Installation de victor
WORKDIR /src
RUN git clone https://github.com/pbillerot/victor.git

WORKDIR /src/victor

# Build avec CGO
RUN GOOS=linux GOARCH=amd64 go build -a -installsuffix cgo -o /src/victor/victor

# ETAPE GENERATION D'UN IMAGE 
# avec le projet
FROM alpine

# Installation de victor...
RUN mkdir -p /src/victor
copy --from=goalpine /src/victor /src/victor

# Uptade OS + hugo + utilitaires
RUN apk add --update nano openssh-client hugo git rsync sshpass lftp

# Ajout du user 1000 afin de partager les fichiers avec le user du serveur hôte
RUN addgroup -g 1000 -S vigo && \
    adduser -u 1000 -S vigo -G vigo

# Point d'entrée
WORKDIR /src/victor
ENTRYPOINT ./victor

# Le port sur lequel notre serveur écoute
EXPOSE 8080
```

## Container Caddy

### /volshare/docker/docker-compose.yaml

Version complète

_À seul ce container présente des ports accessible de l'extérieur (ports 80 et 443)_

```yaml
version: "3.3"
services:

  caddy:
    # https://hub.docker.com/_/caddy?tab=description
    image: caddy:latest
    container_name: caddy
    restart: unless-stopped
    ports:
    - '80:80'
    - '443:443'
    volumes:
    - './caddy/caddyfile.conf:/etc/caddy/Caddyfile'
    - '/volshare/etc:/data'
    - '/volshare/data/store:/srv'
    - '/volshare:/volshare'
    networks:
    - web
    
    filebrowser:
    ...
    
    portainer:
    ...
    
    victor:
    ...

volumes:
  certs:

networks:
  web:
    driver: bridge

```

### /volshare/docker/caddy/caddyfile.conf

Version complète

Dans ce fichier on trouvera :
- l'`email` qui sera utilisé pour demander le certificat SSL du domaine
- le nom du domaine du site
- éventuellement la liste de d'adresses IP indésirables
- les répertoires servis par l'explorateur web intégré à Caddy Server
- les fichiers logs 
- et enfin les différents `reverse_proxy` redirigés vers nos containers (qui ont été détaillés plus haut)

```shell
# Configuration du serveur Caddy
# https://caddyserver.com/docs/

# GLOBAL option
# https://www.ssllabs.com/ssltest/analyze.html?d=mon.domaine.com
{
    email mon.email@gmail.com
}

# HOST
mon.domaine.com

# blacklist - sites indésirables
@blaklist {
    remote_ip 94.130.212.180 134.119.20.10
}
handle @blaklist {
    respond "Refused!" 403
}

# Serveur de fichiers statiques
redir /store /store/
handle_path /store/* {
    root * /volshare/data/store
    file_server browse
}

# Log du trafic (rotation automatique tous les 100 Mo (10 logs) 
log {
    output file /volshare/log/access.log
    format single_field common_log
}

# filebrowser
# ...
# portainer
# ...
# victor
# ...

```

## Procédure d'installation

```shell
cd /volshare/docker
# création/mise à jour des containers avec reconstruction des images
docker-compose up -d --build 
# creation/mise à jour des containers sans reconstruction des images
docker-compose up -d
# nettoyage des images intermédiaires
docker image prune -f
```

## Démarrage de Victor

{{< image image="/media/page-site.png" position="droite" taille="m" >}}

Dans votre navigateur préféré taper l'URL :

[https://mon.domaine.fr/victor](https://mon.domaine.fr/victor)

puis renseigner le code utilisateur et son mot de passe que vous avez configuré

et vous devriez avoir l'écran d'accueil de Victor :

