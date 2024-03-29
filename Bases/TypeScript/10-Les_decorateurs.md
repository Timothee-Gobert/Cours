# Les décorateurs

## Introduction aux décorateurs

### Que sont les décorateurs ?

**Les décorateurs sont une fonctionnalité qui s'utilise avec des classes et permettent de les annoter ou de les modifier.**

Les décorateurs sont pour le moment une proposition qui pourrait être adoptée dans les spécifications *ECMAScript* pour le *JavaScript*.

*TypeScript* permet déjà leur utilisation en mode expérimental, c'est pour cette raison qu'il faut paramétrer le fichier de configuration pour les utiliser.

Dans le fichier de configuration *tsconfig.json*, il faut ajouter :

```ts
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

### La syntaxe des décorateurs

**Les décorateurs peuvent s'utiliser sur la classe elle-même, sur les méthodes, sur les accesseurs (*get / set*) et sur les paramètres.**

La syntaxe des décorateurs est *@decorateur*, où *decorateur* est le nom de la fonction qui sera appelée lors de l'exécution.

Prenons un premier exemple :

```ts
function decorateur(cible: UneClasse) {
  console.log(cible);
}

@decorateur
class UneClasse {}
```

Nous avons ici une classe appelée *UneClasse* sur lequel nous avons déclaré un décorateur.

Lors de l'exécution, et même si nous n'instancions pas la classe, le décorateur est appelé et la fonction `decorateur()` est exécutée.

Elle reçoit un seul argument : le constructeur de la classe cible sur lequel le décorateur est utilisé.

Nous allons voir dans les prochaines leçons de nombreux cas d'utilisation.

## Utilisation des décorateurs sur les classes

### Les factories de décorateurs

*Factory* signifie usine. En programmation lorsque nous parlons de *factory*, nous désignons une fonction constructeur qui va retourner un objet spécifique.

Ici, une *decorator factory* ou usine à décorateurs, va servir à générer un décorateur lors de l'exécution pour pouvoir lui passer des paramètres.

Voici un exemple :

```ts
function factory(param: string) {
  return (cible) =>  {
  }
}
```

La fonction *factory* est l'usine qui va retourner lors de l'exécution une autre fonction qui est le décorateur.

Pourquoi utiliser ce *pattern* ? Pour pouvoir définir des paramètres lors de l'utilisation sur la classe, on peut maintenant faire :

```ts
function factory(param: string) {
  return (cible: UneClasse) =>  {
  }
}

@factory('une valeur')
class UneClasse {}
```

Nous pouvons maintenant passer un argument lors de l'utilisation du décorateur, ce que nous ne pouvons pas faire avec un décorateur sans *factory* car il ne peut pas prendre de paramètre.

En effet, un décorateur reçoit en argument la cible et ne peut pas prendre de paramètres supplémentaires.

### Exemple d'utilisation de la vidéo

Vous pouvez retrouver le code utilisé dans la vidéo :

```ts
function ComponentFactory({ template, selector}: {template: string, selector: string}) {
  return (cible: MyComponent) => {
    const elem = document.querySelector(selector);
    if (elem) elem.innerHTML = template;
  }
}


@ComponentFactory({
  template: '<h1>Hello</h1>',
  selector: 'app'
})
class MyComponent {
  constructor(public name: string) {}
}
```

Notez que `{ template, selector}: {template: string, selector: string}` permet de typer les paramètres et d'effectuer une affectation par décomposition. Vous pouvez revoir les affectations par décomposition dans le cours *JavaScript* si vous ne vous en rappelez plus.

Ici le fait de mettre le code avant ou dans la fonction décorateur n'a aucune incidence car le code est exécuté immédiatement lors de l'exécution et non lors de l'instanciation.

Nous avons montré cet exemple pour vous montrer un peu comment le framework *Angular* utilise les décorateurs. Aucune importance si vous ne connaissez pas le framework et ne comptez pas l'apprendre, il s'agit simplement d'un exemple important d'utilisation.

### Autre exemple de décorateur de classe

Nous allons prendre un second exemple de décorateur de classe.

```ts
function geler(constructor: Function) {
  Object.freeze(constructor);
  Object.freeze(constructor.prototype);
}


@geler
class UneClasse {
  static methode() {
      console.log('Hello')
  }
}

UneClasse.methode(); // Hello

UneClasse.methode = () => { } // Erreur
```

Vous pouvez revoir ce que fait `Object.freeze()` dans le cours *JavaScript* : il permet d'empêcher la modification d'un objet.

Ici, nous nous servons du décorateur pour empêcher la modification de la classe (qui est un type particulier de fonction constructrice comme nous l'avons vu dans le cours *JavaScript*) et du prototype de la classe.

Pour rappel, lorsque nous mettons le mot clé *static* nous définissons la méthode sur la classe elle-même et non sur son prototype.

Comme nous gelons la classe, vous aurez une erreur lorsque vous tenterez une assignation :

```ts
TypeError: Cannot assign to read only property 'methode'
```

Voici un exemple intéressant permettant de geler ses classes plus facilement en utilisant un décorateur.

## Composition de décorateurs

### Utilisation de plusieurs décorateurs

Vous pouvez utiliser autant de décorateurs que vous voulez sur une même classe, par exemple :

```ts
@decorateur1 @decorateur2
class MaClasse {}
```

Ou sur plusieurs lignes :

```ts
@decorateur1
@decorateur2
class MaClasse {}
```

Ou encore avec certains utilisant une *factory*.

Nous pouvons ainsi combiner nos exemples précédents :

```ts
function ComponentFactory({ template, selector}: {template: string, selector: string}) {
  return (cible: MyComponent) => {
    const elem = document.querySelector(selector);
    if (elem) elem.innerHTML = template;
  }
}

function geler(constructor: Function) {
  Object.freeze(constructor);
  Object.freeze(constructor.prototype);
}

@geler
@ComponentFactory({
  template: '<h1>Hello</h1>',
  selector: 'app'
})
class MyComponent {
    constructor(public name: string) {}
}
```

### Ordres d'évaluation et d'exécution des décorateurs

Notez une chose importante : l'ordre d'évaluation et d'exécution des décorateurs ne sont pas les mêmes :

L'évaluation des décorateurs se fait de **haut en bas suivant leur ordre de déclaration.**

L'exécution des décorateurs se fait de **bas en haut suivant leur ordre de déclaration.**

En reprenant notre exemple nous aurions :

```ts
function ComponentFactory({ template, selector}: {template: string, selector: string}) {
    console.log(1)
    return (cible: MyComponent) => {
      console.log(2)
      const elem = document.querySelector(selector);
      if (elem) elem.innerHTML = template;
    }
}

function geler(constructor: Function) {
  console.log(3)
  Object.freeze(constructor);
  Object.freeze(constructor.prototype);
}

@geler
@ComponentFactory({
  template: '<h1>Hello</h1>',
  selector: 'app'
})
class MyComponent {
  constructor(public name: string) {}
}
```

En tout premier nous avons l'évaluation de *@geler*, comme il n'y a pas de *factory* rien n'est exécuté pour le moment.

En deuxième, nous avons l'évaluation de *@ComponentFactory*. A cette étape la fonction *factory* est exécutée car nous sommes lors de l'étape d'évaluation.

Ensuite nous passons en phase d'exécution des décorateurs, comme c'est de bas en haut suivant l'ordre de déclaration, nous avons d'abord le décorateur retourné par *@ComponentFactory* qui est exécuté, puis le décorateur *@geler* qui est déclaré au dessus.

La plupart du temps les ordres d'exécution et d'évaluation ne sont pas importants, mais il faut quand même le savoir car les résultats peuvent être étranges et cela peut être difficile de déboguer.

## Utilisation des décorateurs sur les méthodes et accesseurs

### Utilisation des décorateurs sur les méthodes

Comme pour les décorateurs utilisés sur les classes, les décorateurs utilisés sur les méthodes se déclarent juste avant leur déclaration :

```ts
@DecorateurMethode
uneMethode() {}
```

Le décorateur sera exécuté avec trois arguments :
1. En premier argument, **la cible du décorateur**, qui sera soit le constructeur de la classe si la méthode est statique, ou le prototype de la classe dans le cas contraire.
2. En second argument, **le nom de la méthode.**
3. En troisième argument, **le descripteur de propriété** de la méthode (si vous n'êtes pas familier avec cette notion, revoyez le cours *JavaScript*).

Prenons un exemple :

```ts
function enumerable(valeur: boolean) {
  return function (cible: any, nomMethode: string, descripteur: PropertyDescriptor) {
    descripteur.enumerable = valeur;
  };
}

class UneClasse {
  @enumerable(true)
  static methode() {
    return "Coucou !";
  }
}

for (const prop in UneClasse) {
    console.log(prop); // methode
}
```

Ici nous utilisons un décorateur pour modifier le descripteur d'une méthode statique.

Il s'agit d'une usine de décorateur (*decorator factory*) que nous avons déjà vue. Il est nécessaire de l'utiliser pour pouvoir passer un argument (ici un booléen) au décorateur.

Essayez de passer en argument *true* ou *false*. Si la méthode est énumérable elle sera affichée par le *for … in* qui permet de parcourir les propriétés énumérables de tout objet.

### Utilisation des décorateurs sur les accesseurs

Les décorateurs pour les accesseurs (*get* et *set*) fonctionnent comme pour les méthodes.

A noter qu'il n'est pas possible d'utiliser un décorateur sur les deux accesseurs (*get* et *set*) d'une propriété. La raison est que le décorateur permet de modifier le descripteur de propriété, et qu'il est commun aux deux accesseurs. Il serait donc dangereux d'autoriser deux modifications pouvant être contradictoires.

Les arguments sont exactement les mêmes que pour les méthodes, à savoir :
1. En premier argument, **la cible du décorateur**, qui sera soit le constructeur de la classe si la propriété est statique, ou le prototype de la classe dans le cas contraire.
2. En second argument, **le nom de la propriété**.
3. En troisième argument, **le descripteur de la propriété**.

Prenons un exemple :

```ts
class UneClasse {
  constructor(private _x: number, private _y: number) {}

  @enumerable(true)
  get x() { return this._x; }

  @enumerable(true)
  get y() { return this._y; }
}

function enumerable(valeur: boolean) {
  return function (target: any, nomProp: string, descripteur: PropertyDescriptor) {
    descripteur.enumerable = valeur;
  };
}

const instance = new UneClasse(1, 2);

for (const prop in instance) {
  console.log(prop); // _x _y x y
}
```

Ici, nous rendons nos accesseurs énumérables sur les instances de notre classe grâce à notre décorateur que nous avons déjà utilisé dans l'exemple précédent.

## Utilisation des décorateurs sur les propriétés et les paramètres

### Utilisation des décorateurs sur les propriétés

Un décorateur sur une propriété s'utilise en le déclarant juste avant la propriété.

Il reçoit deux arguments :
1. En premier argument, **la cible du décorateur**, qui sera soit le constructeur de la classe si la propriété est statique, ou le prototype de la classe dans le cas contraire.
2. En second argument, **le nom de la propriété**.

Prenons l'exemple de la vidéo :

```ts
function Prop(cible: any, nomProp: string) {
  console.log(cible, nomProp)
}

class UneClasse {
  @Prop
  public name: string;

  constructor(name: string) {
    this.name = name;
  }

  public greeting(fancy?: boolean) {

  }
}
```

L'utilisation de décorateurs pour les propriétés n'est pas très courant, il peut être utilisé par certains frameworks ou librairies comme par exemple *Typegoose* pour utiliser les types avec *Mongoose* (un *ORM* utilisé avec *MongoDB*).

### Utilisation des décorateurs sur les paramètres

Un décorateur sur un paramètre s'utilise en le déclarant juste avant ce dernier.

Il reçoit trois arguments lors de l'exécution :
1. En premier argument, **la cible du décorateur**, qui sera soit le constructeur de la classe si la propriété est statique, ou le prototype de la classe dans le cas contraire.
2. En second argument, **le nom de la méthode sur lequel le paramètre est déclaré**.
3. En troisième argument, l'**index du paramètre**.

Reprenons l'exemple de la vidéo :

```ts
function Prop(cible: any, nomProp: string) {
  console.log(cible, nomProp);
}

function Param(cible: any, nomMethode: string, indexParam: number) {
  console.log(cible, nomMethode, indexParam);
}

class UneClasse {
  @Prop
  public name: string;

  constructor(name: string) {
    this.name = name;
  }

  public greeting(@Param fancy?: boolean) {
    console.log(`${fancy ? 'Hello' : 'Hi'} ${this.name}`)
  }
}
```

## Utilisation avancée des décorateurs

### Code de la vidéo

Voici le code de l'exemple :

```ts
function ComponentFactory({ template, selector}: {template: string, selector: string}) {
  return <T extends {new (...args: any[]): {}}>(constructor: T): T => {
    return class MyComponentElement extends constructor {
      private selector: string = selector;
      private template: string = template;

      constructor(...args: any[]) {
        super(args);
      }

      render() {
        const elem = document.querySelector(selector);
        if (elem) elem.innerHTML = template;
      }
    }
  }
}

interface Component {
  render: () => void
}


@ComponentFactory({
  template: '<h1>Hello</h1>',
  selector: 'app'
})
class MyComponent implements Component {
  constructor(public name: string) { }

  render() {}
}

const foo = new MyComponent('Jean');
```

Nous allons donner des explications supplémentaires sur le code *TypeScript* assez complexe.

#### Exemple plus simple

Commençons par un exemple un peu plus simple :

```ts
function Decorateur<T extends {new(...args:any[]):{}}>(constructor:T) {
  return class extends constructor {
    nouvelleProp = "nouveau";
    test = "valeur";
  }
}
```

Ici nous avons une fonction qui est une usine de décorateur. Elle va permettre de modifier le constructeur d'une classe sur laquelle est utilisée le décorateur.

Nous typons l'argument reçu par la fonction utilisée comme décorateur **pour restreindre son type à un constructeur de classe :**

```ts
T extends {new(...args:any[]):{}}
```

Ce code permet de restreindre le type générique `T` à un constructeur.

Un constructeur a en effet obligatoirement une fonction `new()` qui reçoit un nombre indéfini d'arguments et retourne un objet qui est une instance de la classe.

Rappelez-vous qu'un décorateur utilisé sur une classe reçoit comme argument le constructeur de la classe.

Ensuite nous avons :

```ts
return class extends constructor {
  nouvelleProp = "nouveau";
  test = "valeur";
}
```

Nous retournons une nouvelle classe qui étend le constructeur originel passé en argument.

#### Explications ligne par ligne

Poursuivons avec les explications du code :

```ts
function ComponentFactory({ template, selector}: {template: string, selector: string}) {
```

Nous avons notre usine de décorateur qui va recevoir en arguments le *template* et le *selector* passé à celui-ci lors de l'utilisation du décorateur ici :

```ts
@ComponentFactory({
  template: '<h1>Hello</h1>',
  selector: 'app'
})
class MyComponent implements Component {
```

L'usine de décorateur retourne ensuite un décorateur qui est ici une fonction anonyme :

```ts
return <T extends {new (...args: any[]): {}}>(constructor: T): T => {
```

Cette fonction reçoit comme seul argument le constructeur de la classe cible que nous typons avec le type générique `T`.

Ensuite, nous restreignons ce type `T` à un objet qui contient une méthode `new()` recevant un nombre indéfini d'arguments et qui retourne un objet. Autrement dit, nous restreignons le type `T` comme étant un constructeur.

```ts
return class MyComponentElement extends constructor {
```

Le décorateur retourné par l'usine de décorateur va lui même retourné une nouvelle classe qui va remplacer la classe originelle. Cette nouvelle classe va étendre le constructeur de la classe originelle.

Ensuite nous déclarons des propriétés et une méthode sur la nouvelle classe retournée :

```ts
private selector: string = selector;
private template: string = template;

constructor(...args: any[]) {
  super(args);
}

render() {
  const elem = document.querySelector(selector);
  if (elem) elem.innerHTML = template;
}
```

Nous n'oublions pas d'appeler le constructeur de la classe originelle en exécutant *super* et en lui passant tous les arguments reçus.

Nous avons vu basiquement comment les décorateurs sont utilisés par exemple par le *framework Angular* pour apporter des fonctionnalités avancées aux classes.

