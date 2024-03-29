# Types natifs, mappés et conditionnels

## Les types natifs

Lorsque vous installez *TypeScript* ou lorsque vous utilisez par exemple *Visual Studio Code* (qui est codé en *TypeScript* et comporte plusieurs fichiers de types préinstallés), vous avez l'autocomplétion pour de très nombreuses librairies natives sans rien avoir à installer.

### Les types natifs pour les versions de *JavaScript*

Pour *JavaScript*, vous avez un fichier de type pour chaque version à partir d'*ES5*.

A savoir, un fichier *lib.es5.d.ts* qui comporte l'intégralité des types pour les spécifications officielles de cette version de *JavaScript*.

Vous ensuite un fichier pour chaque version annuelle : par exemple, *lib.es2015.d.ts* (qui correspond à *ES6*).

Dans chacun de ces fichiers, vous avez ensuite les références vers tous les fichiers de types de chaque nouvelle fonctionnalité, par exemple :

```ts
/// <reference lib="es5" />
/// <reference lib="es2015.core" />
/// <reference lib="es2015.collection" />
/// <reference lib="es2015.iterable" />
/// <reference lib="es2015.generator" />
/// <reference lib="es2015.promise" />
/// <reference lib="es2015.proxy" />
/// <reference lib="es2015.reflect" />
/// <reference lib="es2015.symbol" />
/// <reference lib="es2015.symbol.wellknown" />
```

Ainsi par exemple pour les promesses, leurs types sont situés dans *es2015.promise.d.ts* :

```ts
interface PromiseConstructor {
    /**
     * A reference to the prototype.
     */
    readonly prototype: Promise<any>;

    /**
     * Creates a new Promise.
     * @param executor A callback used to initialize the promise. This callback is passed two arguments:
     * a resolve callback used to resolve the promise with a value or the result of another promise,
     * and a reject callback used to reject the promise with a provided reason or error.
     */
    new <T>(executor: (resolve: (value?: T | PromiseLike<T>) => void, reject: (reason?: any) => void) => void): Promise<T>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3, T4, T5, T6, T7, T8, T9, T10>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>, T4 | PromiseLike<T4>, T5 | PromiseLike<T5>, T6 | PromiseLike<T6>, T7 | PromiseLike<T7>, T8 | PromiseLike<T8>, T9 | PromiseLike<T9>, T10 | PromiseLike<T10>]): Promise<[T1, T2, T3, T4, T5, T6, T7, T8, T9, T10]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3, T4, T5, T6, T7, T8, T9>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>, T4 | PromiseLike<T4>, T5 | PromiseLike<T5>, T6 | PromiseLike<T6>, T7 | PromiseLike<T7>, T8 | PromiseLike<T8>, T9 | PromiseLike<T9>]): Promise<[T1, T2, T3, T4, T5, T6, T7, T8, T9]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3, T4, T5, T6, T7, T8>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>, T4 | PromiseLike<T4>, T5 | PromiseLike<T5>, T6 | PromiseLike<T6>, T7 | PromiseLike<T7>, T8 | PromiseLike<T8>]): Promise<[T1, T2, T3, T4, T5, T6, T7, T8]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3, T4, T5, T6, T7>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>, T4 | PromiseLike<T4>, T5 | PromiseLike<T5>, T6 | PromiseLike<T6>, T7 | PromiseLike<T7>]): Promise<[T1, T2, T3, T4, T5, T6, T7]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3, T4, T5, T6>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>, T4 | PromiseLike<T4>, T5 | PromiseLike<T5>, T6 | PromiseLike<T6>]): Promise<[T1, T2, T3, T4, T5, T6]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3, T4, T5>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>, T4 | PromiseLike<T4>, T5 | PromiseLike<T5>]): Promise<[T1, T2, T3, T4, T5]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3, T4>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>, T4 | PromiseLike<T4>]): Promise<[T1, T2, T3, T4]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2, T3>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>, T3 | PromiseLike<T3>]): Promise<[T1, T2, T3]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T1, T2>(values: readonly [T1 | PromiseLike<T1>, T2 | PromiseLike<T2>]): Promise<[T1, T2]>;

    /**
     * Creates a Promise that is resolved with an array of results when all of the provided Promises
     * resolve, or rejected when any Promise is rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    all<T>(values: readonly (T | PromiseLike<T>)[]): Promise<T[]>;

    // see: lib.es2015.iterable.d.ts
    // all<T>(values: Iterable<T | PromiseLike<T>>): Promise<T[]>;

    /**
     * Creates a Promise that is resolved or rejected when any of the provided Promises are resolved
     * or rejected.
     * @param values An array of Promises.
     * @returns A new Promise.
     */
    race<T>(values: readonly T[]): Promise<T extends PromiseLike<infer U> ? U : T>;

    // see: lib.es2015.iterable.d.ts
    // race<T>(values: Iterable<T>): Promise<T extends PromiseLike<infer U> ? U : T>;

    /**
     * Creates a new rejected promise for the provided reason.
     * @param reason The reason the promise was rejected.
     * @returns A new rejected Promise.
     */
    reject<T = never>(reason?: any): Promise<T>;

    /**
     * Creates a new resolved promise for the provided value.
     * @param value A promise.
     * @returns A promise whose internal state matches the provided promise.
     */
    resolve<T>(value: T | PromiseLike<T>): Promise<T>;

    /**
     * Creates a new resolved promise .
     * @returns A resolved promise.
     */
    resolve(): Promise<void>;
}

declare var Promise: PromiseConstructor;
```

Vous retrouvez des fonctionnalités de *TypeScript* que nous avons vues : surcharge pour la méthode `Promise.all()`, types génériques, modificateurs (*readonly*) etc.

Lisez le attentivement c'est un bon exercice.

Vous comprenez maintenant comment vous avez l'autocomplétion nativement sur *VS Code* ! C'est parce que tous les fichiers pour les types natifs sont préinstallés.

### Les types natifs pour le *DOM*

En plus des versions de *JavaScript*, vous avez bien sûr l'*API* du *DOM* qui est entièrement typée officiellement (heureusement car on parle de 20500 lignes de types).

Vous pouvez vous amuser à le parcourir dans *VS Code*.

Par exemple, voici les types pour l'élément *input* :

```ts
interface HTMLInputElement extends HTMLElement {
    /**
     * Sets or retrieves a comma-separated list of content types.
     */
    accept: string;
    /**
     * Sets or retrieves how the object is aligned with adjacent text.
     */
    /** @deprecated */
    align: string;
    /**
     * Sets or retrieves a text alternative to the graphic.
     */
    alt: string;
    /**
     * Specifies whether autocomplete is applied to an editable text field.
     */
    autocomplete: string;
    /**
     * Sets or retrieves the state of the check box or radio button.
     */
    checked: boolean;
    /**
     * Sets or retrieves the state of the check box or radio button.
     */
    defaultChecked: boolean;
    /**
     * Sets or retrieves the initial contents of the object.
     */
    defaultValue: string;
    dirName: string;
    disabled: boolean;
    /**
     * Returns a FileList object on a file type input object.
     */
    files: FileList | null;
    /**
     * Retrieves a reference to the form that the object is embedded in.
     */
    readonly form: HTMLFormElement | null;
    /**
     * Overrides the action attribute (where the data on a form is sent) on the parent form element.
     */
    formAction: string;
    /**
     * Used to override the encoding (formEnctype attribute) specified on the form element.
     */
    formEnctype: string;
    /**
     * Overrides the submit method attribute previously specified on a form element.
     */
    formMethod: string;
    /**
     * Overrides any validation or required attributes on a form or form elements to allow it to be submitted without validation. This can be used to create a "save draft"-type submit option.
     */
    formNoValidate: boolean;
    /**
     * Overrides the target attribute on a form element.
     */
    formTarget: string;
    /**
     * Sets or retrieves the height of the object.
     */
    height: number;
    indeterminate: boolean;
    readonly labels: NodeListOf<HTMLLabelElement> | null;
    /**
     * Specifies the ID of a pre-defined datalist of options for an input element.
     */
    readonly list: HTMLElement | null;
    /**
     * Defines the maximum acceptable value for an input element with type="number".When used with the min and step attributes, lets you control the range and increment (such as only even numbers) that the user can enter into an input field.
     */
    max: string;
    /**
     * Sets or retrieves the maximum number of characters that the user can enter in a text control.
     */
    maxLength: number;
    /**
     * Defines the minimum acceptable value for an input element with type="number". When used with the max and step attributes, lets you control the range and increment (such as even numbers only) that the user can enter into an input field.
     */
    min: string;
    minLength: number;
    /**
     * Sets or retrieves the Boolean value indicating whether multiple items can be selected from a list.
     */
    multiple: boolean;
    /**
     * Sets or retrieves the name of the object.
     */
    name: string;
    /**
     * Gets or sets a string containing a regular expression that the user's input must match.
     */
    pattern: string;
    /**
     * Gets or sets a text string that is displayed in an input field as a hint or prompt to users as the format or type of information they need to enter.The text appears in an input field until the user puts focus on the field.
     */
    placeholder: string;
    readOnly: boolean;
    /**
     * When present, marks an element that can't be submitted without a value.
     */
    required: boolean;
    selectionDirection: "forward" | "backward" | "none" | null;
    /**
     * Gets or sets the end position or offset of a text selection.
     */
    selectionEnd: number | null;
    /**
     * Gets or sets the starting position or offset of a text selection.
     */
    selectionStart: number | null;
    size: number;
    /**
     * The address or URL of the a media resource that is to be considered.
     */
    src: string;
    /**
     * Defines an increment or jump between values that you want to allow the user to enter. When used with the max and min attributes, lets you control the range and increment (for example, allow only even numbers) that the user can enter into an input field.
     */
    step: string;
    /**
     * Returns the content type of the object.
     */
    type: string;
    /**
     * Sets or retrieves the URL, often with a bookmark extension (#name), to use as a client-side image map.
     */
    /** @deprecated */
    useMap: string;
    /**
     * Returns the error message that would be displayed if the user submits the form, or an empty string if no error message. It also triggers the standard error message, such as "this is a required field". The result is that the user sees validation messages without actually submitting.
     */
    readonly validationMessage: string;
    /**
     * Returns a  ValidityState object that represents the validity states of an element.
     */
    readonly validity: ValidityState;
    /**
     * Returns the value of the data at the cursor's current position.
     */
    value: string;
    /**
     * Returns a Date object representing the form control's value, if applicable; otherwise, returns null. Can be set, to change the value. Throws an "InvalidStateError" DOMException if the control isn't date- or time-based.
     */
    valueAsDate: Date | null;
    /**
     * Returns the input field value as a number.
     */
    valueAsNumber: number;
    /**
     * Sets or retrieves the width of the object.
     */
    width: number;
    /**
     * Returns whether an element will successfully validate based on forms validation rules and constraints.
     */
    readonly willValidate: boolean;
    /**
     * Returns whether a form will validate when it is submitted, without having to submit it.
     */
    checkValidity(): boolean;
    reportValidity(): boolean;
    /**
     * Makes the selection equal to the current object.
     */
    select(): void;
    /**
     * Sets a custom error message that is displayed when a form is submitted.
     * @param error Sets a custom error message that is displayed when a form is submitted.
     */
    setCustomValidity(error: string): void;
    setRangeText(replacement: string): void;
    setRangeText(replacement: string, start: number, end: number, selectionMode?: SelectionMode): void;
    /**
     * Sets the start and end positions of a selection in a text field.
     * @param start The offset into the text field for the start of the selection.
     * @param end The offset into the text field for the end of the selection.
     * @param direction The direction in which the selection is performed.
     */
    setSelectionRange(start: number, end: number, direction?: "forward" | "backward" | "none"): void;
    /**
     * Decrements a range input control's value by the value given by the Step attribute. If the optional parameter is used, it will decrement the input control's step value multiplied by the parameter's value.
     * @param n Value to decrement the value by.
     */
    stepDown(n?: number): void;
    /**
     * Increments a range input control's value by the value given by the Step attribute. If the optional parameter is used, will increment the input control's value by that value.
     * @param n Value to increment the value by.
     */
    stepUp(n?: number): void;
    addEventListener<K extends keyof HTMLElementEventMap>(type: K, listener: (this: HTMLInputElement, ev: HTMLElementEventMap[K]) => any, options?: boolean | AddEventListenerOptions): void;
    addEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | AddEventListenerOptions): void;
    removeEventListener<K extends keyof HTMLElementEventMap>(type: K, listener: (this: HTMLInputElement, ev: HTMLElementEventMap[K]) => any, options?: boolean | EventListenerOptions): void;
    removeEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | EventListenerOptions): void;
}
```

Vous retrouverez également d'autres *API* natives typées dans des fichiers dédiés, comme par exemple l'*API* des *WebWorkers*.

## Les types mappés

### Détails sur le fonctionnement

Les types mappés (*mapped types*) permettent de créer des types à partir d'autres types.

Prenons un exemple très simple pour bien comprendre les types mappés :

```ts
type Props = 'a' | 'b';

type Test = { [K in Props]: boolean };
```

Nous avons un *alias Props* qui contient une union de deux types littéraux.

Nous avons un second *alias Test* qui contient un objet.

Nous utilisons un type indexable [prop: type] pour indiquer un nombre indéfini de propriétés sur l'objet.

Ce type indexable est particulier car nous utilisons une **variable de type générique, appelée par convention `K` (pour *key*, à savoir clé d'une propriété), et le mot clé `in` suivi de l'*alias Props*.**

Cela permet de restreindre les propriétés possibles : elles ne peuvent être contenues que dans *Props*, et dans notre exemple cela signifie que les seules propriétés possibles sont *'a'* ou *'b'*.

Retenez que c'est proche du fonctionnement de *for...in* en *JavaScript*. Cela permet en fait de ne pas avoir à écrire :

```ts
type Flags = {
  a: boolean;
  b: boolean;
}
```

C'est très utile lorsque vous avez de nombreuses propriétés !

A noter que *Props* est simplement un *alias* comme vous le savez.

Vous pouvez donc directement faire :

```ts
type Test = { [K in 'a' | 'b']: boolean };
```

Mais il vaut mieux utiliser un *alias* pour plus de lisibilité à partir de quelques types littéraux.

### Le mot clé *keyof*

Souvent les types mappés utilisent également le mot clé *keyof* que nous avons vu.

Ce mot clé, propre à *TypeScript*, permet simplement d'obtenir les noms de propriétés d'un argument de type passé en paramètre.

### Exemples d'utilisation

Nous avons déjà vu plusieurs exemples dans le chapitre sur les types génériques.

Nous allons en rappeler certains comme par exemple, pour rendre les champs d'une *interface* optionnels ou obligatoires.

#### Le type `Partial<T>`

Le type global générique `Partial<T>` permet de rendre tous les champs optionnels, il utilise en fait un type mappé :

```ts
type Partial<T> = { [P in keyof T]?: T[P] | undefined; }
```

`P in keyof T` signifie que le type générique `P` est une propriété contenue (`in`) dans les propriétés de `T` (`keyof T`).

Ensuite, nous utilisons `?` pour rendre cette propriété `P` optionnelle.

Enfin, nous copions le type avec les modificateurs éventuels (par exemple *readonly*) en faisant `T[P]`. Cela permet de récupérer le type de la propriété `P` sur `T`.

Exemple :

```ts
interface Test {
  readonly name: string;
}

let test: Partial<Test> = {name: 'Paul'}
test.name = 'Jean-Louis'; // Erreur
```

Ici nous avons bien copié le modificateur *readonly* grâce à la notation `T[P]` utilisé par le type global `Partial<T>`.

#### Le type `Readonly<T>`

**Le type global *Readonly* permet de passer toutes les propriétés d'un type passé en argument en *readonly*.**

Il utilise également un type mappé :

```ts
type Readonly<T> = { readonly [P in keyof T]: T[P]; }
```

### Utilisations avancées

Vous avez une *interface* complexe et souhaitez rendre ses propriétés optionnelles et ajouter par exemple deux autres propriétés obligatoires. Cela peut être utile lors de la préparation de données avant une insertion en base de données par exemple.

```ts
interface Todo {
  id: number;
  texte: string;
  auteur: string;
}

type TempData<T> = {
  [P in keyof T]?: T[P];
} & { isValid: boolean }

const tempTodo: TempData<Todo> = {
  isValid: true,
  texte: 'En cours...'
}
```

Ce qui est intéressant ici est l'utilisation d'une intersection `&` pour ajouter une propriété obligatoire, après avoir passé toutes les propriétés en optionnel du type générique `T` passé en argument.

Nous pourrions même rendre une seule ou quelques propriétés obligatoires, cela peut être intéressant pour de grandes *interfaces*. Imaginons que vous ayez une *interface User* avec une trentaine de propriétés. Vous pourriez ainsi faire :

```ts
interface User {
  id: number;
  prop1: string;
  prop2: string;
  prop20: boolean;
}

type TempData<T> = {
  [P in keyof T]?: T[P];
} & { prop20: boolean; prop1: string; }

const tempTodo: TempData<User> = {
  prop20: true,
  prop1: 'Texte'
}
```

Ici nous rendons toutes les propriétés optionnelles de l'*interface User*, sauf deux que nous rendons obligatoires grâce à l'intersection.

## Les types conditionnels

**Les types conditionnels sont un type particulier de types mappés permettant de sélectionner entre deux types suivant une condition.**

Par exemple :

```ts
T extends U ? A : B
```

Ici si le type générique passé en argument `T` étend le type générique `U` alors il se verra attribuer le type `A` et sinon le type `B`.

Prenons l'exemple de la vidéo :

```ts
declare function maFonction<T>(x: T): T extends string ? string : number;

maFonction('12');
```

L'utilisation de *declare* permet ici de déclarer la signature de la fonction sans avoir à l'implémenter immédiatement.

Ici, nous spécifions que l'argument passé est de type `T`.

Nous testons ensuite le type pour savoir s'il étend *string*, autrement dit cela signifie que le type est une chaîne de caractères. Si c'est le cas, nous typons le retour de la fonction comme étant une chaîne de caractères. Dans le cas contraire, nous typons le retour comme étant un nombre

Ainsi, si on passe une chaîne de caractères : `maFonction('12');`, *TypeScript* sait que la valeur de retour de la fonction est une chaîne de caractères.

Si l'argument passé n'est pas une chaîne de caractères, par exemple `maFonction(true);`, alors la valeur de retour de la fonction est un nombre.

### Les types conditionnels distribués

**Les types conditionnels distribués sont des types conditionnels auxquels nous passons un union de types.**

Dans ce cas la condition est testée pour chaque type de l'union et c'est pour cette raison que nous les appelons types conditionnels distribués.

Nous allons commencer par revoir deux exemples de types globaux utilisant les types conditionnels distribués, puis nous verrons un autre exemple.

#### Rappel de l'exemple avec le type `Exclude<T,U>`

`Exclude<T, U>` permet de construire un nouveau type à partir d'un type T en excluant toutes les propriétés de `U`.

Exemple :

```ts
type Exclure = Exclude<1 | 2 | 3, 1 | 2>; //  3
```

`Exclude<T, U>` est un alias pour :

```ts
type Exclude<T, U> = T extends U ? never : T
```

Pour chaque propriété de `T` on vérifie si elle déjà présente dans `U`, si oui on retourne *never*, c'est-à-dire on la supprime, et si elle n'est pas présente on la garde en la retournant.

En détails cela donne :

```ts
1 extends 1 | 2 ? never : 1
2 extends 1 | 2 ? never : 2
3 extends 1 | 2 ? never : 3
```

Dans les deux premiers le type littéral est inclus dans l'union `U` donc on retourne *never*.

Dans le dernier cas, le type littéral 3 n'est pas inclus dans l'union de types littéraux et donc nous retournons le type littéral 3.

#### Rappel de l'exemple avec le type global `Extract<T, U>`

Pour rappel, le type `Extract<T,U>` permet de construire un nouveau type en extrayant d'un argument de type `T` toutes les propriétés qui se trouvent dans `U`.

Exemple :

```ts
type Extraites = Extract<1 | 2 | 3, 1 | 2>; //  1 | 2
```

`Extract<T, U>` est un alias pour :

```ts
type Extract<T, U> = T extends U ? T : never
```

Même chose que pour *Exclude*, cet alias utilise un type conditionnel distribué.

Mais ici c'est exactement l'opposé, si le type est inclus dans l'union `U` on le garde, sinon on l'écarte.

Dans le détails cela donne :

```ts
1 extends 1 | 2 ? 1 : never
2 extends 1 | 2 ? 2 : never
3 extends 1 | 2 ? 3 : never
```

Au final on obtient donc un nouveau type qui est l'*union* des types littéraux 1 et 2 : `1 | 2`.

#### Exemple de type avancé : obtenir les noms de propriétés d'un type qui ne sont pas des fonctions

En combinant les types mappés et les types conditionnels il est possible de construire des types complexes à partir d'autres types.

Par exemple, il est possible de créer un type utilitaire qui ne va récupérer que **les noms de propriétés d'un type qui ne sont pas des fonctions.**

```ts
interface User {
  id: number;
  name: string;
  adresses: string[];
  updateName(newName: string): void;
}

type ProprietesPasFonctions<T> = { [K in keyof T]: T[K] extends Function ? never : K }[keyof T];

type proprietes = ProprietesPasFonctions<User>;
```

Le type *proprietes* contient l'*union* de types littéraux : `"id" | "name" | "adresses"`.

Expliquons pas à pas ce type avancé.

Premièrement, il faut savoir qu'il est possible d'accéder à la valeur d'une propriété d'un objet de types, comme avec un objet classique. A savoir :

```ts
type MonType = { prop1: number }['prop1'];
```

Ce qui revient à assigner à `MonType` le type `number : type MonType = number`.

Il est possible, contrairement aux objets aux classiques, d'obtenir les types de plusieurs propriétés en passant une *union* de propriétés :

```ts
type MonType = { prop1: number, prop2: string, prop3: boolean }['prop1' | 'prop2' | 'prop3'];
```

Cela revient à assigner à `MonType` l'union de types littéraux suivante :

```ts
type MonType = string | number | boolean;
```

Il est possible d'utiliser cette notation avec *keyof* que nous avons déjà vu, nous pouvons ainsi faire directement ceci si nous voulons tous les types de toutes les propriétés :

```ts
type UnType = { prop1: number, prop2: string, prop3: boolean };
type MonType = UnType[keyof UnType];
```

Il faut passer par un type intermédiaire car on ne pas accéder récursivement au type pendant l'assignation.

`MonType` contient comme précédemment `string | number | boolean`.

**Nous pouvons maintenant comprendre `{ [K in keyof T]: K }` :**

```ts
type UnType = { prop1: number, prop2: string, prop3: boolean };
type MonType = { [K in keyof UnType]: K };
```

Ici l'alias `MonType` contient `{ prop1: "prop1"; prop2: "prop2"; prop3: "prop3"; }`.

`[K in keyof T]` permet d'accéder à chaque propriété `K` de `T` l'une après l'autre et : `K` permet de lui assigner la propriété en cours d'itération.

Si nous voulions copier le type nous aurions utilisé un type mappé :

```ts
type UnType = { prop1: number, prop2: string, prop3: boolean };
type MonType = { [K in keyof UnType]: UnType[K] };
```

Nous aurions ainsi eu dans l'*alias* : `{ prop1: number; prop2: string; prop3: boolean; }`.

Passons à l'étape suivante pour comprendre `{ [K in keyof T]: T[K] extends Function ? never : K }` :

```ts
type UnType = { prop1: number, prop2: string, prop3: boolean, prop4: () => void };
type MonType = { [K in keyof UnType]: UnType[K] extends Function ? never : K };
```

Ici nous utilisons un type conditionnel distribué et pour chaque propriété contenu dans le type *unType* nous regardons si le type de la propriété (`UnType[K]`) étend *Function*. Si c'est le cas, nous retournons le type *never*, sinon nous retournons le nom de la propriété.

Nous avons donc dans l'*alias UnType* : `{ prop1: "prop1"; prop2: "prop2"; prop3: "prop3"; prop4: never; }`.

Dernière étape : `[keyof T]` que nous avons vu en premier donc pas de difficulté ! Nous accédons à toutes les propriétés de `T` grâce à *keyof* et les propriétés typées *never* sont automatiquement écartées de l'*union* en sortie :

```ts
type UnType = { prop1: number, prop2: string, prop3: boolean, prop4: () => void };
type MonType = { [K in keyof UnType]: UnType[K] extends Function ? never : K }[keyof UnType];
```

Ce qui fait que nous avons ici dans l'*alias* l'*union* suivante : `"prop1" | "prop2" | "prop3"`.

Il n'y a plus qu'à utiliser un type générique `T` et nous obtenons bien :

```ts
type ProprietesPasFonctions<T> = { [K in keyof T]: T[K] extends Function ? never : K }[keyof T];
```

#### Aller encore plus loin

Nous pourrions maintenant vouloir copier toutes les propriétés d'un type qui ne sont pas de fonction, et ne pas seulement obtenir un *union* de types littéraux contenant les noms de propriétés.

Nous allons simplement utilise le type global utilitaire `Pick<T, K>` qui permet de récupérer les types en passant une *union* de noms de propriétés :

```ts
interface User {
  id: number;
  name: string;
  adresses: string[];
  updateName(newName: string): void;
}


type MonType<T> = Pick<T, { [K in keyof T]: T[K] extends Function ? never : K }[keyof T]>;

type proprietes = MonType<User>;
```

Nous obtenons bien dans l'*alias propriété* le type suivant :

```ts
type proprietes = {
  id: number;
  name: string;
  adresses: string[];
}
```
Ces combinaisons de types mappés et de types conditionnels distribués sont vraiment avancées et vous ne les rencontrerez que dans des librairies de projets très importants.

## L'inférence dans les types conditionnels

Nous allons voir l'inférence pour les types conditionnels. C'est un sujet également très avancé que vous rencontrerez rarement.

### Le mot clé *infer*

Le mot clé *infer* permet de demander à *TypeScript* d'inférer le type d'une variable.

Comme nous l'avons déjà vu au début du cours, l'inférence est le fait pour *TypeScript* de déduire le type d'un élément en fonction de la déclaration, de l'assignation et du contexte.

### Explications sur le type global `ReturnType<T>`

Nous avions vu que le type `ReturnType<T>` permettait de créer un type à partir du type de retour d'une fonction :

```ts
function uneFonction(): boolean | undefined | Todo {
  // ...
  return;
}
type TypeRetour = ReturnType<typeof uneFonction>;
// boolean | Todo | undefined
```

En fait, ce type global est un *alias* qui utilise l'inférence des types conditionnels :

```ts
T extends (...args: any) => infer R ? R : any
```

Il permet pour chaque argument du type `T` passé en argument de type, d'inférer le type de retour (`R` pour *Return*) et si ce n'est pas possible de typer en *any*.

Cela donne :

```ts
type Test1 = ReturnType<() => number>;  // number
type Test2 = ReturnType<() => () => any[]>;  // () => any[]
type Test3 = ReturnType<typeof Math.random>;  // number
type Test4 = ReturnType<typeof Array.isArray>;  // boolean
```

A noter que si nous passons une implémentation de fonction il faut obligatoirement utiliser *typeof* pour extraire le type avant de pouvoir le passer.

