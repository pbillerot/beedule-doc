---
title: "Résumé des commandes de l'Éditeur de texte"
date: 2021-03-20
draft: false
categories:
tags:
- editeur
cover: "/media/markdown.jpg"
style: bee-doc
#menu:
#  page:
#    weight: 30
---
<!--more-->
### Touches fonctions

{{< texte rouge >}}**ctrl+s**{{< /texte >}} : sauvegarde du document  
{{< texte rouge >}}**ctrl+/**{{< /texte >}} : mise en commentaire de la méta-données  
{{< texte rouge >}}**ctrl+espace**{{< /texte >}} : liste et complétion des shortcodes  

### Méta-données
```yaml
title: Exemple page
draft: false 
date: 2020-12-26 
tags: 
- contenu
cover: /media/image.jpg

categories: 
- news
publishDate: 2021-02-10 
expiryDate: 2021-05-01  
_build:
  list: false
style: bee-doc bee-chanson
menu:
  page:
    weight: 10
    parent: shortcode
```

### Shortcode

```
{{</*bouton [icone="nom icone"] [image="image"] titre="titre" [lien="lien"]*/>}}

{{</*cartes taille="s m l xl"*/>}}

{{</*carte image="image" [diapo="diapo"] [titre="titre"] [lien="lien"] [pdf="pdf"] [taille="s m l xl"]*/>}}  
texte...  
{{</*/carte*/>}}

{{</*/cartes*/>}}

{{</*colonnes [two three]*/>}}...<--->...{{</*/colonnes*/>}}

{{</*centre*/>}}...{{</*/centre*/>}}

{{</*diaporama*/>}}

{{</*galerie [sous-repertoire]*/>}}

{{</*icone nomIcone*/>}}

{{</*image image="/..." [lien="lien"] [position="gauche droite"] [taille="s m l xl]" [forme="ronde"]*/>}}

{{</*label label="libellé" [icone="nomIcone"] [lien="lien"]*/>}}

{{</*message [info success warning error]*/>}}message{{</*/message*/>}}

{{</*player "lien_vers_audio" ["titre"] ["boucle"]*/>}}

{{</* players répertoire*/>}}

{{</*repertoire "répertoire"*/>}}

{{</*toc*/>}}

{{</*texte [xs s m l xl] [rouge orange vert cyan bleu pourpre violet marron gris]*/>}}...{{</*/texte*/>}}
```
### Markdown

```markdown
# Titre 1
## Titre 2
### Titre 3
#### Titre 4
##### Titre 5
###### Titre 6

**gras**
_soulignement_
~~barré~~

Liste:
- non ordonnée
- non ordonnée
  - sous-liste non ordonée

1. numérotée
1. numérotée
   1 sous-liste numérotée

[Je suis un lien](https://www.google.com)

![][/image]

| Tables         | Sont            | Frais  |
| -------------- |:---------------:| ------:|
| col 3 est      | aligné à droite | 1600 € |
| col 2 est      | centré          |   12 € |
| lignes zébrées | harmonieuses    |    1 € |

>citation

ligne horizontale
---

```