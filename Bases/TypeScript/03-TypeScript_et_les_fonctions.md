# TypeScript et les fonctions

## Typer les arguments et la valeur de retour

### Déclarer des fonctions

Il n'y a pas de différence en *TypeScript* sur la déclaration de fonctions : vous pouvez les déclarer de manière anonyme ou nommée :

```ts
function maFonction() {
}

const maFonction = () => {};
```

### Typer les arguments

En *TypeScript*, il est fortement recommandé de typer tous les paramètres de vos fonctions :

```ts
function maFonction(param: type) {
  return valeur;
}
```

Vous pouvez typer autant de paramètres que nécessaire :

```ts
function multiplier(x: number, y: number) {
    return x * y;
}
```

### Typer la valeur de retour

Pour typer la valeur de retour d'une fonction il faut faire :

```ts
function maFonction(param: type): type {
  return valeur;
}
```
Remarquez le : *type* après les parenthèses de la déclaration des arguments, c'est le type de la valeur de retour.

Par exemple :

```ts
function multiplier(x: number, y: number): number {
    return x * y;
}
```

Nous indiquons que la fonction retourne un nombre. Dans ce cas précis comme la fonction est très simple, l'inférence de *TypeScript* fonctionne et il connaît le type retourné. 

