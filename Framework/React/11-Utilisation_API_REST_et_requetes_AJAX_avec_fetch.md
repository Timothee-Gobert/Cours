# Utilisation API REST et requêtes AJAX avec fetch

## Introduction aux requêtes AJAX

### Rappels sur _AJAX_

L’_AJAX_ est une technologie permettant de mettre à jour simplement des parties du _DOM_ d'une page _HTML_ au lieu de devoir recharger la page entière.

_AJAX_ permet également d’exécuter du code de manière asynchrone, c'est-à-dire que votre code continue à s'exécuter pendant qu’une ou plusieurs requêtes sont en cours.

_Une bonne connaissance de **async** / **await**, des **promesses** en général et de l'**API fetch** est nécessaire pour suivre ce chapitre._

### Rappels sur _fetch_

_fetch_ est une _Web API_ disponible dans tous les navigateurs qui permet d'envoyer des requêtes _HTTP_.

La syntaxe de l’_API_ est très simple :

```ts
const promesse = fetch(url, [options]);
```

Le premier paramètre est l’_URL_ cible de la requête.

Le second paramètre est un objet d’options que nous étudierons en détails.

La _Web API_ retourne une promesse qui sera tenue si le serveur répond.

La promesse est résolue avec un objet _Response_.

A ce stade, vous pouvez accéder aux propriétés suivantes :

- **url :** url de la requête.
- **redirected :** booléen indiquant si la requête a été redirigée par le serveur.
- **status :** code du statut de la requête.
- **ok :** booléen pour savoir si la requête s’est bien déroulée (_true_ si le code du statut est compris entre _200_ et _299_).
- **type :** type de la requête cors ou basic (nous y reviendrons).
- **statusText :** message du statut de la requête.
- **headers :** headers de la réponse. Il faut utiliser la méthode `get()` et passer le nom du _header_ à récupérer.

### Le service _restapi.fr_

Nous allons utiliser un service de Dyma pour le backend qui permet notamment de sauvegarder des données facilement en utilisant un véritable service _REST_.

Ce service est *www.restapi.fr* et il permet de sauvegarder et de manipuler des données facilement. Il permet également de générer des données suivant un modèle que vous définissez.

## Configuration d'une requête POST

### Modification du composant _App.js_

Dans le composant _App.js_, nous ne créons plus la tâche car nous la créerons dans le composant `AddTodo` :

```ts
function App() {
  const [todoList, setTodoList] = useState([]);

  function addTodo(todo) {
    setTodoList([...todoList, todo]);
  }
// ...
```

### Modification du composant _AddTodo.js_

Dans le composant _AddTodo.js_, nous faisons la requête _HTTP_ de type _POST_ pour ajouter la tâche dans notre base de données :

```ts
import { useState } from "react";

export default function AddTodo({ addTodo }) {
  const [value, setValue] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === "Enter" && value.length) {
      createTodo();
    }
  }

  async function createTodo() {
    try {
      setLoading(true);
      setError(null);
      const reponse = await fetch("https://restapi.fr/api/todo", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          content: value,
          edit: false,
          done: false,
        }),
      });
      if (reponse.ok) {
        const todo = await reponse.json();
        addTodo(todo);
        setValue("");
      } else {
        setError("Oops, une erreur");
      }
    } catch (e) {
      setError("Oops, une erreur");
    } finally {
      setLoading(false);
    }
  }

  function handleClick() {
    if (value.length) {
      createTodo();
    }
  }

  return (
    <>
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
          {loading ? "Chargement" : "Ajouter"}
        </button>
      </div>
      {error && <p style={{ color: "red" }}>{error}</p>}
    </>
  );
}
```

Nous créons deux variables d'état pour afficher l'état de chargement pendant la requête et une éventuelle erreur.

### Remplacement de la propriété _id_ par _\_id_

La base de données _MongoDB_ utilisée par _RestAPI_ retourne en effet une propriété _\_id_ sur les documents créés. Remplacez donc partout dans l'application les propriétés _id_ par _\_id_.

Par exemple dans le composant _TodoList_ :

```ts
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
            key={todo._id}
            todo={todo}
            editTodo={(content) => editTodo(todo._id, content)}
            cancelEditTodo={() => toggleTodoEdit(todo._id)}
          />
        ) : (
          <TodoItem
            key={todo._id}
            todo={todo}
            deleteTodo={() => deleteTodo(todo._id)}
            toggleTodo={() => toggleTodo(todo._id)}
            editTodo={() => toggleTodoEdit(todo._id)}
            selectTodo={() => selectTodo(todo._id)}
          />
        )
      )}
    </ul>
  ) : (
    <p>Aucune tâche en cours </p>
  );
}
```

## Configuration d'une requête GET et useEffect

### Modification du composant _App.js_

Pour le chargement initial des tâches, étant donné que c'est le rendu du composant racine qui doit déclencher la requête, nous devons utiliser un effet :

```ts
import React, { useState, useEffect } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

function App() {
  const [todoList, setTodoList] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let ignore = false;
    async function fetchTodoList() {
      try {
        const reponse = await fetch("https://restapi.fr/api/todo");
        if (reponse.ok) {
          const todos = await reponse.json();
          if (!ignore) {
            if (Array.isArray(todos)) {
              setTodoList(todos);
            } else {
              setTodoList([todos]);
            }
          }
        } else {
          console.error("Oops, une erreur");
        }
      } catch (e) {
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }
    fetchTodoList();
    return () => {
      ignore = true;
    };
  }, []);

  function addTodo(todo) {
    setTodoList([...todoList, todo]);
  }

  function deleteTodo(id) {
    setTodoList(todoList.filter((todo) => todo._id !== id));
  }

  function toggleTodo(id) {
    setTodoList(
      todoList.map((todo) =>
        todo._id === id ? { ...todo, done: !todo.done } : todo
      )
    );
  }

  function toggleTodoEdit(id) {
    setTodoList(
      todoList.map((todo) =>
        todo._id === id ? { ...todo, edit: !todo.edit } : todo
      )
    );
  }

  function editTodo(id, content) {
    setTodoList(
      todoList.map((todo) =>
        todo._id === id ? { ...todo, edit: false, content } : todo
      )
    );
  }

  function selectTodo(id) {
    setTodoList(
      todoList.map((todo) =>
        todo._id === id
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
        {loading ? (
          <p>Chargement en cours...</p>
        ) : (
          <TodoList
            todoList={todoList}
            deleteTodo={deleteTodo}
            toggleTodo={toggleTodo}
            toggleTodoEdit={toggleTodoEdit}
            editTodo={editTodo}
            selectTodo={selectTodo}
          />
        )}
      </div>
    </div>
  );
}

export default App;
```

Notez la variable _ignore_ qui est utile pour ne pas rendre deux fois le composant s'il y a deux requêtes successives.

Le mode développement vous permet de voir facilement le nombre de rendus en cas de chargement multiple du composant. Ici il n'y a aucune chance qu'il soit rendu deux fois car c'est le composant racine.

Mais c'est une bonne pratique d'ajouter une variable _ignore_ pour éviter les _race conditions_ et les rendus inutiles supplémentaires.

Notez que sans le _StrictMode_, le composant racine est rendu deux fois : c'est normal, une première fois avant l'effet et une fois après l'effet avec un traitement par lot pour définir la liste de tâches et définir le chargement comme terminé.

## Configuration d'une requête PATCH

### Modification de _App.js_

Nous pouvons simplifier en remplaçant la plupart des fonctions par une fonction générique de mise à jour d'une tâche :

```ts
import { useState, useEffect } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

function App() {
  const [todoList, setTodoList] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let ignore = false;
    async function fetchTodoList() {
      try {
        const reponse = await fetch("https://restapi.fr/api/todo");
        if (reponse.ok) {
          const todos = await reponse.json();
          if (!ignore) {
            if (Array.isArray(todos)) {
              setTodoList(todos);
            } else {
              setTodoList([todos]);
            }
          }
        } else {
          console.error("Oops, une erreur");
        }
      } catch (e) {
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }
    fetchTodoList();
    return () => {
      ignore = true;
    };
  }, []);

  function addTodo(todo) {
    setTodoList([...todoList, todo]);
  }

  function deleteTodo(id) {
    setTodoList(todoList.filter((todo) => todo._id !== id));
  }

  function updateTodo(newTodo) {
    setTodoList(
      todoList.map((todo) => (todo._id === newTodo._id ? newTodo : todo))
    );
  }

  return (
    <div className="d-flex justify-content-center align-items-center p-20">
      <div className="card container p-20">
        <h1 className="mb-20">Liste de tâches</h1>
        <AddTodo addTodo={addTodo} />
        {loading ? (
          <p>Chargement en cours...</p>
        ) : (
          <TodoList
            todoList={todoList}
            deleteTodo={deleteTodo}
            updateTodo={updateTodo}
          />
        )}
      </div>
    </div>
  );
}

export default App;
```

Notez que la suppression ne fonctionnera plus jusqu'à la prochaine leçon.

### Modification de _TodoList.js_

Nous devons adapter ce composant car il reçoit maintenant une fonction _updateTodo_ en _prop_ :

```ts
import TodoItem from "./TodoItem";
import EditTodo from "./EditTodo";

export default function TodoList({ todoList, deleteTodo, updateTodo }) {
  return todoList.length ? (
    <ul>
      {todoList.map((todo) =>
        todo.edit ? (
          <EditTodo key={todo._id} todo={todo} updateTodo={updateTodo} />
        ) : (
          <TodoItem key={todo._id} todo={todo} updateTodo={updateTodo} />
        )
      )}
    </ul>
  ) : (
    <p>Aucune tâche en cours </p>
  );
}
```

### Modification de _TodoItem.js_

Ce composant reçoit maintenant en prop une fonction _updateTodo_, nous devons donc l'adapter.

Il réalise la requête de mise à jour de la tâche, c'est donc une requête de type _PATCH_.

```ts
import React, { useState } from "react";

export default function TodoItem({ todo, deleteTodo, updateTodo }) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  async function tryUpdateTodo(newTodo) {
    try {
      setLoading(true);
      setError(null);
      const { _id, ...update } = newTodo;
      const reponse = await fetch(`https://restapi.fr/api/todo/${todo._id}`, {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(update),
      });
      if (reponse.ok) {
        const newTodo = await reponse.json();
        updateTodo(newTodo);
      } else {
        setError("Oops, une erreur");
      }
    } catch (e) {
      setError("Oops, une erreur");
    } finally {
      setLoading(false);
    }
  }

  return (
    <li
      className={
        "mb-10 d-flex flex-row justify-content-center align-items-center p-10"
      }
    >
      {loading ? (
        <span className="flex-fill">Chargement...</span>
      ) : (
        <span className="flex-fill">
          {todo.content} {todo.done && "✅"}
        </span>
      )}
      <button
        className="btn btn-primary mr-15"
        onClick={(e) => {
          e.stopPropagation();
          tryUpdateTodo({ ...todo, done: !todo.done });
        }}
      >
        Valider
      </button>
      <button
        className="btn btn-primary mr-15"
        onClick={(e) => {
          e.stopPropagation();
          tryUpdateTodo({ ...todo, edit: true });
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

### Modification de _EditTodo.js_

Même chose pour ce composant :

```ts
import { useState } from "react";

export default function EditTodo({ todo, updateTodo, cancelEditTodo }) {
  const [value, setValue] = useState(todo.content);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  async function tryUpdateTodo(newTodo) {
    try {
      setLoading(true);
      setError(null);
      const { _id, ...update } = newTodo;
      const reponse = await fetch(`https://restapi.fr/api/todo/${todo._id}`, {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(update),
      });
      if (reponse.ok) {
        const newTodo = await reponse.json();
        updateTodo(newTodo);
      } else {
        setError("Oops, une erreur");
      }
    } catch (e) {
      setError("Oops, une erreur");
    } finally {
      setLoading(false);
    }
  }

  function handleChange(e) {
    const inputValue = e.target.value;
    setValue(inputValue);
  }

  function handleKeyDown(e) {
    if (e.key === "Enter" && value.length) {
      tryUpdateTodo({ ...todo, content: value, edit: false });
      setValue("");
    }
  }

  function handleClick() {
    if (value.length) {
      tryUpdateTodo({ ...todo, content: value, edit: false });
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
      <button onClick={handleClick} className="btn btn-primary mr-15">
        Sauvegarder
      </button>
      <button
        onClick={() => tryUpdateTodo({ ...todo, edit: false })}
        className="btn btn-reverse-primary"
      >
        Annuler
      </button>
    </div>
  );
}
```

## Configuration d'une requête DELETE et useReducer

### Modification de _App.js_

Nous utilisons le _hook_ `useReducer` pour l'état de notre liste de tâches au lieu d'un `useState` :

```ts
import { useReducer, useEffect, useState } from "react";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

function todoReducer(state, action) {
  switch (action.type) {
    case "FETCH_TODO": {
      return {
        ...state,
        todoList: action.todoList,
      };
    }
    case "ADD_TODO": {
      return {
        ...state,
        todoList: [...state.todoList, action.todo],
      };
    }
    case "UPDATE_TODO": {
      return {
        ...state,
        todoList: state.todoList.map((t) =>
          t._id === action.todo._id ? action.todo : t
        ),
      };
    }
    case "DELETE_TODO": {
      return {
        ...state,
        todoList: state.todoList.filter((t) => t._id !== action.todo._id),
      };
    }
    default: {
      throw new Error("Action inconnue");
    }
  }
}

function App() {
  const [state, dispatch] = useReducer(todoReducer, { todoList: [] });
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    let ignore = false;
    async function fetchTodoList() {
      try {
        const reponse = await fetch("https://restapi.fr/api/todo");
        if (reponse.ok) {
          const todos = await reponse.json();
          if (!ignore) {
            if (Array.isArray(todos)) {
              dispatch({ type: "FETCH_TODO", todoList: todos });
            } else {
              dispatch({ type: "FETCH_TODO", todoList: [todos] });
            }
          }
        } else {
          console.error("Oops, une erreur");
        }
      } catch (e) {
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }
    fetchTodoList();
    return () => {
      ignore = true;
    };
  }, []);

  function addTodo(newTodo) {
    dispatch({ type: "ADD_TODO", todo: newTodo });
  }

  function deleteTodo(deletedTodo) {
    dispatch({ type: "DELETE_TODO", todo: deletedTodo });
  }

  function updateTodo(updatedTodo) {
    dispatch({ type: "UPDATE_TODO", todo: updatedTodo });
  }

  return (
    <div className="d-flex justify-content-center align-items-center p-20">
      <div className="card container p-20">
        <h1 className="mb-20">Liste de tâches</h1>
        <AddTodo addTodo={addTodo} />
        {loading ? (
          <p>Chargement en cours...</p>
        ) : (
          <TodoList
            todoList={state.todoList}
            deleteTodo={deleteTodo}
            updateTodo={updateTodo}
          />
        )}
      </div>
    </div>
  );
}

export default App;
```

### Modification de _TodoList.js_

Nous modifions ce composant simplement pour passer _deleteTodo_ en _prop_ à _TodoItem_ :

```ts
import TodoItem from "./TodoItem";
import EditTodo from "./EditTodo";

export default function TodoList({ todoList, deleteTodo, updateTodo }) {
  return todoList.length ? (
    <ul>
      {todoList.map((todo) =>
        todo.edit ? (
          <EditTodo key={todo._id} todo={todo} updateTodo={updateTodo} />
        ) : (
          <TodoItem
            key={todo._id}
            todo={todo}
            updateTodo={updateTodo}
            deleteTodo={deleteTodo}
          />
        )
      )}
    </ul>
  ) : (
    <p>Aucune tâche en cours </p>
  );
}
```

### Modification de _TodoItem.js_

Nous effectuons une requête pour la suppression d'une tâche :

```ts
import { useState } from "react";

export default function TodoItem({ todo, deleteTodo, updateTodo }) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  async function tryUpdateTodo(newTodo) {
    try {
      setLoading(true);
      setError(null);
      const { _id, ...update } = newTodo;
      const reponse = await fetch(`https://restapi.fr/api/todo/${todo._id}`, {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(update),
      });
      if (reponse.ok) {
        const newTodo = await reponse.json();
        updateTodo(newTodo);
      } else {
        setError("Oops, une erreur");
      }
    } catch (e) {
      setError("Oops, une erreur");
    } finally {
      setLoading(false);
    }
  }

  async function handleDeleteTodo() {
    try {
      setLoading(true);
      setError(null);
      const reponse = await fetch(`https://restapi.fr/api/todo/${todo._id}`, {
        method: "DELETE",
      });
      if (reponse.ok) {
        deleteTodo(todo);
      } else {
        setError("Oops, une erreur");
      }
    } catch (e) {
      setError("Oops, une erreur");
    } finally {
      setLoading(false);
    }
  }

  return (
    <li
      className={
        "mb-10 d-flex flex-row justify-content-center align-items-center p-10"
      }
    >
      {loading ? (
        <span className="flex-fill">Chargement...</span>
      ) : (
        <span className="flex-fill">
          {todo.content} {todo.done && "✅"}
        </span>
      )}
      <button
        className="btn btn-primary mr-15"
        onClick={(e) => {
          e.stopPropagation();
          tryUpdateTodo({ ...todo, done: !todo.done });
        }}
      >
        Valider
      </button>
      <button
        className="btn btn-primary mr-15"
        onClick={(e) => {
          e.stopPropagation();
          tryUpdateTodo({ ...todo, edit: true });
        }}
      >
        Modifier
      </button>
      <button
        className="btn btn-reverse-primary"
        onClick={(e) => {
          e.stopPropagation();
          handleDeleteTodo();
        }}
      >
        Supprimer
      </button>
    </li>
  );
}
```

[code](https://github.com/dymafr/react-chapitre11-http-effect)
