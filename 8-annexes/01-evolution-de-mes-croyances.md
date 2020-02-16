# Évolution de mes croyances

À la lecture de ce repository, certains pourraient penser que j'ai une pensée dogmatique, que je suis à côté de la plaque ou que je prends pas en compte certains choix.

Une bonne partie des choses que j'énonce ici sont pourtant à l'opposé de ce que je pensais avant.
Ici, je vais essayer d'expliquer pourquoi je crois ce que je crois.

Bien que difficile à faire à soi-même, je m'inspire des principes de l'[entretien épistémique](https://skeptikon.fr/videos/watch/4bf28fd4-ca36-4ec9-983f-842fef879e32).

La majorité de ce qui est écrit dans ce repository n'est pas factuel et ne peut pas l'être (jusqu'à preuve du contraire).
Par exemple, qu'est-ce qui est **objectivement** du clean-code ?
Si quelqu'un a la réponse, je suis preneur, mais j'émettrai beaucoup de doutes sur l'objectivité.
En raison de cela, je ne peux que me contenter de rechercher la source de mes croyances et vous inviter à me challenger.

Remarque, j'écris ces lignes principalement en utilisant ma mémoire avec tous les biais cognitifs possibles.

## Composition de mes équipes

À mon arrivé à Zengularity (aujourd'hui Fabernovel), j'ai travaillé sur des projets front et back.
J'ai travaillé sur un premier projet que j'ai repris seul (après une pause d'environ un an).
Après 3 à 6 mois dessus, d'autres développeurs m'ont rejoint.

Que ce soit l'équipe précédente (avant la pause d'un an) ou l'équipe suivante, nous étions tous des développeurs fullstack, mais avec une appétence back.

Côté Back, j'avais peu de choses à redire.
Par contre, côté front, il y avait de nombreux antipatterns :

* God components
* Structure du code qui ne suit pas la structure des pages
* Des composants redéfinis plusieurs fois avec le même code copié/collé

Je tiens à dire que le code n'était pas spécialement mauvais.
Il était majoritairement simple à comprendre et à modifier.
Les exemples que je prends sont ceux qui m'ont marqué et pas forcément représentatif de la globalité du projet.

Mais comme les développeurs avaient une appétence back, le code front était souvent délaissé au profit du back.
Bien que petit à petit, on ait diminué le code dupliqué, réduit les god components, etc., de nombreuses fois des mauvaises pratiques revenaient
(code dupliqué, structure du code pas claire), parfois attrapées en code review, parfois non.
Cependant, plus le problème était traité niveau conception, moins les problèmes revenaient.

Un exemple frappant sur ce projet était la gestion des `input`.
Il y avait deux composants pour faire un `input[type=text]`, dont une version qui mélangeait `text`, `number`, `date` et `autocomplétion`.
Il y avait également quelques endroits où nous appelions directement `input[type=text]` en injectant les styles… des deux autres composants.

Il y a plusieurs sources du problème, comme toujours. Mais l'une de ces sources aurait pû limiter les problèmes.
En regardant le code, on peut émettre l'hypothèse que la conception ne répondait pas aux questions :

* Quel est le but de mon composant ?
* Comment puis-je utiliser ce composant ?
* Quelles sont les limites de mon composant ?

Une des versions du composant `input[type=text]` était **permissif**.
Il ne répondait pas à un besoin métier, mais permettait d'être réutiliser pour tous les besoins techniques :
Et ça n'a pas manqué, il était utilisé en y injectant de nombreux besoins techniques.

Avant mon départ du projet, je n'ai pas réussi à supprimer la dette technique (je suis resté environ un an sur le projet).
Les raisons dont je me souviens étaient :

* On ne change pas ce qui marche
* On ne peut pas supprimer/corriger des attributs car on ne sait pas comment ils sont utilisés, par exemple :
  * l'attribut `styles` utilisé à toutes les sauces
  * l'attribut `pattern` (qui était utilisé uniquement pour les dates et permettait de faire une conversion en date, mais toutes les dates n'avaient pas le même pattern)
* Des parties de l'application n'avaient pas le même style/comportement (et là, comme ça touche à l'UI/UX, c'est plus compliqué de changer niveau dev)

Je crois aujourd'hui que la source du problème était :

* Les membres de l'équipe sont peu intéressés par le front
* Ou les membres de l'équipe étaient junior, sans accompagnement d'un senior

De ce que j'en retire, pour contrer ce problème, je pense qu'il faut :

* Simplifier le code en donner des outils prêts à l'emploi 
* Réduire au maximum les possibilités techniques et ne donner que des possibilités métier épurées
* On ne peut pas faire confiance à son équipe
  * Je ne dis pas que l'équipe est mauvaise, mais que l'équipe peut ne pas être intéressée par le sujet ou tout simplement junior
  * Comme on ne peut pas faire confiance au front pour les validations back, ça signifie qu'on doit se protéger du pire

## Confusion restrictif, permissif et exhaustif

Quand j'ai débuté le développement et la création de composant, j'étais persuadé qu'il fallait être **permissif**.
On doit donner la possibilité à l'appelant de faire « ce qu'il veut ».

Exemples concrets :

* Toujours ajouter un attribut `style` (ou `className`) permettant à l'appelant de changer les styles du composant ou de faire des corrections techniques
* Toujours retourner les éléments techniques au cas où l'appelant en aurait besoin (ex: passer l'événement DOM à `onChange` et pas uniquement la valeur de l'input interne)
* Permettre d'injecter des composants au cas où (alors que non prévu à date)


Petit à petit, voyant que ça posait plus de problèmes que ça n'en résolvait, je me suis demandé s'il ne fallait pas faire l'inverse.
C'est à dire être **restrictif**.

Cette façon de penser a résolu beaucoup de problèmes. C'était mieux qu'être permissif.
Si un développeur ne peut pas faire quelque chose, alors il ne le fait pas.

Par contre, être trop restrictif peut devenir problématique.
Notamment, lorsqu'on doit répondre à un besoin, mais que les outils dont on dispose ne nous permettent pas de le faire.

Je peux mettre à jour le composant utilisé pour répondre à mon petit besoin (un petit attribut « style » par exemple), mais on redevient **permissif**.
Et très facilement, ce « petit » besoin devient la norme et la même permission est dupliquée.
Ou dans d'autres cas, ce « petit » besoin ouvre une porte trop importante et engendre une dette technique.

Ce que je présente depuis tout à l'heure sont en réalité des solutions **techniques**.
En voulant répondre au besoin immédiat, on oublie quel est vraiment le besoin.
C'est typiquement un biais que nous avons lorsqu'on utilise une solution bottom-up (cf. [Méthodes de développement](../1-methodes/01-top-down-vs-bottom-up.md)).


J'ai longtemps utilisé ce terme « restrictif » pour dire « ne pas être trop permissif » tout en essayant d'expliquer le problème ci-dessus.
C'est en commençant ce repository que j'ai pensé au terme **exhaustif**.

Il me semble permettre d'exprimer qu'il faut bien penser, réfléchir à tous les besoins.
Et si un nouveau besoin émerge, alors il faut revoir l'exhaustivité.

Autrement dit, si nous avons un bouton vert et un rouge, on aura deux variantes d'un composant.
Si pour une raison, un bouton bleu apparaît, il faut revoir le besoin et autoriser trois variantes.
C'est une solution top-down: quel est le besoin et comment je l'implémente.

Il ne faut pas permettre au départ de changer les couleurs (être trop **permissif**),
ni s'interdire de changer l'API ou « juste un peu ici » (être trop **restrictif**, amène à une solution bottom-up).

Il faut bien énoncer le besoin et y répondre en fonction.
