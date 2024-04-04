# Rendu conditionnel et liste

## Introduction au chapitre et Emmet

### Activer _Emmet_ pour _React_

Dans _VS Code_ rendez-vous dans `Ficher > Préférences > Paramètres` puis recherchez _Emmet_.

Au niveau de _Emmet: Include Languages_ cliquez sur _Add item_ puis ajoutez en clé _javascript_ et en valeur _javascriptreact_.

### Projet de départ

Créez une nouvelle application avec _Vite_ :

```bash
npm create vite@latest rendu-conditionnel -- --template react-swc
```

Modifiez le composant _App.jsx_ :

```jsx
import Articles from "./components/Articles";

function App() {
  return (
    <div className="d-flex flex-row justify-content align-items-center p-20">
      <Articles />
    </div>
  );
}

export default App;
```

Créez un dossier `components` dans `src` et créez-y le composant `Articles.jsx` :

```jsx
function Articles() {
  return (
    <div style={{ width: "700px" }}>
      <h1 className="mb-20">Liste des articles</h1>
      <div className="card p-20">
        <h2 className="mb-10">Titre de l'article</h2>
        <p>
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Sequi
          consectetur asperiores dicta iusto inventore eius corrupti eum! Iste
          doloremque delectus voluptas quos. Et at consectetur similique
          suscipit officiis cum asperiores? Sunt maxime repudiandae quaerat
          debitis. Nobis asperiores, soluta perferendis voluptatibus vel
          dignissimos optio ipsam repellat repellendus tenetur illo blanditiis,
          modi placeat non consequuntur quibusdam dolorum, temporibus quod
          suscipit.
        </p>
      </div>
    </div>
  );
}

export default Articles;
```

Créez un dossier `assets` dans `src` et copiez-y le dossier styles.

Modifiez `main.jsx` pour les importer :

```jsx
import React, { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./assets/styles/index.scss";
import App from "./App";

const rootElement = document.getElementById("root");
const root = createRoot(rootElement);

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

## Affichage avec les ternaires et AND

### Qu’est- ce que le rendu conditionnel ?

Le rendu conditionnel d’éléments est simplement **le fait d’afficher ou non un ou plusieurs éléments si une ou plusieurs conditions sont remplies.**

Par exemple, si un utilisateur est connecté, vous voulez que dans le menu il y est l’élément permettant de se déconnecter.

A l’inverse, si il est déconnecté, vous souhaitez lui afficher un bouton de connexion.

### _React_ et le rendu conditionnel

Avec _React_ le rendu conditionnel est très simple grâce à _JSX_.

En effet, vous pouvez utiliser des conditions JavaScript pour déterminer quel(s) élément(s) afficher.

Nous allons voir que nous pouvons utiliser toutes les instructions conditionnelles disponibles en _JavaScript_ : _if_, _else_, _else if_, _&&_ et _? :_ (opérateur ternaire).

### Le rendu conditionnel avec l’opérateur ternaire _? :_

Lorsque vous voulez afficher un élément si une condition est respectée et un autre élément si elle n’est pas respectée, vous pouvez utiliser l’opérateur ternaire conditionnel.

Cet opérateur _JavaScript_ natif s’utilise de la manière suivante :

```
condition ? expr1 : expr2 ;
```

Dans un composant vous pouvez ainsi retourner du _markup_ différent suivant une condition grâce à _JSX_ :

```jsx
function Todo({ text, isDone }) {
  return <li>{isDone ? <del>{text + " ✔"}</del> : text}</li>;
}
```

Ici nous retournons soit le texte de la tâche barré et avec une _checkmark_ si elle est terminée (si _isDone_ vaut _true_) et sinon juste le texte.

### Le rendu conditionnel avec l’opérateur logique &&

Vous connaissez _&&_, il s’agit de l’opérateur logique “et”. Par exemple :

```
expr1 && expr2;
```

Si _expr1_ est _falsy_ (c'est-à-dire que l'expression est convertie implicitement en _false_) alors cette expression retourne _false_.

> _Vous pouvez revoir les valeurs falsy dans la formation JavaScript._

Dans le cas contraire, l’expression retourne _expr2_.

Cette méthode permet donc d’afficher un élément si et seulement si une condition est respectée.

Dans un composant vous pouvez ainsi retourner du _markup_ différent suivant une condition grâce à _JSX_ :

```jsx
function Todo({ text, isDone }) {
  return (
    {text} {isDone && '✔'}
  );
}
```

Ici nous ajoutons la _checkmark_ uniquement si _isDone_ est _true_.

## Affichage avec les if et les variables

### L’utilisation de variables

Grâce au _JSX_ vous pouvez stocker des éléments différents dans des variables facilement suivant une ou plusieurs conditions.

En reprenant notre exemple :

```jsx
function Todo({ text, isDone }) {
  let content = text;
  if (isDone) {
    content = text + " ✔";
  }
  return <li>{content}</li>;
}
```

Vous pouvez même assigner du _markup_ sur plusieurs lignes grâce à l'utilisation des parenthèses :

```jsx
function Todo({ text, isDone }) {
  let content = text;
  if (isDone) {
    content = <del>{text + " ✔"}</del>;
  }
  return <li>{content}</li>;
}
```

### L’utilisation de composants

Il est aussi bien sûr possible de rendre conditionnellement un composant ou rien avec _&&_ :

```jsx
{
  condition && <Composant />;
}
```

Ou avec l'opérateur ternaire :

```jsx
{
  condition ? <Composant /> : null;
}
```

Ou encore d'afficher un composant différent suivant une condition :

```jsx
{
  condition ? <Composant1 /> : <Composant2 />;
}
```

## Afficher des listes avec map

### Rappels sur la méthode _map()_

La méthode _JavaScript_ **map()** permet de créer un nouveau tableau en appliquant une fonction sur chaque élément du tableau sur lequel elle est utilisée.

```jsx
const arr = [1, 2, 3];
const mapArr = arr.map((el) => el * 2); // [2, 4, 6]
```

### _React_ et les listes

Pour afficher des listes avec _React_, nous avons deux moyens.

Le premier est **d’assigner à une variable un nouveau tableau contenant les éléments à afficher :**

```jsx
const nombres = [1, 2, 3];
const elements = nombres.map((n) => <li>{n}</li>);

function Composant() {
  return (
    <div>
      <ul>{elements}</ul>
    </div>
  );
}
```

Le second est directement d’**utiliser _map()_ dans l’expression _JSX_ retournée :**

```jsx
const nombres = [1, 2, 3];

function Composant() {
  return (
    <div>
      <ul>
        {nombres.map((n) => (
          <li> {n * 2}</li>
        ))}
      </ul>
    </div>
  );
}
```

> _N'hésitez pas à aller revoir la leçon I**ntroduction à la programmation fonctionnelle** de la formation **JavaScript** si vous n'êtes pas à l'aise avec **map()**._

## Utilisation de filter, des clés et des fragments

### Les clés (_keys_)

Dans la leçon précédente, si vous ouvriez la console du navigateur, vous auriez l'avertissement suivant :  
`Warning: Each child in a list should have a unique “key” prop.`

Cela signifie que chaque élément d'une liste devrait avoir une _prop_ _key_ unique.

**Les clés permettent à React de savoir à quel élément d'un tableau un élément de liste correspond. C'est important si les éléments d'un tableau peuvent être modifiés, par exemple leur index (lors d'un tri), ou s'ils sont supprimés ou insérés.**

En effet, dans ces cas, il est nécessaire de mettre à jour le _DOM_ et _React_ doit savoir quelles mises à jour sont nécessaires. Pour ce faire, il utilise la clé unique correspondant à chaque élément.

Le plus souvent, la clé unique est un _id_ d'une rangée ou d'un document dans une base de données.

Par exemple :

```jsx
function Articles({}) {
  const articles = [
    { title: "Un titre 1", content: "Contenu 1", id: 1 },
    { title: "Un titre 2", content: "Contenu 2", id: 2 },
    { title: "Un titre 3", content: "Contenu 3", id: 3 },
  ];

  return (
    <div style={{ width: "700px" }}>
      <h1 className="mb-20">Liste des articles</h1>
      {articles.map((article) => (
        <Article
          key={article.id}
          content={article.content}
          title={article.title}
        />
      ))}
    </div>
  );
}
```

### Afficher plusieurs éléments sur le _DOM_ pour chaque élément du tableau

Comme dans notre exemple, le plus souvent vous voudrez afficher plusieurs éléments sur le _DOM_ pour un même élément du tableau qui sera un objet.

Dans ce cas, vous pouvez utiliser un fragment sur lequel vous utiliserez la _prop_ _key_ unique.

Pour pouvoir utiliser la _prop_, il faut utiliser la syntaxe `<Fragment>` et non la syntaxe raccourcie `<>` que nous avions vue :

```jsx
import { Fragment } from "react";

function Articles({}) {
  const articles = [
    { title: "Un titre 1", content: "Contenu 1", id: 1 },
    { title: "Un titre 2", content: "Contenu 2", id: 2 },
    { title: "Un titre 3", content: "Contenu 3", id: 3 },
  ];

  return (
    <div style={{ width: "700px" }}>
      <h1 className="mb-20">Liste des articles</h1>
      {articles.map((article) => (
        <Fragment key={article.id}>
          <div className="card p-20 mb-20">
            <h2 className="mb-10">
              <h2>{article.title}</h2>
            </h2>
            <p>{article.content}</p>
          </div>
        </Fragment>
      ))}
    </div>
  );
}

export default Articles;
```
