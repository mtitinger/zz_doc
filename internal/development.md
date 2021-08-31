# Development guidelines

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
