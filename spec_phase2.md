# Spécification de la phase 2

L'interface pour la phase 2 doit permettre de contrôler et si nécessaire de corriger les éléments du *contexte d'interprétation musicale* 
reconnu par l'OMR. Ces éléments sont

 - les **clefs** : elles permettent d'inférer le nom des notes selon leur position sur la portée
 - les **armures** : donne le nombre de dièses ou de bémols *par défaut*
 - la **métrique** (ou chiffrage de mesure) : indique la division en temps d'une mesure

Ces éléments sont représentés par des symboles affectant une portée spécifique. De plus, les armures et la métrique ne peuvent apparaître qu'en 
début de mesure. En revanche on peut changer de clé n'importe quand, voire plusieurs fois par mesure.

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

## Aperçu de l'inteface

Le projet d'interface est illustré ci-dessous. On affiche en vis-à-vis une page de la partition et la page correspondante du MEI
produite avec Verovio.

Tous les éléments de contexte sont surlignés : clés, armures et métriques. On s'appuie pour cela sur les boites englobantes des symboles fournies par l'OMR, et sur le surlignage du SVG Verovio. Chaque élément de contexte est identifié de manière unique et cohérente, aussi bien sur l'image quand dans le MEI. On peut donc passer d'une représentation à l'autre et vice-versa.

> [!IMPORTANT]  
> L'OMR remonte un degré de confiance sur l'interprétation des symboles. On pourrait envisager de ne surligner que ceux dont
> le niveau de confiance est insuffisant. À voir avoir le partenaire IRISA. 

L'interface doit permettre de saisir une action d'édition (voir ci-dessus) sur un symbole incorrect. On ne peut pas remplacer 
un symbole par n'importe quoi, donc on propose une liste de choix restreints pour remplacer, respectivement, une clé, une armure ou une métriue.

L'affichage avec toutes ces informations ressemblera à ceci:

![phase2](https://github.com/user-attachments/assets/6c29be10-ad2f-4cc5-bb72-eaa84d81cfb9)


