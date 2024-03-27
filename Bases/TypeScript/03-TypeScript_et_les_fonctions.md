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

## Typer une fonction avant sa déclaration

### Typer avant la déclaration

Il est possible de typer une fonction avant de la déclarer, par exemple :

```ts
let multiplier: (nombre1: number, nombre2: number) => number;
```

Ici, la variable multiplier doit forcément prendre une fonction qui prend deux nombres en arguments et retourne un nombre.

**Attention, lors du typage d'une fonction de cette manière il est obligatoire de préciser le type de sa valeur de retour après *=>*.**

Lors de la déclaration il faudra respecter ce typage.

Si la fonction ne retourne rien il faudra utiliser *void* pour le type de la valeur retournée.

### Usage

On utilise par exemple le type fonction notamment pour les objets afin de typer leurs méthodes :

```ts
let monObjet = {
  data: number[],
  multiplier: (nombre1: number, nombre2: number) => number
}
```

### Le type *Function*

Parfois si vous ne savez pas la valeur de retour ou le type des arguments car ils sont dynamiques, vous pourrez utiliser le type plus générique *Function*.

Notez bien la majuscule contrairement aux types *string*, *number* et *boolean*.

```ts
let maFonction: Function;
```

## Les paramètres optionnels

### Les paramètres obligatoires par défaut

En *TypeScript*, tous les paramètres sont par défaut requis pour une fonction.

Cela signifie qu'il faut fournir une valeur pour chaque paramètre lors de l'appel d'une fonction, cette valeur pouvant être *null* ou *undefined* si le type des paramètres le permet.

**En *TypeScript*, par défaut, il faut donc que le nombre de paramètres déclarés coïncide strictement avec le nombre de paramètres passé lors de l'invocation.**

Ainsi par exemple, le compilateur *tsc* va se plaindre si vous passez pas assez ou trop d'arguments :

```ts
function ajouter(x: number, y: number): number {
  return x + y;
}

ajouter(2); // Expected 2 arguments, but got 1
ajouter(1, 2, 3); // Expected 2 arguments, but got 3
```

### Définir des paramètres optionnels

**En *TypeScript*, pour définir des paramètres comme étant optionnels, il suffit d'ajouter un *?* après chaque paramètre optionnel.**

Par exemple :

```ts
function ajouter(x: number, y: number, z?: number): number {
  return z ? x + y +z : x + y;
}
```

**Les paramètres optionnels doivent *toujours* être ajoutés après les paramètres obligatoires.**

Par exemple, il ne faut surtout pas faire :

```ts
function ajouter(x?: number, y: number, z: number): number {
  return x ? x + y + z : y + z;
}
```

## Paramètres par défaut et rest

### Les paramètres par défaut

En *TypeScript*, il est également possible de définir une valeur initiale comme en *JavaScript* pour les paramètres.

Ces paramètres avec des valeurs initiales sont appelées **paramètres initialisés par défaut.**

Par exemple :

```ts
function ajouter(x = 0, y = 0): number {
  return x + y;
}
```

Lorsque vous précisez des valeurs par défaut, vous n'avez pas besoin de typer dans la plupart des cas car *TypeScript* déduit le type des paramètres par inférence.

**Les paramètres initialisés par défaut sont traités comme des paramètres optionnels, vous n'avez donc pas besoin d'ajouter *?*.**

A noter que contrairement aux paramètres optionnels, il est possible d'utiliser un paramètre initialisé par défaut à n'importe quel place :

```ts
function ajouter(x = 0, y: number): number {
  return x + y;
}
```

Bien sûr dans ce cas, le premier paramètre n'est plus optionnel et il faut obligatoirement passer une valeur en premier argument. Pour que *TypeScript* utilise la valeur par défaut, il faut passer *undefined* :

```ts
ajouter(undefined, 42); // 42
```

### Utilisation d'un reste

En *TypeScript* si vous utilisez l'opérateur *rest (...)* comme dernier paramètre d'une fonction, vous pourrez le typer.

Par exemple :

```ts
function ajouter(x: number, ...nbrs: number[]): number {
  return nbrs.reduce((a, b) => a + b, x)
}

ajouter(1,2,3,4); // 10
```

Ici nous avons un tableau de nombres contenant tous les arguments passés après le premier.

Nous typons ce tableau comme étant un tableau de nombres.

Nous passons *x* comme valeur initiale de *reduce* et additionnons ensuite tous les nombres du tableau. 

## La surcharge de fonction

### La surcharge des fonctions

**La surcharge de fonction en *TypeScript* permet de faire varier le type de la valeur de retour en fonction du type de ses arguments.**

Prenons un premier exemple qui peut être beaucoup mieux écrit avec les types génériques :

```ts
function maFonction(param: string): string;
function maFonction(param: number): number;
function maFonction(param: boolean): boolean;
function maFonction(param: any): any {
  return param;
}

maFonction('test').toLocaleLowerCase();
maFonction(42).toFixed();
```

Ici notre fonction a trois surcharges qui permettent à *TypeScript* de savoir que lorsque l'on passe une chaîne de caractères, la fonction retournera une chaîne de caractères, que lorsqu'on passe un nombre, la fonction retournera un nombre et même chose pour un booléen.

Si on passe autre chose la fonction retournera un type indéterminé.

De cette manière, nous pouvons accéder sans problèmes aux bonnes méthodes suivant l'argument passé à la fonction

### Exemple avancé de surcharge

Prenons un exemple plus complexe :

```ts
interface User {}

function getUser(id: number): User;
function getUser(email: string): User;
function getUser(prenom: string, nom: string): User;

function getUser(param1: string | number, param2?: string ): User {

  let user: User;

  if (typeof param1 === 'number') {
      // Récupère le user par id
  } else if (typeof param2 != 'undefined') {
      // Récupère le user par prénom et nom
  } else {
      // Récupère le user par email
  }

  return user;
}
```

Ici, nous avons trois surcharges pour une fonction qui permet de récupérer un utilisateur soit en passant un id, soit en passant un email, soit en passant un nom et prénom.

Nous utilisons des gardes de type pour implémenter une logique différente suivant les arguments passés.

Avec la surcharge nous avons une vision claire de ce que peut recevoir la fonction et de ce qu'elle retourne.

Encore un exemple, où la valeur de retour change en fonction du type d'argument passé :

```ts
interface User {}

function getUsers(id: number): User;
function getUsers(ids: number[]): User[];

function getUsers(param: number | number[]): User | User[] {

  let users: User | User[];

  if( typeof param === 'object') {
      // Récupère les users avec le tableau d'ids
  } else {
      // Récupère le user par son id
  }

  return user;
}

getUsers(4342342);
getUsers([2342342341, 122241423]);
```

Ici *TypeScript* sait que si vous passez un seul id vous aurez en sortie un seul *User*, et que si vous passez un tableau, vous aurez un tableau de *Users*.

Vous avez ici la sécurité et l'autocomplétion tout en conservant en flexibilité puisque vous avez une fonction qui peut prendre plusieurs types en entrée et retourner un type différent suivant ce type. 

