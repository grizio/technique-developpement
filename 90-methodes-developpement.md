# Méthodes de développement

Dans cette page, je vais énoncer quelques méthodes de développement.
Elles vous paraîtront peut-être évidentes mais cela vous permettra peut-être de mettre des mots sur cette évidence.

## Top-down vs Bottom-up

Lors de la création ou la modification d'un code, nous avons souvent deux manières générales de traiter le problème :

* Top-down :
  * Partir du besoin fonctionnel et aller dans le détail technique
  * Ou partir de la fonction appelante et impacter la fonction appelée (récursif)
* Bottom-up :
  * Partir d'une contrainte technique et impacter le besoin fonctionnel
  * Ou partir de la fonction appelée et impacter la fonction appelante (récursif)

Pour comprendre ces deux approches, je vous propose l'exemple suivant :

> Nous souhaitons réaliser un processus qui :
> 
> 1. Récupère un dossier
> 2. Y ajoute une information
> 3. Valide la conformité du dossier
> 4. Envoi le dossier à un service
> 5. Sauvegarde le dossier en base de données

### Top-down

Si nous souhaitons expliciter au mieux ce besoin, nous allons tenter d'avoir une approche top-down.
Remarque: nous ignorons toutes les promises et gardons un aspect « synchrone » pour expliquer le principe sans complexité technique.

```typescript
// application.ts
app.post("/dossier/:id", authenticateUserMiddleware, (request, response) => {
  const dossierId = request.params.id
  const information = request.body
  
  processus(dossierId, information)

  // On devrait gérer les exception, mais inutile pour l'exemple
  response.send(200, "OK")
})

// processus.ts
import * as service from "./service"

function processus(dossierId: string, information: string): Dossier {
  const dossier = getDossier(dossierId)
  const updatedDossier = updateDossier(dossier, information)
  if (validateDossier(updatedDossier)) {
    service.sendDossier(updatedDossier)
    sauvegardeDossier(updatedDossier)
    return dossier
  } else {
    throw "Invalid dossier"
  }
}

function validateDossier(dossier: Dossier): boolean {
  return dossierEstOuvert(dossier) && informationEstAcceptable(dossier.information)
}

function dossierEstOuvert(dossier: Dossier): boolean {
  return dossier.dateFermeture === undefined
}

function informationEstAcceptable(information: string): boolean {
  return informationEstRemplie(information) && informationRespectePattern(information)
}

function informationEstRemplie(information: string): boolean {
  return information.length > 0
}

function informationRespectePattern(information: string): boolean {
  return (/[a-zA-Z0-9]+/).test(information)
}

// […]

// service.tis
export function sendDossier(dossier: Dossier): void {
  fetch("http://servi.ce/dossier", {
    method: "POST",
    body: JSON.stringify(dossier)
  })
}
```

Sans regarder le détail des fonctions, vous pouvez comprendre le processus de haut niveau (`function processus`).
On peut ensuite aller dans le détail et recommencer de manière récursive.

Ici, on va donc définir le besoin de haut niveau, puis détailler chaque besoin sur plusieurs niveaux.
Chaque étape peut bien sûr avoir un nombre de niveaux différent, selon le détail à accorder à chaque niveau.

Par rapport au snippet, nous écrivons les fonctions de haut en bas.
En pratique, on peut aussi dire qu'on écrit le code des couches supérieures vers les couches inférieures.
D'où le terme top-down.

Cette manière de développer permet *généralement* de mettre en évidence les **besoins**.

### Bottom-up

Maintenant, imaginons que notre service nécessite d'utiliser un token lié à l'utilisateur pour déclencher sa requête (parce que pourquoi pas).
On va devoir faire le travail inverse :

```typescript
// applicationRouter.ts
app.post("/dossier/:id", authenticateUserMiddleware, (request, response) => {
  const dossierId = request.params.id
  const information = request.body
  const authenticatedUser = request.authenticatedUser // avec le middleware authenticateUserMiddleware
  
  // On injecte authenticatedUser
  processus(dossierId, information, authenticatedUser)

  // On devrait gérer les exception, mais inutile pour l'exemple
  response.send(200, "OK")
})

// processus.ts
import * as service from "./service"

// On récupère authenticatedUser
function processus(dossierId: string, information: string, authenticatedUser: User): Dossier {
  const dossier = getDossier(dossierId, user)
  const updatedDossier = updateDossier(dossier, information)
  if (validateDossier(updatedDossier)) {
    // On injecte authenticatedUser
    service.sendDossier(updatedDossier, authenticatedUser)
    sauvegardeDossier(updatedDossier)
    return dossier
  } else {
    throw "Invalid dossier"
  }
}

// […]

// service.tis
// On récupère authenticatedUser
export function sendDossier(dossier: Dossier, authenticatedUser: User): void {
  fetch("http://servi.ce/dossier", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${authenticatedUser.token}`
    },
    body: JSON.stringify(dossier)
  })
}
```

Par rapport au snippet, nous avons commencé par écrire le code en bas, puis remonté les besoins aux fonctions supérieures.
En pratique, on peut aussi dire qu'on met à jour le code des couches inférieure et impacte les couches supérieures.
D'où le terme bottom-up.

Cette manière de développer permet *généralement* de mettre en évidence les **contraintes techniques**.

### Autres remarques

Quand on écrit la première fois le code, il est souvent simple d'appliquer la méthode top-down.
Après tout, on a un besoin initial, autant partir de ce dernier.

Par contre, quand on met à jour le code, on peut tomber dans plusieurs contextes.
Ci-dessous quelques contextes identifiés.

#### Ajout d'une étape

Le besoin exprimé met en évidence l'ajout d'une étape dans le processus.
Dit autrement, on va insérer un nouveau sous-processus dans l'existant.

Lorsque nous somme dans ce contexte, on va souvent chercher à développer en mode top-down en partant de l'étape (ou sous-étape) concernée.
Ainsi, nous continuons à exprimer le besoin dans le code.

Pour reprendre l'exemple, ça pourrait être l'appel de `service.sendDossier` qui serait ajouté à postériori.

#### Ajout d'une contrainte technique - réponse technique

Cette fois-ci, le besoin exprimé met en évidence une contrainte technique, mais qui ne change pas le processus.
Dit autrement, la structure du code ne va pas évoluer, mais l'appelé va requérir plus d'information.

C'est typiquement l'exemple donné dans la version `bottom-up`.
À part remonter les contraintes techniques, il est assez complexe (et souvent inutile) de modifier le processus.

#### Ajout d'une contrainte technique - réponse fonctionnelle

Ce cas est plus complexe à mettre en évidence.
Adaptons notre exemple initial :

```typescript
function processus(dossierId: string, fournisseurId: string): Dossier {
  const dossier = getDossier(dossierId)
  const updatedDossier = updateFournisseurDossier(dossier, fournisseurId)
  if (validateDossier(updatedDossier)) {
    service.sendDossier(updatedDossier)
    sauvegardeDossier(updatedDossier)
    return dossier
  } else {
    throw "Invalid dossier"
  }
}

function validateDossier(dossier: Dossier): boolean {
  return dossier.fournisseurId !== undefined && dossier.fournisseurId.length !== 0
}
```

Notre travail cette fois-ci sera de valider le besoin suivant : S'assurer que le fournisseur existe en base de données.

Une approche bottom-up ressemblerait à :

```typescript
function processus(dossierId: string, fournisseurId: string): Dossier {
  const dossier = getDossier(dossierId)
  const updatedDossier = updateFournisseurDossier(dossier, fournisseurId)
  if (validateDossier(updatedDossier)) {
    service.sendDossier(updatedDossier)
    sauvegardeDossier(updatedDossier)
    return dossier
  } else {
    throw "Invalid dossier"
  }
}

// Pas besoin de passer plus d'information, car dossier contient déjà tout.
function validateDossier(dossier: Dossier): boolean {
  return dossier.fournisseurId !== undefined && dossier.fournisseurId.length !== 0
    // On rajoute la validation de l'existence ici
    && fournisseurExists(dossier.fournisseurId)
}
```

Cette solution peut sembler satisfaisante.
Cependant, elle a le défaut (si c'en est un) de faire un appel DB au milieu des validations.

Peut-être qu'on peut résoudre le problème différemment en top-down :

```typescript
function processus(dossierId: string, fournisseurId: string): Dossier {
  const dossier = getDossier(dossierId)
  // On réalise la vérification au plus haut niveau, c'est une partie du processus
  if (fournisseurExists(fournisseurId)) {
    const updatedDossier = updateFournisseurDossier(dossier, fournisseurId)
    if (validateDossier(updatedDossier)) {
      service.sendDossier(updatedDossier)
      sauvegardeDossier(updatedDossier)
      return dossier
    } else {
      throw "Invalid dossier"
    }
  }
}

function validateDossier(dossier: Dossier): boolean {
  return dossier.fournisseurId !== undefined && dossier.fournisseurId.length !== 0
}
```

Quelle est la meilleure solution ici ?

Réponse courte : Ça dépend du contexte.

Réponse longue: il faut prendre en compte plusieurs considérations :

* Separation Of Concern / Single Responsibility Principle
  * Un appel DB pendant une validation est-il acceptable dans le projet ?
* Qu'est-ce qui est important ?
  * Qu'est-ce qui fait partie du processus principal ?
  * Qu'est-ce qui fait partie d'un sous-processus ?
  * Dans quel sous-processus le besoin doit-il s'exprimer ?


Très souvent, notre réflexe sera d'avoir une approche bottom-up car plus simple à écrire et remet en question peu de code.
L'accumulation de cette stratégie peut conduire très rapidement à de la dette technique et complexifier le processus.

À ce jour, je n'ai pas de recette magique à vous conseiller à part mettre en évidence ces deux pratiques
et espérer qu'elles vous servent à mieux comprendre les impacts et les réponses auxquelles elles répondent.

### #MyLife Expérience personnelle de ces deux approches

Personnellement, j'ai compris les dérives d'une accumulation bottom-up sur un projet réel.
Je suis arrivé sur ce projet après deux ans de développement dessus (et après une pause, j'étais à ce moment le seul développeur).
Nous avions dedans une fonction très complexe à comprendre (200 lignes pour la fonction principale et +1000 lignes pour tous les sous-processus).

Premier réflexe: c'est quoi ce code « pas génial » ? je comprends pas grand chose !

![WTF / minute](https://i2.wp.com/commadot.com/wp-content/uploads/2009/02/wtf.png?w=550)

En regardant l'historique git associé à la fonction, chaque changement était pertinent et était le changement intuitif que j'aurai sûrement fait.

Nous avons donc quelque chose d'étrange :

* Tous les changements semblent satisfaisants
* Le résultat est insatisfaisant

À quel moment y a-t-il donc eu un problème ?

En regardant dans le détail, les changements étaient majoritairement réalisés de manière bottom-up.
D'une contrainte technique, nous avons remonté les besoins techniques.
Parfois, c'était justifié, parfois, ça augmentait la complexité et on perdait le besoin.

Plus le code était complexe, plus les changements étaient bottom-up. Le risque de revoir le processus devenait critique.
De même, quand j'ai repris le projet, mes premiers changements étaient tous bottom-up, car trop risqués de les implémenter top-down.
