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
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';

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
import { useState } from 'react';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';

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

Il faut donc d'abord créer un nouveau tableau avec l'opérateur *spread* puis ajouter notre nouvelle tâche à celui-ci.

`crypto.randomUUID()` est une *API Web* qui est compatible tout navigateur et permet de générer un *UUID v4*. C'est typiquement le genre d'identifiant que vous auriez dans une base de données.

Notez bien que nous passons cette fonction de modification au composant enfant `AddTodo` en *prop*. Ce composant pourra ensuite exécuter cette fonction pour ajouter une tâche et donc modifier l'état du composant parent.

Rappelez-vous que **le composant enfant a l'interdiction totale de modifier une *prop* directement**, c'est pour cela qu'en *React* nous passons des fonctions pour que les composants enfants puissent demander la modification de l'état. 

Autrement dit, **un composant enfant ne doit JAMAIS modifier l'état reçu en *prop* directement mais par l'intermédiaire d'une fonction**.

 
#### Modification du composant AddTodo

Le composant `AddTodo` a en état local la valeur du champ avec une double liaison :

- lorsque nous modifions l'état avec `setValue()`, la valeur du champ est modifié grâce à la liaison `value={value}`.
- lorsque l'utilisateur modifie la valeur du champ, l'état local est modifié grâce à `onChange={handleChange}`.

Notez bien que le composant reçoit en *prop* la fonction `addTodo()` et qu'il l'invoque lorsque nous voulons ajouter une tâche, et donc modifier l'état du composant parent.

```jsx
import { useState } from 'react';

export default function AddTodo({ addTodo }) {
  const [value, setValue] = useState('');

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === 'Enter' && value.length) {
      addTodo(value);
      setValue('');
    }
  }

  function handleClick() {
    if (value.length) {
      addTodo(value);
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
      <button onClick={handleClick} className="btn btn-primary">
        Ajouter
      </button>
    </div>
  );
}
```
 
#### Modification du partial _themes.scss

```scss
input {
  border-radius: 4px;
  border: 1px solid var(--gray-2);
  padding: 10px 15px;
}
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
## Implémentation de l'édition

### Modification de `App.jsx`

Nous ajoutons une fonction pour éditer une tâche, elle change le contenu de la tâche et passe le mode édition à *false* :

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
import { useState } from 'react';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';

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

Nous modifions le composant `TodoList` simplement pour passer en *prop* plus bas dans l'arbre la fonction permettant de sélectionner une tâche :

```jsx
import TodoItem from './TodoItem';
import EditTodo from './EditTodo';

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
        todo.selected ? 'selected' : ''
      }  `}
    >
      <span className="flex-fill">
        {todo.content} {todo.done && '✅'}
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

*React* met **dans sa queue de mise à jour,** pour le prochain rendu la nouvelle valeur de notre tâche, par exemple :

```jsx
{
  content: "1",
  done: true,
  edit: false,
  id: "b702f80a-2d23-4f74-8388-5eebf681ffd4",
  selected: false
}
``` 

- ensuite l'événement se propage et remonte sur le *DOM*
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

*React* met **dans sa queue de mise à jour,** pour le prochain rendu la nouvelle valeur de notre tâche, par exemple :

```jsx
{
  content: "1",
  done: false,
  edit: false,
  id: "b702f80a-2d23-4f74-8388-5eebf681ffd4",
  selected: true
}
```

Remarquez bien que *done* est à *false* !

En effet, la valeur de la tâche pour ce rendu n'a pas changé ! Rappelez-vous que les mises à jour d'état se font au prochain rendu donc dans la liste nous avons toujours l'ancienne tâche et donc *done* est à *false* !

Au prochain rendu, *React* exécute sa queue de mise à jour et prend la dernière mise à jour de la tâche et nous obtenons donc :

```jsx
{
  content: "1",
  done: false,
  edit: false,
  id: "b702f80a-2d23-4f74-8388-5eebf681ffd4",
  selected: true
}
```

C'est assez complexe à comprendre au début mais c'est vraiment essentiel à comprendre pour maîtriser *React*, aussi n'hésitez pas à relire 2 / 3 fois les explications.

 
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

## A quoi sert *Context* ?

Nous avons vu jusqu'à maintenant que pour passer des informations d'un composant parent vers un composant enfant il suffisait d'utiliser des *props*.

Mais passer des *props* peut devenir très verbeux et pénible s'il faut les passer très loin en bas de l'arbre des composants ou si plusieurs branches ont besoin des mêmes *props*.

Par exemple, nous avons besoin d'une information dans tous les composants en bleu. Nous devons donc passer en *prop* cette information 8 fois :

![Diagram with a tree of ten nodes, each node with two children or less. The root node contains a bubble representing a value highlighted in purple. The value flows down through the two children, each of which pass the value but do not contain it. The left child passes the value down to two children which are both highlighted purple. The right child of the root passes the value through to one of its two children - the right one, which is highlighted purple. That child passed the value through its single child, which passes it down to both of its two children, which are highlighted purple.](/00-assets/images/React/passing_data_prop_drilling.webp)

***Context* permet de rendre des informations disponibles dans tous les composants enfant d'un composant même s'ils sont très éloignés.**

![Diagram with a tree of ten nodes, each node with two children or less. The root parent node contains a bubble representing a value highlighted in orange. The value projects down directly to four leaves and one intermediate component in the tree, which are all highlighted in orange. None of the other intermediate components are highlighted.](/00-assets/images/React/passing_data_context_far.webp)


### Utilisation du *Context*

#### 1 - Créer le *Context*

Pour créer un *Context*, on utilise un fichier pour le déclarer avec `createContext()` et on l'exporte.

**On passe la valeur que l'on souhaite partager en argument.**

Par convention le nom du fichier est suffixé par *Context*, par exemple `ExempleContext.js` :

```jsx
import { createContext } from 'react';

export const ExempleContext = createContext(42);
```

#### 2 - Utiliser le *Context*

Pour ensuite l'utiliser dans n'importe quel composant il suffit de l'importer et d'utiliser le *hook* `useContext()` :

```jsx
import { useContext } from 'react';
import { ExempleContext } from './ExempleContext.js';

export default function UnComposant() {
  const data = useContext(ExempleContext);
}
```

**Comme tous les *hooks* il est obligatoire d'utiliser `useContext()` au premier niveau d'un composant (ni imbriqué dans un bloc conditionnel, ni dans une fonction).**

#### 3 - Fournir un autre *Context*

Pour fournir (*provide*) un *Context* il faut l’importer et utiliser le composant `<NomContext.provider value={valeur}>` :

```jsx
import ExempleContext from './ExempleContext';
import UnComposantEnfant from './UnComposantEnfant';

export default function UnComposantParnet() {
  return (
    <ExempleContext.Provider value={100}>
      <UnComposantEnfant />
    </ExempleContext.Provider>
  );
}
```

**Cela permet de changer la valeur fournie pour tous les composants situés plus bas dans l'arbre.**

*Pour comprendre le fonctionnement du **Context** vous pouvez faire le parallèle avec l'héritage des propriétés **CSS**. En **CSS**, quand vous appliquez une propriété sur un élément, par exemple `color: blue` sur une `<div>`, tous les éléments en-dessous seront bleu sauf si vous écrasez cette propriété pour un autre élément plus bas sur le **DOM**.*

**Le *Context* n'est pas forcément une valeur statique vous pouvez très bien le lier à une valeur dynamique, par exemple un état :**

```jsx
import ExempleContext from './ExempleContext';
import UnComposantEnfant from './UnComposantEnfant';
import { useState } from 'react';

export default function UnComposantParnet() {
  const [valeur, setValeur] = useState(42);

  return (
    <ExempleContext.Provider value={valeur}>
      <UnComposantEnfant />
    </ExempleContext.Provider>
  );
}
```
 
### Quand utiliser le *Context* ?

Il ne faut pas abuser du *Context*, en *React* il n'est pas rare que des dizaines de *props* soient passées le long de l'arbre des composants sur une application complexe. Ce n'est pas une raison pour les remplacer par *Context*.

**En effet, avec les *props* il est facile de voir d'où elles viennent et où elles vont. Elles sont donc plus facilement lisibles et maintenables.**

**Voici des exemples d'usages recommandés du *Context* par l'équipe *React* :**

- **Les thèmes :** si votre application a deux ou plusieurs thèmes (par exemple un mode sombre), vous pouvez utiliser un *Context* pour indiquer le mode choisi aux composants qui ont besoin d'être adapté suivant le mode.
- **L'utilisateur connecté :** de nombreux composants partout dans l'application ont souvent besoin d'avoir des informations sur l'utilisateur connecté. Cela permet de rendre disponible ces informations partout dans l'application.
- ***Routing* :** la plupart des solutions de *React* pour le routing utilise en fait le *Context*. Nous verrons cela dans la suite de la formation.
- **Gérer l'état global :** nous verrons que dans les applications importantes il peut être efficace de gérer l'état en combinant *Context* et le *hook* `useReducer()`.

