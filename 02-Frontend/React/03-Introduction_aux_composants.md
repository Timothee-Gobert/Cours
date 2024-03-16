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

## Communication d'un composant parent vers un composant enfant

### Modification de *App.jsx*

Commençons par le composant racine :

```jsx
import { useState } from 'react';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';

function App() {
  const [todoList, setTodoList] = useState([]);

  function addTodo(content) {
    const todo = { id: crypto.randomUUID(), done: false, edit: false, content };
    setTodoList([...todoList, todo]);
  }

  function deleteTodo(id) {
    setTodoList(todoList.filter((todo) => todo.id !== id));
  }

  return (
    <div className="d-flex justify-content-center align-items-center p-20">
      <div className="card container p-20">
        <h1 className="mb-20">Liste de tâches</h1>
        <AddTodo addTodo={addTodo} />
        <TodoList todoList={todoList} deleteTodo={deleteTodo} />
      </div>
    </div>
  );
}

export default App;
```

Notez bien que la fonction de suppression utilise la méthode `filter()` **qui retourne un nouveau tableau sans modifier le tableau dans la variable d'état *todoList*.** Nous respectons donc bien la règle de ne pas modifier directement l'état !

Nous passons la fonction de suppression à notre composant `TodoList` en *prop*.

### Modification de `TodoList.jsx`

Dans le composant `TodoList` nous affichons un message s'il n'y a pas de tâches. S'il y a au moins un tâche dans le tableau *todoList* nous parcourons le tableau et créons un composant `TodoItem` pour chaque tâche.

Nous utilisons l'identifiant unique de la tâche comme *key*.

Nous utilisons une *closure* pour créer une fonction anonyme retournant la fonction de suppression liée à la valeur de l'identifiant unique de la tâche.

>*Si vous avez besoin d'une petite mise au point sur le concept de fermeture (**closure**), n'hésitez pas à vous rendre sur la formation JavaScript.*

```jsx
import TodoItem from './TodoItem';

export default function TodoList({ todoList, deleteTodo }) {
  return todoList.length ? (
    <ul>
      {todoList.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          deleteTodo={() => deleteTodo(todo.id)}
        />
      ))}
    </ul>
  ) : (
    <p>Aucune tâche en cours </p>
  );
}
```
 
### Modification de `TodoItem.jsx`

Dans le composant d'affiche de chaque tâche nous avons simplement :

```jsx
export default function TodoItem({ todo, deleteTodo }) {
  return (
    <li className="mb-10 d-flex justify-content-center align-items-center p-10">
      <span className="flex-fill">{todo.content}</span>
      <button className="btn btn-primary mr-15">Valider</button>
      <button className="btn btn-primary mr-15">Modifier</button>
      <button className="btn btn-reverse-primary" onClick={deleteTodo}>
        Supprimer
      </button>
    </li>
  );
}
```

## Valider et éditer une todo

### Modification de `App.jsx`

Dans le composant racine, nous ajoutons une fonction pour passer une tâche en mode édition.

En effet, quand la propriété *edit* d'une tâche vaut *true* nous voulons afficher le formulaire d'édition de cette tâche et quand elle vaut *false*, nous voulons afficher la tâche.

Nous devons donc ajouter cette mécanique pour déclencher le mode édition.

```jsx
import { useState } from 'react';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';

function App() {
  const [todoList, setTodoList] = useState([]);

  function addTodo(content) {
    const todo = { id: crypto.randomUUID(), done: false, edit: false, content };
    setTodoList([...todoList, todo]);
  }

  function deleteTodo(id) {
    setTodoList(todoList.filter((todo) => todo.id !== id));
  }

  function toggleTodo(id) {
    setTodoList(
      todoList.map((todo) =>
        todo.id === id ? { ...todo, done: !todo.done } : todo
      )
    );
  }

  function toggleTodoEdit(id) {
    setTodoList(
      todoList.map((todo) =>
        todo.id === id ? { ...todo, edit: !todo.edit } : todo
      )
    );
  }

  return (
    <div className="d-flex justify-content-center align-items-center p-20">
      <div className="card container p-20">
        <h1 className="mb-20">Liste de tâches</h1>
        <AddTodo addTodo={addTodo} />
        <TodoList
          todoList={todoList}
          deleteTodo={deleteTodo}
          toggleTodo={toggleTodo}
          toggleTodoEdit={toggleTodoEdit}
        />
      </div>
    </div>
  );
}

export default App;
```
 
### Modification de `TodoList.jsx`

Dans *TodoList*, nous affichons un composant différent suivant que nous sommes en mode édition ou non.

Comme d'habitude, nous faisons une *closure* et la passons en *prop* pour permettre au composant `TodoItem` de déclencher le mode édition pour une tâche donnée :

```jsx
import TodoItem from './TodoItem';
import EditTodo from './EditTodo';

export default function TodoList({
  todoList,
  deleteTodo,
  toggleTodo,
  toggleTodoEdit,
}) {
  return todoList.length ? (
    <ul>
      {todoList.map((todo) =>
        todo.edit ? (
          <EditTodo key={todo.id} todo={todo} />
        ) : (
          <TodoItem
            key={todo.id}
            todo={todo}
            deleteTodo={() => deleteTodo(todo.id)}
            toggleTodo={() => toggleTodo(todo.id)}
            editTodo={() => toggleTodoEdit(todo.id)}
          />
        )
      )}
    </ul>
  ) : (
    <p>Aucune tâche en cours </p>
  );
}
```
 
### Modification de `TodoItem.jsx`

Dans `TodoItem`, nous passons simplement notre *prop* `editTodo` en gestionnaire d'événement pour le clic sur le bouton "Modifier" :

```jsx
export default function TodoItem({ todo, deleteTodo, toggleTodo, editTodo }) {
  return (
    <li className="mb-10 d-flex justify-content-center align-items-center p-10">
      <span className="flex-fill">
        {todo.content} {todo.done && '✅'}
      </span>
      <button className="btn btn-primary mr-15" onClick={toggleTodo}>
        Valider
      </button>
      <button className="btn btn-primary mr-15" onClick={editTodo}>
        Modifier
      </button>
      <button className="btn btn-reverse-primary" onClick={deleteTodo}>
        Supprimer
      </button>
    </li>
  );
}
```

