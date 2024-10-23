# Spécification de la phase 3: objets musicaux

L'interface pour la phase 3 doit permettre de contrôler et si nécessaire de corriger les 
objets musicaux, définis comme *tout objet ayant une durée*. Cela couvre:

 - les **notes** : on les sélectionne par la tête de note
 - les **accords** : on sélectionne une des têtes de note de l'accord
 - les **silences**  : on sélectionne le symbole du silence

On ne peut que corriger un élément existant en modifiant certaines de ses propriétés (durée, hauteur, etc.)
ou supprimer un élément existant. On ne peut pas ajouter d'élément.

## Obtenir les annotations liant un  objet musical et sa région sur l'image

Connaissant la reférence d'un opus (par exemple ``all:collabscore:saintsaens-ref:C006_0``), obtient la liste 
des notes, clés et simlences
sous la forme d'annotations liant l'identifiant de l'objet dans le XML et la région sur l'image.
Le modèle d'annotation est ``image-region``, et le concept d'annotation est ``note-region``.

Voici un exemple de l'URL d'appel pour l'opus all:collabscore:saintsaens-ref:C006_0.

https://neuma.huma-num.fr/rest/collections/all:collabscore:saintsaens-ref:C006_0/_annotations/image-region/note-region/


## Obtenir les annotations indiquant une demande de correction sur un objet musical 

Le système OMR remonte des demandes de corrections sur des objets quand le niveau
de certitude est en dessous d'un seuil. Ces demandes peuvent également être obtenus sous
forme d'annotations avec le modèle d'annotation ``omr-error``. La requête REST est 
donc dans ce cas:

https://neuma.huma-num.fr/rest/collections/all:collabscore:saintsaens-ref:C006_0/_annotations/omr-error/_all/

## Principe de l'interface

L'interface doit permettre de collecter un ensemble d'éditions qui seront appliquées au résultat de l'OMR. Ces éditions sont de plusieurs types:

  -  edition d'une note
  - édition d'un silence
  - suppression d'un objet

Par application des éditions au résultat "brut" de l'OMR avec le service ``apply_editions``, 
on obtient une partition corrigée des erreurs de l'OMR.

La granularité des tâches proposées à l'utilisateur pour cette phase est, comme pour la phase 1, *la partition complète* (contrairement à ce qui avait été envisagé initialement). On peut parier, sur la base des premières expériences, que le nombre d'erreurs sera très limité et les corrections très rapides.

## L'interface

L'interface est identique à celle de la phase 2. On affiche en vis-à-vis une page de la partition et la page correspondante 
produite avec Verovio.

Ce qui change, c'est le type
des objets que l'on peut modifier: uniquement les notes, silences et accords. 
Tous ces objets sont identifiés de manière unique dans le XML, et dans le SVG correspondant.

On va surligner par défaut les objets pour lesquels une demande de correction a été 
remontée de l'OMR. Mais on laissera l'utilisateur effectuer une modification sur tous les
objets.

L'interface doit permettre de saisir une action d'édition sur un objet. Elle consiste à 
proposer un formulaire avec un ensemble de propriétés, en proposant comme valeurs
par défaut celles issues de l'OMR.

### Formulaire pour les silences

C'est le plus simple: on ne peut modifier que la durée. La figure ci-dessous montre l'interface de
MuseScore: on peut sélectionner des durées allant de la quadruple croche à la ronde. 

On peut aussi indiquer qu'un groupe de notes est un triolet (ou un n-olet en général). Pour cela
on pointe le premier objet du groupe et on choisit dans un menu déroulant le type de groupement.

![form-musescore](https://github.com/user-attachments/assets/f28f4fb3-e92e-40e2-bdef-00c2b9f8ebea)

### Formulaire pour les notes

On peut modifier les propriétés suivantes:

  - la durée
  - la hauteur
  - l'altération

Pour changer la hauteur, idéalement, on bouge la note de haut en bas sur le verovio... Sans doute difficile.
On peut aussi indiquer l'intervalle de transposition: une tierce au-dessus, une seconde en dessous.... Il 
faudrait refléter le changement immédiatement dans l'affichage.

Pour changer l'altération on peut choisir dans un menu déroulant: voir ci-dessus.


### Formulaire pour les accords

On peut modifier chaque note de l'accord. *Mais, attention**: si on modifie la durée d'une des notes,
alors la durée de toutes les autres doit être modifiée également.

### Exemples d'actions utilisateur:

 - on clique sur une note dans l'affichage Verovio ; c'est une noire alors que l'image montre que la réalité est une croche; le formulaire permet de corriger la durée
 -  ou bien: c'est un la 4,  alors que l'image montre que la réalité est do4 ; le formulaire permet de corriger la hauteur de la note

### Les remplacements possibles

On va limiter (au moins dans un premier temps) la liste des valeurs possibles pour chaque élément

 - les clés:
     - Codage des clés en MEI:
       ```xml
                <clef shape="F" line="4"/>
       ```
       Shape peut être "F" pour clé de Fa, "C" pour clé d'ut, et "G" pour sol
     - Codage des clés en MusicXML. Même chose mais avec des éléments
       ```xml
                <clef id="clef_1223_1710">
                   <sign>G</sign>
                   <line>2</line>
                 </clef>
        ```
     - clé de fa 4ème  En MEI:   clef shape="F" line="4" 
     - clé de fa 3ème ligne:  clef shape="F" line="3" 
     - clé d'ut 4ème: clef shape="C" line="4" 
     - clé d'ut 3ème: clef shape="C" line="3" 
     - clé d'ut 2ème: clef shape="C" line="2" 
     - clé d'ut 1ère ligne:  clef shape="C" line="1" 
     - clé de sol 2ème ligne. En MEI:   clef shape="G" line="2" 
     - clés de sol octaviées (haut et bas) : ... je cherche ...


 - les armures : de 0 à 7 dièses, de 0 à 7 bémols.
   - Codage des armures en MEI:
     ```xml
               <keySig xml:id="k10l74wn" sig="2f"/>
     ``` 
     Quand il y a X bémols, c'est un attribut sig="Xf", par exemple  sig="2f"
     pour deux bémols. Quand il y a X dièses, même chose avec sig="Xs",
     par exemple  sig="2s" pour deux dièses
   - Codage des armures en MusicXML: oin indique un élément ``fifths``
     qui vaut de -7 à 7. Les négatifs sont pour les bémols, les positifs
     pour les dièses. Exemple pour deux bémols:
     ```xml
              <key id="ks_1323_1721">
                <fifths>-2</fifths>
              </key>
     ```        
 - métriques: permettre la saisie d'une fraction d'entiers, plus un indicateur "lettre" (un 4/4 peut s'affiche en C, un 2/2 en C barré)

# Codages des annotations

Chaque opération d'édition doit être codée en JSON avec les paramètres nécessaires. Voir le document https://github.com/collabscore/callico/blob/main/editions.md pour la liste des éditions.

## Codage des propriétés d'une note

Exemple pour une clé de sol 2ème ligne.

```json
   {
     "label": "G",
     "line": 2
   }
```
