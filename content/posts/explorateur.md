---
title: "L'explorateur du contenu"
date: 2021-03-30
draft: false
categories:
tags:
- explorateur
cover: "/media/page-site.png"
style: bee-doc
#menu:
#  page:
#    weight: 20
---

<!--more-->
{{< diaporama >}}
{{< toc >}}

{{< image image="/media/page-site.png" >}}

## Un explorateur de fichiers
{{< colonnes >}}

**Victor** est tout simplement un explorateur de fichiers
qui permet de créer/visualiser/modifier/supprimer les fichiers et sous-répertoires du répertoire `content` du site
<--->
```
content
├── media
│   ├── agile.jpg
│   ├── bee.svg
│   ├── bg-bee.svg
│   └── icone-bee.svg
├── posts
│   └── premier.md
└── site
    └── config.yaml -> ../../config.yaml
```
{{< /colonnes >}}

l'action pour changer de répertoire ou visualiser/modifier un fichier se fera par un **simple clic**.

Dans un répertoire, {{< icone file plus >}} pour créer un **nouveau fichier**, {{< icone folder outline >}} pour **créer un sous-répertoire**, {{< icone upload >}} pour **déverser des fichiers** (_à noter que le glisser-déposer à partir de votre explorateur de fichier local est possible_)

Pour manipuler un élément, il faudra effectuer **un clic long** et faire apparaître les boutons d'action sur l'élément sélectionné : {{< icone pencil alternate >}} **renommer** / {{< icone copy outline >}}**copier** / {{< icone arrow right >}}**déplacer** / {{< icone trash alternate outline >}} **supprimer** / {{< icone file download >}} **télécharger**.

Exemple ci-après sur `bee.svg`

{{< image image="/media/clic-long.png" >}}
<br/>
{{< colonnes >}}
Le bouton {{< icone check >}} va permettre de passer en mode multiple pour sélectionner plusieurs éléments et réaliser des opérations groupées.

Le bouton {{< icone eye blue >}} pour visualiser le site de test.
<--->
{{< image image="/media/site-test.png" >}}
{{< /colonnes >}}

## États des documents

4 états pour un document :
- le document est **en ligne**
- le document sera **prochainement en ligne** (la date de publication est marquée en rouge) 
- le document est **en brouillon** (une icône rouge derrière le titre du document)
- le document est **expiré** (la date d'expiration est marquée en rouge) (À noter que la date d'expiration apparaît aussi en bas du document visualisé sur le site de test - voir l'image ci-dessus)

{{< image image="/media/apercu-doc.png" >}}

