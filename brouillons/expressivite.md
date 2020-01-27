# Être le plus expressif

Lorsque je lis le code existant, je souhaite pouvoir accéder rapidement à l'information intéressante.
Je ne souhaite pas perdre du temps sur des informations secondaires ou négligeables.

Je n'ai à ce jour pas trouvé la forme idéale s'appliquant à l'ensemble des contextes.
Je pense que ce jour n'arrivera pas et c'est d'ailleurs un aspect de notre métier qui m'intrigue.

Partons d'un exemple pour illustrer mon propos.
J'ai une barre d'actions contenant trois boutons : annuler, sauvegarder et envoyer.

Quelle est la version du code me permettant de mettre le plus en avant les actions ?

## Exemples

### Version 1

```typescript jsx
function ActionsBar({ cancel, save, send, sendDisabled }: Props) {
  return (
    <>
      <Button className="primary medium" onClick={cancel}>
        Cancel
      </Button>
    
      <Button className="secondary medium icon-save" onClick={save}>
        Save
      </Button>
    
      <Button
          className={"primary medium icon-send " + (sendDisabled ? "disabled" : "")}
          onClick={send}
      >
        Send
      </Button>
    </>
  )
}
```

Remarque: L'objectif n'est pas de savoir si `disabled` pourra désactiver effectivement le `Button`, mais de regarder la lisibilité.

### Version 2

```typescript jsx
function ActionsBar({ cancel, save, send, sendDisabled }: Props) {
  return (
    <>
      <Button type="secondary" size="medium" onClick={cancel}>
        Cancel
      </Button>
    
      <Button type="secondary" size="medium" icon="save" onClick={save}>
        Save
      </Button>
    
      <Button
          type="primary"
          size="medium"
          icon="send"
          disabled={sendDisabled}
          onClick={send}
      >
        Send
      </Button>
    </>
  )
}
```

### Version 3

```typescript jsx
function ActionsBar({ cancel, save, send, sendDisabled }: Props) {
  return (
    <>
      <MediumSecondaryButton onClick={cancel}>
        Cancel
      </MediumSecondaryButton>
    
      <MediumSecondaryButton icon="save" onClick={save}>
        Save
      </MediumSecondaryButton>
    
      <MediumPrimaryButton
          icon="send"
          disabled={sendDisabled}
          onClick={send}
      >
        Send
      </MediumPrimaryButton>
    </>
  )
}
```

## Réflexions

Notre réflexion ici va tourner autour de la lisibilité du code (et uniquement de celle-ci).
Que pouvons-nous dire de chacun des trois exemples ?


Pour la première version, nous avons l'élément principal (`Button`), ses caractéristiques juxtaposées, puis l'action qu'il effectue.
Pour les cas simples, c'est assez facile de trouver l'information principale, les caractéristiques et l'action impliquée.
Cependant, pour les cas complexes (ici une conditions), le calcul des caractéristiques se fait via des constructions de chaînes et est moins simple à lire.
Lorsque nous connaissons les différentes variantes (ici type, taille et désactivé), c'est rapide à interprêter.
Toutefois, plus le nombre de variantes augmente, plus il sera compliqué de toutes les avoir en tête.

Pour la seconde version, nous avons l'élément principal (`Button`), ses caractéristiques explicitées, puis l'action qu'il effectue.
Il est plus facile de comprendre l'intérêt de chaque caractéristique car elles sont toutes nommées.
Même avec un nombre plus important et des constructions conditionnelles, il est simple de raisonner car elles sont toutes isolées dans des attributs spécifiques.
Nous pouvons alors avoir des variantes plus importantes sans que la complexité en souffre de trop.

Pour la troisième version, nous avons deux caractéristiques (`Medium` et `Primary`), l'élément principale (`Button`), d'autres caractéristiques (`icon` et `disabled`) puis l'action.
Il semble plus complexe de trouver l'élément principal que les deux autres versions, le format venant de l'anglais où les adjectifs sont devant le nom.
J'ai été inspiré à utiliser ce genre de naming via les [code styles dart](https://dart.dev/guides/language/effective-dart/design#prefer-putting-the-most-descriptive-noun-last).
Si vous respectez toujours ce format, on peut aisément apprendre à lire « de droite à gauche ».
Cependant, il y a dans le nom deux caractéristique et l'élément principal.
Ensuite, nous avons comme dans la deuxième version les autres caractéristiques sous la forme d'attributs explicites et ses avantages.
Par contre, il sera difficilement possible de construire des variantes pour les deux premières caractéristiques.


Cependant, est-ce que ça nous aide à vraiment choisir une forme plutôt qu'une autre ?
Est-ce que la description de la structuration nous permet de déterminer quoi choisir ?
La réalité est bien sûr plus complexe.

À choisir, je prendrais la troisième version pour ces raisons :

* Je lis très vite que c'est un `Button` car je suis habitué à ce code style pour le naming
* L'espace visuel pour les caractéristiques UI obligatoires est réduit et en dehors des autres caractéristiques
  * Ceci est d'autant plus vrai pour le troisième `Button` avec 5 caractéristiques.
* Ce qui est le plus important, pour moi, est l'action (`onClick`) associée à chaque `Button` et le texte (`children`)
  * Dans le premier cas, c'est d'ailleurs les seuls éléments mis en évidence
* Le code résultant est concis et explicite

Pourquoi mettre moins d'importance sur les caractéristiques UI ?
Ces dernières sont généralement connues car on a une image de l'application et on sait plus ou moins implicitement quelles sont ces caractéristiques.
Ce qui m'intéresse, c'est trouver le `Button` qui déclenche une action (`onClick`).
Je l'identifie soit par le nom du bouton (son `children`) ou le nom de l'action (`onClick`).
Dans la lecture de tous les jours, les caractéristiques UI sont des configurations nécessaires mais avec une valeur moindre.

Remarque, nous pouvons nous permettre d'avoir ce genre de naming (`SizeTypeButton`) car :
* La taille et le type sont obligatoires
* le nombre de possibilités est faible (dans la plupart de mes projets, on avait 3 tailles * 3 types = 9 noms).


Mes conclusions sur cet aspect :

* Définir quelle est l'importance des différentes caractéristiques mises en jeu et leurs priorité
  * Quel est l'objectif du composant et son utilisation (ex: button = déclencher une action) ==> grande priorité
  * Quelles sont les aspects UI obligatoires et aisément identifiables (ex: type et taille) ==> faible priorité
* Réfléchir à comment permettre de mettre en avant les caractéristiques les plus importantes
* Éviter d'avoir des constructions complexes (ex: `disabled` du premier exemple)