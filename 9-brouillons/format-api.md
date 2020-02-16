# Format d'API

On peut également se poser la question du format de notre DS.
Ce format est également une part importante, car elle va influencer la conception du DS.

Revenons à nous boutons.
Nous allons simplifier le besoin et supposer que nous n'avons qu'un seul format de bouton.
Cependant, ce bouton peut être soit un `<button onClick={function}>`, soit un `<a href="link">`.
L'action associé au bouton ne peut donc pas être la même dans ces deux cas.

Quelle version d'API vous semble la meilleure ?

## Version 1 (API)

```typescript jsx
type ButtonProps = {
  disabled?: boolean
  icon?: string
  onClick: () => void
}
function Button(props: ButtonProps)

type ButtonLinkProps = {
  disabled?: boolean
  icon?: string
  href: string
}
function ButtonLink(props: ButtonLinkProps)


// Utilisation
<Button onClick={() => console.log("click")} />
<Link href="/some/where" />
```

## Version 2 (API)

```typescript jsx
type ButtonProps = {
  disabled?: boolean
  icon?: string
} & ActionProps

type ActionProps = ActionButtonProps | ActionLinkProps
type ActionButtonProps = { onClick: () => void }
type ActionLinkProps = { href: string }

function Button(props: ButtonProps)


// Utilisation

<Button onClick={() => console.log("click")} />
<Button href="/some/where" />

// Impossible:
<Button href="/some/where" onClick={() => console.log("click")} />
```

Une alternative ici serait d'autoriser `onClick` et `href` en les rendant tous deux optionnel, mais on casse l'aspect bug-free-by-design.

## Version 3 (API)

```typescript jsx
type ButtonProps = {
  disabled?: boolean
  icon?: string
  action: string | () => void
}
function Button(props: ButtonProps)


// Utilisation

<Button action={() => console.log("click")} />
<Button action="/some/where" />
```

## Réflexions

La première version sépare bien les deux cas possibles et est plutôt explicite.
Cependant, on se retrouve avec deux composants qui sont visuellement identiques et ont la majorité de leurs priopriétés en commun.

La deuxième version est plus complexe pour la déclaration de l'API.
À l'utilisation, cependant, nous gardons un même composant et ne pouvons utiliser qu'une des deux actions.
Selon l'action utilisée (`onClick` ou `href`), nous obtiendrons un `<button>` ou un `<a>` de manière explicite.

La troisième version est une version simplifiée de la deuxième.
Au lieu d'utiliser 4 types composés, nous n'en gardons qu'un seul et aggrégeons les actions en une seule (`onClick` + `href` => `action`).
Cette action est soit un lien (`string`), soit un listener (`() => void`).
Selon le type de l'action fourni, nous obtiendrons un `<button>` ou un `<a>` de manière implicite.


En ce qui me concerne, la version 2 est celle que je préfère le plus.
Nous avons un seul composant visuel nommé et nous définissons ensuite les cas d'usage autorisés.
Nous gardons un attribut explicite qui nous permet de définir le comportement sous-jacent à la lecture.

Mais ce n'est pas l'API que j'utiliserai dans mes projets professionnels.
L'API est très complexe et il faut déployer trop d'énergie intellectuelle pour comprendre l'ensemble des cas d'usage pour la valeur que cela apporte.

Je n'utiliserai pas non plus la version 3.
Celle-ci a le défaut de ne pas permettre de rajouter des attributs spécifiques, tel que `download` pour les `<a>`.
De même, aggréger des types dans des attributs peut également être complexe à voir et comprendre.

Ce qui nous ramène à la version 1 par élimination.
Elle est sûrement la plus verbeuse et elle explicite qu'il y a deux composants distincts.
Seul le nom peut donner l'indice que ces deux composants sont visuellement identiques.


Mes conclusions sur cet aspect :

* Ce que je considère comme étant le meilleur choix ne l'est pas forcément pour l'équipe entière… et l'équipe future
  * Prendre en compte les débutants (juniors ou ceux qui commencent Typescript)
  * Prendre en compte les nouveaux arrivants (cette API est une parmi d'autres et la complexité ne fait que croître)
* Ne pas chercher à toujours masquer la technique
  * Si par conception, on doit avoir deux éléments, créer deux composants