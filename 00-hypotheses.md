# Hypothèses

Avant de commencer la réflexion, il est nécessaire d'en poser les hypothèses.
Ces hypothèses sont inaltérables et définissent le périmètre dans lequel nous nous basons.
Si vous n'êtes pas en accord avec ou concerné par celles-ci, le reste de l'article devient caduque.

## Cible

La cible du Design System est un ensemble de projets restreint et connu.

Ça peut être :
* l'ensemble des projets de votre entreprise, dans le cas d'un concepteur d'applications (ex: Lucca)
* l'ensemble des projets d'un seul de vos clients (ex: Fabernovel pour Engie)

Ça ne peut pas être :
* Un DS ouvert pour le grand public (ex: Material, Ant)
* Un DS pour un client et d'autres fournisseurs (l'ensemble des projets devient alors potentiellement non restreint et inconnu)

Remarque : travaillant plutôt en mode projet (et non en mode produit), je vais souvent énoncer des éléments dans ce sens.

## Objectif

Les objectifs principaux du Design System sont :

* Pouvoir réutiliser des composants (design et dev) afin d'accélérer le design et l'implémentation ==> réduction des coûts
* Avoir une interface homogène à travers les applications concernées
* Pouvoir se concentrer sur les **besoins métiers** et mettre l'énergie sur la valeur ajoutée

Les développeurs ont tendance à mettre de la valeur sur la technique et la promouvoir à tort (moi le premier).
Cette technique a beaucoup d'impact dans la conception et implémentation des projets et peut impacter les coûts finaux.
Mais rares sont les clients qui souhaitent payer cette technique, car souvent ils ne la comprennent pas.

Les designers ont tendance à mettre de la valeur sur le rendu esthétique et la promouvoir.
Cette esthétique a beaucoup d'impact sur la satisfaction utilisateur et sur l'implémentation des projets et peut impacter les coûts finaux.
Mais rares sont les clients qui souhaitent payer plus cher pour une application *un peu* plus belle si une équivalence plus simple (=moins chère) existe.

Il faut donc garder en tête quelle est la valeur que nous apportons et est-ce que cette valeur est pour nous (designer et développeurs) ou pour le client.

Nous allons également voir ça dans les autres parties, mais il faut aussi se demander si une solution :

* A un impact sur le client (=ça coûte plus cher) ?
* A un impact sur nous (=on développe plus vite, on est plus satisfait, on ajoute plus facilement plus de valeur) ?
* A un impact sur les utilisateurs (=expérience utilisateur) ?

Parfois, une solution est positive pour les trois parties, parfois il faut faire un choix.

⚠️ Cela peut paraître évident, mais d'expérience, il est très facile de tomber dans le piège et l'oublier.
Pire, il est très facile de justifier nos croyances comme bénéfiques au client ou aux utilisateurs, sans qu'elles ne soient prouvée, voire complètement fausses.

## Méthodologie, organisation

Le DS est développé en parallèle des projets l'utilisant.
Ce sont les équipes projet qui maintiennent le DS.
Tout développeur d'une de ces équipes peut mettre à jour le DS.
Chaque modification doit être approuvée par une partie ou l'ensemble des autres équipes.

Cela implique :

* Le DS ne fonctionne pas en boîte noire (on a accès à l'API et l'implémentation)
* Le DS n'est pas modifiable au sein d'un projet (pas de copie du DS, mais une dépendance)
* Le DS doit rester le plus rétrocompatible au fur et à mesure des évolutions (=limiter au mieux les versions majeures)

## Technologies

Dans tout l'article, les exemples seront en Typescript.
Cela signifie qu'on explicite les types et que le compilateur valide le code.

Je pars également du principe qu'on cherche à s'approcher du [bug-free by design](http://www.changit.fr/bug-free-by-design).

Nous utliserons React comme Framework de composants, mais l'article peut s'appliquer à d'autres.