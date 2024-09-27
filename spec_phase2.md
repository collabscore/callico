# Spécification de la phase 2

L'interface pour la phase 2 doit permettre de contrôler et si nécessaire de corriger les éléments du *contexte d'interprétation musicale* 
reconnu par l'OMR. Ces éléments sont

 - les **clefs** : elles permettent d'inférer le nom des notes selon leur position sur la portée
 - les **armures** : donne le nombre de dièses ou de bémols *par défaut*
 - la **métrique** (ou chiffrage de mesure) : indique la division en temps d'une mesure

Ces éléments sont représentés par des symboles affectant une portée spécifique. De plus, les armures et la métrique ne peuvent apparaître qu'en 
début de mesure. En revanche on peut changer de clé n'importe quand, voire plusieurs fois par mesure.

## Les données

Connaissant la reférence d'un opus (par exemple all:collabscore:saintsaens-ref:C006_0), obtient la liste des clés, armures et métriques sous la forme d'annotations liant l'identifiant de l'objet dans le MEI et la région sur l'image. Le modèle d'annotation est ``image-region``, et le concept d'annotation est ``symbol-region``.

Voici un exemple de l'URL d'appel pour l'opus all:collabscore:saintsaens-ref:C006_0.

https://neuma.huma-num.fr/rest/collections/all:collabscore:saintsaens-ref:C006_0/_annotations/image-region/symbol-region/

## Principe de l'interface

On va s'interdire d'ajouter ou de supprimer un symbole. Le but en effet est de corriger (si besoin) le résultat de l'OMR, mais pas
de se substituer à lui en développant des fonctionnalités d'édition de partition. 

L'interface doit permette de collection un ensemble d'éditions qui seront appliquées au résultat de l'OMR. Ces éditions sont de plusieurs
types:

  - remplacement d'une clé
  - remplacement d'une armure
  - remplacement d'une métrique
  - indication d'une erreur d'interprétation: un symbole a été considéré à tort comme étant un élément de contexte.

Par application des éditions au résultat "brut" de l'OMR, on obtient une partition corrigée des erreurs de l'OMR, à l'exception de
la non-reconnaissance d'un symbole de contexte (par exemple une clé non reconnue ou confondues avec une tête de note ou signe de reprise). On mesurera ces erreurs résiduelles comme indicateur de qualité de l'ensemble du processus CollabScore.

La granularité des tâches proposées à l'utilisateur pour cette phase est, comme pour la phase 1, *la partition complète* (contrairement à ce qui avait été envisagé initialement). On peut parier, sur la base des premières expériences, que le nombre d'erreur sera très limité et les corrections très rapides.

## Aperçu de l'interface

Le projet d'interface est illustré ci-dessous. 

### Affichage 

On affiche en vis-à-vis une page de la partition et la page correspondante du MEI
produite avec Verovio.

Tous les éléments de contexte sont surlignés : clés, armures et métriques. On s'appuie pour cela sur les boîtes englobantes des symboles, fournies par l'OMR, ansi que sur le surlignage du SVG Verovio. Chaque élément de contexte est identifié de manière unique et cohérente, aussi bien sur l'image quand dans le MEI. On peut donc passer d'une représentation à l'autre et vice-versa.

> [!IMPORTANT]  
> L'OMR remonte un degré de confiance sur l'interprétation des symboles. On pourrait envisager de ne surligner que ceux dont
> le niveau de confiance est insuffisant. À voir avoir le partenaire IRISA. 

L'interface doit permettre de saisir une action d'édition (voir ci-dessus) sur un symbole incorrect. On ne peut pas remplacer 
un symbole par n'importe quoi, donc on propose une liste de choix restreints pour remplacer, respectivement, une clé, une armure ou une métriue.

L'affichage avec toutes ces informations ressemblera à ceci:

![phase2](https://github.com/user-attachments/assets/6c29be10-ad2f-4cc5-bb72-eaa84d81cfb9)

Exemples d'actions utilisateur:

 - on clique sur une clé dans l'affichage Verovio ; c'est une clé de fa alors que la source indique une clé de sol -> on sélectionne dans la fenêtre des clés, la clé de sol, et on obtient l'action de remplacement à stocker dans l'annotation.
 - idem pour les armures et les métriques.

### Les remplacements possibles

On va limiter (au moins dans un premier temps) la liste des valeurs possibles pour chaque élément

 - les clés:
     - clé de fa 4ème  En MEI:   clef shape="F" line="4" 
     - clé de fa 3ème ligne:  clef shape="F" line="3" 
     - clé d'ut 4ème: clef shape="C" line="4" 
     - clé d'ut 3ème: clef shape="C" line="3" 
     - clé d'ut 2ème: clef shape="C" line="2" 
     - clé d'ut 1ère ligne:  clef shape="C" line="1" 
     - clé de sol 2ème ligne. En MEI:   clef shape="G" line="2" 
     - clés de sol octaviées (haut et bas) : ... je cherche ...

      - Codage des clés en MEI:
                <clef shape="F" line="4"/>
         
 - les armures : de 0 à 7 dièses, de 0 à 7 bémols. En MEI:
    -  Quand il y a X bémols, c'est un élément keySig xml:id="ks_1323_1721" sig="Xf", par exemple  sig="2f" pour deux bémols
    -  Quand il y a X dièses, même chose avec sig="Xs", par exemple  sig="2s" pour deux dièses
 - métriques: permettre la saisie d'une fraction d'entiers, plus un indicateur "lettre" (un 4/4 peut s'affiche en C, un 2/2 en C barré)

# Codages des annotations

Chaque opération d'édition doit être codée en JSON avec les paramètres nécessaires. Voir le document https://github.com/collabscore/callico/blob/main/editions.md pour la liste des éditions.


## Codage d'une clé

Exemple pour une clé de sol 2ème ligne.

```json
   {
     "label": "G",
     "line": 2
   }
```

## Codage d'une armure

Exemple pour 2 dièses

```json
   {
     "nb_sharps": 2
   }
```

Exemple pour 3 bémols:

```json
   {
     "nb_flats": 3
   }
```

> [!NOTE]  
> On aura soit ``nb_sharps``, soit ``nb_flats``, pas les deux en même temps.

## Codage d'une métrique

Exemple en 3/4:

```json
   {
     "time": 3,
      "unit": 4
   }
```

Exemple en 2/2 avec affichage lettre (C barré)

```json
   {
     "time": 2,
      "unit": 2,
      "type": "letter"
   }
```




