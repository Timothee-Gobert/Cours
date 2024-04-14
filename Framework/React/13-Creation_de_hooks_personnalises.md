# Création de hooks personnalisés

## Qu'est ce qu'un hook personnalisé

### Les _hooks_ personnalisés

Les _hooks_ personnalisés permettent d’extraire la logique d’un composant sous forme de fonctions réutilisables.

**Ce sont des fonctions dont le nom commencent par _use_ et qui appellent d'autres _hooks_.**

**Ils permettent la composition de _hooks_.** La composition de fonctions est un paradigme puissant de la programmation fonctionnelle qui permet de combiner des fonctions simples pour obtenir des comportements plus complexes.

### Règles pour les _hooks_ personnalisés

Les règles des _hooks_ s'appliquent également pour les _hooks_ personnalisés :

- Pas d'imbrication dans des boucles, des conditions ou des fonctions imbriquées.
- Appeler uniquement les _hooks_ au plus haut niveau des composants ou dans un _hook_ personnalisé.

## Extraction de logique avec un hook

### Création de `useTrackMouse`

Nous repartons d'une application vierge.

Créez un dossier **hooks** dans **src**.

Dans ce dossier, créez le fichier _useTrackMouse.js_ :

```ts
import { useEffect, useState } from "react";

export function useTrackMouse() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function update(e) {
      setPosition({ x: e.pageX, y: e.pageY });
    }
    window.addEventListener("mousemove", update);
    return () => window.removeEventListener("mousemove", update);
  }, []);

  return position;
}
```

Notez que nous utilisons un tableau vide comme dépendance de l'effet pour ne déclarer l'écouteur d'événement qu'une seule fois et pas à chaque rendu.

Nous n'oublions pas de nettoyer l'effet en retournant une fonction de nettoyage qui détruit le gestionnaire d'événement.

Nous retournons la position de la souris à chaque fois qu'elle est modifiée.

### Modification de _App.js_

Nous utilisons simplement le hook personnalisé que nous avons créé :

```ts
import { useTrackMouse } from "./hooks/useTrackMouse";

function App() {
  const position = useTrackMouse();

  return (
    <div
      className="d-flex justify-content-center align-items-center"
      style={{ backgroundColor: "#fefefe", height: "100vh", width: "100%" }}
    >
      <h1>
        {position.x} : {position.y}
      </h1>
    </div>
  );
}

export default App;
```

## Utilisation d'un hook avec fetch

### Création de `useFetchRecipes`

Dans le dossier **hooks**, créez le fichier _useFetchRecipes.js_ :

```ts
import { useState, useEffect } from "react";

export function useFetchRecipes() {
  const [recipes, setRecipes] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchData() {
      try {
        const response = await fetch("https://restapi.fr/api/recipes");
        if (response.ok) {
          const fetchedData = await response.json();
          setRecipes(Array.isArray(fetchedData) ? fetchedData : [fetchedData]);
        } else {
          setError("Error");
        }
      } catch (e) {
        setError("Error");
      } finally {
        setIsLoading(false);
      }
    }
    fetchData();
  }, []);

  return { recipes, isLoading, error };
}
```

Nous créons un _hook_ personnalisé pour notre requête.

### Modification de _App.js_

Nous utilisons ensuite le hook personnalisé pour notre requête.

```ts
import { useFetchRecipes } from "./hooks/useFetchRecipes";

function App() {
  const { recipes, isLoading, error } = useFetchRecipes();

  return (
    <div
      className="d-flex flex-row justify-content-center align-items-center"
      style={{ backgroundColor: "#fefefe", height: "100vh", width: "100%" }}
    >
      {isLoading ? (
        <p>Chargement ....</p>
      ) : (
        <ul>
          {recipes.map((r) => (
            <li key={r._id}>{r.title}</li>
          ))}
        </ul>
      )}
      {error && <p>{error}</p>}
    </div>
  );
}

export default App;
```

## Rendre un hook générique pour le réutiliser

### Modification de `useFetchRecipes`

Nous renommons le fichier _useFetchRecipes.js_ en _useFetchData.js_.

Nous rendons notre _hook_ plus générique pour pouvoir le réutiliser si besoin dans d'autres composants.

```ts
import { useState, useEffect } from "react";

export function useFetchData(url) {
  const [data, setData] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let ignore = false;
    async function fetchData() {
      try {
        const response = await fetch(url);
        if (response.ok) {
          if (!ignore) {
            const fetchedData = await response.json();
            setData(Array.isArray(fetchedData) ? fetchedData : [fetchedData]);
          }
        } else {
          setError("Error");
        }
      } catch (e) {
        setError("Error");
      } finally {
        if (!ignore) {
          setIsLoading(false);
        }
      }
    }
    fetchData();
    return () => {
      ignore = true;
    };
  }, []);

  return {
    data,
    isLoading,
    error,
  };
}
```

Nous créons un _hook_ personnalisé pour notre requête.

### Modification de _App.js_

Nous utilisons maintenant notre _hook_ modifié :

```ts
import React from "react";
import { useFetchData } from "./hooks/useFetchData";

function App() {
  const {
    data: recipes,
    isLoading,
    error,
  } = useFetchData("https://restapi.fr/api/recipes");

  return (
    <div
      className="d-flex flex-row justify-content-center align-items-center"
      style={{ backgroundColor: "#fefefe", height: "100vh", width: "100%" }}
    >
      {isLoading ? (
        <p>Chargement ....</p>
      ) : (
        <ul>
          {recipes.map((r) => (
            <li key={r._id}>{r.title}</li>
          ))}
        </ul>
      )}
      {error && <p>{error}</p>}
    </div>
  );
}

export default App;
```

[Code sur Github](https://github.com/dymafr/react-chapitre13-custom-hook)
