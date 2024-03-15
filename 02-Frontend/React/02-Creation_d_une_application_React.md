# Création d'une application React

## Utilisation de create-vite et structure d'une application React

### Utilisation de create-vite

*Vite* fournit plusieurs *templates* maintenus officiellement pour démarrer un projet *React*.

Nous allons utiliser la version utilisant *swc*.

Créez un dossier pour suivre cette formation, par exemple *dyma-react*, et ouvrez le dans *VS Code*.

Ouvrez ensuite un terminal et faites :

```bash
npm create vite@latest
```

Entrez `y` puis `entrée`.

Entrez un nom de projet. Par exemple, *dyma-react*.

Choisissez *React*.

Choisissez *JavaScript + SWC*.

 
### Le fichier package.json

Ce fichier au format *JSON* contient les dépendances du projet nécessaires en production sous *dependencies*, les dépendances nécessaires en développement sous *devDependencies*, ainsi que leurs versions *SemVer*.

Il contient également des *scripts* que vous pouvez lancer facilement depuis un terminal : par exemple `npm run dev`.

#### Dépendances de production (dependencies) :

- **react :** cœur de la librairie *React*.
- **react-dom :** librairie qui fournit des méthodes spécifiques au *DOM* pour *React*, permettant de manipuler le *DOM*.

#### Dépendances de développement (devDependencies) :

- **@types/react :** contient les déclarations de types *TypeScript* pour *React*. Ceci est utilisé pour avoir un support de type lors de l'écriture de composants React en *TypeScript* mais aussi pour l’auto-complétion et la détection d'erreurs statiques dans *VS Code*.
- **@types/react-dom :** comme *@types/react*, mais pour *react-dom*. 
- **@vitejs/plugin-react-swc :** un plugin *Vite* pour utiliser *SWC* avec des projets *React*. *SWC* est le compilateur de code source très rapide écrit en *Rust*. 
- **eslint :** un outil de *linting* pour *JavaScript* et *TypeScript* qui aide à détecter et corriger les erreurs de syntaxe, les mauvaises pratiques de codage, et à assurer la cohérence du style de code.
- **eslint-plugin-react :** un *plugin ESLint* spécifique pour les projets *React*, qui ajoute des règles de *linting* propres à *React*.
- **eslint-plugin-react-hooks :** un *plugin ESLint* qui ajoute des règles de *linting* pour les *Hooks* de *React*, afin de s'assurer qu'ils sont utilisés correctement.
- **eslint-plugin-react-refresh :** un *plugin ESLint* pour intégrer *React Fast Refresh*, une fonctionnalité qui permet de recharger à chaud les composants *React* sans perdre leur état.

#### Les scripts

Voici une explication rapide pour chaque *script* du fichier *package.json* :

- **"dev": "vite" :** exécute *Vite* en mode développement, démarrant un serveur de développement local pour l'application. Les modifications apportées au code source sont reflétées en temps réel grâce au rechargement à chaud.

- **"build": "vite build" :** utilise *Vite* pour construire l'application pour la production. Cela génère une version optimisée du site, prête à être déployée.

- **"lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0" :** exécute *ESLint* sur tous les fichiers *.js* et *.jsx* dans le projet. Le *script* signale également les directives de désactivation d'*ESLint* non utilisées et échoue si des avertissements sont trouvés (grâce à l'option *--max-warnings 0*).

- **"preview": "vite preview" :** exécute un serveur local pour prévisualiser la version de production (*build*) de l'application. Cela est utile pour tester la version de production localement avant le déploiement.

### Le système de gestion sémantique des versions

**Le système de gestion sémantique des versions** (SemVer) permet de gérer facilement les versions d'une librairie avec trois nombres séparés par des points : *MAJEUR.MINEUR.CORRECTIF* :

1. le numéro de version MAJEUR quand il y a des changements non rétro-compatibles,
2. le numéro de version MINEUR quand il y a des ajouts de fonctionnalités rétro-compatibles (par exemple ajout d'une fonctionnalité supplémentaire),
3. le numéro de version de CORRECTIF quand il y a des corrections d’anomalies rétro-compatibles.

### Installation des dépendances et lancement de l'application en développement

Dans un terminal, dans le dossier de votre projet, installez les dépendances :

```bash
npm i
```

Puis lancez le serveur de développement de *Vite* :

```bash
npm run dev
```

Cette commande va exécuter un serveur de développement *Node.js* grâce à *Vite* et le lancer sur le *port 5173* par défaut.

Vous pouvez donc retrouver votre première application sur : http://localhost:5173/.

 
### Création du build pour la production

On appelle *build* l'ensemble des fichiers générés par *Vite* pour la **production**, c'est-à-dire **la distribution de votre application *React* par un serveur *Web* à vos utilisateurs**.

Pour générer un *build* de votre application faites la commande :

```bash
npm run build
```

Nous verrons à la fin du cours comment configurer un serveur *Web* pour servir le *build* de l'application.

### Les autres dossiers et fichiers du projet

#### Le fichier package-lock.json

Ce fichier permet de connaitre la version exacte utilisée sur votre projet pour chaque librairie, ainsi que toutes les versions des dépendances de ces librairies.

C’est un outil très puissant pour gérer toutes les versions de toutes les dépendances et leurs relations. 

Vous n’aurez jamais à modifier ou à lire ce fichier. Il s’agit uniquement d’un fichier utilitaire qui sera utilisé par *npm* lors des installations, des mises à jour et des désinstallations des librairies qui sont des dépendances de votre projet.

Il est utile pour le travail en équipe ou sur plusieurs environnements car il permet d'installer exactement les mêmes dépendances quelque soit l'environnement lorsqu'on exécute la commande *npm install*.

#### Le fichier .gitignore

Ce fichier permet de lister les fichiers qui doivent être ignorés par *Git*.

Ce fichier est déjà bien configuré pour la plupart des applications, vous n’y toucherez donc probablement pas.

#### Le fichier .eslintrc.cjs

Ce fichier de configuration *ESLint* spécifie les règles et les paramètres pour l'analyse statique du code source d'un projet *React*.

- **root: true** : indique que ce fichier est la configuration racine et *ESLint* ne devrait pas chercher d'autres fichiers de configuration dans les répertoires parents.
- **env** : définit l'environnement d'exécution du code, ici le navigateur et *ECMAScript 2020* sont activés.
- **extends** : utilise plusieurs configurations recommandées et prédéfinies pour *ESLint* et des *plugins React*, y compris les *hooks React* et le *runtime JSX*.
- **ignorePatterns** : *ESLint* ignorera les fichiers dans les répertoires *dist* et le fichier de configuration *ESLint* actuel.
- **parserOptions** : définit la version *ECMAScript* comme la dernière et indique que le code source utilise le type de module *ECMAScript*.
- **settings** : spécifie la version de *React* utilisée dans le projet.
- **plugins** : inclut le plugin *react-refresh*.
- **rules** : définit une règle spécifique du plugin *react-refresh*, qui émet un avertissement si certains types de composants *React* ne sont pas exportés correctement, avec une option pour autoriser l'exportation de constantes.

#### Le dossier public

Ce dossier comporte tous les éléments qui ne sont pas optimisés par *Vite*.

#### Le fichier index.html

Ce fichier est la seule page *HTML* de votre application !

Oui, rappelez vous nous construisons une *SPA* !

Il contient les imports notamment du favicon de votre application et du point d'entrée pour le code source de l'application *React* (*main.jsx*).

#### Le dossier src

Ce dossier est la source de votre application. Il contiendra toute l’application que vous développerez avec *React*.

#### Le fichier src/main.jsx

Ce fichier est le premier fichier *JavaScript* à s’exécuter.

Nous y reviendrons en détails, mais il permet notamment d’injecter votre application *React* dans l’*index.html* avec :

```jsx
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

#### Le fichier src/index.css

Ce fichier contient les styles globaux de l'application qui seront disponibles dans toute l'application.

#### Le fichier src/App.jsx

Ce fichier contient la partie logique du composant racine de l’application.

Nous allons étudier en détails dans les prochains chapitres ce que sont les composants *React* et pourquoi ce qui semble être du *HTML* est écrit dans une fonction *JavaScript*.

#### Le fichier src/App.css

Ce fichier contient le style du composant.

Nous étudierons également en détails les manières dont le *CSS* peut être organisé dans les applications *React*.

## Comment fonctionne React ?

### Le DOM et le DOM virtuel

Pour rappel, le *DOM* (Document Object Model) est une *API* pour les documents *HTML*. 

Il permet de **représenter de manière structurée un document sous la forme d'un arbre**. Il définit également comment cette structure peut être manipulée par du code.

Le *DOM* est constitué de nœuds et d'objets. Les nœuds peuvent avoir des gestionnaires d'événements qui se déclenchent lorsqu'un événement se produit et activer notamment des scripts pour manipuler les pages web.

Avant d’afficher le composant sur le *DOM*, il existe en réalité une étape intermédiaire : la création d’un *DOM virtuel*.

Dans les applications web modernes, il peut y avoir des centaines ou des milliers de nœuds sur lesquels sont enregistrés de nombreux gestionnaires d'événements.

En conséquence, de nombreuses mises à jour du *DOM* doivent être réalisées.

Or, **ces mises à jour sont coûteuses en performances car plus le *DOM* est grand, plus les recherches et les changements sont coûteux**.

*React* utilise donc un *DOM virtuel* qui est une représentation du *DOM* en *JavaScript*. Le *DOM virtuel* n'est donc qu'un simple objet *JavaScript*.

Le *DOM virtuel* est modifié avant que les modifications sur le *DOM* ne soient effectuées.

Le *DOM virtuel* est une représentation légère du *DOM* en *JavaScript* qui peut être utilisée par des algorithmes très performants de comparaison des différences entre *DOM virtuel* et *DOM*.

Dès lors que le *DOM* doit être modifié, un patch est appliqué par *React* de manière optimisé pour ne réaliser que les changements absolument nécessaires.

Par exemple, si aucune valeur affichée ne change, alors aucune mise à jour du *DOM* n'est déclenchée ce qui économise beaucoup en performance.

En résumé, *React* va créer ou récupérer le *DOM virtuel* puis le comparer de manière extrêmement optimisé au *DOM* puis va effectuer les insertions, les modifications et les suppressions d’éléments *HTML* nécessaires.

### Le fichier main.jsx

Regardons une nouvelle fois le fichier racine de notre application React :

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

Commençons par étudier *ReactDOM* et *render*.

### ReactDOM et render

#### Que fait la méthode ReactDOM.createRoot() ? 

Elle permet de définir le nœud racine du *DOM* sur lequel toute l'application *React* sera rendue.

#### Que fait la méthode render() ? 

Elle permet de rendre l’élément *React* passé en premier argument sur l'élément passé à `ReactDOM.createRoot()`.

Cela permet donc ici d'afficher notre composant `App`, appelé **composant racine**, dans l’élément *HTML* qui a pour *id root*.

Cet élément se trouve dans le *body* de *index.html* : nous avons en effet :

```html
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
```

Cela veut donc dire que *React* va afficher votre premier composant dans une div dans le body de notre page *HTML*, tout simplement.

#### Que fait `<App />` ?

`<App />` permet d'indiquer où placer le **composant** *React App* qui est situé dans le fichier *App.jsx* :

```jsx
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <div>
        <a href="https://vitejs.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.jsx</code> and save to test HMR
        </p>
      </div>
      <p className="read-the-docs">
        Click on the Vite and React logos to learn more
      </p>
    </>
  )
}

export default App
```

Nous verrons en détail les composants plus tard. Remarquez simplement que le composant *App* est une fonction qui retourne du *JSX*.

Nous verrons plus en détails le langage *JSX* dans les prochaines leçons.

### Le mode strict

*React.StrictMode* permet de détecter les problèmes potentiels d’une application. Il active des **vérifications et avertissements supplémentaires** pour toute l'application React car nous lui passons l'élément racine, tout en haut de l'arbre des composants.

Les vérifications du mode strict sont effectuées uniquement durant le développement.

## createRoot, render et le JSX

### Les éléments React

Les éléments *React* sont des objets qui décrivent ce que vous voulez afficher à l’écran.

*React* utilise ces objets pour construire le *DOM* et le mettre à jour.

Pour créer un élément *React* il suffit d’utiliser la méthode prévue :

```jsx
React.createElement(
  type,
  [props],
  [...children]
)
```

Le type peut être un élément *HTML* par exemple *div* et comme nous le verrons plus tard, un composant ou un fragment *React*.

Vous pouvez lui passer des propriétés, comme par exemple une classe, en deuxième paramètre.

Vous pouvez enfin lui passer des enfants (*children*) qui peuvent être une chaîne de caractère ou d’autres éléments *React*.
 
### Le langage JSX

*React* recommande l’utilisation de *JSX*. 

*JSX* est une extension syntaxique à JavaScript qui permet d’utiliser du *HTML* amélioré dans du *JavaScript*.

Nous pouvons réécrire l’exemple précédent de la manière suivante :

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
const element = <h1>Bonjour de Dyma</h1>;

root.render(element);
```

## Le langage JSX

### Rappels sur JSX 

*JSX* est un langage construit par dessus le *JavaScript*.

Il ne peut pas être exécuté par les navigateurs qui ne peuvent exécuter que du *JavaScript*.

Il est donc nécessaire de transpiler le *JSX* vers du *JavaScript* natif.

C’est *Vite* qui réalise cette tâche.

```jsx
const element = <h1>Bonjour de Dyma</h1>;
```

Est donc transpilé en *JavaScript* ce qui donne le code suivant, avant l’exécution par le navigateur :

```jsx
const element = React.createElement(
  'h1',
  null,
  'Bonjour de Dyma'
);
```
 
### Utilisation d’expression JavaScript

**Avec *JSX* vous pouvez utiliser des expressions *JavaScript* directement en utilisant des accolades :**

```jsx
const prenom = 'Jean';
const element = <h1>Bonjour, {prenom}</h1>;

ReactDOM.render(element);
```

**Il est possible d’utiliser toute expression *JavaScript* qui peut être convertie en chaîne de caractères.**

Vous pouvez utiliser des **parenthèses** si votre élément fait plusieurs lignes.

Par exemple :

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));

const direBonjour = () => 'Bonjour !';
const obj = {
  prenom: 'Jean',
};

const element = (
  <div>
    <p> Vous pouvez utiliser des nombres : {2 + 3}</p>
    <p> Vous pouvez utiliser des fonctions : {direBonjour()}</p>
    <p> Vous pouvez utiliser des propriétés d'objet : {obj.prenom}</p>
    <p> Vous NE POUVEZ PAS utiliser des objet directement.</p>
  </div>
);

root.render(element);
```

### Utilisation d’expressions JSX

Du code *JSX* est une expression valide qui peut être utilisée également.

L’expression sera de toute façon transpilée en *JavaScript*.

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
const monExpressionJx = () => {
  if (true) {
    return <div>Il est possible d'utiliser une expression JSX !</div>;
  } else {
    return <div>Il est possible d'utiliser une expression JSX !</div>;
  }
};

const element = monExpressionJx();

root.render(element);
```
 
### Définir des attributs avec JSX

Vous pouvez définir des attributs en utilisant du *JSX* par exemple assigner la valeur de l’attribut `src` dans une image :

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));

const lien = "https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Collage_of_Six_Cats-02.jpg/1024px-Collage_of_Six_Cats-02.jpg";

// JSX permet de définir des attributs
// Si l'élément est vide vous pouvez utiliser la noation raccourcie />

const element = <img src={lien}/>;

root.render(element);
```
 
### Utilisation de la notation raccourcie pour les éléments vides

Lorsqu’un élément HTML est vide, par exemple :

```html
<img></img>
```

Vous pouvez utiliser une notation raccourcie permise par *JSX* :

```jsx
<img />
```
 
### Utilisation de fragments

En React, il est obligatoire que l'élément rendu ait un élément parent unique. C'est-à-dire qu'il faut qu'il y ait un élément contenant s'il y a plusieurs éléments enfants.

On ne peut donc pas faire :

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));

const element = (
  <div>
    <h1>Bonjour de Dyma</h1>
  </div>
  <p>Un paragraphe</p>
);

root.render(element);
```

Vous aurez l'erreur *JSX expressions must have one parent element*.

Comme il est courant pour un composant de **renvoyer plusieurs éléments. Les fragments nous permettent de grouper une liste d’enfants sans ajouter de nœud supplémentaire au *DOM***.

Il faut utiliser des balises vides :

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));

const element = (
  <>
    <div>
      <h1>Bonjour de Dyma</h1>
    </div>
    <p>Un paragraphe</p>
  </>
);

root.render(element);
```
 
### Immutabilité des éléments React

Les éléments *React* sont immuables (immutables), cela veut dire qu’une fois défini ils ne peuvent être modifiés lors de l’exécution.

L’expression est évaluée et affichée et n’est pas mise à jour par défaut, par exemple :

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));

const element = (
  <div>
    <h1>Bonjour de Dyma</h1>
    <h2>Il est exactement {new Date().toLocaleTimeString()} lors de l'exécution.</h2>
  </div>
);

root.render(element);
```

Même si nous recréions l’élément, il ne serait pas mis à jour car il a été évalué une seule fois et est immutable :

```jsx
import React from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));

const element = (
  <div>
    <h1>Bonjour de Dyma</h1>
    <h2>
      Il est exactement {new Date().toLocaleTimeString()} lors de l'exécution.
    </h2>
  </div>
);

setInterval(() => root.render(element), 1000);
```
 
### Mise à jour des éléments React 

Le seul moyen de mettre à jour un élément est d’en créer un nouveau.

Il sera réévalué en un nouvel objet *JavaScript* avec une référence distincte.

Dans l’exemple, nous isolons l’élément dans une *const* qui a une portée de bloc pour que chaque exécutions de l’intervalle crée un tout nouvel élément *JavaScript* sans aucune référence au précédent. 

Dans ce cas, le *DOM* est mis à jour mais pas entièrement !

Et oui, rappelez-vous que React utilise un *DOM virtuel* et il va utiliser des algorithmes de comparaison très performants pour comparer le *DOM virtuel* avant et après la modification pour ne mettre à jour que l’élément *HTML* qui a été modifié sur le *DOM* :

## Utilisation alternative de React

Vous pouvez passer rapidement cette leçon si vous êtes intéressé par l'utilisation de *React* comme *framework Web* complet.

Ici, nous allons voir comment utiliser *React* pour une petite partie d'une application.

 
### Installation de Live Server

Dans *VS Code*, allez dans l'onglet *Extensions* et recherchez *Live Server*. 

Installez l'extension.

 ### Création de l'application

Créez un dossier `test` et un fichier `index.html` dans ce dernier.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script
      crossorigin
      src="https://unpkg.com/react@18/umd/react.development.js"
    ></script>
    <script
      crossorigin
      src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"
    ></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  </head>
  <body>
    <div id="app"></div>
    <script type="text/babel">
      const root = ReactDOM.createRoot(document.getElementById('app'));
      const searchBar = <input placeholder="rechercher" />;

      root.render(searchBar);
    </script>
  </body>
</html>
```
Lancez ensuite le fichier avec *Live Server*. 