# Idées en vrac

* Éviter l'attribut className
    * C'est cool, ça permet de tout faire et corriger des bugs ou gérer des cas simples sans créer d'autres éléments html
    * Peut-on faire confiance à tous les développeurs ?
    * Si on a besoin de modifier la structure interne du composant, comment va se comporter le className ?

Exemple:

```
type Props = {
  value: string
  maxLength: number
  className: string
}

// Avant
function Textarea({ value, maxLength, className }: Props) {
  return <textarea maxLength={ maxLength } className={ className }>{ value }</textarea>
}

// Après
// Où placer le className ?
// Sur le textarea ?
// Sur la div encapsulante ?
// Avec `border: red` ?
// Avec `align-self: right` ?
function Textarea({ value, maxLength, className }: Props) {
  return <div>
    <textarea maxLength={ maxLength }>{ value }</textarea>
    <span>{ value.length } / { maxLength }</span>
  </div>
}
```
