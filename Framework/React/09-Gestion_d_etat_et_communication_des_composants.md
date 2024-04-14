# Introduction à la gestion d'état et aux communications des composants

## Les relations entre composants

Voici un premier aperçu général des relations entre composants, nous le détaillerons tout au long du chapitre :

- **Parent -> Enfant :** nous avons déjà vu que pour passer des données dans cette direction, il fallait simplement utiliser des _props_.
- **Enfant -> Parent :** dans ce cas il faut également utiliser des _props_. La composant parent **passe une fonction en** _prop_ au composant enfant qui permet de modifier l'état du composant parent. Le composant enfant n'a qu'à exécuter la fonction lorsqu'il a besoin de modifier l'état du composant parent.
- **Composants frères** (de même niveau, _siblings_) : dans ce cas il faut remonter l'état dans un **composant ancêtre commun des composants frères**. Cet ancêtre commun pourra passer des données en _props_ aux composants frères, et le composants frères pourront modifier l'état en exécutant des fonctions passées en _prop_ par l'ancêtre commun.
- **Composants éloignés dans l'arbre :** comment passer des données entre un composant parent et un composant très en dessous dans l'arbre des composants ? Une solution est de passer en _prop_ tout le long de l'arbre les données et les fonctions de modification. Cette solution est très répétitive et souvent illisible si elle est multipliée pour de nombreuses _props_. Nous verrons que nous pourrons utiliser _createContext()_ et _useContext()_ pour répondre à cette problématique.
- **Etat complexe :** souvent lorsque l'application devient un peu plus importante les composants en haut de l'arbre ont des états complexes et des modifications plus avancées des états locaux interviennent. Nous verrons qu'un _hook_ permet de répondre à cette problématique : _useReducer()_.

### Création de l'application de liste de tâches

Pour apprendre les notions de ce chapitre, nous allons réaliser une liste de tâches puis nous utiliserons ces notions dans notre projet.

Commencez par créer une nouvelle application :

```bash
npm create vite@latest todo-list -- --template react-swc
```

Copiez ensuite le dossier contenant les styles que nous avons utilisé dans tous les chapitres précédents et placez-le dans `src/assets/styles`.

Créez un dossier `components` dans le dossier `src`.

Créez-y les composants suivants : `TodoList.jsx`, `AddTodo.jsx`, `TodoItem.jsx` et `EditTodo.jsx`.

#### Modification de App.jsx

Dans le composant racine, mettez ce code de base :

```jsx
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

function App() {
  return (
    <div className="d-flex justify-content-center align-items-center p-20">
      <div className="card container p-20">
        <h1 className="mb-20">Liste de tâches</h1>
        <AddTodo />
        <TodoList />
      </div>
    </div>
  );
}

export default App;
```

#### Modification de EditTodo.jsx

Dans chaque composant du dossier `components` mettez le code suivant en adaptant bien sûr le nom du composant :

```jsx
export default function EditTodo() {
  return <h2>EditTodo</h2>;
}
```

## Communication d'un composant enfant vers un composant parent

#### Modification de App.jsx

Dans le composant racine, nous aurons la liste des tâches dans l'état local car c'est le composant qui est le premier ancêtre commun des composants `AddTodo` et `TodoList`.

Or, ces composants ont besoin de modifier / d'afficher la liste des tâches.

```jsx
import { useState } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

function App() {
  const [todoList, setTodoList] = useState([]);

  function addTodo(content) {
    const todo = { id: crypto.randomUUID(), done: false, edit: false, content };
    console.log(todo);
    setTodoList([...todoList, todo]);
  }

  return (
    <div className="d-flex justify-content-center align-items-center p-20">
      <div className="card container p-20">
        <h1 className="mb-20">Liste de tâches</h1>
        <AddTodo addTodo={addTodo} />
        <TodoList />
      </div>
    </div>
  );
}

export default App;
```

Remarquez bien que la fonction permettant d'ajouter une tâche ne modifie pas le tableau contenu dans l'état directement !

Rappelez-vous que **l'état local doit être considéré comme en lecture seule !**

Il faut donc d'abord créer un nouveau tableau avec l'opérateur _spread_ puis ajouter notre nouvelle tâche à celui-ci.

`crypto.randomUUID()` est une _API Web_ qui est compatible tout navigateur et permet de générer un _UUID v4_. C'est typiquement le genre d'identifiant que vous auriez dans une base de données.

Notez bien que nous passons cette fonction de modification au composant enfant `AddTodo` en _prop_. Ce composant pourra ensuite exécuter cette fonction pour ajouter une tâche et donc modifier l'état du composant parent.

Rappelez-vous que **le composant enfant a l'interdiction totale de modifier une _prop_ directement**, c'est pour cela qu'en _React_ nous passons des fonctions pour que les composants enfants puissent demander la modification de l'état.

Autrement dit, **un composant enfant ne doit JAMAIS modifier l'état reçu en _prop_ directement mais par l'intermédiaire d'une fonction**.

#### Modification du composant AddTodo

Le composant `AddTodo` a en état local la valeur du champ avec une double liaison :

- lorsque nous modifions l'état avec `setValue()`, la valeur du champ est modifié grâce à la liaison `value={value}`.
- lorsque l'utilisateur modifie la valeur du champ, l'état local est modifié grâce à `onChange={handleChange}`.

Notez bien que le composant reçoit en _prop_ la fonction `addTodo()` et qu'il l'invoque lorsque nous voulons ajouter une tâche, et donc modifier l'état du composant parent.

```jsx
import { useState } from "react";

export default function AddTodo({ addTodo }) {
  const [value, setValue] = useState("");

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === "Enter" && value.length) {
      addTodo(value);
      setValue("");
    }
  }

  function handleClick() {
    if (value.length) {
      addTodo(value);
      setValue("");
    }
  }

  return (
    <div className="d-flex justify-content-center align-items-center mb-20">
      <input
        type="text"
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        value={value}
        className="mr-15 flex-fill"
        placeholder="Ajouter une tâche"
      />
      <button onClick={handleClick} className="btn btn-primary">
        Ajouter
      </button>
    </div>
  );
}
```

#### Modification du partial \_themes.scss

```scss
input {
  border-radius: 4px;
  border: 1px solid var(--gray-2);
  padding: 10px 15px;
}
```

## Communication d'un composant parent vers un composant enfant

### Modification de _App.jsx_

Commençons par le composant racine :

```jsx
import { useState } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

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

Notez bien que la fonction de suppression utilise la méthode `filter()` **qui retourne un nouveau tableau sans modifier le tableau dans la variable d'état _todoList_.** Nous respectons donc bien la règle de ne pas modifier directement l'état !

Nous passons la fonction de suppression à notre composant `TodoList` en _prop_.

### Modification de `TodoList.jsx`

Dans le composant `TodoList` nous affichons un message s'il n'y a pas de tâches. S'il y a au moins un tâche dans le tableau _todoList_ nous parcourons le tableau et créons un composant `TodoItem` pour chaque tâche.

Nous utilisons l'identifiant unique de la tâche comme _key_.

Nous utilisons une _closure_ pour créer une fonction anonyme retournant la fonction de suppression liée à la valeur de l'identifiant unique de la tâche.

> _Si vous avez besoin d'une petite mise au point sur le concept de fermeture (**closure**), n'hésitez pas à vous rendre sur la formation JavaScript._

```jsx
import TodoItem from "./TodoItem";

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

En effet, quand la propriété _edit_ d'une tâche vaut _true_ nous voulons afficher le formulaire d'édition de cette tâche et quand elle vaut _false_, nous voulons afficher la tâche.

Nous devons donc ajouter cette mécanique pour déclencher le mode édition.

```jsx
import { useState } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

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

Dans _TodoList_, nous affichons un composant différent suivant que nous sommes en mode édition ou non.

Comme d'habitude, nous faisons une _closure_ et la passons en _prop_ pour permettre au composant `TodoItem` de déclencher le mode édition pour une tâche donnée :

```jsx
import TodoItem from "./TodoItem";
import EditTodo from "./EditTodo";

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

Dans `TodoItem`, nous passons simplement notre _prop_ `editTodo` en gestionnaire d'événement pour le clic sur le bouton "Modifier" :

```jsx
export default function TodoItem({ todo, deleteTodo, toggleTodo, editTodo }) {
  return (
    <li className="mb-10 d-flex justify-content-center align-items-center p-10">
      <span className="flex-fill">
        {todo.content} {todo.done && "✅"}
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

## Implémentation de l'édition

### Modification de `App.jsx`

Nous ajoutons une fonction pour éditer une tâche, elle change le contenu de la tâche et passe le mode édition à _false_ :

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

  function editTodo(id, content) {
    setTodoList(
      todoList.map((todo) =>
        todo.id === id ? { ...todo, edit: false, content } : todo
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
          editTodo={editTodo}
        />
      </div>
    </div>
  );
}

export default App;


Modification de TodoList.jsx

Dans le composant TodoList, nous passons en prop au composant EditTodo la fonction permettant d'éditer la tâche en la liant à l'id de la tâche encore grâce à une closure.

Nous passons également en prop la fonction permettant de modifier le mode édition de la tâche. Notez que nous la renommons en cancelEditTodo car ici la propriété edit vaut forcément true et ne peut être passé qu'à false. Nous renommons pour être plus spécifique et gagner en lisibilité et donc maintenabilité du code.

import TodoItem from './TodoItem';
import EditTodo from './EditTodo';

export default function TodoList({
  todoList,
  deleteTodo,
  toggleTodo,
  toggleTodoEdit,
  editTodo,
}) {
  return todoList.length ? (
    <ul>
      {todoList.map((todo) =>
        todo.edit ? (
          <EditTodo
            key={todo.id}
            todo={todo}
            editTodo={(content) => editTodo(todo.id, content)}
            cancelEditTodo={() => toggleTodoEdit(todo.id)}
          />
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


Modification de EditTodo.jsx

Le composant EditTodo est très similaire au composant pour ajouter une tâche :

import { useState } from 'react';

export default function EditTodo({ todo, editTodo, cancelEditTodo }) {
  const [value, setValue] = useState(todo.content);

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === 'Enter' && value.length) {
      editTodo(value);
      setValue('');
    }
  }

  function handleClick() {
    if (value.length) {
      editTodo(value);
      setValue('');
    }
  }

  return (
    <div className="d-flex justify-content-center align-items-center mb-10">
      <input
        type="text"
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        value={value}
        className="mr-15 flex-fill"
        placeholder="Ajouter une tâche"
      />
      <button onClick={handleClick} className="btn btn-primary mr-15">
        Sauvegarder
      </button>
      <button onClick={cancelEditTodo} className="btn btn-reverse-primary">
        Annuler
      </button>
    </div>
  );
}
```

## Selectionner une todo

### Modification de `App.jsx`

Nous ajoutons une fonction pour sélectionner une tâche et désélectionner les autres :

```jsx
import { useState } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

function App() {
  const [todoList, setTodoList] = useState([]);

  function addTodo(content) {
    const todo = {
      id: crypto.randomUUID(),
      done: false,
      edit: false,
      selected: false,
      content,
    };
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

  function editTodo(id, content) {
    setTodoList(
      todoList.map((todo) =>
        todo.id === id ? { ...todo, edit: false, content } : todo
      )
    );
  }

  function selectTodo(id) {
    setTodoList(
      todoList.map((todo) =>
        todo.id === id
          ? { ...todo, selected: !todo.selected }
          : { ...todo, selected: false }
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
          editTodo={editTodo}
          selectTodo={selectTodo}
        />
      </div>
    </div>
  );
}

export default App;
```

### Modification de `TodoList.jsx`

Nous modifions le composant `TodoList` simplement pour passer en _prop_ plus bas dans l'arbre la fonction permettant de sélectionner une tâche :

```jsx
import TodoItem from "./TodoItem";
import EditTodo from "./EditTodo";

export default function TodoList({
  todoList,
  deleteTodo,
  toggleTodo,
  toggleTodoEdit,
  editTodo,
  selectTodo,
}) {
  return todoList.length ? (
    <ul>
      {todoList.map((todo) =>
        todo.edit ? (
          <EditTodo
            key={todo.id}
            todo={todo}
            editTodo={(content) => editTodo(todo.id, content)}
            cancelEditTodo={() => toggleTodoEdit(todo.id)}
          />
        ) : (
          <TodoItem
            key={todo.id}
            todo={todo}
            deleteTodo={() => deleteTodo(todo.id)}
            toggleTodo={() => toggleTodo(todo.id)}
            editTodo={() => toggleTodoEdit(todo.id)}
            selectTodo={() => selectTodo(todo.id)}
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

Nous modifions le composant `TodoItem` :

```jsx
export default function TodoItem({
  todo,
  deleteTodo,
  toggleTodo,
  editTodo,
  selectTodo,
}) {
  return (
    <li
      onClick={selectTodo}
      className={`mb-10 d-flex flex-row justify-content-center align-items-center p-10 ${
        todo.selected ? "selected" : ""
      }  `}
    >
      <span className="flex-fill">
        {todo.content} {todo.done && "✅"}
      </span>
      <button
        className="btn btn-primary mr-15"
        onClick={(e) => {
          e.stopPropagation();
          toggleTodo();
        }}
      >
        Valider
      </button>
      <button
        className="btn btn-primary mr-15"
        onClick={(e) => {
          e.stopPropagation();
          editTodo();
        }}
      >
        Modifier
      </button>
      <button
        className="btn btn-reverse-primary"
        onClick={(e) => {
          e.stopPropagation();
          deleteTodo();
        }}
      >
        Supprimer
      </button>
    </li>
  );
}
```

Pourquoi avons-nous besoin de stopper la propagation des clics dans les gestionnaires d'événement des boutons ?

La réponse est le traitement par lots des mises à jour avant le prochain rendu.

En fait ce qu'il se passe, par exemple si nous cliquons sur le bouton "Valider" par exemple :

- `toggleTodo()` est exécutée :

```jsx
function toggleTodo(id) {
  setTodoList(
    todoList.map((todo) =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    )
  );
}
```

_React_ met **dans sa queue de mise à jour,** pour le prochain rendu la nouvelle valeur de notre tâche, par exemple :

```jsx
{
  content: "1",
  done: true,
  edit: false,
  id: "b702f80a-2d23-4f74-8388-5eebf681ffd4",
  selected: false
}
```

- ensuite l'événement se propage et remonte sur le _DOM_
- `selectTodo()` est donc exécutée :

```jsx
function selectTodo(id) {
  setTodoList(
    todoList.map((todo) =>
      todo.id === id
        ? { ...todo, selected: !todo.selected }
        : { ...todo, selected: false }
    )
  );
}
```

_React_ met **dans sa queue de mise à jour,** pour le prochain rendu la nouvelle valeur de notre tâche, par exemple :

```jsx
{
  content: "1",
  done: false,
  edit: false,
  id: "b702f80a-2d23-4f74-8388-5eebf681ffd4",
  selected: true
}
```

Remarquez bien que _done_ est à _false_ !

En effet, la valeur de la tâche pour ce rendu n'a pas changé ! Rappelez-vous que les mises à jour d'état se font au prochain rendu donc dans la liste nous avons toujours l'ancienne tâche et donc _done_ est à _false_ !

Au prochain rendu, _React_ exécute sa queue de mise à jour et prend la dernière mise à jour de la tâche et nous obtenons donc :

```jsx
{
  content: "1",
  done: false,
  edit: false,
  id: "b702f80a-2d23-4f74-8388-5eebf681ffd4",
  selected: true
}
```

C'est assez complexe à comprendre au début mais c'est vraiment essentiel à comprendre pour maîtriser _React_, aussi n'hésitez pas à relire 2 / 3 fois les explications.

Modification de `_themes.scss`

```scss
.selected {
  border: 2px solid var(--primary);
  border-radius: 4px;
}

li {
  cursor: pointer;
}
```

## Présentation du hook `useContext()` et de `createContext()`

## A quoi sert _Context_ ?

Nous avons vu jusqu'à maintenant que pour passer des informations d'un composant parent vers un composant enfant il suffisait d'utiliser des _props_.

Mais passer des _props_ peut devenir très verbeux et pénible s'il faut les passer très loin en bas de l'arbre des composants ou si plusieurs branches ont besoin des mêmes _props_.

Par exemple, nous avons besoin d'une information dans tous les composants en bleu. Nous devons donc passer en _prop_ cette information 8 fois :

![IMAGE Diagram with a tree of ten nodes, each node with two children or less. The root node contains a bubble representing a value highlighted in purple. The value flows down through the two children, each of which pass the value but do not contain it. The left child passes the value down to two children which are both highlighted purple. The right child of the root passes the value through to one of its two children - the right one, which is highlighted purple. That child passed the value through its single child, which passes it down to both of its two children, which are highlighted purple.](../../assets/images/React/passing_data_prop_drilling.webp)

**_Context_ permet de rendre des informations disponibles dans tous les composants enfant d'un composant même s'ils sont très éloignés.**

![IMAGE Diagram with a tree of ten nodes, each node with two children or less. The root parent node contains a bubble representing a value highlighted in orange. The value projects down directly to four leaves and one intermediate component in the tree, which are all highlighted in orange. None of the other intermediate components are highlighted.](../../assets/images/React/passing_data_context_far.webp)

### Utilisation du _Context_

#### 1 - Créer le _Context_

Pour créer un _Context_, on utilise un fichier pour le déclarer avec `createContext()` et on l'exporte.

**On passe la valeur que l'on souhaite partager en argument.**

Par convention le nom du fichier est suffixé par _Context_, par exemple `ExempleContext.js` :

```jsx
import { createContext } from "react";

export const ExempleContext = createContext(42);
```

#### 2 - Utiliser le _Context_

Pour ensuite l'utiliser dans n'importe quel composant il suffit de l'importer et d'utiliser le _hook_ `useContext()` :

```jsx
import { useContext } from "react";
import { ExempleContext } from "./ExempleContext.js";

export default function UnComposant() {
  const data = useContext(ExempleContext);
}
```

**Comme tous les _hooks_ il est obligatoire d'utiliser `useContext()` au premier niveau d'un composant (ni imbriqué dans un bloc conditionnel, ni dans une fonction).**

#### 3 - Fournir un autre _Context_

Pour fournir (_provide_) un _Context_ il faut l’importer et utiliser le composant `<NomContext.provider value={valeur}>` :

```jsx
import ExempleContext from "./ExempleContext";
import UnComposantEnfant from "./UnComposantEnfant";

export default function UnComposantParnet() {
  return (
    <ExempleContext.Provider value={100}>
      <UnComposantEnfant />
    </ExempleContext.Provider>
  );
}
```

**Cela permet de changer la valeur fournie pour tous les composants situés plus bas dans l'arbre.**

_Pour comprendre le fonctionnement du **Context** vous pouvez faire le parallèle avec l'héritage des propriétés **CSS**. En **CSS**, quand vous appliquez une propriété sur un élément, par exemple `color: blue` sur une `<div>`, tous les éléments en-dessous seront bleu sauf si vous écrasez cette propriété pour un autre élément plus bas sur le **DOM**._

**Le _Context_ n'est pas forcément une valeur statique vous pouvez très bien le lier à une valeur dynamique, par exemple un état :**

```jsx
import ExempleContext from "./ExempleContext";
import UnComposantEnfant from "./UnComposantEnfant";
import { useState } from "react";

export default function UnComposantParnet() {
  const [valeur, setValeur] = useState(42);

  return (
    <ExempleContext.Provider value={valeur}>
      <UnComposantEnfant />
    </ExempleContext.Provider>
  );
}
```

### Quand utiliser le _Context_ ?

Il ne faut pas abuser du _Context_, en _React_ il n'est pas rare que des dizaines de _props_ soient passées le long de l'arbre des composants sur une application complexe. Ce n'est pas une raison pour les remplacer par _Context_.

**En effet, avec les _props_ il est facile de voir d'où elles viennent et où elles vont. Elles sont donc plus facilement lisibles et maintenables.**

**Voici des exemples d'usages recommandés du _Context_ par l'équipe _React_ :**

- **Les thèmes :** si votre application a deux ou plusieurs thèmes (par exemple un mode sombre), vous pouvez utiliser un _Context_ pour indiquer le mode choisi aux composants qui ont besoin d'être adapté suivant le mode.
- **L'utilisateur connecté :** de nombreux composants partout dans l'application ont souvent besoin d'avoir des informations sur l'utilisateur connecté. Cela permet de rendre disponible ces informations partout dans l'application.
- **_Routing_ :** la plupart des solutions de _React_ pour le routing utilise en fait le _Context_. Nous verrons cela dans la suite de la formation.
- **Gérer l'état global :** nous verrons que dans les applications importantes il peut être efficace de gérer l'état en combinant _Context_ et le _hook_ `useReducer()`.

## Exemple d'utilisation du hook `useContext()`

### Création de `ThemeContext.js`

Dans notre application, créez un dossier `context` dans `src`.

Dans ce dossier, créez le fichier `ThemeContext.js` :

```jsx
import { createContext } from "react";

const ThemeContext = createContext("primary");
export default ThemeContext;
```

### Création du composant `Button.jsx`

Dans le dossier `components`, créez le fichier `Button.jsx` :

```jsx
import { useContext } from "react";
import ThemeContext from "../context/ThemeContext";

export default function Button({ text, className, ...props }) {
  const theme = useContext(ThemeContext);

  return (
    <button
      {...props}
      className={`btn btn-${theme} ${className ? className : ""}`}
    >
      {text}
    </button>
  );
}
```

**{ text, className, ...props } :** permet de récupérer les _props_ _text_ et _className_ grâce à la déstructuration et de mettre le reste des _props_ dans une propriété _props_ grâce à l'opérateur **rest (...)**.

> _N'hésitez pas à revoir l'opérateur **rest** -également appelé paramètres du reste-, qui ne doit pas être confondu avec l'opérateur **spread** -également appelé opérateur de décomposition-, dans la formation **JavaScript**._

**{ ...props } :** permet de copier toutes les propriétés de l'objet _props_ (grâce à l'opérateur _spread_) et de les passer comme _props_ au composant natif _button_.

Notez que nous récupérons le `Context` avec le hook `useContext()`.

Nous l'utilisons pour dynamiquement changer la classe de nos boutons.

### Modification du partial `_themes.scss`

```scss
.btn-secondary {
  background-color: #2980b9;
  color: white;
  border: 2px solid #2980b9;
}
```

### Modification du composant `AddTodo.jsx`

Nous utilisons notre nouveau composant `Button` :

```jsx
import { useState } from "react";
import Button from "./Button";

export default function AddTodo({ addTodo }) {
  const [value, setValue] = useState("");

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === "Enter" && value.length) {
      addTodo(value);
      setValue("");
    }
  }

  function handleClick() {
    if (value.length) {
      addTodo(value);
      setValue("");
    }
  }

  return (
    <div className="d-flex justify-content-center align-items-center mb-20">
      <input
        type="text"
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        value={value}
        className="mr-15 flex-fill"
        placeholder="Ajouter une tâche"
      />
      <Button text="Ajouter" onClick={handleClick} />
    </div>
  );
}
```

### Modification du composant `TodoItem.jsx`

Nous utilisons notre nouveau composant `Button` :

```jsx
import Button from "./Button";

export default function TodoItem({
  todo,
  deleteTodo,
  toggleTodo,
  editTodo,
  selectTodo,
}) {
  return (
    <li
      onClick={selectTodo}
      className={`mb-10 d-flex flex-row justify-content-center align-items-center p-10 ${
        todo.selected ? "selected" : ""
      }  `}
    >
      <span className="flex-fill">
        {todo.content} {todo.done && "✅"}
      </span>
      <Button
        text="Valider"
        className="mr-15"
        onClick={(e) => {
          e.stopPropagation();
          toggleTodo();
        }}
      />
      <Button
        text="Modifier"
        className="mr-15"
        onClick={(e) => {
          e.stopPropagation();
          editTodo();
        }}
      />
      <Button
        text="Supprimer"
        onClick={(e) => {
          e.stopPropagation();
          deleteTodo();
        }}
      />
    </li>
  );
}
```

### Modification du composant `EditTodo.jsx`

Nous utilisons notre nouveau composant `Button` :

```jsx
import { useState } from "react";
import Button from "./Button";

export default function EditTodo({ todo, editTodo, cancelEditTodo }) {
  const [value, setValue] = useState(todo.content);

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === "Enter" && value.length) {
      editTodo(value);
      setValue("");
    }
  }

  function handleClick() {
    if (value.length) {
      editTodo(value);
      setValue("");
    }
  }

  return (
    <div className="d-flex justify-content-center align-items-center mb-10">
      <input
        type="text"
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        value={value}
        className="mr-15 flex-fill"
        placeholder="Ajouter une tâche"
      />
      <Button text="Sauvegarder" className="mr-15" onClick={handleClick} />
      <Button text="Annuler" className="mr-15" onClick={cancelEditTodo} />
    </div>
  );
}
```

### Modification du composant `App.jsx`

Dans le composant racine nous ajoutons une liste d'options permettant de modifier le thème des boutons.

La liste est liée de manière bidirectionnelle avec l'état local _theme_ grâce à `value={theme} onChange={handleThemeChange}`.

Nous utilisons ensuite un fournisseur de _Context_ pour lier la valeur de la variable d'état _theme_ au _Context_ fourni aux composants enfants du composant racine (donc tous les autres composants) grâce à `<ThemeContext.Provider value={theme}>`.

```jsx
import { useState } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";
import ThemeContext from "./context/ThemeContext";

function App() {
  const [todoList, setTodoList] = useState([]);
  const [theme, setTheme] = useState("primary");

  function addTodo(content) {
    const todo = {
      id: crypto.randomUUID(),
      done: false,
      edit: false,
      selected: false,
      content,
    };
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

  function editTodo(id, content) {
    setTodoList(
      todoList.map((todo) =>
        todo.id === id ? { ...todo, edit: false, content } : todo
      )
    );
  }

  function selectTodo(id) {
    setTodoList(
      todoList.map((todo) =>
        todo.id === id
          ? { ...todo, selected: !todo.selected }
          : { ...todo, selected: false }
      )
    );
  }

  function handleThemeChange(e) {
    setTheme(e.target.value);
  }

  return (
    <ThemeContext.Provider value={theme}>
      <div className="d-flex justify-content-center align-items-center p-20">
        <div className="card container p-20">
          <h1 className="mb-20 d-flex justify-content-center align-items-center">
            <span className="flex-fill">Liste de tâches</span>
            <select value={theme} onChange={handleThemeChange}>
              <option value="primary">Rouge</option>
              <option value="secondary">Bleu</option>
            </select>
          </h1>
          <AddTodo addTodo={addTodo} />
          <TodoList
            todoList={todoList}
            deleteTodo={deleteTodo}
            toggleTodo={toggleTodo}
            toggleTodoEdit={toggleTodoEdit}
            editTodo={editTodo}
            selectTodo={selectTodo}
          />
        </div>
      </div>
    </ThemeContext.Provider>
  );
}

export default App;
```

## Présentation de `useReducer()`

### Qu'est-ce qu'un Reducer ?

Les composants qui ont de nombreux _hooks_ d'état et qui ont des mises à jour nombreuses de ces états peuvent devenir rapidement complexes à maintenir et à organiser.

**Pour ces composants, vous pouvez utiliser un _hook_ spécifique qui permet de gérer toutes les mises à jour de l'état du composant dans une fonction unique, extérieure au composant, appelée _reducer_.**

Le _hook_ `useReducer()` vient en remplacement de plusieurs _hooks_ `useState()` dans un composant.

La syntaxe d'un reducer est :

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- **state :** état courant contenant un objet avec des propriétés. _Vous pouvez le voir comme la fusion de tous les états locaux des `useState()`._
- **dispatch :** fonction permettant d'envoyer une _action_ au _reducer_.
- **reducer :** fonction permettant de mettre à jour l'état (_state_) en fonction d'une _action_.
- **initialState :** état initial pour le premier rendu du composant.

### Les actions

Une _action_ est l'objet passé à la fonction `dispatch()` et qui représente ce que l'utilisateur a fait.

Une _action_ doit représenter une seule interaction même si elle a plusieurs effets.

Par convention, une _action_ a une **propriété** _type_ qui décrit ce qu'il s'est passé et qui sert au _reducer_ pour l'identifier, et **optionnellement d'autres propriétés avec des données.**

### Le reducer

Le _reducer_ est une **fonction pure** qui reçoit en arguments l'**état courant** et l'_action_.

**Il doit retourner le nouvel état résultant des modifications engendrées par l'_action_.**

Comme pour `useState()`, **l'état doit toujours être considéré en lecture seul et c'est un nouvel objet qui doit être retourné comme nouvel état.**

**Par convention, on utilise une instruction _switch / case_ avec un cas pour chaque type d'_action_.**

Exemple :

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "increment_age": {
      return {
        name: state.name,
        age: state.age + 1,
      };
    }
    case "change_name": {
      return {
        name: action.newName,
        age: state.age,
      };
    }
  }
  throw Error("Action inconnue : " + action.type);
}
```

_Le nom de **reducer**, vient de la programmation fonctionnelle, et plus précisément de la fonction `reduce()` dont le fonctionnement est similaire._

_`reduce()` prend en premier argument la valeur courante de l'accumulation et en second argument l'accumulateur et retourne la nouvelle valeur de l'accumulateur._

*Les **reducers** de **React** ont la même logique : ils prennent l'état actuel et une *action* et retourne le nouvel état. Ils "accumulent" les *actions* au cours du temps et retourne à chaque fois la nouvelle valeur de l'état qui est l'accumulateur.*

### Exemple basique

Par exemple, si nous avons un bouton qui incrémente un compteur contenu dans l'état, nous pourrions avoir une _action_ qui a une propriété _type_ **'inc_count'** :

```jsx
function handleClick() {
  dispatch({ type: "inc_count" });
}
```

Voici un exemple complet basique pour comprendre le fonctionnement :

```jsx
import { useReducer } from "react";

function reducer(state, action) {
  if (action.type === "inc_count") {
    return {
      count: state.count + 1,
    };
  }
  throw Error("Action inconnue.");
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      <button
        onClick={() => {
          dispatch({ type: "inc_count" });
        }}
      >
        +1
      </button>
      <p>Le compteur vaut {state.count}.</p>
    </>
  );
}
```

Evidemment, dans cet exemple trivial, utiliser `useReducer()` n'a aucun intérêt, mais nous devons commencer par un exemple très simple pour bien comprendre les étapes.

L'état initial est `{count: 0}`.

Lorsque l'utilisateur clique sur le bouton, une _action_ `{type: 'inc_count'}` est envoyée en utilisant la fonction _dispatch_ (qui signifie justement expédier / envoyer).

Le _reducer_ reçoit alors en premier argument l'état actuel en premier argument et l'_action_ en second argument.

Il met à jour l'état en fonction de la propriété _type_ de l'_action_ et des éventuelles données contenues dans celle-ci.

Il doit retourner le nouvel état.

Pour résumer :

**Etat initial => dispatch(action) => reducer(etatActuel, action) => nouvel état**

## Exemple d'utilisation de useReducer()

### Modification de *App.jsx*

L'utilisation d'un *reducer* va grandement simplifier notre composant racine.

En effet, nous allons extraire toute la logique des fonctions pour se contenter d'envoyer des *actions* en fonction des interactions de l'utilisateur.

Comme nous l'avons vu dans la leçon précédente, chaque *action* contient un *type* et éventuellement une propriété permettant de modifier l'état.

```ts
import { useReducer } from 'react';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';
import ThemeContext from './context/ThemeContext';
import todoReducer from './reducers/todoReducer';

function App() {
  const [state, dispatch] = useReducer(todoReducer, {
    theme: 'primary',
    todoList: [],
  });

  function addTodo(content) {
    dispatch({
      type: 'ADD_TODO',
      content,
    });
  }

  function deleteTodo(id) {
    dispatch({
      type: 'DELETE_TODO',
      id,
    });
  }

  function toggleTodo(id) {
    dispatch({
      type: 'TOGGLE_TODO',
      id,
    });
  }

  function toggleTodoEdit(id) {
    dispatch({
      type: 'TOGGLE_EDIT_TODO',
      id,
    });
  }

  function editTodo(id, content) {
    dispatch({
      type: 'EDIT_TODO',
      id,
      content,
    });
  }

  function selectTodo(id) {
    dispatch({
      type: 'SELECT_TODO',
      id,
    });
  }

  function handleThemeChange(e) {
    dispatch({
      type: 'SET_THEME',
      theme: e.target.value,
    });
  }

  return (
    <ThemeContext.Provider value={state.theme}>
      <div className="d-flex justify-content-center align-items-center p-20">
        <div className="card container p-20">
          <h1 className="mb-20 d-flex justify-content-center align-items-center">
            <span className="flex-fill">Liste de tâches</span>
            <select value={state.theme} onChange={handleThemeChange}>
              <option value="primary">Rouge</option>
              <option value="secondary">Bleu</option>
            </select>
          </h1>
          <AddTodo addTodo={addTodo} />
          <TodoList
            todoList={state.todoList}
            deleteTodo={deleteTodo}
            toggleTodo={toggleTodo}
            toggleTodoEdit={toggleTodoEdit}
            editTodo={editTodo}
            selectTodo={selectTodo}
          />
        </div>
      </div>
    </ThemeContext.Provider>
  );
}

export default App;
```
 
### Création de *todoReducer.js*

Dans le dossier **src**, créez un dossier **reducers** et placez-y un fichier *todoReducer.js* :

```ts
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO': {
      return {
        ...state,
        todoList: [
          ...state.todoList,
          {
            id: crypto.randomUUID(),
            content: action.content,
            edit: false,
            done: false,
            selected: false,
          },
        ],
      };
    }
    case 'DELETE_TODO': {
      return {
        ...state,
        todoList: state.todoList.filter((t) => t.id !== action.id),
      };
    }
    case 'TOGGLE_TODO': {
      return {
        ...state,
        todoList: state.todoList.map((t) =>
          t.id !== action.id ? t : { ...t, done: !t.done }
        ),
      };
    }
    case 'TOGGLE_EDIT_TODO': {
      return {
        ...state,
        todoList: state.todoList.map((t) =>
          t.id !== action.id ? t : { ...t, edit: !t.edit }
        ),
      };
    }
    case 'EDIT_TODO': {
      return {
        ...state,
        todoList: state.todoList.map((t) =>
          t.id !== action.id
            ? t
            : { ...t, content: action.content, edit: false }
        ),
      };
    }
    case 'SELECT_TODO': {
      return {
        ...state,
        todoList: state.todoList.map((t) =>
          t.id !== action.id
            ? { ...t, selected: false }
            : { ...t, selected: true }
        ),
      };
    }
    case 'SET_THEME': {
      return {
        ...state,
        theme: action.theme,
      };
    }
    default: {
      throw new Error('action inconnue');
    }
  }
}

export default todoReducer;
```

Remarquez bien que l'état précédent n'est jamais modifié directement, il faut retourner un nouvel état.

## Architecture avec les hooks useContext() et useReducer()

L'objectif va être ici de combiner tout ce que nous avons vu sur les *reducers* et les *Contexts*.

Nous avons vu que nous pouvions fournir n'importe quelle valeur dans un *Context*.

**Nous pouvons donc passer l'état et la fonction d'envoi d'*action* (*dispatch*) retournés par un *reducer* dans un *Context* pour les rendre disponible à tous les composants plus bas dans l'arbre.**

### Création de TodoContext.js

Dans le dossier **context** commencez par créez un fichier *TodoContext.js* :

```ts
import { createContext } from 'react';

export const TodoStateContext = createContext(null);
export const TodoDispatcherContext = createContext(null);
```
 
### Modification de *App.jsx*

Dans le composant racine nous allons fournir l'état et la fonction *dispatch* à tous les composants plus bas dans l'arbre grâce au *Context* :

```ts
import { useReducer } from 'react';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';
import ThemeContext from './context/ThemeContext';
import { TodoStateContext } from './context/TodoContext';
import { TodoDispatcherContext } from './context/TodoContext';
import todoReducer from './reducers/todoReducer';

function App() {
  const [state, dispatch] = useReducer(todoReducer, {
    theme: 'primary',
    todoList: [],
  });

  function handleThemeChange(e) {
    dispatch({
      type: 'SET_THEME',
      theme: e.target.value,
    });
  }

  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatcherContext.Provider value={dispatch}>
        <ThemeContext.Provider value={state.theme}>
          <div className="d-flex justify-content-center align-items-center p-20">
            <div className="card container p-20">
              <h1 className="mb-20 d-flex justify-content-center align-items-center">
                <span className="flex-fill">Liste de tâches</span>
                <select value={state.theme} onChange={handleThemeChange}>
                  <option value="primary">Rouge</option>
                  <option value="secondary">Bleu</option>
                </select>
              </h1>
              <AddTodo />
              <TodoList />
            </div>
          </div>
        </ThemeContext.Provider>
      </TodoDispatcherContext.Provider>
    </TodoStateContext.Provider>
  );
}

export default App;
```

Remarquez que nous n'avons plus aucune *prop* à passer aux composants enfants car nous allons utiliser le *hook* `useContext()` pour accéder à l'état et à *dispatch* plus bas dans l'arbre des composants.

### Modification de *TodoList.jsx*

Dans le composant *TodoList* nous pouvons grandement simplifier, nous n'avons plus de *prop* à passer depuis le composant parent vers les composants enfants.

Remarquez que nous accédons à l'état, et plus particulièrement à la liste des tâches, en utilisant le *hook* `useContext()` et en récupérant l'état fourni.

```ts
import { useContext } from 'react';
import TodoItem from './TodoItem';
import EditTodo from './EditTodo';
import { TodoStateContext } from '../context/TodoContext';

export default function TodoList() {
  const state = useContext(TodoStateContext);

  return state.todoList.length ? (
    <ul>
      {state.todoList.map((todo) =>
        todo.edit ? (
          <EditTodo key={todo.id} todo={todo} />
        ) : (
          <TodoItem key={todo.id} todo={todo} />
        )
      )}
    </ul>
  ) : (
    <p>Aucune tâche en cours </p>
  );
}
```
 
### Modification de *AddTodo.jsx*

Dans le composant *AddTodo* nous accédons à la fonction *dispatch* grâce au *Context* et nous pouvons directement envoyer l'*action* permettant d'ajouter une tâche :

```ts
import { useState, useContext } from 'react';
import Button from './Button';
import { TodoDispatcherContext } from '../context/TodoContext';

export default function AddTodo() {
  const [value, setValue] = useState('');
  const dispatch = useContext(TodoDispatcherContext);

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === 'Enter' && value.length) {
      dispatch({
        type: 'ADD_TODO',
        content: value,
      });
      setValue('');
    }
  }

  function handleClick() {
    if (value.length) {
      dispatch({
        type: 'ADD_TODO',
        content: value,
      });
      setValue('');
    }
  }

  return (
    <div className="d-flex justify-content-center align-items-center mb-20">
      <input
        type="text"
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        value={value}
        className="mr-15 flex-fill"
        placeholder="Ajouter une tâche"
      />
      <Button text="Ajouter" onClick={handleClick} />
    </div>
  );
}
```

### Modification de *TodoItem.jsx*

Dans le composant *TodoItem* nous accédons à la fonction *dispatch* grâce au *Context* et nous pouvons directement envoyer les *actions* permettant de sélectionner une tâche, de la modifier, de la supprimer ou de la valider ou non :

```ts
import React, { useContext } from 'react';
import Button from './Button';
import { TodoDispatcherContext } from '../context/TodoContext';

export default function TodoItem({ todo }) {
  const dispatch = useContext(TodoDispatcherContext);

  return (
    <li
      onClick={() =>
        dispatch({
          type: 'SELECT_TODO',
          id: todo.id,
        })
      }
      className={`mb-10 d-flex flex-row justify-content-center align-items-center p-10 ${
        todo.selected ? 'selected' : ''
      }  `}
    >
      <span className="flex-fill">
        {todo.content} {todo.done && '✅'}
      </span>
      <Button
        text="Valider"
        className="mr-15"
        onClick={(e) => {
          e.stopPropagation();
          dispatch({
            type: 'TOGGLE_TODO',
            id: todo.id,
          });
        }}
      />
      <Button
        text="Modifier"
        className="mr-15"
        onClick={(e) => {
          e.stopPropagation();
          dispatch({
            type: 'TOGGLE_EDIT_TODO',
            id: todo.id,
          });
        }}
      />
      <Button
        text="Supprimer"
        onClick={(e) => {
          e.stopPropagation();
          dispatch({
            type: 'DELETE_TODO',
            id: todo.id,
          });
        }}
      />
    </li>
  );
}
```

### Modification de *EditTodo.jsx*

Dans le composant *EditTodo* nous accédons à la fonction *dispatch* grâce au *Context* et nous pouvons directement envoyer les *actions* permettant d'éditer une tâche et d'annuler le mode édition :

```ts
import { useState, useContext } from 'react';
import Button from './Button';
import { TodoDispatcherContext } from '../context/TodoContext';

export default function EditTodo({ todo }) {
  const [value, setValue] = useState(todo.content);
  const dispatch = useContext(TodoDispatcherContext);

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === 'Enter' && value.length) {
      dispatch({
        type: 'EDIT_TODO',
        id: todo.id,
        content: value,
      });
      setValue('');
    }
  }

  function handleClick() {
    if (value.length) {
      dispatch({
        type: 'EDIT_TODO',
        id: todo.id,
        content: value,
      });
      setValue('');
    }
  }

  return (
    <div className="d-flex justify-content-center align-items-center mb-10">
      <input
        type="text"
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        value={value}
        className="mr-15 flex-fill"
        placeholder="Ajouter une tâche"
      />
      <Button text="Sauvegarder" className="mr-15" onClick={handleClick} />
      <Button
        text="Annuler"
        className="mr-15"
        onClick={() =>
          dispatch({
            type: 'TOGGLE_EDIT_TODO',
            id: todo.id,
          })
        }
      />
    </div>
  );
}
```

## Amélioration de l'architecture

### Modification de *TodoContext.js*

Nous utilisons des fonctions utilitaires pour faciliter l'usage des *Contexts* en ayant qu'un seul import à faire dans les composants qui les utilisent :

```ts
import { createContext, useContext } from 'react';

export const TodoStateContext = createContext(null);
export const TodoDispatcherContext = createContext(null);

export const useTodos = () => useContext(TodoStateContext);
export const useTodoDispatcher = () => useContext(TodoDispatcherContext);
```

### Création de *TodoProvider.jsx*

Créez un composant *TodoProvider.jsx* dans le dossier **components**.

Nous utilisons ce composant pour fournir tous les *Contexts* et affichons la *prop children* reçue :

```ts
import { useReducer } from 'react';
import ThemeContext from '../context/ThemeContext';
import { TodoStateContext } from '../context/TodoContext';
import { TodoDispatcherContext } from '../context/TodoContext';
import todoReducer from '../reducers/todoReducer';

function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, {
    theme: 'primary',
    todoList: [],
  });

  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatcherContext.Provider value={dispatch}>
        <ThemeContext.Provider value={state.theme}>
          {children}
        </ThemeContext.Provider>
      </TodoDispatcherContext.Provider>
    </TodoStateContext.Provider>
  );
}

export default TodoProvider;
```
 
### Création de *TodoFeature.jsx*

Créez un composant *TodoFeature.jsx* dans le dossier **components**.

Le *JSX* de ce composant sera passé en *children* au composant *TodoProvider.jsx*.

```ts
import { useTodoDispatcher, useTodos } from '../context/TodoContext';
import AddTodo from './AddTodo';
import TodoList from './TodoList';

function TodoFeature() {
  const dispatch = useTodoDispatcher();
  const state = useTodos();

  function handleChange(e) {
    dispatch({
      type: 'SET_THEME',
      theme: e.target.value,
    });
  }

  return (
    <div className="d-flex flex-row justify-content-center align-items-center p-20">
      <div className="card container p-20">
        <h1 className="mb-20 d-flex flex-row justify-content-center align-items-center">
          <span className="flex-fill">Todo list</span>
          <select value={state.theme} onChange={handleChange}>
            <option value="primary">Rouge</option>
            <option value="secondary">Bleu</option>
          </select>
        </h1>
        <AddTodo />
        <TodoList />
      </div>
    </div>
  );
}

export default TodoFeature;
```

### Modification de *App.jsx*

Nous modifions le composant racine, qui est maintenant très concis :

```ts
import TodoFeature from './components/TodoFeature';
import TodoProvider from './components/TodoProvider';

function App() {
  return (
    <TodoProvider>
      <TodoFeature />
    </TodoProvider>
  );
}

export default App;
```