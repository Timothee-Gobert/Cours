# Les interfaces

## Introduction aux interfaces

### Qu'est-ce qu'une interface ?

**Une interface est un contrat qui définit la forme que doit prendre un objet *JavaScript* (objets littéraux, classes et fonctions).**

Il est important de noter qu'une interface n'est pas compilée : aucun code *JavaScript* ne sera créé à partir de l'interface.

Elle sert uniquement pour l'*IDE*, c'est-à-dire *VS Code*, (auto-complétions et erreurs de types) et lors de la compilation avec *tsc* (analyse statique des types).

### Notre première interface

Prenons un premier exemple avec un objet :

```ts
interface User {
  prenom: string;
}
```

Ici, notre objet *User* doit obligatoirement avoir une propriété *prenom* contenant une chaîne de caractères.

Ensuite nous pouvons utiliser l'*interface* pour typer par exemple un paramètre d'une fonction :

```ts
function printUser(user: User) {
  console.log(user.prenom);
}
```

Ici, la fonction *printUser* doit recevoir en argument un objet respectant l'*interface User*.

Par exemple :

```ts
let paul = {prenom: 'Paul'};

printUser(paul);
```

**Attention ! Lorsque vous typez le paramètre de cette manière *user: User***, cela revient à dire que le paramètre **doit au moins avoir le ou les propriétés de l'interface *User* et donc avoir une propriété *prenom*.**

**En revanche, lorsque vous typez un objet avec une interface, il faut que l'objet respecte exactement l'interface.** C'est-à-dire qu'il doit avoir exactement les propriétés définies, ni plus ni moins.

Voici un exemple pour bien comprendre :

```ts
interface User {
  prenom: string;
}

function printUser(user: User) {
  console.log(user.prenom);
}

const paul = {prenom: 'Paul', nom: 'Dupont'};

printUser(paul); // Cela fonctionne car paul a au moins une propriété nom

const jean: User = {prenom: 'Jean', nom: 'Duprey'}; // Erreur
```

Dans le cas de l'objet *Jean*, *TypeScript* nous signale l'erreur suivante : 
>*Type '{ prenom: string; nom: string; }' is not assignable to type 'User'. Object literal may only specify known properties, and 'nom' does not exist in type 'User'.*

Autrement dit, il se plaint car *jean* a une propriété *nom* qui ne respecte pas le contrat imposé par l'interface *User*. 

## Propriétés optionnelles et en lecture seule

### Les propriétés optionnelles avec `?`

Il est possible de définir des **propriétés optionnelles** comme nous l'avons vu pour les arguments de fonction en utilisant `?`.

Ces propriétés sont utilisables également dans les *interfaces*.

```ts
interface User {
  nom?: string;
}
```

Ici, nous établissons comme contrat que les objets qui devront respecter l'*interface User* devront avoir obligatoirement une propriété *prenom* contenant une chaîne de caractères, et une propriété optionnelle *nom*, qui si elle existe, devra également contenir une chaîne de caractères.

En outre, les propriétés optionnelles permettent l'autocomplétion et l'autocorrection.

Par exemple :

```ts
interface User {
  prenom: string;
  nom?: string;
}

function printUser(user: User) {
  console.log(user.prnom);
}
```

Ici l'autocorrection vous signalera : 
>*Property 'prnom' does not exist on type 'User'. Did you mean 'prenom'?*.

### Propriété en lecture seule avec readonly

Certaines propriétés ne doivent être modifiables que lorsqu'un objet est créé pour la première fois.

Comme nous l'avons vu, vous pouvez spécifier qu'une propriété est en lecture seule en mettant *readonly* devant le nom de la propriété :

```ts
interface User {
  readonly prenom: string;
}
```

Ici, les objets typés avec l'*interface User* devront contenir une propriété *prenom* en lecture seule.

Exemple :

```ts
interface User {
  readonly prenom: string;
}

const jean: User = {prenom: 'Jean'};
jean.prenom = 'Paul'; // Erreur
```

*TypeScript* vous signale :
>*Cannot assign to 'prenom' because it is a read-only property.*.

Autrement dit que la propriété est en lecture seule et ne peut être modifiée après sa première assignation.

La lecture seule fonctionne également pour les tableaux, par exemple :

```ts
interface User {
  prenom: string;
  allergies: readonly string[]
}

const jean: User = {prenom: 'Jean', allergies: ['pollen', 'poussière']};
jean.allergies.push('poils de chat'); // Erreur
```

Ici, vous aurez l'erreur suivante : 
>*Property 'push' does not exist on type 'readonly string[]'.*

Les tableaux en readonly se déclarent également en faisant :

```ts
ReadonlyArray<string>
```

En fait, avec les tableaux en lecture seule, *TypeScript* retire toutes les méthodes permettant la modification des tableaux, cela vous permet d'être certain de ne pas modifier votre tableau.

## Les types indexables

Les types indexables permettent de définir le type de clé de propriétés d'un objet. Pour rappel, les clés d'un objet en *JavaScript* peuvent être une chaîne de caractères ou un nombre.

Par exemple :

```ts
interface User {
  prenom: string;
  [prop: string]: string;
}
```

Ici vous signifiez que *User* pourra avoir une propriété *nom* ou *adresse* ou *email* ou tout autre propriété qui sera dynamiquement créée comme ceci :

```ts
interface User {
  prenom: string;
  [index: string]: string;
}

const jean: User = {prenom: 'Jean'};
jean.nom = 'Dupont';
```

Ici, *TypeScript* ne se plaindra pas car nous lui avons dit que les *User* peuvent avoir des propriétés dont la clé est une chaîne de caractères et la valeur une chaîne de caractères.

### Autre exemple

Supposons que nous voulions utiliser une *interface* pour typer des tableaux contenant des chaînes de caractères, au lieu de faire *string[]*.

Nous procéderions de cette manière :

```ts
interface TableauString {
  [index: number]: string;
}
```

Ici, les clés sont des nombres car nous sommes dans un tableau, et les valeurs des chaînes de caractères :

```ts
let myArray: TableauString = ["Jean", "Paul"];
```

### Recommandations

L'utilisation d'un type indexable oblige que toutes les propriétés respectent le type de l'index et la valeur de retour

Par exemple :

```ts
interface User {
  [index: string]: number;
  age: number;
  prenom: string; // Erreur
}
```

Ici vous aurez l'erreur : 
>*Property 'prenom' of type 'string' is not assignable to string index type 'number'.*, car *prenom* ne retourne pas un nombre.

Or, un type indexable type l'ensemble des propriétés de l'objet.

Les types indexables sont utiles lorsque vous aurez des dictionnaires dans certains objets.

Par exemple un dictionnaire de l'id des utilisateurs actifs :

```ts
interface ActiveUsersId {
  [index: string]: string;
}
```

## Interfaces et fonctions

### Typer les fonctions avec des interfaces

Nous avions vu comment typer les arguments des fonctions et définir leur signature.

Il est possible de créer des *interfaces* pour nos fonctions.

Prenons un premier exemple :

```ts
interface FonctionDeRecherche {
  (critere1: number, critere2: string): boolean;
}
```

Ici, cette fonction acceptera en premier paramètre un critère de type nombre et en second paramètres un critère de type chaîne de caractère et retournera un booléen.

Vous pouvez ainsi ensuite typer et implémenter une telle fonction :

```ts
let rechercher: FonctionDeRecherche = (crit1: number, crit2: string) => {
  // recherche...et si trouvé on retourne true :
  return true;
};
```

Notez que le nom des paramètres n'est pas important avec l'utilisation d'une *interface*, ce que *TypeScript* vérifie c'est : le nombre et les types des paramètres et la valeur de retour.

### Typer les objets contenant des méthodes

Ce qui vous sera utile très souvent est de typer les méthodes de vos objets :

```ts
interface User {
  prenom: string;
  direBonjour(nom:string): void;
}
```

Avec par exemple :

```ts
const user1: User = {
  prenom: "Jean",
  direBonjour(nom: string) {
    console.log(`Bonjour, je m’appelle ${this.prenom} ${nom}`);
  }
}
```

## Compositions d'interfaces et classes

### Les classes peuvent implémenter des interfaces

**Une classe peut implémenter une interface. Cela permet d'indiquer explicitement qu'une classe répond à un contrat particulier, c'est-à-dire à une *interface*.**

Prenons un premier exemple :

```ts
interface User {
  prenom: string;
  nom: string;
  direBonjour(): void;
}

class NamedUser implements User {
  direBonjour() {
    console.log(`Bonjour, je m’appelle ${this.prenom} ${this.nom}`);
  }
  constructor(public prenom:string, public nom: string) {}
}
```

Ici nous devons avoir sur notre classe deux propriétés *prenom* et *nom* et une méthode *direBonjour()*. Les types des propriétés et la valeur retournée par la méthode doivent également respecter l'*interface*.

### L'extension d'interface

Suivant le même principe que pour les classes, il est également possible d'étendre une interface qui en héritera.

Par exemple si nous avons l'*interface* :

```ts
interface User {
  prenom: string;
  direBonjour(): void;
}
```

Nous pouvons créer une seconde *interface* qui hérite de l'*interface* et qui l'étend :

```ts
interface RegisteredUser extends User {
  nom: string;
  adresse: string;
  mail: string;
}
```

Nous pouvons ensuite typer des objets avec l'*interface* fille :

```ts
const jean: RegisteredUser = {
  prenom: 'Jean',
  nom: 'Dupont',
  adresse: '2 rue du chateau 92220',
  mail:'jean.dupont@aol.com',
  direBonjour() {
    console.log(`Bonjour, je m’appelle ${this.prenom} ${this.nom}`);
  }
}
```

### L'extension multiple d'interfaces

Contrairement aux classes qui ne peuvent pas hériter, et donc étendre qu'une seule autre classe, il est possible d'effectuer de l'héritage multiple avec les *interfaces*.

La raison est simple : une classe *TypeScript* ne peut avoir qu'une seule classe parente à cause de l'héritage prototypal (*revoir le chapitre sur les classes **JavaScript** si besoin*), alors qu'une *interface* ne sera pas compilée, il n'y a donc aucune limite sur l'héritage multiple.

Si vous ne pouvez pas faire :

```ts
class A {}

class B {}

class C extends A, B {} // Erreur
```

Vous aurez l'erreur : *Classes can only extend a single class.*

Vous pouvez en revanche faire :

```ts
interface A {
  propA: string;
}

interface B {
  propB: string
}

interface C extends A, B {}

const obj: C = {
  propA: 'A',
  propB: 'B',
  propC: 'C' // Erreur
}
```

Ici notre objet ne respecte l'*interface C* car il a une propriété qui n'est pas présente sur *les interfaces A* et *B* que *l'interface C* étend.

L'erreur est bien : 
>*Type '{ propA: string; propB: string; propC: string; }' is not assignable to type 'C'. Object literal may only specify known properties.*

### Différence entre interfaces et classes abstraites

Vous pourriez vous demander la différence entre une classe abstraite et une classe dans le contexte du typage de classes.

La différence est qu'une **classe abstraite est un contrat (comme une *interface*), mais aussi une implémentation partielle d'une classe.**

Ce qui fait qu'une classe abstraite, contrairement à une *interface*, est compilée et présente lors de l'exécution.

**En général nous recommandons l'utilisation des interfaces sauf dans deux cas.**

Premièrement, lorsque vous devez vérifier qu'une instance est une instance d'un certain type lors de l'exécution, et que vous devez donc par exemple utiliser *instanceof* :

```ts
if (x instanceof ClasseAbstraite)
```

Dans ce cas il est obligatoire d'utiliser une classe ou une classe abstraite car une interface n'est pas disponible lors de l'exécution.

Deuxièmement, lorsque vous voulez spécifier comment des méthodes seront utilisées, c'est-à-dire d'implémenter certaines méthodes et pas seulement de définir un contrat :

```ts
abstract class ClasseAbstraite {
  protected abstract methodeA(): void
  protected abstract methodeB(): void
  methodeC() {
    this.methodeA()
    this.methodeB()
  }
}

class A extends ClasseAbstraite {
  methodeA() {
    console.log('A');
  }
  methodeB() {
    console.log('B');
  }
}

const instance = new A();
instance.methodeC();
```

C'est une situation assez avancée que vous rencontrerez très rarement, mais dans ce cas il faudra utiliser une classe abstraite.

## Extension d'une classe par une interface

### Une *interface* peut étendre une classe

**Une *interface* peut hériter d'une classe et l'étendre.**

Dans ce cas, l'*interface* hérite de la classe, à savoir de la forme des propriétés et méthodes de la classe mais pas de leur implémentation.

Prenons un premier exemple proche de la vidéo :

```ts
class Player {
  constructor(public isPlaying: boolean) {}
}

interface PlayerBasic extends Player {
  play(): void;
}
```

Ici si nous voulons respecter la forme de *Player* et la forme de *PlayerBasic* nous devons faire :

```ts
class Game implements PlayerBasic {
  play() {}

  constructor(public isPlaying: boolean) {}
}
```

La classe *Game* implémente bien une méthode *play()* qui ne retourne rien, et a bien une propriété publique *isPlaying* qui est un booléen.

### Forcer l'héritage d'une classe

Là où cela devient intéressant est qu'une *interface* qui étend une classe avec des modificateurs *private* ou *protected* va obliger que l'implémentation de ces propriétés ou méthodes privées ou protégées soit effectuées par la classe ou une classe fille (dans le cas d'une propriété protégée).

Prenons un exemple pour bien comprendre :

```ts
class Player {
  constructor(private isPlaying: boolean) {}
}

interface PlayerBasic extends Player {
  play(): void;
}
```

Cette fois-ci, la propriété est privée ! Pour implémenter correctement l'*interface PlayerBasic* il faut maintenant obligatoirement que l'implémentation soit faites par la classe *Player*.

Cela signifie qu'une classe doit obligatoirement étendre la classe *Player* et en hériter pour respecter l'*interface PlayerBasic*.

Ici vous aurez une erreur :

```ts
class Game implements PlayerBasic {
  play() {}
  constructor(private isPlaying: boolean) {}
}
```

L'erreur est la suivante : 
>*Class 'Game' incorrectly implements interface 'PlayerBasic'. Types have separate declarations of a private property 'isPlaying'*.

Autrement dit, la propriété *isPlaying* n'est pas implémentée par la classe *Player* et donc l'*interface PlayerBasic* n'est pas respectée, il faudrait faire :

```ts
class Game extends Player implements PlayerBasic {
  play() {}
}
```

Cela fonctionne aussi pour les méthodes, prenons un exemple :

```ts
class Player {
  protected record() {}
}

interface PlayerBasic extends Player {
  play(): void;
}

class Game implements PlayerBasic {
  play() {}

  record {}
}
```

Ici vous aurez une erreur :
>*Class 'Game' incorrectly implements interface 'PlayerBasic'. Property 'record' is protected but type 'Game' is not a class derived from 'Player'*.

Autrement dit, la méthode *record()* n'est pas implémentée par la classe *Player* ou une classe fille de la classe *Player* et donc l'*interface PlayerBasic* n'est pas respectée, il faudrait faire :

```ts
class Game extends Player implements PlayerBasic {
  play() {}
}
```

