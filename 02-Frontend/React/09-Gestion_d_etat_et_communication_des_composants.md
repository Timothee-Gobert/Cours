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
