# Spécifications de l'interface collaborative

## Organisation des campagnes en 4 étapes

Les campagnes se succèdent dans un ordre strict. 
  
Chaque campagne s'effectue à une certaine granularité et produit une séquence d'opérations d'éditions.

Après chaque campagne on recalcule le MEI à partir de la sortie OMR et des opérations
d'édition.
  
### Première étape - Manifeste

On spécifie les parties: instruments, abbréviations (granularité: Opus)

On associe chaque portée (simple ou double) à l'une des parties (granularité : page)

Opérations d'édition:
  - specify_part (décrire une partie) 
  - merge_staves (fusionner 2, voire 3 portées sur une même partie)
  - assign_staff_to part 

### Etape 2 - Contexte interprétation

On demande à l'utilisateur de vérifier tous les éléments qui conditionnent l'interprétation 
des notes. 

Ces éléments sont

 - Armures (en cas de niveau de confiance assez faible)
 - Clefs (en cas de niveau de confiance assez faible)
 - Métrique (en cas de doute sur la durée de la mesure)

La granularité est la page

### Etape 3 - éléments locaux

En supposant les parties bien identifiées et localisées sur les portées (étape 1), et 
les éléments d'interprétation corrects (étape 2), on demande à l'utilisateur de corriger 
les éléments locaux.

  - Durée, hauteur, altération des notes
  - Durée des silences
  - Eventuellement ajout ou suppression d'éléments

La granularité est la page 

### Etape 4 (optionnelle) - les paroles

À compléter. S'inspirer de la saisie séquentielle dans MuseScore.

### Etape 5 (optionnelle) - vérification et validation

À compléter.

