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
