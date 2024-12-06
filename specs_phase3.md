# Spécification de la phase 3: objets musicaux

L'interface pour la phase 3 doit permettre de contrôler et si nécessaire de corriger les 
objets musicaux, définis comme *tout objet ayant une durée*. Cela couvre:

 - les **notes** : on les sélectionne par la tête de note
 - les **accords** : on sélectionne une des têtes de note de l'accord
 - les **silences**  : on sélectionne le symbole du silence

On ne peut que corriger un élément existant en modifiant certaines de ses propriétés (durée, hauteur, etc.)
ou supprimer un élément existant. On ne peut pas ajouter d'élément.

## Obtenir les annotations liant un  objet musical et sa région sur l'image

Connaissant la reférence d'un opus (par exemple ``all:collabscore:saintsaens-ref:C006_0``), on obtient la liste 
des notes, clés et silences
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

<img width="270" alt="Capture d’écran 2024-10-24 à 11 54 35" src="https://github.com/user-attachments/assets/ade110c5-5a15-47ea-8a3f-4828fd52bac5">

L'interface doit permettre de collecter un ensemble d'éditions qui seront appliquées au résultat de l'OMR. Ces éditions sont de plusieurs types:

  - édition d'une note ou d'un accord
  - édition d'un silence
  - suppression d'un objet

Par application des éditions au résultat "brut" de l'OMR avec le service ``apply_editions``, 
on obtient une partition corrigée des erreurs de l'OMR. **Mais** il est très facile également d'effectuer directement la modification sur le document Verovio puisque l'impact de la modification n'est que local, contrairement au cas des clés, armures et métriques.

La granularité des tâches proposées à l'utilisateur pour cette phase est, comme pour la phase 1, *la partition complète* (contrairement à ce qui avait été envisagé initialement). On peut parier, sur la base des premières expériences, que le nombre d'erreurs sera très limité et les corrections très rapides.

## L'interface

L'interface est identique à celle de la phase 2. On affiche en vis-à-vis une page de la partition et la page correspondante produite avec Verovio. **On peut, au moins dans un premier temps
intégrer les deux phases en déclenchant le widget de phase 2 sur les clés, armures et métriques,
et le widget de phase 3 sur les notes, accords et silences**.

Ce qui change, c'est le type
des objets que l'on peut modifier: uniquement les notes, silences et accords. 
Tous ces objets sont identifiés de manière unique dans le XML, et dans le SVG correspondant.

On va surligner par défaut les objets pour lesquels une demande de correction a été 
remontée de l'OMR. Mais on laissera l'utilisateur effectuer une modification sur tous les
objets.

L'interface doit permettre de saisir une action d'édition sur un objet. Elle consiste à 
proposer un formulaire avec un ensemble de propriétés, en proposant comme valeurs
par défaut celles issues de l'OMR.

### Les modifications autorisées

## Les altérations

On pointe une note, y compris si elle est dans un accord, et on ajoute/supprime des dièses
ou des bémols. **Le nombre d'options est limité**: 0, 1 ou 2 dièses, 1 ou deux bémols.

## La hauteur

On pointe une note, y compris si elle est dans un accord, et on modifie sa hauteur en
se déplaçant dans la séquence. Le déplacement correspond au nombre de lignes et 
interlignes vers le haut ou le bas.

Etant donné ce déplacement, il faut 
être capable de déterminer le nouveau codage de la note. Ce codage est constitué 
de deux valeurs: le *pitch name* (A, B, C, D, E, F, G) et l'octave (voir https://en.wikipedia.org/wiki/Scientific_pitch_notation 
par exemple). 

Un déplacement positif indique une progression vers la droite, un déplacement négatif un 
déplacement vers la gauche. Quand on atteint l'extrémité de droite (donc un Gx), on ajoute +1 
à l'octave et on obtient A(x+1). Quand on atteint l'extrémité de gauche (donc un Ax) 
c'est l'inverse, on obtient G(x-1).

Exemple:
  - À partir d'un A4, avec une transposition de +1, on obtient B4
  - À partir d'un A4, avec une transposition de -1, on obtient G3

En résumé, c'est une séquence qui va de C0 à C9. Ne pas dépasser ces bornes.
On doit pouvoir coder une fonction javascript qui fait ça.

### Pour les durées

Il y a une séquence qui va de la quadruple croche à la ronde. De plus chaque note
après la triple-croche peut être pointée. On va coder cette séquence 
commme suit (par exemple, 'tc' indique
une triple croche, et 'tp-p' une triple croche pointée).

(qc, tc, tc-p, dc, dc-p, c, c-p n, n-p, b, b-p, r) 

Quand on sélectionne une note, on doit positionner le curseur dans l'interface sur la durée
actuelle de la note, prise comme valeur par défaut. On peut se déplacer ensuite vers
la droite ou la gauche en restant dans les limites du tableau ci-dessus. Idéalement
il faudrait montrer immédiatement à l'utilisateur l'effet d'un déplacement en
affichant la valeur courante, dans le widget ou directement dans l'affichage Verovio.

Pour modifier le MusicXML en  fonction du choix effectué c'est un peu compliqué...

Il y a un attribut global *divisions* dans le document XML qui indique le nombre maximal de divisions possibles pour une *noire*. Donc, une valeur de 1 indique qu'on ne peut pas décomposer la noire, une valeur de 4 qu'on ne peut pas la décomposer au-delà des doubles-croches, etc. Cet attribut *divisions*
se trouve en début de mesure (exemple ci-dessous). S'il n'est pas présent, c'est le dernier recontré qui fait foi (oui, c'est chiant).

```xml
<measure id="m1-1" implicit="no" number="1">
<attributes>
<divisions>10080</divisions>
<clef id="clef_1223_1710">
<sign>G</sign>
<line>2</line>
</clef>
</attributes>
<note id="F1Part2M11r1">
<rest/>
<duration>40320</duration>
<type>whole</type>
</note>
</measure>
```
Ensuite, pour chaque note, silence ou accords, on a un attribut *duration* qui indique le nombre de divisions. En supposant que ``divisions`` vaut 4, on a par exemple:

  - ``duration=1`` : une double-croche
  - ``duration=4``: une noire
  - ``duration=6``: une noire pointée
  - ``duration=8``: une blanche
  - ``duration=16``: une ronde

Voici donc l'algo pour parcourir les durées autorisées

  - **Quand on monte**: on divise  ``duration`` par ``divisions'', et on multiplie par ``1,5 x divisions``.

Exemples (toujours en supposant que ``divisions`` vaut 4).

  - Si ``duration=4`` (une noire), on calcule (4/4) x 1,5 x 4 = 6, soit une noire pointée
  - Si  
Et ainsi de suite. Cela correspond aux codes MusicXML.

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

## Le codage des remplacements.



# Codages des annotations

Chaque opération d'édition doit être codée en JSON avec les paramètres nécessaires. Voir le document https://github.com/collabscore/callico/blob/main/editions.md pour la liste des éditions. Dans 
à peu près tous
les cas (sauf les triolets) on indique un identifiant d'objet et les propriétés à modifier.

Quelques exemples de codage:

Changement de la durée. 

```json
   {
     "duration": "16th"
   }
```

Changement de la hauteur. 

```json
   {
     "pitch": "D6"
   }
```


Changement de la durée, de la hauteur et ajout d'un dièse

```json
   {
     "pitch": "D6",
     "duration": "16th",
      "alter": 1
   }
```

