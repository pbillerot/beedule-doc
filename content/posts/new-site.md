---
title: "Déclaration d'un site Hugo"
date: 2021-01-20
draft: false
#categories:
tags:
- technique
cover: "/media/hugo.png"
style: bee-doc
#menu:
#  page:
#    parent: technique
#    weight: 20
---
<!--more-->
{{< toc >}}

## Préalable

Le site Hugo devra être créé par les procédures décrites dans la documentation en ligne [Quick Start](https://gohugo.io/getting-started/quick-start/).

Vous pouvez aussi regarder la documentation d'installation du thème [beedream](https://www.billerot.eu/beedream/site/installation/).

En résultat vous devriez avoir une structure qui ressemble à cela :

```
beefree/              (répertoire racine du site)
├── archetypes
│   └── default.md    (le modèle des documents)
├── config.toml       (fichier de configuration et de personnalisation du site)
│   ou config.yaml
├── content           (le contenu du site)
│   ├── rep-1
│   │   └── doc-1.md 
│   └── rep-2
│       └── doc-2.md 
├── data              (données éventuelles)
├── layouts           (modèles de présentation de certaines pages)
├── data              (données éventuelles)
├── resources         (fichiers de resources)
│   └── _gen
│       ├── assets
│       └── images
├── static            (fichiers css js image)
└── themes            (répertoire des thèmes)
    └── beedream
```

## Déclaration du site Hugo

La déclaration se fera dans le fichier de configuration 

### hugo.yaml

```yaml
# Liste des webapp victor à gérer
hugoapp:
  # nom du site
  - name: beefree
    # titre du site dans l'application Victor (et nom pas le titre du site Hugo)
    title: "Beefree Admin"
    # répertoire racine du répertoire hugo
    folder: "/volshare/beefree"
    # chemin d'accès du site en PRODUCTION
    baseurl: "/beefree/" # ou "/"
    # theme du site Hugo
    theme: "beedream"
    # aide en ligne du thème
    themehelp: "https://www.billerot.eu/beedream/"
    # script de déploiement de la production sur un autre site (commandes ftp, rsync ...)
    deploy: "/volshare/docker/beefree/deploy.sh"
    # Libellé dans Victor du menu de lancement du déploiement
    deploylabel: "Deployer la production sur le site OVH..."

  # autre site...
  - name: hyde
    title: "Hyde Admin"
    folder: "/volshare/hyde"
    baseurl: "/hyde/"
    theme: "hyde"
    themehelp: "https://github.com/spf13/hyde"
```

## Les fichiers de configurations de Hugo

Le fichier de configuration du site est localisé directement sous la racine. 

```
racine
├── content
│   ...
│   └── site
│       └── config.yaml -> ../../config.yaml
├── config.yaml
    
```
Il peut être intéressant de donner la possiblité de le modifier directement dans l'interface Victor.

Pour se faire, vous pouvez créer un lien symbolique par exemple sous `content/site/config.yaml` qui pointerai sur le fichier de config du site.

```shell
mkdir content/site
cd content/site
ln -s ../../config.yaml config.yaml
```
{{< image image="/media/lien-config.png" >}}

## Script de déploiement

Si le site de développement n'est pas sur le site de déploiement, l'administrateur technique devra écrire un script de déploiement.

Lors des étapes de **test**, Victor pilote Hugo pour générer les fichiers dans le répertoire `private`. Ce répertoire est actualisé à chaque modification/ajout/suppression de fichiers dans `content`

La génération du site de **production** se fera dans le répertoire `public` (répertoire par défaut de Hugo).
```
racine
├── content
│   ...
├── private
│   ... fichiers du site de test 
├── public
│   ... fichiers du site de production
├── config.yaml
    
```

Le script de déploiement devra par conséquent recopier le répertoire `public` sur le site de déploiement.

Lors de l'exécution du script Victor se positionne sous le répertoire racine de site Hugo. 

Exemple de script en utilisant le script lftp

```shell
#!/bin/bash

echo Deploiement en cours...

lftp sftp://user:password@site-production.com -e "mirror --delete --reverse public /www/beefree; quit"

echo Deploiement end..

```

