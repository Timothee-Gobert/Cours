# Les types

## Introduction aux types

### Les types

Comme son nom l'indique et comme nous l'avons vu *TypeScript* est avant tout un langage pour typer. Il est possible de typer absolument tout comme nous le verrons : que ce soit les variables mais aussi les retour de fonction ou des situations encore plus complexes.

Typer au début peut sembler fastidieux : il faut ajouter un peu de code à chaque fois.

Mais les bénéfices se remarquent très vite et dans cet ordre souvent :
1. Vous aurez moins de bugs lors de l'exécution car beaucoup auront été signalés lors de la compilation : typo dans les noms, erreurs dans les conditions, dans les arguments passés, dans les retours de fonction etc.
2. VS Code ou tout autre IDE vous fournira avec un survol de la souris plein d'informations sur vos fonctions : quels arguments sont attendus etc, et vous fera gagner un temps fou grâce à l'autocomplétion (car *TypeScript* saura toutes les propriétés sur vos objets etc).
3. Lorsque vous serez sur un projet à plusieurs, vous pourrez lire et comprendre beaucoup plus vite le code des autres développeurs car vous verrez facilement ce que prend et ce que retourne chaque fonction, quels sont les types acceptés par chaque variable etc.

Une fois les premiers jours où cela semblera être une surcouche embêtante passés, vous ne pourrez plus faire sans !

### Les types primitifs

Nous allons commencer par les types les plus basique : les primitives !

Pour spécifier un type en *TypeScript* nous utilisons la plupart du temps (nous verrons en détails les autres cas) cette syntaxe :

```js
variable: type;
```

Il est possible de simplement typer une variable sans rien assigner ou typer et assigner directement :

```js
variable: type;
variable: type = x;
```

#### Les chaînes de caractères : *string*

Les chaînes de caractères en *TypeScript* comprennent exactement comme en *JavaScript* les guillemets simples ou doubles et les accents graves (*backticks*) :

```ts
let prenom: string = "Jean";
prenom = 'Paul';
prenom = `Patrick`
```

#### Les nombres : *number*

Tous les nombres valides en *JavaScript* le sont en *TypeScript* :

```ts
let decimal: number = 42;
let flottant: number = 42.22;
let hexadecimal: number = 0x2A;
let binaire: number = 0b101010;
let octal: number = 0o52;
```

#### Les booléens : *boolean*

Le type *boolean* n'accepte que **true** ou **false** :

```ts
let actif: boolean;
actif = true;
actif = false;
```
 
## Les types any et object

### Le type *any*

Il est possible de ne pas connaître le type de variables.

Ces valeurs peuvent provenir d'un contenu dynamique, par exemple de l'utilisateur ou d'une bibliothèque tierce.

Il est également possible que vous souhaitez typer plus tard votre fonction ou variable ou que vous soyez en train de convertir un fichier *JavaScript* en *TypeScript* et que vous mettiez les types peu à peu.

Dans ces cas, nous voulons désactiver la vérification de type. Pour ce faire, nous les étiquetons avec le type *any*.

```ts
let pasSur: any = 4;
pasSur = ‘en fait une string’;
```

Lorsque vous commencerez en *TypeScript* vous aurez tendance à beaucoup trop utiliser *any*. A chaque fois que vous ne saurez pas trop le type vous mettrez *any*.

Au début ce n'est pas très grave mais il faut vous forcer à faire l'effort de tout bien typer pour garder les bénéfices de l'usage de *TypeScript*.

Nous vous recommandons de ne quasiment jamais utiliser *any* sauf dans les cas vus ci-dessus.

### Le type *object*

Le type *objet* permet simplement de représenter le type **non primitif** : tout **ce qui n'est pas** *number*, *string*, *boolean*, *bigint*, *symbol*, *null* ou *undefined*.

Cela peut donc être un tableau, un objet littéral etc.

```ts
let monObj: object;
monObj = {};
```

Notez que *object* et *{}* sont deux manières de typer un objet, nous pouvons écrire :

```ts
let monObj: {};
monObj = {};
```

Ce type est peu utilisé directement et nous verrons plus tard pourquoi.

## Les types array et tuple

### Le type array

**Il y a deux syntaxes en *TypeScript* pour typer les tableaux.**

**En *TypeScript* un tableau est un tableau *JavaScript* contenant un nombre indéfini d'éléments.**

C'est la différence avec les *tuples*, voir plus bas.

**Première possibilité :**

```js
type[];
```

Où type est le type des éléments contenus dans un tableau. Il est possible d'accepter par exemple que les nombres :

```js
let liste: number[] = [1, 2, 3];
```

Tous les types :

```js
let liste: any[] = [1, true, {}];
```

Deuxième possibilité :

```js
Array<type>
```

Par exemple :

```js
let liste: Array<number>  = [1, 2, 3];
```

**Attention !**

```js
let liste:[];
```

Signifie en *TypeScript* un tableau vide uniquement. Vous ne pourrez assigner à liste qu'un tableau vide avec ce type seul, il faut donc toujours préciser le type (ou les types comme nous le verrons plus tard) contenu dans le tableau.

### Les *tuples*

**Les *tuples* sont des tableaux avec un nombre prédéfinis d'éléments.**

Par exemple si vous souhaitez toujours avoir un nombre puis une string :

```js
// Déclaration d’un type tuple
let x: [string, number];
// Initialisation
x = [“salut”, 3];
// Erreur si on ne respecte pas l’ordre et les types déclarés :
x = [3, “salut”]; // Erreur de compilation
```

## Le type enum

### Les *enum*

Ce type n'existe pas du tout en *JavaScript*, c'est un apport de *TypeScript*.

*enum* vient d'énumération.

Une énumération est un moyen de nommer des ensembles de valeurs, le plus souvent numériques, afin de ne pas se tromper et aussi pour plus de lisibilité dans le code.

Voici un exemple :

```js
enum Role {ADMIN, READ_ONLY, READ_WRITE};

if (user.role === Role.ADMIN) {
  // …
}
```

### Changer la numérotation de départ

Par défaut, les énumérations commencent à numéroter leurs membres à partir de 0.

Vous pouvez modifier ceci en définissant manuellement la valeur d'un de ses membres. Par exemple, nous pouvons commencer l'exemple précédent à 10 au lieu de 0 :

```js
enum Couleur {Bleu = 10, Blanc, Rouge = 5};
let a: Couleur = Couleur.Bleu;
let b: Couleur = Couleur.Blanc;
let c: Couleur = Couleur.Rouge;
console.log(a); // 10
console.log(b); // 11 car incrémenté à partir de la valeur précédente
console.log(c); // 5
```

### Utiliser des chaînes de caractères comme valeur pour les *enum*

Par défaut les *enum* prennent des valeurs numériques de 0 à n.

Mais vous pouvez leur attribuer n'importe quel valeur.

Un exemple plus utile, par exemple pour définir la palette de couleurs de son interface :

```js
enum Couleur { Bleu = '#2980b9', Vert = "#27ae60", Rouge = '#c0392b' };
console.log(Couleur.Bleu); // #2980b9
```

## Les types null, void, undefined et never

### Le type *void*

*void* est un peu comme le contraire de *any*: il signifie l'absence de tout type.

**On l'utilise uniquement pour le typage de retour de fonctions** que nous verrons très en détails :

```ts
function direBonjour(): void { // la fonction ne retourne rien, la valeur de retour est donc void
  console.log(‘Bonjour !’);
```

### Les types *null* et *undefined*

Dans Typescript,*undefined* et *null* ont également leur propre type.

```ts
let n: null = null;
let u: undefined = undefined;
```

Par défaut *null* et *undefined* sont des sous-types de tous les autres types.

Cela signifie que vous pouvez assigner *null* et *undefined* par exemple à *number* :

```ts
let nombre: number = null;
```

Toutefois, lorsque vous utilisez l'option de compilation *strictNullChecks*, *null* et *undefined* ne peuvent être affectés qu'à *any* et à leurs types respectifs.

A noter qu'*undefined* peut être également assigné au type *void* avec ou sans l'option.

Nous reverrons cette option de compilation recommandée également en détails.

### Le type *never*

Le type *never* représente le type de valeurs qui ne se produisent jamais.

Par exemple, *never* est le type de retour pour une expression de fonction qui lève toujours une exception ou celles qui ne retourneront jamais rien.

Les variables acquièrent également le type *never* lorsqu'il leur est attribué un type qui ne peut jamais être rempli.

```ts
function error(message: string): never {
    throw new Error(message);
}

function echec(): never {
    return error(‘Problème !’);
}

function boucleInfinie(): never {
    while(true) {
    }
}
```

On ne se sert jamais de ce type sauf cas particulier.

## L’inférence et l'assertion de types

### L'inférence de type

**L'inférence de type est une fonctionnalité extrêmement puissante de *TypeScript* : elle permet de déduire le type de vos assignations.**

Par exemple :

```ts
let x = 42;
```

Par l'inférence de type, *TypeScript* déclare implicitement que la variable est de type *number*, vous ne pourrez donc pas lui assigner une chaîne de caractères plus tard dans votre code.

Ce type d'inférence se produit lors des assignations de variable, lors de la déclaration de paramètres par défaut pour des fonctions et pour les retours de fonctions.

C'est cette fonctionnalité qui pousse les débutants à mettre *any* partout car le compilateur va se plaindre d'assignation successible de types différents sans que vous ayez indiqué que la variable pouvait avoir plusieurs types.

### L'inférence contextuelle de type

**L'inférence contextuelle est l'inférence de type effectuée par *TypeScript* en fonction de la localisation du code :**

```ts
window.onmousedown = (e) => {
    console.log(e.button); // fonctionne
    console.log(e.random); // Erreur
};
```

Ici *TypeScript* détecte le type de la fonction *onmousedown()*, en effet tous les objets et fonctions natifs, notamment du *DOM*, sont typés par *TypeScript* !

Grâce à cela il peut déduire le type de la fonction déclaré et qu'elle reçoit un *mouseEvent* natif. *TypeScript* sait donc qu'il existe bien une propriété *button* mais pas une propriété *random* sur un *mouseEvent*.

Dans l'exemple suivant, *TypeScript* saura qu'un *scroll* ne retourne pas un *mouseEvent* mais un *UIEvent* sur lequel il n'y a pas de propriété *button* :

```ts
window.onscroll = (e) => {
    console.log(e.button); // Erreur
}
```

**L'inférence est l'une des raisons principales pour utiliser *TypeScript* : sans aucune ligne de code supplémentaire vous bénéficiez d'un contrôle déjà très puissant de l'*IDE* et du compilateur sur tous les types natifs et toutes vos assignations.**

### L'assertion

Dans de rares cas, vous aurez besoin de signaler à *TypeScript* que vous êtes sûr d'un type et qu'il n'a pas à s'en préoccuper.

Autrement dit, vous lui désignez un type et vous l'obligez à le respecter, c'est ce qu'on appelle l'assertion.

L'assertion doit être maniée avec prudence car vous perdez le bénéfice du contrôle de type avant l'exécution.

Pour l'assertion il faut utiliser :

```ts
as type
```

Par exemple :

```ts
let maVar: any = "Une chaîne";

let longueur: number = (maVar as string).length;
```

Ici nous forçons *TypeScript* à reconnaître *maVar* comme étant une chaîne de caractères pour pouvoir utiliser l'autocomplétion et la propriété *length*.

A noter que vous verrez aussi parfois :

```ts
let longueur: number = (<string>maVar).length;
```

Cette seconde notation n'est pas celle à privilégier et elle est dépréciée, vous pourrez cependant la rencontrez avant *TypeScript 3.9* donc il faut la connaitre. 

