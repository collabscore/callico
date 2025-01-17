# Spécification de la phase 3: objets musicaux

L'interface pour la phase 3 doit permettre de contrôler et si nécessaire de corriger les 
objets musicaux, définis comme *tout objet ayant une durée*. Cela couvre:

 - les **notes** : on les sélectionne par la tête de note
 - les **accords** : on sélectionne une des têtes de note de l'accord
 - les **silences**  : on sélectionne le symbole du silence

On ne peut que corriger un élément existant en modifiant certaines de ses propriétés (durée, hauteur, etc.) ou supprimer un élément existant. On ne peut pas ajouter d'élément.

## Comment récupérer les données

### Obtenir les annotations liant un  objet musical et sa région sur l'image

Connaissant la reférence d'un opus (par exemple ``all:collabscore:saintsaens-ref:C006_0``), on obtient la liste des notes, clés et silences sous la forme d'annotations liant l'identifiant de l'objet dans le XML et la région sur l'image.
Le modèle d'annotation est ``image-region``, et le concept d'annotation est ``note-region``.

Voici un exemple de l'URL d'appel pour l'opus all:collabscore:saintsaens-ref:C006_0.

https://neuma.huma-num.fr/rest/collections/all:collabscore:saintsaens-ref:C006_0/_annotations/image-region/note-region/

### Obtenir les annotations indiquant une demande de correction sur un objet musical 

Le système OMR remonte des demandes de corrections sur des objets quand le niveau
de certitude est en dessous d'un seuil. Ces demandes peuvent également être obtenus sous
forme d'annotations avec le modèle d'annotation ``omr-error``. La requête REST est 
donc dans ce cas:

https://neuma.huma-num.fr/rest/collections/all:collabscore:saintsaens-ref:C006_0/_annotations/omr-error/_all/

## Principe de l'interface

<img width="270" alt="phase3" src="https://github.com/user-attachments/assets/48fe53c2-ff12-465a-baae-a0a77a445a00" />

L'interface doit permettre de collecter un ensemble d'éditions qui seront appliquées au résultat de l'OMR. Ces éditions sont de plusieurs types:

  - édition d'une note ou d'un accord
  - édition d'un silence
  - suppression d'un objet

Par application des éditions au résultat "brut" de l'OMR avec le service ``apply_editions``, 
on obtient une partition corrigée des erreurs de l'OMR. **Mais** il est très facile également d'effectuer directement la modification sur le document Verovio puisque l'impact de la modification n'est que local, contrairement au cas des clés, armures et métriques.

La granularité des tâches proposées à l'utilisateur pour cette phase est, comme pour la phase 1, *la partition complète* (contrairement à ce qui avait été envisagé initialement). On peut parier, sur la base des premières expériences, que le nombre d'erreurs sera très limité et les corrections très rapides.

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

### Modification des altérations

On pointe une note, y compris si elle est dans un accord, et on ajoute/supprime des dièses
ou des bémols. **Le nombre d'options est limité**: 1 ou 2 dièses, 1 ou 2 bémols, 0 altérations.

Dans le MusicXML on a ``accidental`` => ce qui est montré et ``alter``, l'altération de la note qu'on veut entendre. On va oublier ``accidental`` et on va se contenter de modifier ``alter``. Voici une note avec un bémol.

```xml
<pitch>
  <step>B</step>
  <alter>-1</alter>
  <octave>2</octave>
</pitch>
```
Voici un élément avec 2 dièses. 

```xml
<pitch>
  <step>G</step>
  <alter>2</alter>
  <octave>4</octave>
</pitch>
```

L'interface doit se positionner par défaut en tenant compte de la valeur courante de ``alter``
(donc on montre un dièse si ``alter``  vaut 1). Ensuite on peut incrémenter de 1 ou -1, mais
en restant toujours dans la liste [-2, -1, 0, 1, 2].  On conserve la valeur validée par l'utilisateur 
pour la transmettre au service d'édition.

### Modification de la hauteur

On pointe une note, y compris si elle est dans un accord, et on modifie sa hauteur en
se déplaçant dans la séquence. Le déplacement correspond au nombre de lignes et 
interlignes vers le haut ou le bas.

Etant donné ce déplacement, il faut 
être capable de déterminer le nouveau codage de la note. Ce codage est constitué 
de deux valeurs: le *pitch name* (A, B, C, D, E, F, G) et l'octave (voir https://en.wikipedia.org/wiki/Scientific_pitch_notation 
par exemple). 

Un déplacement positif indique une progression vers la droite dans cette liste, un déplacement négatif un 
déplacement vers la gauche. Quand on atteint l'extrémité de droite (donc un Gx), on ajoute +1 
à l'octave et on obtient A(x+1). Quand on atteint l'extrémité de gauche (donc un Ax) 
c'est l'inverse, on obtient G(x-1).

Exemple:
  - À partir d'un A4, avec une transposition de +1, on obtient B4
  - À partir d'un A4, avec une transposition de -1, on obtient G3
  - À partir de G4 avec une transposition de +1 on obtient A5

En résumé, c'est une séquence qui va de C0 à C9. Ne pas dépasser ces bornes.
On doit pouvoir coder une fonction javascript qui fait ça.

### Modification de la durée

Il y a une séquence de durées acceptables
qui va de la quadruple croche à la ronde (ce serait bien de la paramétrer
pour pouvoir la changer facilement). De plus chaque note
après la triple-croche peut être pointée. On va coder cette séquence 
commme suit (par exemple, 'tc' indique
une triple croche, et 'tp-p' une triple croche pointée).

(qc, qp-p, tc, tc-p, dc, dc-p, c, c-p, n, n-p, b, b-p, r) 

Quand on sélectionne une note, on doit positionner le curseur dans l'interface sur la durée
actuelle de la note, prise comme valeur par défaut. On peut se déplacer ensuite vers
la droite ou la gauche en restant dans les limites du tableau ci-dessus. Idéalement
il faudrait montrer immédiatement à l'utilisateur l'effet d'un déplacement en
affichant la valeur courante, dans le widget ou directement dans l'affichage Verovio,
comme on le fait déjà pour les clefs ou les armures.

La figure ci-dessous montre la liste des valeurs proposées dans MuseScore
![duree-musescore](https://github.com/user-attachments/assets/4eefdca6-de78-4450-9710-de055a63a8a2)

Il faut y ajouter des options:
  - on peut transformer une note en silence ou un silence en note ; un case à cocher "silence" peut faire l'affaire
  - Enfin on peut indiquer qu'une note ou un silence  est le début d'un n-olet (triolet, etc.)

**Comment modifier le MusicXML**.  
Il y a un attribut global *divisions* dans le document XML qui indique le nombre maximal de divisions possibles pour une *noire*. Donc, une valeur de 1 indique qu'on ne peut pas décomposer la noire, une valeur de 4 qu'on ne peut pas la décomposer au-delà des doubles-croches, etc. Cet attribut *divisions* se trouve en début de mesure (exemple ci-dessous). S'il n'est pas présent, c'est le dernier recontré qui fait foi (oui, c'est chiant).

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
Ensuite, pour chaque note, silence ou accord, on a un attribut *duration* qui indique le nombre de divisions. En supposant que ``divisions`` vaut 4, il faut
quatre unités pour faire une noire. On a donc par exemple:

  - ``duration=1`` : une double-croche
  - ``duration=2`` : une croche
  - ``duration=4``: une noire
  - ``duration=6``: une noire pointée
  - ``duration=8``: une blanche
  - ``duration=16``: une ronde

La formule générale pour calculer la valeur de ``duration`` est donc
``divisions x facteur_durée``, le facteur durée étant donné ci-dessous.

```JSON
{
  "qc": 1/16,
   "qp-p": 3/32,
  "tc": 1/8,
  "tc-p": 3/16,
  "dc": 1/4,
  "dc-p": 3/8,
 "c":  1/2,
  "c-p": 3/4,
  "n": 1,
   "n-p": 3/2,
  "b": 2,
   "b-p": 3,
"r": 4   
}
```

### Le cas des triolets

On peut aussi indiquer qu'un groupe de notes est un triolet (ou un n-olet en général). Pour cela
on pointe le premier objet du groupe et on choisit dans un menu déroulant le type de groupement: une valeur entre 2 et 10.

### Le cas des accords

On peut modifier chaque note de l'accord. *Mais, attention*: si on modifie la durée d'une des notes,
alors la durée de toutes les autres doit être modifiée également.

### Le cas des silences

Pour les silences, on ne peut modifier que la durée

# L'appel au service d'édition

On appelle le service ``apply_editions`` (en passant le paramètre HTTP ``format=json``) pour 
un recalcul immédiat de la partition. Comme d'habitude le JSON des éditions doit 
être stocké dans l'annotation de Callico pour être ensuite intégré définitivement dans Neuma.

Voici des exemples de codage. Il faut se référer au document https://github.com/collabscore/callico/blob/main/editions.md pour la liste des services. Tout
ce qui suit correspond aux paramètres du service ``update_music_element``.

### Codage des changements de hauteur

On insère dans l'édition un attribut ``pitch_change`` avec un entier non nul indiquant le
nombre de déplacements.

**Exemples**:

 On a déplacé  la note d'un cran vers le haut:
```json
   {
     "pitch_change": 1
   }
```

 On a déplacé  la note de deux crans vers le bas:
 
```json
   {
     "pitch_change": -2
   }
```

### Codage des changements d'altération

Même principe que pour le changement de hauteur: On insère dans l'édition un attribut ``alter`` 
qui indique le nombre de dièses ou de bémols
de la note modifiée (soit la valeur de l'élément ``alter``  de MusicXML). C'est toujours un
entier appartenant à [-2, -1, 0, 1, 2];

**Exemples**:

 La note n'a ni dièse ni bémol
 
```json
   {
     "alter": 0
   }
```

La note a deux dièses

```json
   {
     "alter": 2
   }
```

La note a un bémol

```json
   {
     "alter": -1
   }
```

## Codage de la durée

Il faut s'abstraire du codage XML pour les annotations, car on travaille
directement sur la forme des notes. Le plus simple est d'envoyer la paire
``(divisions, duration)`` et je me débrouillerai avec ça. La valeur
de ``duration`` est calculée comme indiquée ci-dessus. 
`
### Quelques exemples complets

Se référer au document 
https://github.com/collabscore/callico/blob/main/editions.md et à la description
du service ``update_music_element``.
