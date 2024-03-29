# Unions, intersections, nouveaux opérateurs et alias

## L'union de types

### Les combinaisons de types

Nous avons vu comment combiner des interfaces en les étendant, nous avons vu également comment implémenter des interfaces avec des classes.

*TypeScript* propose d'autres fonctionnalités permettant de combiner tous les types (y compris les interfaces) : les unions et les intersections.

### Les unions de type

**Les unions de type permettent de combiner deux ou plusieurs types et s'utilise avec `|`.**

Les unions s'utilisent dans toutes les situations.

#### Unions de type et variables

Prenons un premier exemple :

```ts
let uneVar: string | number;
```

Ici, nous effectuons une union entre les types chaîne de caractères et nombre.

La variable pourra ainsi recevoir l'un des deux types.

#### Unions de type et fonctions

Vous pouvez également typer les paramètres des fonctions en utilisant des unions :

```ts
function ajouter(a: string | number, b: string | number) {
  return Number(a) + Number(b);
}
```

Ici par exemple notre fonction peut ajouter des nombres et des chaînes de caractères. Nous voulons donc utiliser des unions pour les paramètres de la fonction.

Vous pouvez également typer les retours de fonctions en utilisant des unions :

```ts
function ajouter(isAdmin: boolean): string | void {
  if (isAdmin) {
    return 'secret';
  } else {
    return;
  }
}
```

Ici, la fonction peut retourner une *chaîne de caractères* ou *void*.

D'ailleurs il est intéressant de noter que *void* est lui-même une union des types *undefined* et *null* !

#### Unions de type et interfaces

Nous pouvons également utiliser les unions avec les interfaces, et n'importe quelle combinaison entre types primaires, interfaces et classes.

Par exemple :

```ts
interface User {
  name: string;
  email: string;
}

interface Moderator extends User {
  role: 'moderator';
  editMessage: (msg: string) => string
}

let user: User | Moderator;
```

Il est possible d'avoir une union entre un type et une interface :

```ts
interface User {
  name: string;
  email: string;
}

let user: User |null;
```

Ici notre user peut se voir assigner un *User* respectant notre interface, ou *null*. 

## Les unions discriminantes et gardes

### Une problématique fréquente

Lorsque l'on utilise des unions d'*interfaces* on peut rencontrer le problème suivant :

```ts
interface User {
  name: string;
  email: string;
}

interface Moderator extends User {
  role: 'moderator';
  editMessage: (msg: string) => string
}

function uneFonction(user: User | Moderator) {
  user.editMessage('un message')
}
```

Ici, nous avons une erreur suivante : 
>*Property 'editMessage' does not exist on type 'User | Moderator'. Property 'editMessage' does not exist on type 'User'*.

Autrement dit ici *TypeScript* ne peut pas savoir si la fonction va recevoir un utilisateur ou un modérateur, il ne peut donc pas savoir si la méthode `editMessage()` existera sur l'argument passé lors de l'exécution.

*TypeScript* nous le signale et nous affiche donc une erreur.

### Les unions discriminantes

Pour résoudre cette problématique, la solution recommandée officiellement est l'utilisation d'unions discriminantes.

Le principe est simple : il faut ajouter une propriété différenciante sur l'*interface* qui permette à *TypeScript* de savoir quelle *interface* utiliser lors d'une union.

**Cette propriété doit contenir un type littéral, à savoir une seule valeur possible.**

Exemple :

```ts
interface User {
  role: 'user';
  name: string;
  email: string;
}

interface Moderator {
  role: 'moderator';
  name: string;
  email: string;
  editMessage: (msg: string) => string
}

function uneFonction(user: User | Moderator) {
  if (user.role === 'moderator') {
    user.editMessage('un message')
  }
}
```

Ici *TypeScript* peut savoir que nous avons forcément un modérateur dans le bloc *if*.

**Prenons l'exemple le plus commun et détaillons :**

```ts
interface Carre {
  forme: "carre";
  cote: number;
}
interface Rectangle {
  forme: "rectangle";
  largeur: number;
  hauteur: number;
}
interface Cercle {
  forme: "cercle";
  rayon: number;
}

type Forme = Carre | Rectangle | Cercle;

function calcAire(e: Forme) {
  switch (e.forme) {
    case "carre":
      return e.cote * e.cote;
    case "rectangle":
      return e.largeur * e.hauteur;
    case "cercle":
      return Math.PI * e.rayon ** 2;
  }
}
```

C'est le cas d'utilisation le plus fréquent. Ici nous avons une fonction qui peut prendre trois types de figures et retourner son aire. Pour typer correctement la fonction nous avons besoin d'une union discriminante qui utilise :
1. **Une propriété qui est un type littéral pouvant avoir une seule valeur, appelé élément discriminant ou différenciant.** Dans notre exemple c'est la propriété *forme*.
2. **Un alias de type qui est une union de plusieurs *interfaces*.** Dans notre exemple c'est l'alias *Forme*.
3. **Des gardes sur le type pour la propriété discriminante.** Ces gardes peuvent être, comme ici, une instruction *switch* ou des instructions *if* et *else if*.

## Les gardes

### Les gardes de type

Toutes les techniques de garde de type ne sont pas spécifiques du tout à *TypeScript*, leur usage est très répandu en *JavaScript* et vous les utilisez sûrement déjà pour la plupart.

Les gardes de type permettent en *JavaScript* qu'un objet a une propriété, en *TypeScript* il s'agit de vérifier qu'un objet est d'un type particulier.

### Les instructions *if*

La plus commune est l'instruction *if* qui va vérifier la présence ou la valeur d'une ou plusieurs propriétés, nous avons utiliser ce type de garde ici :

```ts
function uneFonction(user: User | Moderator) {
  if (user.role === 'moderator') {
    user.editMessage('un message')
  }
}
```

Une différence entre *JavaScript* et *TypeScript* est que nous pourrions faire en *JavaScript* une vérification sur l'existence de la méthode :

```ts
function uneFonction(user: User | Moderator) {
  if (user.editMessage) {
    user.editMessage('un message')
  }
}
```

En *TypeScript* ce code génère une erreur car pour lui vous essayez d'accéder à une méthode qui n'est pas sûre d'être présente.

Nous allons voir plusieurs méthodes. La première que nous avons déjà vu est l'utilisation d'une propriété discriminante comme vu plus haut : *user.role === 'moderator'*.

### Utiliser des prédicats

**En informatique, une fonction de prédicat, souvent abrégée en *predicate* en anglais, est une fonction qui prend un élément en paramètre et retourne un booléen suivant que l'élément satisfasse une condition.**

Nous en avons vu beaucoup avec par exemple `includes()`, `some()`, `every()` etc. Il y en a de très nombreux en *JavaScript*.

Ici, notre prédicat va simplement vérifier qu'un élément est d'un certain type et va le dire à *TypeScript*. En reprenant le même exemple nous pouvons faire :

```ts
interface User {
  role: 'user';
  name: string;
  email: string;
}

interface Moderator {
  role: 'moderator';
  name: string;
  email: string;
  editMessage: (msg: string) => string
}

function uneFonction(user: User | Moderator) {
  if (isMod(user)) {
    user.editMessage('un message')
  }
}

function isMod(user: User | Moderator): user is Moderator {
  return (user as Moderator).editMessage !== undefined;
}
```

Ici, nous vérifions que notre utilisateur possède la méthode `editMessage()` lors de l'exécution et nous retournons *true* ou *false* en fonction du résultat.

Notez l'utilisation de `user is Moderator` dans le type de retour de notre prédicat.

**L'opérateur `is` de *TypeScript* permet simplement de réduire le type possible : c'est ce qu'on appelle un *prédicat de type*.**

Le compilateur de *TypeScript* sait que nous testons qu'une valeur est d'un type particulier et considère donc que l'accès à la méthode est sécurisé.

### Utiliser l'opérateur `in`

Une autre manière est d'utiliser l'opérateur *JavaScript* `in`

**L'opérateur `in` renvoie un booléen suivant qu'une propriété donnée appartient ou non à un objet.**

Il vérifie également sur la chaîne de prototype que la propriété est présente ou non.

En reprenant notre exemple nous pourrions faire :

```ts
interface User {
  role: 'user';
  name: string;
  email: string;
}

interface Moderator {
  role: 'moderator';
  name: string;
  email: string;
  editMessage: (msg: string) => string
}

function uneFonction(user: User | Moderator) {
  if ('editMessage' in user) {
    user.editMessage('un message')
  }
}
```

Ici, *TypeScript* ne se plaindra plus car il comprend que nous vérifions l'existence de la méthode avant de tenter d'y accéder.

### Utiliser l'opérateur `instanceof`

Pour les classes, il est possible de tester le type des instances facilement et donc de pouvoir créer des gardes de type :

```ts
class User {
  constructor(public role: 'user', public name: string, public email: string) {}
}

class Moderator {
  constructor(public role: 'moderator', public name: string, public email: string) {}

  editMessage(msg: string){}
}

function uneFonction(user: User | Moderator) {
  if (user instanceof Moderator) {
    user.editMessage('un message')
  }
}
```

## Les intersections

**Les intersections permettent de combiner plusieurs types en un seul en utilisant `&`.**

Elles permettent de spécifier qu'un objet doit respecter plusieurs *interfaces* et donc avoir l'ensemble des types définis dans ces *interfaces*.

Prenons un premier exemple :

```ts
interface User {
    name: string;
    email: string;
  }

  interface Moderator {
    role: 'moderator';
    editMessage: (msg: string) => string
  }


  let moderateur: User & Moderator;
```

Ici pour respecter l'intersection des deux *interfaces*, il faudra avoir un objet de la forme :

```ts
moderateur = {
    name: 'Jean',
    email: 'jean@gmail.com',
    role: 'moderator',
    editMessage : (message: string) => 'Message édité'
  }
```

### Nouveaux opérateurs

L'avantage de *TypeScript* est qu'il permet d'utiliser toutes les dernières fonctionnalités de *JavaScript* car il est ensuite compilé suivant la *target* pour rendre le code compatible avec tous les navigateurs.

Nous allons voir deux nouveaux opérateurs *JavaScript* que vous pouvez déjà utiliser en *TypeScript*, ou en *JavaScript* en utilisant *Webpack*.

### L'opérateur de coalescence des *nulls* : `??`

**L'opérateur `??` permet de renvoyer l'expression à sa droite, si l'expression à sa gauche vaut *null* ou *undefined*.**

Vous avez dû rencontrer déjà ce pattern d'**affectation d'une valeur par défaut** :

```ts
let utilisateur = user || 'inconnu';
```

L'opérateur `||` permet de forcer l'évaluation de `user` en booléen.

Si c'est une valeur *falsy*, `'inconnu'` sera retourné, si c'est une valeur *truthy* alors ce sera `user` qui sera assigné.

Pour rappel les valeurs **falsy** sont *false*, *null*, *undefined*, *0*, *NaN*, ''*,* "" et *``*.

Il y a un risque d'erreur dans certaines situations par exemple :

```ts
let unTexte = '';

let texte = unTexte || 'Texte par défaut';
```

Dans ce cas nous aurons le texte par défaut alors que nous souhaitions peut être avoir une chaîne de caractères vide.

Dans ce cas, il est plus facile d'utiliser le nouvel opérateur qui permet de raisonner plus facilement :

```ts
let texte = unTexte ?? 'Texte par défaut';
```

### L'opérateur de chaînage optionnel

Vous avez l'habitude de l'opérateur de chaînage qui permet d'accéder à une propriété d'un objet, et ce même sur plusieurs niveaux :

```ts
const monObj = {
  prop1: {
    prop2: true
  }
}

monObj.prop1.prop2;
```

Or, en *JavaScript*, tenter d'accéder à une propriété d'un objet qui n'existe pas et vaut *undefined* renvoie une erreur.

Exemple :

```ts
const monObj2 = {}

monObj2.prop1.prop2;
```

Ici vous aurez l'erreur suivante : 
>*TypeError: Cannot read property 'prop2' of undefined*.

Il est donc commun d'avoir ce pattern :

```ts
if (user && user.local) {
  user.local.email
}
```

Nous vérifions l'existence de l'objet et de la propriété de l'objet avant de tenter un accès à une propriété d'un objet plus profonde.

**L'opérateur de chaînage optionnel `?` permet de renvoyer *undefined* et non pas une erreur si on tente d'accéder à une propriété d'un objet qui n'existe pas.**

Nous pouvons ainsi directement faire :

```ts
if (user?.local?.email) {
  user.local.email
}
```

**De même pour l'accès aux méthodes d'objets,** si nous faisions :

```ts
const a = { b: {}}
a.b.hello();
```

Nous aurions l'erreur 
>*TypeError: a.b.hello is not a function*.

Alors que si nous utilisons l'opérateur de chaînage optionnel :

```ts
let uneVal = a.b?.hello?.();
```

Nous n'aurons plus de problème si *prop* ou *methode* ne sont pas définies, *uneVal* vaudra simplement *undefined* mais nous n'aurons plus d'erreur de type et donc plus de crash.

## Les alias de type

**Les alias de type sont simplement un nom donné à un type.**

Ce type peut être n'importe quoi : un autre type, une union d'interfaces, une intersection d'un type primaire et d'une interface etc.

La syntaxe est très simple :

```ts
type nomDuType = LES_TYPES
```

Par exemple :

```ts
type Chaine = string;
```

Nous avons un alias pour le type primaire *string* et nous pouvons l'utiliser partout :

```ts
let maVar: Chaine;
```

Le plus souvent vous les utiliserez pour **les unions de type littéral**.

Au lieu de devoir faire par exemple à chaque fois :

```ts
let animal: 'chat' | 'chien' | 'koala';

function sortir(animal: 'chat' | 'chien' | 'koala') {}
```

Il est préférable de déclarer un alias et de le réutiliser :

```ts
type Animal = 'chat' | 'chien' | 'koala';

let animal: Animal;

function sortir(animal: Animal) {}
```

### Alias vs *interfaces*

Les types n'existent pas non plus, comme les interfaces, après la compilation.

Un type peut s'utiliser comme une *interface*, vous pouvez créer un type pour un objet, une fonction etc comme vous le feriez avec une *interface*.

Nous allons voir ensemble les quelques différences entre les types et les interfaces :

**Premièrement, les *interfaces* créent un nom dans l'*IDE***.. La conséquence est double : d'abord, lorsque vous passez la souris sur une interface vous aurez son nom, alors que lorsque vous passez la souris sur un alias vous aurez les types que l'alias recouvre. Ensuite, lorsque vous aurez un message d'erreur de compilation, dans le cas d'une *interface*, l'erreur la mentionnera, alors que pour un alias vous aurez simplement le ou les types qui ne sont pas respectés.

Faites l'essai en copiant le code suivant dans *VS Code* :

```ts
type Alias = { nombre: number };
interface Interface {
  nombre: number;
}
```

Essayez également de créer des erreurs :

```ts
let a: Alias = {a: true};
let b: Interface = {a: true};
```

**Deuxièmement, contrairement aux *interfaces*, il n'est pas possible de fusionner des alias.**

La fusion d'*interface*, rarement utilisée, permet d'effectuer plusieurs déclarations d'une même *interface* dont les propriétés sont fusionnées automatiquement :

```ts
interface User {
  age: number;
  name: string;
}

interface User {
  address: string;
}
```

Dans cet exemple, pour respecter l'*interface User*, il faudra avoir les propriétés *age*, *name* et *address* : les deux déclarations sont fusionnées.

Cette fusion n'est pas possible pour les alias, une seule déclaration est possible.

### Recommandation

Notre recommandation est de toujours utiliser les *interfaces* dans votre application pour tous vos modèles.

Nous vous conseillons de réserver les alias pour les unions de types littéraux vus précédemment. C'est un excellent cas d'utilisation.