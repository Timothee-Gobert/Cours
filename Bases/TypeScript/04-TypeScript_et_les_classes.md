# TypeScript et les classes

## Généralités sur les classes avec TypeScript

### *TypeScript* et les classes

Depuis la version *ES6* de *JavaScript*, les classes sont disponibles nativement en *JavaScript*.

Cependant, il existe plusieurs spécificités à *TypeScript* qui sont très utiles pour la programmation orienté objets.

### La déclaration des propriétés

**Première spécificité, avec *TypeScript*, il est obligatoire de déclarer des propriétés à l'extérieur du *constructor*** (sauf avec la notation raccourcie comme nous le verrons plus tard) :

```ts
class Personne {
  nom: string;
  // ...
}
```

Ce qui donne par exemple :

```ts
class Personne {
  nom: string;

  constructor(nom: string) {
      this.nom = nom;
  }

  sePresenter() {
      return `Bonjour, je m'appelle ${this.nom} !`;
  }
}
```

Cette obligation permet à *TypeScript* de connaitre les propriétés et leurs types sur les classes déclarées.

### L'héritage

Pas de différence avec les classes *JavaScript* concernant l'héritage :

```ts
class Voiture {
  rouler() {
    console.log('Je roule');
  }
}

class Sportive extends Voiture {
  faireLaCourse() {
    console.log('Je fais la course');
  }
}
```

Pas non plus de différence d'utilisation avec l'utilisation de `super()` pour accéder aux propriétés et méthodes de la classe parente.

Il est de même obligatoire d'appeler `super()` pour exécuter le constructeur de la classe parente si on utilise un constructeur dans la classe fille :

```ts
class Voiture {

  sieges: number;

  constructor(sieges: number) {
    this.sieges = sieges;
  }

  rouler() {
    return 'Je roule.';
  }
}

class Sportive extends Voiture {

  chevaux: number;

  constructor(sieges: number, chevaux: number) {
    super(sieges);
    this.chevaux = chevaux;
  }

  faireLaCourse() {
    console.log(`${super.rouler()} Et je fais la course.`);
  }

}
const monBolide = new Sportive(4, 960) ;

console.log(monBolide);// {sieges: 4, chevaux: 960}
monBolide.faireLaCourse(); // Je roule. Et je fais la course.
```

## Les modificateurs public, private, protected et readonly

### Les propriétés *publiques* par défaut

**Par défaut, toutes les propriétés et méthodes déclarés sur une classe sont publiques.**

**Lorsqu'une propriété d'une classe est *public*, nous pouvons y accéder à l'extérieur de la classe.**

Si nous reprenons notre exemple précédent, il est donc strictement équivalent avec :

```ts
class Personne {
  public nom: string;

  constructor(nom: string) {
      this.nom = nom;
  }

  public sePresenter() {
      return `Bonjour, je m'appelle ${this.nom} !`;
  }
}
```

### Le modificateur *private*

Lorsqu'une propriété d'une classe est marquée comme *private*, il n'est pas possible d'y accéder à l'extérieur de sa classe.

```ts
class Personne {
  private nom: string;

  constructor(leNom: string) {
    this.nom = leNom;
  }

  private sePresenter() {
      return `Bonjour, je m'appelle ${this.nom} !`;
  }
}
const personne = new Personne("Jean");
personne.nom; // Erreur
personne.sePresenter(); // Erreur
```

Dans les deux cas *TypeScript* vous signalera que la propriété est privée, par exemple : 
>*Property 'sePresenter' is private and only accessible within class 'Personne'*.

### Le modificateur *protected*

**Lorsqu'une propriété d'une classe est marquée comme *protected* elle peut uniquement y être accédée depuis sa classe ou depuis les classes héritant de celle-ci.**

```ts
class Personne {
  protected nom: string;

  constructor(nom: string) {
    this.nom = nom;
  }
}

class Employe extends Personne {
  private service: string;

  constructor(nom: string, service: string) {
    super(nom); this.service = service;
  }

  public presentation() {
    return `Bonjour, je m’apelle ${this.nom} et je travaille au service ${this.service}.`;
  }
}

let jean = new Employe("Jean", "comptable");
console.log(jean.presentation()); // Bonjour, je m’apelle Jean et je travaille au service comptable.
```

### Le modificateur *readonly*

Certaines propriétés ne doivent être modifiables que lors de l'initialisation.

Vous pouvez spécifier qu'une propriété est en lecture seule en mettant *readonly* devant le nom de la propriété.

**Cette propriété ne pourra être modifiée que lors de sa déclaration ou son initialisation dans le constructeur de la classe.**

```ts
class Personne {
  readonly nom: string;

  constructor(nom: string) {
    this.nom = nom;
  }
}
```

**Le modificateur *readonly* peut être combiné avec les autres modificateurs.**

Par exemple :

```ts
class Personne {
  private readonly nom: string;

  constructor(leNom: string) {
    this.nom = leNom;
  }

  private renommer(nom:string) {
    this.nom = nom; // Erreur
  }
}
const personne = new Personne("Jean");
```

Ici *TypeScript* vous signalera : 
>*Cannot assign to 'nom' because it is a read-only property.*

## Propriétés statiques et notation raccourcie

### Notation raccourcie

Il existe une notation raccourcie avec *TypeScript* pour la déclaration de propriétés.

Le nom officiel est **propriétés paramètres** car ce sont des propriétés passées dans le constructeur de la classe.

**La notation raccourcie permet d'effectuer la déclaration et l'assignation des propriétés dans le contructeur de la classe, ce qui fait beaucoup moins de code :**

```ts
class Personne {
  constructor(public nom: string) {}
}
```

Équivaut strictement à :

```ts
class Personne {
  public nom: string;
  constructor(nom: string) {
    this.nom = nom;
  }
}
```

Notez que pour utiliser la notation raccourcie, ***il faut obligatoirement préciser un modificateur*** (*public*, *private*, *protected* ou *readonly*) **pour chaque propriété définie de cette manière.**

### Propriétés et méthodes statiques

Comme en *JavaScript*, il est possible d'utiliser des propriétés et des méthodes statiques, il n'y a aucune différence d'utilisation :

```ts
class Personne {
  constructor(public prenom: string, public nom: string) {}

  static decrire () {
    console.log('Personne est une classe pour créer des personnes.');
  }
}

const dylan = new Personne('Bob', 'Dylan');

Personne.decrire(); //  Personne est une classe pour créer des personnes.
dylan.decrire(); // Error : dylan.decrire is a static member of type 'Personne'
```

## Les classes abstraites

### Les classes abstraites

Les classes abstraites sont des classes qui servent de base à d'autres classes grâce à l'héritage.

**Elles ne sont pas instanciables.**

**Elles servent à définir la structure de classes sans les implémenter.**

```ts
abstract class Personne {
  constructor(public genre: string) {}
}

const personne = new Personne(); // Cannot create an instance of an abstract class

class Homme extends Personne {
  constructor() {
    super('homme');
  }
}

const homme = new Homme();
```

### Les méthodes abstraites

**Au sein d'une classe abstraite**, il est possible de définir des **méthodes abstraites**, ces méthodes servent à définir comment elles doivent être créées dans les méthodes des classes héritées :

```ts
abstract class Personne {
  constructor(public genre: string) {}

  abstract direBonjour(): void

  direAuRevoir() {
    console.log('Au revoir !')
  }
}


class Homme extends Personne {
  constructor() {
    super('homme');
  }

  direBonjour() {
    console.log('Bonjour !')
  }
}

const homme = new Homme();
homme.direAuRevoir();
homme.direBonjour();
```

Ici en déclarant une méthode abstraite dans la classe abstraite *Personne*, nous obligeons toutes les classes héritant de *Personne* à définir une méthode `direBonjour()` qui ne doit rien retourner.

### Typer les instances avec la classe comme type

En *TypeScript* il est possible de typer les instances d'une classe en utilisant le nom de la classe comme type.

Par exemple, ici, la variable ne peut recevoir qu'une instance de la classe *Personne* :

```ts
const user: Personne;
```