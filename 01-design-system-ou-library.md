# Design System ou library

Quelle est la différence entre un DS et une library ?

## Cas d'utilisation

Pour essayer de répondre à cette question, nous allons concevoir une library de modal de confirmation.
Le but de cette library est de permettre au développeur :

* d'afficher une modal de confirmation
* de réaliser une action selon si l'utilisateur clique sur « confirmer » ou « annuler »

Tout au long de cette partie, nous ne verrons que la partie API (=les types), sans l'implémentation.

Il y aura à chaque fois de nombreuses alternatives d'API possibles, nous en proposons une, pas la meilleure, ni la pire.

## Définition de la library

### 1. Cas nominal

Commençons avec l'interface la plus simple :

```typescript jsx
type Props = {
  message: string
}
export function confirmModal(props: Props): Promise<void>
```

Lorsque nous appelons la fonction, ça va :

* afficher la modal et bloquer l'écran en attente d'une réponse
* renvoyer la réponse sous la forme d'une `Promise` (`resolve` = confirmé, `reject` = annulé)

### 2. Respecter les pratiques des différents développeurs

Certains développeurs dans la communauté JS ne sont pas habitués aux Promises.
Nous pouvons alors ajouter une autre version :

```typescript jsx
type Props = {
  message: string
}
export function confirmModal(props: Props): Promise<void>
export function confirmModal(props: Props, onConfirm: () => void, onCancel: () => void): void
```

Ainsi, nous repondons aux besoins de ces développeurs.

### 3. De nouveaux attributs optionels

Maintenant, le besoin d'ajouter potentiellement un titre apparaît et changer le texte des boutons d'action.
Nous changeons alors notre API ainsi :

```typescript jsx
type Props = {
  message: string
  title?: string
  confirmLabel?: string
  cancelLabel?: string
}
export function confirmModal(props: Props): Promise<void>
export function confirmModal(props: Props, onConfirm: () => void, onCancel: () => void): void
```

### 4. Styles de l'overlay

Avoir un overlay pour bloquer l'écran de l'utilisateur est plutôt une bonne chose.
Par contre, il serait intéressant de pouvoir en modifier la couleur pour être plus en phase avec notre charte graphique.

```typescript jsx
type Props = {
  message: string
  title?: string
  confirmLabel?: string
  cancelLabel?: string
  overlayColor?: string
}
export function confirmModal(props: Props): Promise<void>
export function confirmModal(props: Props, onConfirm: () => void, onCancel: () => void): void
```

### 5. Ajout d'animation

Un affichage brusque de la modal n'est pas forcément intéressant.
On peut surement ajouter des animations.
Par contre, laquelle ?
Il existe bien des animations classiques, qu'on peut intégrer, mais peut-être qu'il en faudra des plus complexes selon les projets.

Laissons donc le choix au développeur :

```typescript jsx
type BuiltInAnimations = "fadeIn" | "slide-top" | "…"
type CustomAnimations = () => void
type Props = {
  message: string
  title?: string
  confirmLabel?: string
  cancelLabel?: string
  overlayColor?: string
  animation?: BuiltInAnimations | CustomAnimations
}
export function confirmModal(props: Props): Promise<void>
export function confirmModal(props: Props, onConfirm: () => void, onCancel: () => void): void
```

### 6. Plusieurs styles de modal

Nous pouvons avoir besoin de plusieurs styles de modal de confirmation dans nos applications :

* Classique: Confirmer simplement une action
* Warning: L'action a des conséquences importantes
* Danger: L'action a des conséquences irréversibles
* …

En tant que concepteur de library, je ne peux pas penser à tous les cas.
De même, le rendu pourra aussi dépendre de la charte graphique.

Pour répondre à ce besoin, je vais proposer la nouvelle API suivante :

```typescript jsx
type BuiltInAnimations = "fadeIn" | "slide-top" | "…"
type CustomAnimations = () => void
type Props = {
  message: string
  title?: string
  confirmLabel?: string
  cancelLabel?: string
  animation?: BuiltInAnimations | CustomAnimations
  // ça peut être un object, mais restons simple, le type ici n'est pas pertinent
  titleStyles?: string
  messageStyles?: string
  modalStyles?: string
  // remplace overlayColor
  overlayStyles?: string
}
export function confirmModal(props: Props): Promise<void>
export function confirmModal(props: Props, onConfirm: () => void, onCancel: () => void): void
```

### 7. Contenu complexe

Jusqu'à maintenant, le contenu de notre modal était plutôt simple: un titre optionel et un message.
Par contre, cela peut ne pas être suffisant.
Certains développeurs souhaitent pouvoir afficher un contenu plus complexe (listes à puces, images, …).

Voici une solution pour y répondre :

```typescript jsx
type BuiltInAnimations = "none" | "fadeIn" | "slide-top" | "…"
type CustomAnimations = () => void
type Props = {
  // remplace message - on peut aussi utiliser le type de React
  children: string | React.Element
  title?: string
  confirmLabel?: string
  cancelLabel?: string
  animation?: BuiltInAnimations | CustomAnimations
  titleStyles?: string
  messageStyles?: string
  modalStyles?: string
  overlayStyles?: string
}
export function confirmModal(props: Props): Promise<void>
export function confirmModal(props: Props, onConfirm: () => void, onCancel: () => void): void
```

### 8. Et le spectacle continue !

Nous allons nous arrêter ici, nous avons déjà couvert de nombreux cas d'utilisation mais ce travail peut continuer encore et encore et n'être que le début d'accord d'accord.

## Et dans le cas d'un Design System ?

Revenons maintenant à notre Design System.
Nous allons revoir l'ensemble des étapes de la partie précédente et vérifier :

* leur pertinence
* la solution technique proposée
* les alternatives techniques et conceptuelles

Nous allons définir une nouvelle API (ou pas) selon les hypothèses que l'on posera.

Rappel: Je respecte les [hypothèses posées](00-hypotheses.md) quand je parle de DS.
Ces hypothèses ont un **impact fort** sur le choix des solutions ci-dessous.

### 1. Cas nominal

Dans ce cas, on peut difficilement faire autrement, donc gardons l'API telle quelle :

```typescript jsx
type Props = {
  message: string
}
export function confirmModal(props: Props): Promise<void>
```

### 2. Respecter les pratiques des différents développeurs

Dans ce cas, est-ce qu'on veut :

* Favoriser plusieurs pratiques ?
* Imposer une seule pratique ?

Si on reprend nos [hypothèses](00-hypotheses.md), nous sommes dans un contexte où l'homogénéité est plus favorable.
Ce qui signifie qu'il est pertinent de ne garder qu'une version.

Chez nous, nous favoriserons plus les `Promises` que les callback.

L'API reste inchangée :

```typescript jsx
type Props = {
  message: string
}
export function confirmModal(props: Props): Promise<void>
```

### 3. De nouveaux attributs optionels

Les attributs ajoutés à ce niveau sont plutôt courant.
Cependant, doivent-ils tous être optionnels ?

Par exemple, dans notre contexte, `confirmLabel` et `cancelLabel` auront généralement les mêmes labels (« OK », « Annuler »).
Par contre, on veut pouvoir les modifier (« Oui, je confirme vouloir supprimer cet utilisateur », « Non, c'est une erreur »).

Quant au titre, la question n'est pas aussi simple.
Souhaitons-nous avoir des modales avec titre et d'autres sans titre ?
Il existe *peut-être* des cas où nous voudrions les deux versions.
Mais tant que ces cas n'existent pas, devons-nous ajouter cette possibilité ?
Et si cette possibilité existe (ex: dans les maquettes), est-ce bien un choix conscient, réfléchi et pas une erreur ?

Autant de questions qu'il faut d'abord répondre avant de choisir la solution.

Dans le doute, il est préférable de vérifier l'existant et de ne choisir qu'une seule des deux versions.
Ici, nous supposons que nous n'avons pas besoin de titre, nous allons donc utiliser la version sans.

```typescript jsx
type Props = {
  message: string
  confirmLabel?: string
  cancelLabel?: string
}
export function confirmModal(props: Props): Promise<void>
```

### 4. Styles de l'overlay

Nous souhaitons avoir une homogénéisation envers l'ensemble des projets ([cf. hypothèses](00-hypotheses.md)).

Pourquoi aurions-nous besoin d'avoir plusieurs styles de celui-ci ?

Autant ne pas l'ajouter dans l'API et garder la même que précédemment.

```typescript jsx
type Props = {
  message: string
  confirmLabel?: string
  cancelLabel?: string
}
export function confirmModal(props: Props): Promise<void>
```

### 5. Ajout d'animation

Nous sommes dans le même cas de figure que précédemment : Y a-t-il une pertinence d'avoir plusieurs animations ?
Le cas échéant, avons-nous besoin de combien de types d'animations ?

Supposons que nous avons deux variantes identifiées :

* Fade in après déclenchement d'une action utilisateur synchrone (ex: clique sur un bouton)
* Aucune après déclenchement d'une action asynchrone (ex: retour serveur)

Ces deux variantes sont identifiées et peuvent donc être implémentées :

```typescript jsx
type Props = {
  message: string
  confirmLabel?: string
  cancelLabel?: string
  animation?: "none" | "fadeIn"
}
export function confirmModal(props: Props): Promise<void>
```

### 6. Plusieurs styles de modal

Pour voir comment répondre à ce besoin, revenons… au besoin.

> Nous pouvons avoir besoin de plusieurs styles de modal de confirmation dans nos applications :
> * Classique: Confirmer simplement une action
> * Warning: L'action a des conséquences importantes
> * Danger: L'action a des conséquences irréversibles
> * …

En fait, nous identifions explicitement des variantes à utiliser.
Ces variantes sont déjà nommées (`classique`, `warning`, `danger`) et identifie des cas d'utilisation.
Autant en profiter et les réutiliser.

```typescript jsx
type Props = {
  message: string
  confirmLabel?: string
  cancelLabel?: string
  animation?: "none" | "fadeIn"
  // combine en interne titleStyles, messageStyles, modalStyles, overlayStyles
  variante?: "warning" | "danger"
}
export function confirmModal(props: Props): Promise<void>
```

Si nous identifions une nouvelle variante, rien ne nous empêchera de l'ajouter à posteriori.

### 7. Contenu complexe

Nous arrivons à notre dernière modification.
Pour y répondre, nous allons de nouveau vérifier les besoins.

* Soit nous avons besoin des contenus complexes
* Soit nous n'en avons pas besoin

Dans le premier cas, l'API ne change pas.
Dans le second cas, l'API change comme pour la partie « library » :

```typescript jsx
type Props = {
  // remplace message - on peut aussi utiliser le type de React
  children: string | React.Element
  confirmLabel?: string
  cancelLabel?: string
  animation?: "none" | "fadeIn"
  variante?: "warning" | "danger"
}
export function confirmModal(props: Props): Promise<void>
```

### 8. Et le spectacle continue !

La même méthode peut être utilisée pour tous les autres cas d'utilisation.

## Résumé

Concevoir un *Design System* est très différent de concevoir une *library*.
Nous ne pouvons pas obtenir le même résultat car nous ne répondons pas aux mêmes besoins (du moins avec les [hypothèses posées](00-hypotheses.md)).

Chaque situation est particulière et n'a pas forcément de bonne réponse et chaque réponse émise ici pourrait être invalidée selon le contexte.
Cependant, nous pouvons émettre deux axes de réflexion spécifiques :

* Dans le cas d'une library, on doit répondre à des **besoins non identifiés** et donc être **permissif**
* Dans le cas d'un Design System, on doit répondre à des **besoins identifiés** et donc être **exhaustif**

Je fais bien une distinction entre **permissif** et **exhaustif**.
Ces deux termes peuvent sembler proches mais ne sont pas identiques.

> Permissif
> Tolérant à l'égard des comportements non conformes, sans contraintes, sans interdits, indulgent, complaisant.
> — https://www.linternaute.fr/dictionnaire/fr/definition/permissif/ 


> Exhaustif:
> Qui traite un sujet dans sa totalité, de manière complète, sans rien oublier.
> — https://www.linternaute.fr/dictionnaire/fr/definition/exhaustif/


Le fonctionnement et la conception qui en découle est donc opposée :

* Permissif : comment donner les billes techniques pour définir de nouveaux besoins
* Exhaustif : depuis l'ensemble des besoins, comment les décrire techniquement


## Contre-argumentations

Dans cette partie, je vais essayer de mettre à l'épreuve cette conception.
N'hésitez pas à mettre en défaut mes réflexions.

### Contexte projet vs contexte DS

Certains pourraient noter :

> C'est un processus à faire pour un projet, pas pour un Design System.
> Le contexte est différent.

Si j'essaye de formaliser l'argumentation (en essayant d'éviter l'homme de paille), j'obtiens :

* Le processus donné est pour un projet
* Nous ne sommes pas dans un contexte projet
* Donc le processus doit être différent

Si c'est bien cette argumentation que vous soutenez, la logique est malheureusement invalide.
La conclusion peut être juste, mais l'argument ne l'est pas.
Pour plus d'information : https://skeptikon.fr/videos/watch/11799932-f73c-421a-8b4e-e0b97219c643 

Pour répondre à ce point, je vous réfère encore une fois aux [hypothèses](00-hypotheses.md).
Comme énoncé, nous sommes dans un contexte où l'ensemble des projets sont connus.
Nous pouvons donc traiter le Design System comme un projet et l'ensemble des projets comme un macro-projet.

Dans ce cas, si vous considérez que le processus est à faire (ou faisable) pour un projet, pourquoi ne serait-il pas à faire (ou faisable) dans le Design System ?


### Contexte DS vs contexte library

On peut aussi dire :

> Le Design System sera utilisé comme une library

Je suis d'accord, c'est techniquement une library de composants Figma et npm (ou autres outils design et dev).
Mais là, ne confondons-nous pas deux sujets : la conception et l'implémentation ?
Nous pouvons aussi parler de l'organisation, du cycle de vie des projets, versionning, etc.

Encore une fois, quel est l'objectif du Design System ?
Est-ce que c'est apporter de la valeur métier aux clients et utilisateurs ou est-ce d'avoir une interface spécifique pour chaque application ?
Avons-nous besoin de pouvoir paramétrer **fortement** le Design System ?
Souhaitons-nous avoir des interfaces spécifiques, homogènes ?

Il n'y a pas de réponse universelle et il n'est pas possible d'en donner une qui ne sera pas invalidée selon les objectifs.

Voici quelques pistes de réflexion (non-exhaustives bien sûr) :

* Le client et les utilisateurs veulent-ils des interfaces homogènes ?
* Est-ce que l'UI est plus importante que les processus métier (et la réponse peut être oui, même si ça semble non intuitif dit ainsi) ?
* Comment est piloté (design, dev et projet) le Design System ?
* Y a-t-il des composants différents sur les applications et pourquoi ?
* Y a-t-il des besoins (très) différents, voire incompatibles entre les applications et pourquoi ?

La structure du Design System sera à adapter à votre contexte.
D'ailleurs, peut-être que ce n'est pas un Design System dont vous avez besoin, mais de libraries de composants.

### On ne peut pas répondre à tous les besoins

J'ai plein de contre-exemples :

* Comment je peux corriger un bug technique qui n'est que sur mon projet ?
* Comment je peux modifier les styles pour un cas particulier sur mon projet ?
* …


Ce sont des questions très intéressantes, mais y répondre nécessite de poser des hypothèses.

Pour trouver une réponse, je vous conseille toujours de vous poser les questions :

* Quel est l'objectif souhaité
* Quels sont les impacts possibles, positifs et négatifs (non traité ici mais très important, j'essayerai de détaillé ce point)
* Y a-t-il une manière de gérer le besoin par conception









