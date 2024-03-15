# Introduction aux composants

## Introduction aux composants

### Définition d’un composant

**Un composant est une brique réutilisable et isolée.**

Les composants sont les briques fonctionnelles sur lesquelles repose votre application.

**Un composant *React* est une fonction qui peut optionnellement recevoir en paramètres des propriétés (les *props* que nous étudierons) et qui retourne un élément *React*.** Cet élément *React* peut par exemple être du *markup JSX*.

Un composant est responsable d’une portion de l’interface utilisateur et décrit comment elle doit apparaître.

Ainsi par exemple, voici un composant qui affiche un simple bouton :

```jsx
function MonBouton() {
  return (
    <button>Cliquez moi</button>
  );
}
```

**Le nom d'un composant doit commencer par une lettre majuscule.**

Il est possible ensuite de l'afficher avec `render()` :

```jsx
root.render(<MonBouton />);
```

Notez la syntaxe raccourcie : vous pouvez directement écrire `<MonBouton />` au lieu de `<MonBouton></MonBouton>`

## Exporter et importer des composants

### Export par défaut d'un composant

Les applications *React* sont découpées en un très grand nombre de composants qui sont répartis logiquement dans des fichiers.

Pour exporter un composant depuis un fichier, vous pouvez l'**exporter par défaut.**

Dans ce cas, créez un fichier en *UpperCamelCase* avec le nom du composant par exemple *MonSuperComposant* et utilisez l'extension *.js* ou *.jsx*.

```jsx
export default function MyApp() {
  return (
    <h1>Bienvenue !</h1>
  );
}
```

Notez l'utilisation du mot clé *default*.

Vous pouvez ensuite l'importer dans un autre fichier de cette manière :

```jsx
import App from './App';
```

Notez qu'un fichier ne peut avoir qu'un seul export par défaut.

### Export nommé d'un composant

Vous pouvez également exporté un composant de manière nommé, sans utilisation de *default* :

```jsx
export function MyApp() {
  return (
    <h1>Bienvenue !</h1>
  );
}
```

Dans ce cas **il faut utiliser des accolades lors de l'import et vous devez obligatoirement utiliser le même nom que le nom de la fonction exportée :**
```jsx
import { App } from './App';
```

**Avec React, le plus souvent on utilise un export par défaut lorsqu'un fichier exporte un seul composant et on utilise les exports nommés lorsqu'un fichier exporte plusieurs composants et / ou valeurs.**

>*Si vous voulez en savoir plus sur le détail des imports et des exports de modules JavaScript, référez-vous aux leçons dédiées de la formation JavaScript.*

## Passer des props aux composants

### A quoi servent les props ?

Nous avons vu que l’objectif des composants *React* est d’être réutilisable.

Pour que cela soit possible il faut pouvoir leur passer des propriétés afin qu’ils puissent les utiliser pour retourner des éléments *JSX* différents en fonction de celles-ci. Ces propriétés s’appellent les *props*.

**Il s’agit du moyen de *React* pour rendre les composants dynamiques en leur passant des données, d’un composant parent vers un composant enfant.**

Le flux de données est unidirectionnel comme nous allons le voir, c’est à dire qu’il va **toujours dans le sens composant parent vers composant enfant.**

Un composant parent est un composant qui contient un ou plusieurs composants appelé(s) enfant(s).

 
### Passer des props depuis des composants parents

*props* est un objet passé à un composant lors de son utilisation.

Par exemple :

```jsx
<Composant prenom='Paul'/>
```

Ici nous utilisons un composant *Composant* et nous lui passons une propriété *prenom*.

Dans le composant nous pourrons y accéder sur *props* avec *props.prenom*, car *props* est **toujours un objet**.

#### Les props ne sont pas modifiables dans les composants enfants

Il faut garder à l’esprit que **les props ne sont jamais modifiables (elles sont immuables ou *immutables*)**, il ne faut surtout pas que les composants recevant la pr*op la modifie.

Cela ne veut pas dire que les *props* ne peuvent pas changer au cours du temps, le composant parent peut envoyer de nouvelles valeurs au cours du temps !

Nous verrons également que les composants enfants peuvent demander aux composants parents de modifier la valeur des *props*.

 #### Passer des props qui ne sont pas des chaînes de caractères

**Pour passer des *props* qui ne sont pas des chaînes de caractères il faut utiliser des accolades simples :**

```jsx
<Composant name="Jean" age={12} />
```

**Pour passer des props qui sont des objets il faut utiliser des accolades doubles :**

```jsx
<Composant name="Jean" age={12} adresse={{ville: 'Paris', cp: '75017'}} />
```
 
### Lire les props dans les composants enfants

**Pour lire les *props* dans un composant enfant, vous pouvez utiliser l'objet *props* passé en argument :**

```jsx
export function Composant(props) {
  return (
    <h2>
      Hello {props.name}! Tu as {props.age} ans
    </h2>
  );
}
```

Le plus souvent on utilise la **décomposition** *JavaScript* pour plus de lisibilité et de concision :

```jsx
export function Composant({name, age}) {
  return (
    <h2>
      Hello {name}! Tu as {age} ans
    </h2>
  );
}
```

Vous pouvez également, grâce à cette syntaxe, assigner des valeurs par défaut :

```jsx
export function Composant({name = 'Jean', age = 42}) {
  return (
    <h2>
      Hello {name}! Tu as {age} ans
    </h2>
  );
}
```

## Passage de props et propriété children

### Passage de props le long de plusieurs composants

Parfois vous voudrez simplement passer l'ensemble des *props* plus bas d'en l'arbre des composants, par exemple `Parent > Enfant > PetitEnfant`.

C'est-ce qu'on appelle le passage de props (*props forwarding*) et on utilise pour cela l'opérateur *JavaScript spread* (**...**).

```jsx
function Enfant(props) {
  return (
    <div className="container">
      <PetitEnfant {...props} />
    </div>
  );
}
```

Cela permet de ne pas avoir à copier manuellement l'ensemble des *props*.

### Passer du *JSX* grâce à la propriété *children*

**Il est possible d'imbriquer du *JSX* à l'intérieur des balises d'un composant, comme vous le feriez avec du *HTML* :**

```jsx
<Composant>
  <button>Cliquez</button>
</Composant>
```

**Dans ce cas, le composant pourra accéder et afficher au contenu *JSX* passé grâce à la propriété *children* :**

```jsx
export function Composant({ children }) {
  return (
    <>
      <h1>Hello world !</h1>
      {children}
    </>
  );
}
```
Il est possible de passer toute expression *JSX*, y compris un composant :

```jsx
<Composant>
  <AutreComposant />
</Composant>
```