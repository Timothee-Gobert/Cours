# Les types génériques

## Introduction aux types génériques

### A quoi servent les génériques ?

**Les génériques sont une fonctionnalité de *TypeScript* permettant une grande flexibilité combinée à une sécurité du typage.**

Lorsque vous débuterez avec *TypeScript* vous rencontrerez deux travers :
1. **Trop typer en any.** Cela vous donne une grande flexibilité mais vous perdez totalement en sécurité et en autocomplétion.
2. **Tout typer correctement sans utiliser de génériques.** Cela conférera à votre code la sécurité et l'autocomplétion permis par le typage fort mais vous perdrez en flexibilité, ce qui provoquera beaucoup de duplications et de lourdeurs dans votre code.

Les génériques sont le parfait compromis entre ces deux situations et nous allons détailler tous les cas d'utilisation dans ce chapitre.

En résumé, l'objectif des génériques est de permettre d'utiliser des éléments (fonctions, classes, interfaces etc) qui puissent fonctionner avec une diversité de types tout en conservant l'utilité de TypeScript, à savoir la sécurité et l'autocomplétion.

### Deux exemples de types génériques natifs

Avant de voir en détails les utilisations des types génériques, nous allons donner deux exemples de types génériques utilisés nativement.

#### Le typage des tableaux

Nous avons vu comment typer des tableaux pour qu'ils n'acceptent qu'un seul type en faisant :

```ts
const tableau: string[] = ['Des', 'chaînes', 'de', 'caractères'];
```

Nous avions vu également que la notation alternative était :

```ts
const tableau: Array<string> = ['Des', 'chaînes', 'de', 'caractères'];
```

Si vous passez la souris sur `Array` vous verrez *interface Array\<T>*.

Cela signifie que `Array` prend en fait un argument appelé *T* par convention pour type.

L'interface *Array\<T>* utilise donc un type générique, nous pouvons passer n'importe quel type comme argument et le tableau devra comporter le type défini.

Exemple avec une union de types :

```ts
const tableau: Array<string | boolean> = ['Des', 'chaînes', 'de', 'caractères', true];
```

Autre exemple en utilisant une *interface* :

```ts
interface User {
    name: string;
  }

  const tableau: Array<User> = [{name: 'Paul'}, {name: 'Jean'}];
```

**Nous passons en argument n'importe quel type à l'*interface Array\<T>*, et TypeScript nous obligera à le respecter.**

#### Autre exemple : les promesses

Prenons un exemple avec une promesse :

```ts
let condition: boolean;

  const promesse = new Promise((resolve, reject) => {
    if (condition) {
      resolve(42)
    } else {
      reject('Erreur');
    }
  });
```

Dans *VS Code*, si vous passez la souris sur la constante, vous aurez *Promise\<unknown>*.

Ici, *TypeScript* utilise également un générique pour typer la valeur retournée par la promesse. Autrement dit, il utilise une *interface Promise\<T>*, et donc les génériques.

Ici, il ne peut pas détecter le type par inférence et précise donc que la valeur retournée est de type inconnu : *unknown*.

En revanche, si nous passons un argument, *TypeScript* saura que les valeurs retournées dans les promesses seront du type spécifié :

```ts
let condition: boolean;

  const promesse: Promise<number | string> = new Promise((resolve, reject) => {
    if (condition) {
      resolve(42)
    } else {
      reject('Erreur');
    }
  });
```

Ici, nous indiquons à *TypeScript* les valeurs potentiellement retournées dans la promesse ce qui permet la sécurité et l'autocomplétion.

A savoir que si vous essayez d'utiliser par exemple une méthodes disponibles sur les tableaux ou les objets sur la valeur de retour, *TypeScript* vous renverra une erreur.

Par exemple :

```ts
let condition: boolean;

  const promesse: Promise<number | string> = new Promise((resolve, reject) => {
    if (condition) {
      resolve(42)
    } else {
      reject('Erreur');
    }
  });

  promesse.then((val) => {
   val.map()
  })
```

Ici, vous aurez une erreur *Property 'map' does not exist on type 'string | number'*.

De même, *VS Code* vous proposera en autocomplétion sur *val* les méthodes et propriétés partagées par les types nombre et chaînes de caractères.

Si vous mettiez `Promise<any>`, *TypeScript* ne ferait plus de contrôle sur les valeurs de retour, sur les méthodes que l'on tente d'accéder etc.

```ts
let condition: boolean;

  const promesse: Promise<any> = new Promise((resolve, reject) => {
    if (condition) {
      resolve(42)
    } else {
      reject('Erreur');
    }
  });

  promesse.then((val) => {
   val.map()
  })
```

Ici aucun problème lors de la compilation et dans *VS Code*, pourtant l'accès à la méthode `map()` provoquera une erreur lors de l'exécution.

Retenez pour le moment que les génériques sont très importants pour maintenir la sécurité et l'autocomplétion permises par *TypeScript*. 

## Fonctions et types génériques

### Utilisation des génériques avec les fonctions

Prenons un exemple que les débutants en *TypeScript* ont tendance à faire :

```ts
function identique(arg: any): any {
  return arg;
}
```

Cette fonction ne nous donne aucune information particulière sur le type d'argument ni sur la valeur de retour de la fonction. Par exemple, si nous passons un nombre, nous n'avons aucune garantie sur le type de la valeur de retour.

Nous perdons tout l'intérêt des types : aucune vérification et aucune autocomplétion ne seront possibles par *TypeScript*.

Pour avoir une telle information tout en gardant une flexibilité sur le type de l'argument, il faudrait pouvoir capturer ce type dans une variable pour pouvoir l'utiliser et indiquer le type de la valeur de retour. C'est ici qu'interviennent les génériques !

```ts
function identique<T>(arg: T): T {
  return arg;
}
```

Par convention on utilise la lettre **T** (pour type comme nous l'avons vu) mais il est possible d'utiliser n'importe quel nom,, l'important est l'ajout des <> qui indique que le type passé doit être conservé.

**Ici nous ajoutons une variable de type appelée T qui permet de capturer le type passé à la fonction et le garder pour typer le paramètre et la valeur de retour.**

Autrement dit, nous indiquons que si nous passons *number* comme type à la fonction, alors son paramètre devra recevoir un nombre et elle retournera un nombre.

**Attention ! Comme toutes les fonctionnalités *TypeScript*, cela n'aura aucune incidence lors de l'exécution du code.**

La capture et l'utilisation de la variable contenant le type, `T`, se fait statiquement par *VS Code* et par *tsc* lors de la compilation.

Vous pouvez utiliser une telle fonction de deux manières.

Soit en **spécifiant explicitement la valeur de *T***, par exemple :

```ts
let nombre = identique<number>(2);
```

Soit en **utilisant l'inférence de TypeScript qui va automatiquement détecter le type passé et l'utiliser comme valeur de T** :

```ts
let nombre = identique(2);
```

Dans la plupart des cas vous utiliserez l'inférence car cela fait tout simplement moins à taper et cela ne fait aucune différence. Dans certains cas avancés, vous aurez besoin de spécifier explicitement le type.

Maintenant que vous avez compris le principe, voyons un autre exemple plus réaliste très courant :

```ts
function fusionner(a: object, b: object) {
  return Object.assign(a, b);
}

const obj = fusionner({name: 'Jean'}, {email: 'jean@gmail.com'});

if (obj.name) { // Erreur

}
```

Ici, nous utilisons une fonction qui permet de fusionner deux objets. Le problème est que nous ne donnons pas d'informations sur les paramètres autre que le fait que ce soient des objets.

*TypeScript* ne peut pas inférer leur type et donc on ne peut pas accéder à la propriété *name* sur l'objet résultant de la fusion. Pour *TypeScript* il n'est pas certain que la propriété soit présente et il signale :
>*Property 'name' does not exist on type 'object'*.

Maintenant utilisons les types génériques et observons la différence :

```ts
function fusionner<T, U>(obj1: T, obj2: U) {
  return Object.assign(obj1, obj2);
}

const obj = fusionner({name: 'Jean'}, {email: 'jean@gmail.com'});

if (obj.name) {

}
```

Première différence, *TypeScript* infère que le retour de la fonction fusionné est l'intersection de *T & U*. Il comprend ici contextuellement ce qui est retourné.

Si la fonction était plus complexe, nous pourrions typer nous-mêmes la valeur de retour sans problème :

```ts
function fusionner<T, U>(obj1: T, obj2: U): T & U {
  return Object.assign(obj1, obj2);
}
```

Nous indiquons que le type de l'objet retourné comporte tous les types du premiers arguments et tous les types du second argument.

Seconde différence, *TypeScript* sait maintenant grâce à l'inférence que l'objet retourné, ici *obj*, comportera les propriétés des objets passés en argument.

*TypeScript* regarde que le premier argument comporte la propriété *name* et que le second objet possède la propriété *email*, il en déduit grâce aux types génériques que l'objet retourné a ces deux propriétés.

### Utilisation des génériques pour les signatures de fonctions

Les génériques fonctionnent bien évidemment également avec les signatures de fonction, à savoir le typage d'une fonction avant son implémentation :

```ts
let fonction: <T>(arg: T) => T;
```

Ici la fonction recevra un type `T` en argument et retournera une valeur du même type. 

## Classes et types génériques

### Interfaces et types génériques

Nous allons voir brièvement les interfaces et les types génériques car leur usage est assez rare.

Par exemple, vous pouvez créez des *interface* génériques pour vos fonctions :

```ts
interface FonctionGenerique<T, U> {
  (arg: T): U;
}

let fonction1: FonctionGenerique<string, number>;

fonction1 = function(arg: string): number {
  return Number(arg)
}
```

Vous pouvez, avec cette *interface* forcer un type en paramètre et un type en valeur de retour lors de la déclaration en les passant en arguments.

Bien sûr, cet usage n'est pas limité aux fonctions, vous pouvez l'utiliser pour n'importe quoi :

```ts
interface PaireCleValeur<T, U> {
  cle: T;
  valeur: U;
}

const paire: PaireCleValeur<number, string> = { cle: 1, valeur: "Steve" };
const paires: PaireCleValeur<string, string>[] = [{ cle: '42', valeur: "Steve" }];
```

### Classes et types génériques

L'utilisation des types génériques avec les classes et assez similaires qu'avec les *interfaces*.

Nous allons prendre plusieurs exemples pour bien comprendre, en commençant par la stack qui est une structure de données *LIFO* (dernier rentré, premier sorti) :

```ts
class Stack<T> {
  items: T[] = [];

  push(item: T) {
    this.items.push(item)
  }

  pop(): T | undefined {
    return this.items.length ? this.items.pop() : undefined;
  }

  display() {
    console.log(this.items)
  }
}

let stack1 = new Stack<string>();
stack1.push('test');
stack1.push('test2');
stack1.pop();
stack1.display();
```

Nous pouvons créer une pile qui ne contiendra que le type passé en argument de type lors de l'initialisation de la classe : `new Stack<string>()`.

Prenons un second exemple :

```ts
class Store<T> {
  items: T[] = [];

  addItems(items: T[] | T): T[] {
      Array.isArray(items) ? this.items.push(...items) : this.items.push(items);
      return this.items;
  }

  removeItems(items: T[] | T): T[] {
      this.items = Array.isArray(items) ?
          this.items.filter(item => !items.includes(item)) :
          this.items.filter(item => item !== items);
      return this.items;
  }

  display() {
    console.log(this.items)
  }
}

const storage = new Store<number>();
storage.addItems(1);
storage.addItems([2, 3, 4]);
storage.display(); // [1, 2, 3, 4]

storage.removeItems(2);
storage.removeItems([1, 3]);
storage.display(); // [4]
```

Ici nous créons une fonction qui permet d'ajouter un ou plusieurs éléments d'un même type à un tableau et de supprimer un ou plusieurs éléments de ce type de ce tableau.

Grâce aux génériques nous pouvons simplement faire un *Store* pouvant accepter un type particulier.

Nous pourrions très bien l'utiliser par exemple avec une *interface User* ou autre.

### Contraindre les types génériques

Parfois, vous souhaiterez utiliser des types génériques pour obtenir de la flexibilité mais souhaiterez les limiter principalement pour des raisons d'accès à des méthodes ou à des propriétés.

Prenons un exemple :

```ts
function maFunction<T>(arg: T): T {
  arg.length; // Erreur
}
```

Et un second :

```ts
function maFunction<T>(arg: T): T {
  arg.reverse(); // Erreur
}
```

Dans les deux cas, *TypeScript* protège l'accès à la propriété *length* et à la méthode *includes* car il n'a aucun moyen d'être certain que sur le type générique `T` que vous passerez en argument de type, il y aura cette propriété ou cette méthode.

Il est donc nécessaire de restreindre le type `T`, en lui appliquant des contraintes.

Pour résoudre le premier exemple, nous pourrions faire :

```ts
interface HasLength {
  length: number;
}

function maFunction<T extends HasLength>(arg: T): T {
  arg.length;
  return arg;
}
```

Ici nous contraignons le type `T` générique avec *extends*, pour faire en sorte que T ait obligatoirement une propriété *length*. Dans ce cas *TypeScript* n'aura plus à craindre une erreur d'accès à une propriété *undefined*.

Pour le second exemple, nous pourrions faire :

```ts
function maFunction<T extends Array<any>>(arg: T): T {
  arg.reverse(); // Erreur
  return arg;
}
```

Ici nous contraignons le fait que T soit un tableau contenant tout type. Dans ce cas, la méthode `reverse()` sera bien accessible.

Nous voyons bien l'intérêt de contraindre les types génériques.

### Contraindre un type générique à partir des propriétés d'un objet

Nous allons voir un cas d'utilisation avancé de la contrainte des types génériques.

Nous souhaitons que si nous passons un premier argument de type à une fonction, le deuxième argument de type utilisé ait au moins les propriétés du premier type.

Pour ce faire, nous allons toujours utiliser *extends* mais en utilisant cette fois-ci en plus *keyof*.

***keyof* permet de créer une union de types littéraux avec toutes les propriétés d'un objet.**

Par exemple :

```ts
interface Todo {
id: number;
texte: string;
auteur: string;
}

type TodoProps= keyof Todo; // "id" | "texte" | "auteur"
```

Cela permet de réaliser des patterns très puissants, comme celui-ci :

```ts
function getProperty<T, U extends keyof T>(obj: T, cle: U) {
  return obj[cle];
}

const obj = { prop1: 1, prop2: 2, prop3: 3, prop4: 4 };

getProperty(obj, "prop1");
```

Ici nous avons une fonction qui permet simplement de retourner une propriété d'un objet. Son intérêt est qu'elle empêche d'accéder à une propriété qui n'existe pas sur l'objet sans avoir à créer d'*interface* !

Comme est-ce possible ? Tout simplement car *TypeScript* détecte automatiquement le type du premier argument passé et l'assigne à *T*, ici *T* vaut donc :

```ts
T: {
  prop1: number;
  prop2: number;
  prop3: number;
  prop4: number;
}
```

Ensuite, avec *keyof T* on obtient un union de types littéraux :

```ts
keyof T; // "prop1" | "prop2" | "prop3" | "prop4"
```

Enfin grâce à *extends* on force le paramètre de type *U* à avoir toutes ces propriétés.

Au final nous sommes certains de ne pas accéder à une propriété non déclarée. 

### Collections et types génériques

*TypeScript* propose des types utilitaires pour aider dans vos déclarations de type.

La plupart utilisent des types génériques.

Ils sont tous disponibles de façon globale.

### Le type `Partial<T>`

Comme son nom l'indique, ***Partial* permet de prendre un type en argument et de retourner un nouveau type où toutes les propriétés sont optionnelles.**

Cela vous sera souvent utile par exemple pour des mises à jour :

```ts
interface Todo {
  id: number;
  texte: string;
  auteur: string;
}

function majTodo(todo: Todo, update: Partial<Todo>) {
    return { ...todo, ...update };
}

let todo1: Todo = {
    id: 1,
    texte: 'Apprendre JavaScript',
    auteur: 'Jean'
}

todo1 = majTodo(todo1, {
    texte: 'Apprendre TypeScript',
});
```

Ici en utilisant `Partial<Todo>`, nous faisons en fait ceci :

```ts
type Partial<T> = { [P in keyof T]?: T[P] | undefined; }
```

Nous passons un type générique en argument : `Partial<T>`.

Nous utilisons un type indexable [prop: type] pour indiquer un nombre indéfini de propriétés sur l'objet.

`P in keyof T` signifie que le type générique `P` est une propriété contenue (`in`) dans les propriétés de `T` (`keyof T`).

Ensuite, nous utilisons ? pour rendre cette propriété `P` optionnelle.

Enfin, nous lui assignons la valeur définie `T[P]` ou *undefined*.

Comme pour les autres types utilitaires, nous utilisons en fait simplement un alias qui est un raccourci pour un typage pratique mais un peu complexe à définir soit-même.

### Le type `Readonly<T>`

Comme son nom l'indique, ***Readonly* permet de passer toutes les propriétés d'un type passé en argument en *readonly*.**

Exemple :

```ts
interface Todo {
  id: number;
  texte: string;
  auteur: string;
}

const todo: Readonly<Todo> = {
    id: 1,
    texte: 'Apprendre JavaScript',
    auteur: 'Jean'
};

todo.id = 2; // Erreur: cannot reassign a readonly property
```

Même chose, `Readonly<T>` est en fait un alias pour :

```ts
type Readonly<T> = { readonly [P in keyof T]: T[P]; }
```

Nous passons un type générique en argument : `Readonly<T>`.

Nous utilisons un type indexable [prop: type] pour indiquer un nombre indéfini de propriétés sur l'objet.

`P in keyof T` signifie que le type générique P est une propriété contenue (`in`) dans les propriétés de `T` (`keyof T`).

Ensuite, nous utilisons *readonly* pour rendre cette propriété `P` en lecture seule.

Enfin, nous lui assignons la valeur définie `T[P]`.

### Le type `Record<K,T>`

*Record* permet de prendre deux types en arguments, le premier type étant un alias de types littéraux et le second étant un type quelconque.

Il permet d'utiliser les types littéraux du premier type comme propriétés pour créer un objet dont les valeurs respectent le type du second argument de type.

`Record<K,T>` est un alias pour :

```ts
type Record<K extends string | number | symbol, T> = { [P in K]: T; }
```

`K` est le premier argument de type passé, il doit être d'un type accepté pour les propriétés d'objet, autrement dit il doit respecter l'union de types *string | number | symbol*.

Pour tous les types littéraux de `K`, nous créons une propriété `P` et lui assignons le type `T` passé en deuxième argument.

Prenons un exemple :

```ts
interface PageInfo {
    title: string;
}

type Page = 'home' | 'about' | 'contact';

const x: Record<Page, PageInfo> = {
    about: { title: 'A propos' },
    contact: { title: 'Contact' },
    home: { title: 'Accueil' },
};
```

Ici, l'argument de type `K` est *Page* qui est bien un alias de types littéraux pouvant être des propriétés d'objet.

L'argument de type `T` est *PageInfo* et permet de typer la valeur de chaque propriété contenues dans `K`.

### Le type `Pick<K,T>`

L'alias `Pick<K, T>` permet de récupérer une ou plusieurs propriétés, appelées `K`, d'un type `T`.

Exemple :

```ts
interface Todo {
  id: number;
  texte: string;
  auteur: string;
}

type TodoFront = Pick<Todo, 'texte' | 'auteur'>;

const todo: TodoFront = {
    texte: 'Apprendre TypeScript',
    auteur: 'Paul',
};
```

Nous avons une *Todo* en front que nous n'avons pas encore sauvegardée et n'avons donc pas encore d'*id*.

Nous pouvons utiliser *Pick* pour spécifier les propriétés que nous devons respecter.

`Pick<K, T>` est un alias pour :

```ts
type Pick<T, K extends keyof T> = { [P in K]: T[P]; }
```

`T` est ici *Todo*, à savoir l'argument de type depuis lequel nous allons sélectionner certaines propriétés.

`K extends keyof T` signifie que les propriétés que nous allons récupérer doivent forcément être contenues dans les propriétés de `T`.

Nous utilisons un type indexable [prop: type] pour indiquer un nombre indéfini de propriétés sur l'objet.

Chaque propriété `P` doit être contenu dans `K`, à savoir dans les propriétés de `T`.

Ici, cela signifie que les propriétés sélectionnées doivent être dans l'`union 'texte' | 'auteur' | 'id'`.

Pour la valeur de chaque propriété c'est simplement la valeur correspondant à la propriété `T[P]`.

### Le type `Omit<T, K>`

L'alias `Omit<T, K>` permet à l'inverse d'omettre certaines propriétés, appelées `K`, d'un type `T`.

En reprenant notre exemple, nous aurions :

```ts
interface Todo {
  id: number;
  texte: string;
  auteur: string;
}

type TodoFront = Omit<Todo, 'id'>;

const todo: TodoFront = {
    texte: 'Apprendre TypeScript',
    auteur: 'Paul',
};
```

`Omit<K, T>` est un alias pour :

```ts
type Omit<T, K extends string | number | symbol> = { [P in Exclude<keyof T, K>]: T[P]; }
```

`T` est l'objet sur lequel nous allons omettre certaines propriétés.

`K` est une propriété d'objet est doit donc respecter l'*union string | number | symbol*.

`Exclude<keyof T, K>` permet de construire un nouveau type à partir d'un type T en excluant toutes les propriétés de `K`.

Ici cela signifie que nous prenons toutes les propriétés de `T` (`keyof T`) et que nous enlevons les propriétés de `K`.

Nous vérifions ensuite que chaque propriété du nouvel objet respecte ce nouveau type (`P in Exclude<keyof T, K>`).

Enfin, pour la valeur de chaque propriété c'est simplement la valeur correspondant à la propriété `T[P]`.

### Le type `Exclude<T,U>`

`Exclude<T, U>` permet de construire un nouveau type à partir d'un type T en excluant toutes les propriétés de `U`.

Exemple :

```ts
type Exclure = Exclude<1 | 2 | 3, 1 | 2>; //  3
```

`Exclude<T, U>` est un alias pour :

```ts
type Exclude<T, U> = T extends U ? never : T
```

Nous verrons les types conditionnels distribués dans le prochain chapitre, mais c'est assez simple à comprendre car cela fonctionne comme un ternaire *JavaScript* dans ce cas.

Pour chaque propriété de `T` on vérifie si elle déjà présente dans `U`, si oui on retourne *never*, c'est-à-dire on la supprime, et si elle n'est pas présente on la garde en la retournant.

En détails cela donne :

```ts
1 extends 1 | 2 ? never : 1
2 extends 1 | 2 ? never : 2
3 extends 1 | 2 ? never : 3
```

Dans les deux premiers le type littéral est inclus dans l'union `U` donc on retourne *never*.

Dans le dernier cas, le type littéral 3 n'est pas inclus dans l'union de types littéraux et donc nous retournons le type littéral 3.

Nous reverrons les types conditionnels en détails.

### Le type `Extract<T, U>`

Le type `Extract<T,U>` permet de construire un nouveau type en extrayant d'un argument de type `T` toutes les propriétés qui se trouvent dans `U`.

Exemple :

```ts
type Extraites = Extract<1 | 2 | 3, 1 | 2>; //  1 | 2
```

`Extract<T, U>` est un alias pour :

```ts
type Extract<T, U> = T extends U ? T : never
```

Même chose que pour *Exclude*, cet alias utilise un type conditionnel distribué.

Mais ici c'est exactement l'opposé, si le type est inclus dans l'union *U* on le garde, sinon on l'écarte.

Dans le détails cela donne :

```ts
1 extends 1 | 2 ? 1 : never
2 extends 1 | 2 ? 2 : never
3 extends 1 | 2 ? 3 : never
```

Au final on obtient donc un nouveau type qui est l'*union* des types littéraux 1 et 2 : `1 | 2`.

### Le type `NonNullable<T>`

Le type `NonNullable<T>` permet de filtrer tous les types *null* et *undefined* de l'argument de type `T` :

```ts
type PasNul = NonNullable<string | number | undefined>; // string | number
```

Le type `NonNullable<T>` est un alias pour :

```ts
type NonNullable<T> = T extends null | undefined ? never : T
```

`NonNullable<T>` équivaut exactement à `Exclude<T, null | undefined>`.

Notre exemple peut être réécrit en :

```ts
type PasNul = Exclude<string | number | undefined, null | undefined>;
```

Le fonctionnement est donc le même et nous n'allons pas le redétailler.

### Le type `Parameters<T>`

Le type `Parameters<T>` permet de créer un *tuple* contenant dans l'ordre chaque type de chaque paramètre d'une fonction.

Par exemple à partir **d'une signature de fonction** :

```ts
type MaFonction = (arg1: string, arg2: number, arg3: { prop1: string }) => boolean;

type Parametres = Parameters<MaFonction>;
// [string, number, { prop1: string; }]
```

Autre exemple, en utilisant **l'implémentation d'une fonction et l'opérateur *typeof*** :

```ts
function maFonction(arg1: string, arg2: number, arg3: { prop1: string }): boolean {
    if (arg1 && arg2 && arg3) {
        return true
    } else {
        return false;
    }
}

type Parametres = Parameters<typeof maFonction>;
// [string, number, { prop1: string; }]
```

L'opérateur *typeof* permet simplement de récupérer le type d'un élément. Ici, en lui passant une fonction, on récupère la signature de la fonction.

### Le type `ConstructorParameters<T>`

Le type `ConstructorParameters<T>` permet d'extraire tous les types des paramètres d'un constructeur.

```ts
class Todo {
  constructor(public id: number, public texte: string, public auteur: string) {}
}

type Parametres = ConstructorParameters<typeof Todo>;
// [number, string, string]
```

### Le type `ReturnType<T>`

Le type `ReturnType<T>` permet de créer un type à partir du type de retour d'une fonction :

```ts
function uneFonction(): boolean | undefined | Todo {
    // ...
    return;
}
type TypeRetour = ReturnType<typeof uneFonction>;
// boolean | Todo | undefined
```

### Le type `InstanceType<T>`

Le type `InstanceType<T>` permet de créer un type à partir du type d'une fonction constructrice.

```ts
class Todo {
  constructor(public id: number, public texte: string, public auteur: string) {}
}

type InstanceTodo = InstanceType<typeof Todo>;
```

### Le type `Required<T>`

Comme son nom l'indique, ***Required* permet de prendre un type en argument et de retourner un nouveau type où toutes les propriétés sont obligatoires.**

Prenons un exemple fréquent :

```ts
interface Todo {
  id?: number;
  texte: string;
  auteur: string;
}

type SavedTodo = Required<Todo>;
```

Vous avez une Todo qui n'est pas forcément sauvegardée en base de données et n'a donc pas encore d'*id*.

Vous voulez ensuite créer un nouveau type, *SavedTodo*, pour les todos sauvegardées qui ont toutes les propriétés en obligatoire car vous êtes sûr qu'après la sauvegarde votre todo se sera vue attribuer un *id* par la base de données.

Notez que nous omettons trois alias globaux car leurs cas d'utilisation sont trop rares : `ThisParameterType`, `OmitThisParameter` et `ThisType<T>`. Ils ont tous à voir avec l'utilisation de *this* dans certains patterns de fonctions utilisant `apply()`, `bind()` ou `call()`. 