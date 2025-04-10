# Managing editions in CollabScore

Music scores in CollabScore are obtained via Optical Music Recognition (OMR)
applied to IIIF *sources* (see https://github.com/collabscore/documents/blob/main/devdoc/sources.md). 
The OMR proceeds in two phases

 - a JSON file reporting the music symbols found in the score image is produced by the OMR software DMOS
 - a transcription of the JSON content in a standard music score format

The result of the two-phases process  is an XML file (MusicXML or MEI). Since
OMR is inherently error-prone, this file needs to be completed and/or corrected.

  - *completion* may be adding metadata, or adjusting the regions recognized by ORM
  - *correction* is any action that fixes a faulty OMR output

Both completion and corrections are referred to as *editions* in CollabScore.
generally speaking, and edition is an operation applied during the transcription
of the JSON file to the XML file. This operation has a *name* and a list
of *parameters*.

##  Example of an edition

A music score consists of *parts*, i.e., a set of instructions given to 
a performer playing an instrument. In a score one may find a piano part, a
violin part, etc.

In general OMR cannot extract the description of a part, and it has therefore
to be supplied via and external source (a user generally). The description
of a part consists of

 - its instrument (taken from a taxonomy, in order to identify transposing instruments)
 - its name (free text), to be displayed generally at the beginning of the first part's system(s) in the score 
 - its abbreviation, to be used in subsequent systems.

Specifying the description of a part is done with an operation named ``describe_part`` 
that takes as parameter the target part and the above attributes. Its JSON description is as follows:

```json
{
	"name": "describe_part",
        "target": "Part2",
	"params": {
                 "instrument": "Chant",
                 "name": "Ténor",
                 "abbreviation": "T."
	}
}
```

So the operation has a ``name``, a ``target`` referring to the
object(s) the edition applies to, and a ``params``
dictionary, with three attributes. 

The names and types of the parameters changes from one operation to the other,
but an operation has *always* a ``name`` and a ``target``, from which the expected action  can be decoded.

## Range of editions

An edition can have a *range* of measures. As measure is referred to
by a triplet *(page_number, system_number, measure_number)*. In JSON,
a range has therefore the following form:

```json
{
	"from_page: 2,
	"from_system: 2,
	"from_measure: 1,
	"to_page: 3,
	"to_system: 99,
	"to_measure: 99
}
```
The above range goes from measure 1 of system 2 of page 2 to the
last measure of the last system of page 3. Range can be specified
for some editions.

If a parameter is unspecified, the default value is 1 for ``from`` 
values, and 99 for ``to`` values. The following range applies
to all measures from measure 1 of the third system of page 2.


```json
{
	"from_page: 2,
	"from_system: 3
}
```

## Services on editions

The CollabScore server exchanges operations via REST services. 
The serialization is based on JSON. An operation is always send/received
to/from the ``iiif`` source of a Opus. In general, such a source
is referred to by combining the Opus ref, the keyword ``_sources``
and the source code (``iiif`` in our case). Here is an example:

```
all:collabscore:saintsaens-ref:C006_0/_sources/iiif/
```

The Opus ref can be replaced by its database id, e.g., 

```
22468/_sources/iiif/
```

See the document on sources for explanations. We use this URL for illustrating the services below. Editions can only be accessed by authorized users. The credentials must be given with each call.

Editions are always exchanges as a list. In order to *add* an edition
to an existing list, you must first GET the current edition list, 
append the JSON objet(s) and finally PUT the extended list.

### Retrieving editions

The list of editions for a source can be obtained with a GET

```
curl -u login:password -X GET
http://neuma.huma-num.fr/rest/collections/22468/_sources/iiif/_editions/
```

The services returns an array of editions:

```json
[
  {
	"name": "describe_part",
         "target": "Part2",
	"params": {
                 "instrument": "Chant",
                  "name": "Ténor",
                  "abbreviation": "T."
	}
},
  {
	"name": "describe_part",
         "target": "Part1",
	"params": {
                 "instrument": "Piano",
                 "name": "Piano",
                 "abbreviation": "P."
		}
	}
   }
]
```


### Applying editions

The ``apply_editions`` services allows to apply a list of
editions to an IIIF source. It returns the MusicXML document
resulting from the IIIF parsing completed with editions.

The editions must be sent as a JSON array.

```
curl -u login:password -X GET
http://neuma.huma-num.fr/rest/collections/22468/_sources/iiif/_apply_editions/
-d @editions.json  -H "Content-Type: application/json"
```


### Replacing editions

Editions are replaced by sending a ``PUT`` request.

```
curl -u login:password -X PUT
http://neuma.huma-num.fr/rest/collections/22468/_sources/iiif/_editions/
-d @new_editions.json  -H "Content-Type: application/json"
```

## List of editions

### Describe parts

Defines metadata qualifying a part.
See the first example of this document.

### Assign staff to part

Tells that a staff is allocated to a part in a given range. The
target is the part and the parameters are

  - A staff number
  - A range

Example: the edition specified below tells that staff ``Staff1``
is allocated to part ``P1``, from the second system of page 3.

```json
{
	"name": "assign_staff_to_part",
         "target": "Part1",
	"params": {
		"staff_number": 1
		},
	"range": {
	    "from_page: 3,
	     "from_system: 2
         }
}
```


###  Merging parts

Two distinct parts can be *merged* (typically, staves interpreted as dictinct
parts, that actually correspond to a piano part). The target
is (by convention) the "score" keyword, and the single parameter
is an array of part's ids. 

Example: the edition specified below merges parts ``p1``  and
``p2`` in ``p1``. 

```json
{
	"name": "merge_parts",
         "target": "score",
	"params": {
		"parts": ["p1", "p2"]
		}
}
```

### Remove an object

The removed element will be ignored during the production of the MusicXML file. Its type
is not specified (clef, signature, note...)

```json
{
	"name": "remove_object",
         "target": "my_id",
         "params": {}
 }
```

###  Replace a clef

A clef identified by its id can be replaced. Example:

```json
{
	"name": "replace_clef",
        "target": "clef_1029_209",
         "params": {
                 "label": "G",
                 "line": 2
	}
 }
```

Accepted values for ``label`` are "G", "F", "C".
Accepted values from ``line``  are 
  - 2 (if label is G),
  - 4 or 3 (if label if F)
  - 1, 2, 3, 4 (if label if C)

###  Replace a key signature

A key signature identified by its id can be replaced. Example:

```json
{
	"name": "replace_keysign",
         "target": "ks_1929_1092",
         "params": {
                 "nb_sharps": 2,
                  "nb_flats": 0
	}
 }
```

###  Replace a time signature

A time signature identified by its id can be replaced. Example:

```json
{
	"name": "replace_timesign",
         "target": "ts_1929_1092",
         "params": {
                "time": 3,
                "unit": 4
	}
 }
```

Les valeurs autorisées pour le numérateur sont tous les entiers positifs.
Les valeurs autorisées pour le dénominateur sont les puissances de 2: 1, 2, 4, 8, 16, 32. Et ça suffit.

###  Replace a music element

A music element (note or rest) identified by its id can be updated. The values 
produced by the interface are described in https://github.com/collabscore/callico/blob/main/specs_phase3.md.
Here is a list of examples:

The pitch is moved up two levels:

```json
{
	"name": "replace_music_element",
         "target": "nh_1929_1092",
         "params": {
               "pitch_change": 2
	}
 }
```

The pitch is moved down one level, and the number of flats must be set to 1:

```json
{
	"name": "replace_music_element",
         "target": "nh_1929_1092",
         "params": {
               "pitch_change": -1,
               "alter": -1
	}
 }
```

The duration is a dotted quarter:

```json
{
	"name": "replace_music_element",
         "target": "nh_1929_1092",
         "params": {
               "duration": "quarter",
               "dots": 1
	}
 }
```

The duration is  a dotted eighth, and alteration is set to one sharp.

```json
{
	"name": "replace_music_element",
         "target": "nh_1929_1092",
         "params": {
               "duration": "eighth",
               "dots": 1,
                "alter": 1
	}
 }
```

The note (or rest, or chord) is assigned to the second voice:

```json
{
	"name": "replace_music_element",
         "target": "nh_1929_1092",
         "params": {
               "voice": 2
	}
 }
```

###  Comment a music element

Send a message to associate a comment to a music element

```json
{
	"name": "comment_element",
         "target": "ks_1929_1092",
         "params": {
                 "comment": "This is text of the comment, properly encoded for JSON"
	}
 }
```
