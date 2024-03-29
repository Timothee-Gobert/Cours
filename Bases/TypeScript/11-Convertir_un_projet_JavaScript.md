# Convertir un projet JavaScript

## Introduction et ajout de TypeScript au projet

### Code de départ

Nous allons convertir le projet todo en utilisant _TypeScript_. Vous pouvez retrouver le code de départ sur [Github](https://github.com/dymafr/javascript-chapitre13-todo).

### Ajout de _TypeScript_

Pour ajouter _TypeScript_ au projet, il suffit de l'installer et d'installer le _loader Webpack_ correspondant :

```sh
npm i typescript ts-loader
```

Nous allons ensuite initialiser le fichier de configuration pour le compilateur _TypeScript_ :

```sh
tsc --init
```

`tsc` est comme vous le savez la commande du compilateur _tsc_ de _TypeScript_.

L'option `--init` permet d'initialiser un fichier de configuration _tsconfig.json_ avec certaines options activées par défaut, et toutes les autres options commentées. Cela va nous être bien utile pour découvrir les options ensemble.

## Premières options de configuration de TypeScript

### Compilation du projet

Nous allons commencer par renommer notre fichier _src/index.js_ en _src/index.ts_ afin que nous puissions commencer à utiliser _TypeScript_.

#### Configuration de _Webpack_

Pour le moment nous avons une configuration basique pour le compilateur _TypeScript_, mais nous n'avons pas relié _TypeScript_ à _Webpack_.

En effet, pour le moment, _Webpack_ ne sait pas qu'il doit charger les fichiers _TypeScript_ avec l'extension _.ts_ et les compiler en fichiers _JavaScript_ en utilisant le compilateur _tsc_.

C'est ce que nous allons faire maintenant en modifiant le fichier de configuration de _Webpack_, _webpack.config.js_ :

```ts
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: {
    main: path.join(__dirname, "src/index.ts"),
  },
  output: {
    path: path.join(__dirname, "dist"),
    filename: "[name].bundle.js",
  },
  module: {
    rules: [
      {
        test: /.ts/,
        exclude: /(node_modules)/,
        use: ["ts-loader"],
      },
      {
        test: /.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, "./index.html"),
    }),
  ],
  stats: "minimal",
  devtool: "source-map",
  mode: "development",
  devServer: {
    open: false,
    static: path.resolve(__dirname, "./dist"),
    port: 4000,
  },
};
```

Nous utilisons simplement une règle pour charger les fichiers avec l'extension _.ts_ et nous les chargeons avec _ts-loader_ qui va s'occuper de leur compilation.

#### Configuration de _tsc_

Nous passons ensuite à la configuration nécessaire pour que le projet puisse être compilé :

```json
{
  "compilerOptions": {
    "incremental": true,
    "target": "ES5",
    "module": "ESNext",
    "esModuleInterop": true,
    "sourceMap": true,
    "removeComments": true,
    "lib": ["es6", "dom"],
    "rootDir": "src",
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "downlevelIteration": true
  }
}
```

Nous allons voir maintenant les options.

#### Corrections des premières erreurs de compilation

Dès la première compilation nous avons certaines erreurs qui apparaissent et nous allons voir pourquoi.

Première erreur :

> _Property 'value' does not exist on type 'Element'.ts._

Ici l'erreur est que la méthode native `document.querySelector()` retourne un _Element_ et que sur cet élément il n'y a pas forcément de propriété _value_.

Il faut indiquer à _TypeScript_ que le type de l'élément récupéré est un `HTMLInputElement` et qu'il a donc forcément une propriété _value_, c'est ce qu'on appelle restreindre le type :

```ts
const input: HTMLInputElement = document.querySelector("form > input");
```

Maintenant _TypeScript_ sait que notre méthode retournera tout le temps un élément _input_ et ne se plaint plus.

Deuxième erreur :

> _Argument of type '{ text: any; done: false; }' is not assignable to parameter of type '{ text: string; done: boolean; editMode: boolean; }'.
> Property 'editMode' is missing in type '{ text: any; done: false; }' but required in type '{ text: string; done: boolean; editMode: boolean; }'_

L'erreur est simple : le compilateur nous dit que l'argument de type `{ text: any; done: false; }` ne peut pas être assigné à `{ text: string; done: boolean; editMode: boolean; }` car la propriété _editMode_ est manquante.

La raison de cette erreur est que _TypeScript_ a inféré le type de nos _Todos_ lors de l'assignation :

```ts
const todos = [
  {
    text: "je suis une todo",
    done: false,
    editMode: true,
  },
  {
    text: "faire du JavaScript",
    done: true,
    editMode: false,
  },
];
```

Et qu'il voit qu'on essaye d'ajouter une _todo_ sans préciser la propriété _editMode_ ici :

```ts
const addTodo = (text) => {
  todos.push({
    text,
    done: false,
  });
  displayTodo();
};
```

Pour corriger l'erreur il suffit de faire :

```ts
const addTodo = (text) => {
  todos.push({
    text,
    done: false,
    editMode: false,
  });
  displayTodo();
};
```

### Les options du compilateur _TypeScript_

Nous allons voir une partie des options, les moins intéressantes ou celles que nous avons déjà vues maintenant, puis nous verrons les autres dans la leçon suivante.

Nous vous conseillons de les parcourir rapidement pour savoir qu'elles existent sans vous y attarder, vous en aurez rarement besoin.

#### L'option _incremental_

Cette option permet de spécifier à _TypeScript_ de conserver des informations sur le graphe des dépendances du projet lors de la dernière compilation.

_TypeScript_ utilisera des fichiers cachés pour accélérer les temps de compilation. C'est une option recommandée pour accélérer les compilations lors du développement.

#### L'option _target_

La _target_ est la version de _JavaScript_ ciblée après la compilation.

Si votre application doit être compatible tous navigateurs récents, on choisira _ES6_.

Si votre application est en _Node.js_ on choisira _ES2019_ (version minimale 12) ou _ES2020_.

Il vaut mieux choisir toujours la version la plus récente possible pour éviter du code supplémentaire lors de la compilation et donc des pertes de performance.

Bien sûr il faut prendre en compte votre cible, si votre code doit absolument supporter de très vieux navigateurs abandonnés (Internet Explorer par exemple) il faudra choisir ES5.

#### L'option _module_

Permet de définir le système de modules pour le projet : _CommonJS_ (pour un projet _Node.js_ par exemple), _UMD_, _AMD_, _System_, _ESNext_ ou _ES2020_.

Pour un nouveau projet on utilisera _ES2020_ sauf sur _Node.js_.

#### L'option _lib_

Cette option permet de spécifier les librairies de types natifs incluses par _TypeScript_.

Par défaut, les librairies incluses vont varier suivant la _target_ définie. Par exemple, si la _target_ est _ES5_ vous aurez d'inclus : _DOM_, _ES5_ et _ScriptHost_.

Si la _target_ est _ES6_ vous aurez : _DOM_, _ES6_, _DOM.Iterable_, _ScriptHost_.

Lorsque votre environnement est le _Web_ vous n'aurez probablement pas à toucher cette option.

Vous pouvez modifier cette option si vous travaillez par exemple avec _Node.js_ et que vous n'avez donc pas besoin d'inclure les définitions de type pour le _DOM_.

Vous pouvez également modifier cette option si vous souhaitez utiliser une _target_ plus basse tout en utilisant des _polyfills_ pour certaines fonctionnalités.

Par exemple : vous souhaitez avoir une _target ES5_ pour supporter tous les navigateurs, même les versions anciennes, et vous utilisez des promesses car vous avez un _polyfill ES5_ pour ces dernières.

Si vous mettez _target ES5_ et ne modifiez pas l'option _lib_ vous aurez des erreurs de compilation car les types des promesses ne seront pas inclus (car la cible est _ES5_ qui ne contient pas les promesses).

Il faut donc laissez _target_ à _ES5_ mais spécifiez dans _lib_ _ES6_ : vous n'aurez ainsi plus l'erreur de type.

#### L'option _allowJs_

Cette option permet de spécifier que des fichiers _JavaScript_, donc sans l'extension _.ts_, puissent être inclus dans votre projet. sans erreur.

#### L'option _checkJs_

Cette option s'utilise en combinaison avec _allowJs_ : si vous acceptez les fichiers _JavaScript_ en mettant `allowJs: true`, vous pouvez contrôler si _TypeScript_ va vérifier les types natifs et les types par inférence dans vos fichiers _JavaScript_ avec cette option.

Si vous mettez `checkJs: false`, _TypeScript_ ne fera aucune vérification de type, si vous mettez l'option à _true_, _TypeScript_ pourra émettre des erreurs de compilation.

#### L'option _declaration_

Cette option permet de générer automatiquement des fichiers de déclaration de type pour tous les fichiers _TypeScript_ et _JavaScript_ de votre projet.

C'est utile lorsque vous créez une librairie et souhaitez décrire les APIs externes de votre module. Grâce à cela, _TypeScript_ pourra proposer l'autocomplétion et la vérification des types.

Vous n'utiliserez donc pas la plupart du temps cette option.

#### L'option _sourceMap_

Cette option est très intéressante et nous recommandons de l'utiliser lors du développement.

Elle permet d'activer la génération de fichiers permettant aux débogueurs de faire le lien entre code source et code compilé.

Ainsi, lors d'une erreur, lorsque vous cliquerez sur la source de l'erreur, vous n'aurez pas le fichier compilé mais bien le fichier source. Ce qui vous permettra d'avoir le numéro de ligne correspondant à celui dans votre éditeur.

Cette option facilite donc grandement le débogage de votre code.

#### L'option _outFile_

Cette option permet de faire un _bundle_ d'un seul fichier.

Attention cette option n'est compatible qu'avec les types de module _None_, _System_ ou _AMD_. Dans les faits vous n'utiliserez probablement jamais cette option.

#### L'option _outDir_

Par défaut, les fichiers _JavaScript_ émis lors de la compilation sont placés au même niveau que les fichiers d'origine _TypeScript_, donc dans le même dossier.

L'option _outDir_ permet de spécifier un dossier pour placer les fichiers après la compilation :

```json
{
  "compilerOptions": {
    "outDir": "dist"
  }
}
```

Comme nous utilisons _Webpack_ cela n'a pas d'importance car c'est _Webpack_ qui va placer le _bundle_ dans le dossier spécifié dans sa configuration après la compilation.

#### L'option _rootDir_

Attention c'est option n'est pas du tout pour indiquer quels fichiers doivent être inclus dans la compilation !

Cette option permet de spécifier à partir de quel dossier l'architecture des dossiers et des fichiers sera définie.

Ainsi si nous avons cette architecture :

```sh
├── src
│   └── index.ts
├── fichier.ts
└── tsconfig.json
```

Et que nous avons cette configuration :

```json
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

Nous aurons une erreur car _outDir_ oblige tous les fichiers et dossiers à se trouver dans _dist_ après compilation.

Or, _rootDir_ indique ici que les chemins doivent être calculés à partir de _src/_ donc que _fichier.ts_ doit être un niveau au-dessus par rapport à _src_. Après compilation il doit donc être placé dans _../dist_.

Comme ces deux règles sont contradictoires, vous aurez une erreur.

Si vous ne voulez pas que le fichier _fichier.ts_ soit dans la compilation vous pourriez par exemple faire :

```json
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

Vous n'auriez ainsi plus d'erreur.

#### L'option _removeComments_

Cette option est très utile et permet de supprimer tous les commentaires de vos fichiers _TypeScript_ lors de leur compilation en fichiers _JavaScript_.

#### L'option _files_

Permet de spécifier un tableau de fichier _Typescript_ à inclure dans la compilation, par exemple :

```json
{
  "compilerOptions": {},
  "files": ["index.ts", "app.ts", "router.ts"]
}
```

Si l'un des fichiers n'est pas trouvé par le compilateur une erreur sera soulevée.

#### L'option _include_

Cette option permet de spécifier un tableau de fichiers ou de patterns à compiler, par exemple :

```json
{
  "include": ["src/**/*", "index.ts"]
}
```

Un _pattern_ est comme une expression régulière pour matcher les fichiers et / ou les dossiers respectant le chemin spécifié.

`**/` permet de matcher n'importe quel dossier de n'importe quel niveau. Cela revient à dire tous les dossiers et tous les sous-dossiers quel que soit leur niveau inclus dans _src_.

`*` permet de matcher n'importe quel caractère et un nombre indéfini de fois. Ici cela revient à matcher tous les fichiers.

#### L'option _exclude_

Permet de spécifier des fichiers ou des patterns qui seront exclus de la compilation mêmes s'ils matchent dans _include_.

En revanche si un fichier est déclaré dans _files_, il sera compilé même si il est exclus.

#### L'option _baseUrl_

Permet de spécifier le chemin de base à partir duquel sont résolus les chemins relatifs.

#### L'option _paths_

Permet de spécifier le chemin pour certains modules. TypeScript s'en servira pour résoudre les imports.

Il faut avoir défini l'option _baseUrl_ qui permettra de calculer les _paths_ à partir de ce chemin.

Par exemple, si nous avons :

```sh
├── shared
│   └── unModule
│       └── index.ts
├── src
│   └── index.ts
└── tsconfig.json
```

Et que nous configurons :

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "unModule": ["./shared/unModule"]
    }
  }
}
```

Nous pouvons ensuite directement faire ceci dans nos fichiers :

```ts
import { quelquechose } from "unModule";
```

_TypeScript_ saura où aller regarder pour trouver le module et vous n'avez pas besoin de donner le chemin entier.

Cela peut être utile dans certains cas avancés.

#### Quelques options supplémentaires moins utiles

- **L'option _noEmit_** est utilisée lorsque vous ne voulez pas que _TypeScript_ se charge de la compilation (cela peut être utile dans certains cas, lorsque vous avez plusieurs outils de _build_).
- **L'option _importHelpers_** permet d'importer les _polyfills_ depuis _tslib_ et non pas d'insérer directement le code dans les fichiers _JavaScript_. Cela peut éviter des duplications de code. Vous pouvez l'activer mais il faut avoir installé _tslib_ (`npm i tslib`).
- **L'option _downlevelIteration_** permet de mieux gérer les itérations récentes du langage (_spread_, _for … of_ etc) lorsque vous ciblez _ES5_ ou moins. C'est intéressant d'activer l'option si votre _target_ est _ES3_ ou _ES5_.
- **L'option _isolatedModules_** est utile lorsque vous n'utilisez pas _TypeScript_ pour la compilation et que vous souhaitez que _TypeScript_ émettre des avertissements si vous utilisez des fonctionnalités des modules qui ne sont pas compris par d'autres outils de _build_ (par exemple les _namespaces_).
- **Les options _types_ et _typeRoots_** permettent de restreindre les dépendances de types (précédées par _@types_) qui seront incluses lors de la compilation. C'est très rarement utile.
- **L'option _skipLibCheck_** permet d'empêcher le contrôle des types des librairies externes. Cette option est rarement utilisée.

## Le renforcement du contrôle des types

Il existe un ensemble d'options _TypeScript_ visant à assurer un meilleur contrôle des types de votre code.

Ces options ne sont pas activées pour des raisons de rétrocompatibilité : imaginez un immense projet qui se retrouverait à devoir corriger des centaines (voir milliers) d'erreurs de type avant de pouvoir compiler !

Pour cette raison, l'équipe _TypeScript_, tout en les recommandant, les a laissé à _false_ par défaut. Nous vous encourageons à toutes les mettre à _true_.

C'est plus exigeant, car vous aurez plus d'erreurs de types, et passerez plus de temps au début, mais votre code sera de meilleure qualité à la fin !

### L'option _strict_

C'est une "super-option" dans le sens où elle permet d'activer toutes les options de contrôle de type renforcé d'un coup.

Cette option est à privilégier car elle permet d'activer **les prochaines options de contrôle de type avancé automatiquement à chaque nouvelle version de _TypeScript_.**

Nous allons bien sûr voir en détails toutes les options qu'elle active. Vous pouvez également sélectionner manuellement les options.

#### L'option _alwaysStrict_

Cette option n'a rien à voir avec la précédente : elle permet de s'assurer que tous les fichiers _JavaScript_ utilisent bien le mode _JavaScript strict_.

#### L'option _noImplicitAny_

Lorsque _TypeScript_ ne parvient pas à inférer un type et qu'il n'est pas indiqué, il va par défaut mettre le type _any_.

Avec l'option, _TypeScript_ va vous donner une erreur à la place, c'est pour cette raison qu'elle s'appelle pas d'_any_ implicite (pour implicitement inféré).

#### L'option _strictNullChecks_

Cette option permet de s'assurer que toutes les variables sont testées avec _null_ et _undefined_ et permet de lever une erreur s'il n'y a pas de garantie que la valeur existe avant d'essayer de l'utiliser.

Cela vous force à soit mettre des gardes contre _null / undefined_, soit à mieux typer votre code, soit à assurer à _TypeScript_ manuellement que la valeur sera présente (avec _as_).

#### L'option _strictFunctionTypes_

Cette option permet d'effectuer un contrôle de type plus avancé sur les paramètres.

Par exemple sans l'option ce code compile alors qu'il peut résulter en des erreurs :

```ts
type Personne = (age: string | number) => string;
const personne: Personne = (age: string) => `Bonjour, j’ai ${age} ans`;
```

Avec l'option vous aurez l'erreur suivante :

> _Types of parameters 'age' and 'age' are incompatible. Type 'string | number' is not assignable to type 'string'._

_TypeScript_ détecte bien que le type `string | number` n'est pas la même chose que simplement _string_.

#### L'option _strictBindCallApply_

Cette option permet de contrôler correctement les valeurs passées aux méthodes _JavaScript_ `bind()`, `call()` et `apply()`.

#### L'option _strictPropertyInitialization_

Cette option permet de s'assurer que toutes les propriétés d'une classe sont bien initialisées dans le constructeur.

Par exemple, grâce à cette option le code suivant renverra une erreur :

```ts
class Personne {
  nom: string;
  age: string;
  constructor(nom: string, age: string) {
    this.nom = nom;
  }
}
```

_TypeScript_ renverra cette erreur :

> _Property 'age' has no initializer and is not definitely assigned in the constructor._

#### L'option _noImplicitThis_

Cette option permet de mieux détecter les erreurs de la valeur de _this_ dans de nombreuses situations.

### Les options de linting

Les _linters_ sont des outils permettant d'améliorer la qualité du code en effectuant des analyses statiques.

Toutes ces options sont recommandées et nous vous invitons à les utiliser.

#### L'option _noFallthroughCasesInSwitch_

Cette option va vérifier que tous les cas dans un _switch / case_ contiennent un _break_ ou un _return_ pour vous assurer que plusieurs cas, ou un cas et le _default_, ne soient pas exécutés pour une valeur donnée.

#### L'option _noImplicitReturns_

Cette option permet de s'assurer que toutes les branches de vos fonctions retournent une valeur.

Exemple :

```ts
const personne = (nom: string) => {
  if (nom) {
    return `Coucou, ${nom}`;
  }
};
```

Ici la fonction ne retourne implicitement rien dans le cas où nom est _falsy_, le compilateur renverra donc une erreur si l'option est activée.

#### L'option _noUnusedLocals_

Cette option permet de vous avertir si des variables locales sont déclarées et non utilisées.

#### L'option _noUnusedParameters_

Cette option permet de vous avertir si des paramètres sont déclarés et non utilisés.

## Utilisation de TypeScript dans le projet

### Modification de l'architecture

Nous remontons l'_index.html_ d'un niveau et nous créons un dossier _style_ dans _src_ où nous plaçons _style.css_.

Nous modifions en conséquence le chemin d'import dans _index.ts_ :

```ts
import "./style/style.css";
```

Nous modifions également le chemin dans _webpack.config.json_ :

```ts
...
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, './index.html'),
    }),
  ],
…
```

### Typage d'_index.ts_

Nous allons maintenant expliquer les types lignes par ligne.

#### Les éléments du _DOM_

Commençons par les références aux éléments du _DOM_ :

```ts
const ul: HTMLUListElement = document.querySelector("ul")!;
const form: HTMLFormElement = document.querySelector("form")!;
const input: HTMLInputElement =
  document.querySelector<HTMLInputElement>("form > input")!;
```

Ici, nous n'aurions pas besoin de typer `ul: HTMLUListElement`, `form: HTMLFormElement` et `input: HTMLInputElement`.

En effet, _TypeScript_ a par défaut les types du _DOM_ comme nous l'avons vu et il sait infère le type automatiquement : il sait que `document.querySelector('ul')` retourner `HTMLUListElement | null`, et même chose pour les autres.

Vous bénéficiez donc de l'autocomplétion et du contrôle des types sans avoir rien à faire vous même.

Nous vous conseillons cependant, lorsque vous débutez de tout typer : il vaut mieux trop typer que pas assez.

D'une part, car cela n'a aucun impact sur les performances vu que les types disparaissent lors de la compilation, et d'autre part car lorsqu'on débute il vaut mieux maximiser la lisibilité plutôt que la concision. Lorsque vous aurez plusieurs mois d'expériences avec _TypeScript_, vous ne typerez plus les éléments natifs.

Ensuite, pourquoi ajoutons-nous des points d'exclamations ?

En _TypeScript_,** un point d'exclamation est l'opérateur d'assertion non nul**. Cela indique à _TypeScript_ qu'une valeur ne peut pas être nulle et qu'il n'a pas à contrôler que la valeur est possiblement _null_ ou _undefined_.

Dans notre cas, nous sommes certains que les éléments sont présents car ils ne sont pas ajoutés dynamiquement. Nous pouvons donc le signaler à _TypeScript_.

Enfin, `document.querySelector<HTMLInputElement>` permet de _caster_ l'élément comme étant un `HTMLInputElement` car _TypeScript_ ne peut pas inférer le type de l'élément tout seul. La raison est simple : nous n'utilisons pas un élément _HTML_ directement mais un sélecteur _CSS_ plus complexe avec une imbrication. _TypeScript_ peut donc simplement inférer qu'il s'agit d'un _Element_. Or, nous savons qu'il s'agit forcément d'un `HTMLInputElement` et nous avons besoin de le dire à _TypeScript_ pour qu'il nous laisse accéder à la propriété _value_ de l'élément.

#### L'écouteur et le gestionnaire d'événement du formulaire

Première méthode que nous typons :

```ts
form.addEventListener("submit", (event: Event): void => {
  event.preventDefault();
  const value = input.value;
  input.value = "";
  addTodo(value);
});
```

Ici nous avons simplement à typer l'événement reçu en argument comme étant un _Event_, type natif des événements du _DOM_. Nous typons également le retour de la fonction comme étant _void_ car le gestionnaire d'événement ne retourne pas de valeur.

#### Interface pour nos todos

Nous allons créer une interface pour nos todos. Nous allons donc créer un dossier _src/interfaces_ dans lequel nous allons créer un fichier _todo.interface.ts_ :

```ts
export interface Todo {
  text: string;
  done: boolean;
  editMode: boolean;
}
```

L'_interface_ est simple : nous avons trois propriétés.

Il faut bien penser à l'exporter pour pouvoir l'utiliser dans notre _index.ts_.

#### Utilisation de l'interface

De retour dans _index.ts_ nous importons et utilisons l'_interface_ :

```ts
const todos: Todo[] = [
  {
    text: "je suis une todo",
    done: false,
    editMode: true,
  },
  {
    text: "faire du JavaScript",
    done: true,
    editMode: false,
  },
];
```

Nous continuous avec la méthode pour afficher les todos :

```ts
const displayTodo = () => {
  const todosNode: HTMLLIElement[] = todos.map((todo: Todo, index: number) => {
    if (todo.editMode) {
      return createTodoEditElement(todo, index);
    } else {
      return createTodoElement(todo, index);
    }
  });
  ul.innerHTML = "";
  ul.append(...todosNode);
};
```

Nous typons les arguments reçu par la méthode `map()` lors de l'itération de chaque élément de notre liste de _todo_.

#### Les types pour la méthode de création de nos todos

Nous passons ensuite à la méthode `createTodoElement` :

```ts
const createTodoElement = (todo: Todo, index: number): HTMLLIElement => {
  const li: HTMLLIElement = document.createElement("li");
  const buttonDelete: HTMLButtonElement = document.createElement("button");
  buttonDelete.innerHTML = "Supprimer";
  const buttonEdit: HTMLButtonElement = document.createElement("button");
  buttonEdit.innerHTML = "Edit";
  buttonDelete.addEventListener("click", (event: MouseEvent) => {
    event.stopPropagation();
    deleteTodo(index);
  });
  buttonEdit.addEventListener("click", (event: MouseEvent) => {
    event.stopPropagation();
    toggleEditMode(index);
  });
  li.innerHTML = `
    <span class="todo ${todo.done ? "done" : ""}"></span>
    <p>${todo.text}</p>
  `;
  li.addEventListener("click", () => {
    toggleTodo(index);
  });
  li.append(buttonEdit, buttonDelete);
  return li;
};
```

Nous typons toutes les références à des éléments du _DOM_, tous les paramètres et toutes les valeurs de retour.

Pour les événements de type _click_, nous utilisons l'interface native `MouseEvent` qui est plus spécifique qu'`Event` et permet d'avoir les bonnes propriétés sur l'événement.

#### Typage des autres méthodes

Nous continuons avec les autres méthodes :

```ts
const addTodo = (text: string): void => {
  todos.push({
    text,
    done: false,
    editMode: false,
  });
  displayTodo();
};

const deleteTodo = (index: number): void => {
  todos.splice(index, 1);
  displayTodo();
};

const toggleTodo = (index: number): void => {
  todos[index].done = !todos[index].done;
  displayTodo();
};

const toggleEditMode = (index: number): void => {
  todos[index].editMode = !todos[index].editMode;
  displayTodo();
};

const editTodo = (index: number, input: HTMLInputElement): void => {
  const value = input.value;
  todos[index].text = value;
  todos[index].editMode = false;
  displayTodo();
};
```

Les types sont très simples : il nous suffit de typer les arguments, et comme toutes ces méthodes ne retournent pas de valeur nous typons leur valeur de retour à _void_.

### Code final

Vous pouvez retrouver le code final sur [Github](https://github.com/dymafr/typescript-chapitre11-todo).

## Déboguer un projet TypeScript

### Activer l'option _sourceMap_

Utiliser le débogueur de _VS Code_ pour un projet _TypeScript_ est très similaire à un projet _JavaScript_.

Il faut cependant ne pas oublier de **passer l'option _sourceMap_ à _true_** dans le fichier _tsconfig.json_ pour que cela fonctionne.

Une fois que c'est fait, les étapes sont les mêmes que pour un projet _JavaScript_, nous allons les remettre ici.

### Ouvrir l'onglet de _debug_ sur _VS Code_

Pour ouvrir l'onglet de débug sur _VS Code_ :

Sur Mac, faites _Cmd + Maj + d_.

Sur Linux ou Windows, faites _Ctrl + Maj + d_.

### Installer l'extension _Debugger for Chrome_

Nous allons télécharger une extension qui va nous permettre de faire exactement tout ce que nous avons vu dans la leçon précédente mais directement dans _VS Code_, sans avoir besoin d'aller dans Chrome.

Ouvrez l'onglet extension de _VS Code_ et installez l'extension _Debugger for Chrome_.

### Créer une configuration

Dans le menu cliquez sur la roue _Configure launch.json_ :

Rentrez ensuite :

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome against localhost",
      "url": "http://localhost:4000",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```

L'_URL_ permet de préciser le port où est servi l'application par le serveur de développement _Node.js_ de _Webpack_.

Dans notre cas, le port est _4000_.

Commencer par lancer _Webpack dev server_ avec `npm start`.

Vous pouvez ensuite lancer le _débugger_ soit dans l'onglet _Debug_ en cliquant en haut à gauche sur le bouton play vert.

Soit avec le raccourci en bas qui commence par un petit bouton play suivi de _Launch Chrome…_ qui correspond à la propriété _name_ de la configuration.

Cela lance _Chrome_ sur le port _4000_.

Cela fait également apparaître la barre de _debug_ :

Le premier bouton permet de continuer au prochain _breakpoint_.

Le second permet de _step over_ l'exécution d'une fonction, en exécutant toutes ses instructions et en passant à la prochaine fonction.

Le troisième, _step in_ permet de rentrer dans le corps d'une fonction et d'exécuter instruction par instruction dans celle-ci.

Le quatrième _step out_ permet de ressortir du corps d'une fonction en exécutant les instructions restantes.

Le cinquième permet de relancer le navigateur.

Le dernier permet de stopper le débogage.

### Utiliser des _breakpoints_ ou des _breakpoints_ conditionnels

**Pour ajouter un _breakpoint_ classique il faut cliquer à gauche du numéro de ligne.**

Un point rouge marquant le _breakpoint_ apparaîtra alors.

**Pour ajouter un _breakpoint_ conditionnel il faut effectuer un clic droit, à gauche du numéro de ligne.**

Vous pourrez voir également les _breakpoints_ à droite dans la navigation par défilement rapide.

### L'inspection des variables et l'évaluation d'expression

Vous retrouvez exactement les mêmes possibilités que sur _DevTools_.

En mode débogage vous pouvez retrouver **une partie _VARIABLES_ qui contient les variables _locales_, _globales_ et les _closures_**.

Vous retrouver également une partie _WATCH_ qui permet de suivre l'évolution d'évaluations.
