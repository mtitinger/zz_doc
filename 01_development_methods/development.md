# Development guidelines

## Feature and Maturity Grades

### Features and Requirements
The product features are modelled and categorized as follows:

| Requirements | Description |
| --- | --- |
| Product Requirements (**PRS**)|expressions of the required product features and properties, as required by the customer and end-users in their field of application|
| System Requirements (**SRS**) |translation of the product features in Functional and non-functional system blocs and constraints|

| Testable Features | Description | JIRA Ticket Type |
| --- | --- |
| High-level Features (**HLF**) |  translation of the SRS into a high level work breakout item, that undergo Functional Testing (FT) and Qualification Testing (QT)  as per Test Strategy and Release Strategy| EPICS |
| Low-level Features (**LLF**) | breakout of HLF into development items that undergo Unit Testing (UT), and FT as per Test Strategy| Sub-task of an EPIC |

### Maturity Grades

This project defines a very simplified set of QA Gates, called Maturity Grades (MG)

| MG | Name | Description |
| --- | --- |
|0|Concept|Product level modelling and conceptual design leading to "GO", done in v5, not addressed here|
|10|PRS defined|PRS are written and reviewed, product is defined for this PRS, done in v5, not addressed here|
|20|SRS defined|System Architecture is defined, SRS are written and reviewed, system constraints and Functional blocs (FB) are defined, done in v5, not addressed here|
|30|Software or Hardware Architecture|SW/HW Architecture is defined and reviewed for a FB, done in v5, updated in "SFD" delivered to customer|
|40|Software detailed design (HLF)|HLF are defined and documented for breakout (EPIC in JIRA)|
|60|Low Level Feature is Implemented and Validated (UT/FT)|LLF that realize the HLF is Implemented, has a testsuite and FT/UT reports are provided|
|70|HLF is implemented|HLF is documented in [internal project doc](https://schneider-electric.pages.boost.open.global/passerelle_refonte/Software/bsp/sisgateway-internal-documentation/) and FT/UT report is available |
|79|HLF is qualified|HLF has a QT report, traceability matrix proves that LLF/TestCase/TestReport-OK can be provided.|
|80|HLF is release|HLF is part of a documented release, as per Release Strategy, with a complete matching entry in the Release Note.|


NOTE : As a developer contributing to a HLF/LLF, you should at least check that MG60 and MG70 gates pass, meaning that you can proof you have FT test items, and the delivered HLF is documented and tested.

## Development process

### General Quality and Progress Management Culture
The Software development process is a simple and loosely based on the Agile processes, but officially advertised as a V-cycle, meaning that: 

* we loosely define a sprint as being one week, with a weekly backlog meeting.
* we define Releases matching macroscopic technical milestones, in order to prioritize tasks
* High-level features (HLF), meaning those features that are the technical breakout of the System Requirements (SRS) are managed as Epics. HLF are tracked in their status using a Maturity Grade (MG) criteria.
* HLF are analyzed so as to break them into Low-Level Features (LLF), tracked as Sub-tasks (stories as per Scrum, sub-tasks as per JIRA)

### Software Development Process Activities

* Architecture : we write Architecture and High-LEvel specification in the HLF Jira Ticket
* Detailed Specification : detailed implementation notes, usage and testing note in the LLF tickets
* Code writing : see Coding rules chapter
* Code contribution : see COmmitting rules
* Code Review and Static Analysis : see Committing rules
* Test construction and testing : See Test Strategy
* Validation and Qualification : See Test Strategy
* Release of Deliverables and Branching : See Release Strategy
* Maintenance and Fixes : See Release and Maintenance Strategy


## Coding Rules

### Vital rules

* R0: .h files are headers, APIs exports (extern...) are defines in headers, not in C files
* R1: .c files are modules, where the function bodies are located, don't code non trivial/inline functions in .h files, if you do so, stop it.
* R2: function shared by multiples apps should go to a library
* R3: globals should not be used, and confined to the main modules/application, and not in libraries, unless the intent was to have shared memory, in which case, you are most of the time just missing the use of an IPC of some sort.

### Elementary coding rules

* R10 : return values of function calls must be tested versus error cases
* R11 : NO EXIT() amidst libraries, or as stray exit in function calls
* R12 : an application has only one exit point, assuming R0 and R1 are applied. Upon failure, a readable message is output using a standard output, like stdout, or syslog.
* R13 : Under Linux, by convention, APIs return: ZERO (0) upon success, non-ZERO, preferably negative, upon failure
* R14 : function calls have only one exit point, NO MULTIPLE RETURNS, no longjumps, no atexit etc...
* R15 : immediate values use a define that provides some meaning and reference for that value:

```
#define DEFAULT_REG1	0x356
...
myvar = DEFAULT_REG1;
```

instead of: 

```
myvar = 0x356 /* send me 500$ if you want to know why this value */
```

* R16: defines are in FULLUPPERCASE, variables are NEVER in full-uppercase.

As a reference, you may also read https://github.com/torvalds/linux/blob/master/Documentation/process/coding-style.rst

### Project specific coding rules

* R100: use yoda-code style for conditional statements: 

```
 if (MYVAL == myvar)
```

which prevent against unwanted assignement (if using '=' rather than '=='), instead of: 

```
 if (myvar == MYVAL)
```

## Regles de commit

Les regles ci-dessous devront ^tre formalisees dans le plan de dev, mais on rappelle l'essentiel ici:
* toute modif de code est commitee via git, avec les commandes suivantes (exemples)

```
git add {mon-fichier-modifie}
git add {mon autre modif en rapport avec le meme JIRA}
```

* le commit doit se faire _avec signed-off-by:_ grace a l'option '-s'
* le commit doit comporter une premiere ligne de 'brief' qui commence par le numero du ticket JIRA:

```
git commit -sm "doc: provide more context for Open build server"
```

* exemple sans ticket JIRA justifiant la modif (doit rester exceptionnel), dans ce cas, on indique un "topic", 'doc:' ici:
```
git commit -sm "SEPASSRFNT-123: demonstrate how to write a brief"
```

* les commits doivent contenir un ensemble coherent de modif (pas un fourre-tout de plein de sujets)
* un commit doit etre 'propre' et compiler, si un commit est pousse avec des typo ou grosses erreur, on le rework avec 'amend' ou 'squash' et on le repousse
* notez que l'on peut repousser sur sa branch, mais pas une fois que le merge sur master a ete fait, donc les commits doivent etre clean avant la pullrequest.

Dans le cas d'un commit necessitant plus d'explications qu'un brief, on peut sauter une ligne, et donner plus de detail avec la command 'git commit' sans options

## Regles de branching

* Nous conservons "master" comme branch d'integration, cible des Pull Request.
* ON NE COMMIT PAS DIRECTEMENT SUR MASTER
* chaque contribution doit etre justifiee, specifiee, decrite par un JIRA, en general l'EPIC, pour contribuer ses modif, il faut donc creer une branche àartir de laquelle la PR est faire.
* une branche de pre-integration est nommee de la facon suivante:

```
git branch feature/SEPASSRFNT-456_le_topic_ma_modif
```

* on a donc le numero du JIRA et une courte description de ce que cette branch apporte.

[Back](toc.md)
