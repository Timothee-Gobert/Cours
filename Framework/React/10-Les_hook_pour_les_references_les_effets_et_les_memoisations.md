# Les hooks pour les références, les effets et les mémoïsations

## Introduction aux hooks useRef(), useEffect(), useMemo() et useCallback()

### Le _hook_ `useRef()`

**`useRef()` est un _hook_ natif permettant de référencer une valeur modifiable qui n'est pas utilisée pour le rendu.**

**Autrement dit, `useRef()` permet de créer une variable contenant une valeur disponible entre les différents rendus d'un composant et que la modification de cette valeur ne déclenche pas un nouveau rendu.**

Nous verrons plus en détail dans le chapitre que `useRef()` est utile :

- pour référencer une valeur dont nous n'avons pas besoin pour le rendu du composant (contrairement à `useState()` et `useReducer()` donc) comme par exemple un id de timeout ou d'interval.
- pour manipuler le _DOM_ directement en utilisant une référence.
- pour éviter de recréer des contenus inutilement.

### Le _hook_ `useEffect()`

**`useEffect()` est un _hook_ natif permettant aux composants de se synchroniser avec des systèmes externes en permettant d'exécuter du code après le rendu d'un composant.**

**Pour rappel, le corps d'un composant doit être pur car _React_ a absolument besoin que le rendu d'un composant soit pur.**

**Les effets permettent de créer des effets de bord qui sont causés par le rendu lui-même et non par des interactions utilisateurs.**

Nous verrons plus en détail dans le chapitre que `useEffect()` est utile :

- pour contrôler un composant non géré par _React_ (provenant par exemple d'une librairie tierce).
- pour effectuer une connexion à un serveur (par exemple une connexion _Websocket_ à un chat).
- pour effectuer des requêtes _HTTP_ de type _Ajax_ avec `fetch()` afin de récupérer des données.

### Le _hook_ `useMemo()`

`useMemo()` est un _hook_ natif permettant de mettre en cache une valeur pour gagner en performance (on appelle cela une valeur mémoïsée).

Nous verrons que ce hook est utilisé uniquement pour améliorer les performances dans certains cas.

### Le _hook_ `useCallback()`

**`useCallback()` est un _hook_ natif permettant de mettre en cache une fonction et de conserver la même référence à cette fonction entre les rendus (_on appelle cela une fonction de rappel mémoïsée_).**

Nous verrons que ce _hook_ est utilisé uniquement pour améliorer les performances et éviter des rendus inutiles lorsque l'on passe des fonctions de rappel en _prop_ à des composants enfants.

## Présentation du hook useRef()

### Introduction au _hook_ `useRef()`

**`useRef()` est un _hook_ natif permettant de référencer une valeur modifiable qui n'est pas utilisée pour le rendu.**

Autrement dit, `useRef()` permet de créer une variable contenant une valeur disponible entre les différents rendus d'un composant et que **la modification de cette valeur ne déclenche pas un nouveau rendu**.

Comme pour les autres _hooks_ il faut déclarer `useRef()` au premier niveau des composants.

**La syntaxe de `useRef()` est :**

```ts
import { useRef } from "react";

const ref = useRef(valeurInitiale);
```

**Le _hook_ retourne un objet _ref_ qui contient une unique propriété _current_ :**

```ts
{
  current: valeurInitiale;
}
```

**Cette propriété _current_ contient la valeur initiale passée en argument.**

Pour les rendus suivants, `useRef()` va retourner le même objet. Vous pouvez donc voir l'objet _ref_ comme un espace mémoire en dehors du composant et qui n'est pas utilisé pour la gestion des rendus.

### A retenir sur le _hook_ `useRef()`

Voici les trois choses à retenir absolument pour l'utilisation de ce _hook_ :

- il est utile pour **stocker de l'information entre les rendus** dans un objet ref.
- **l'information contenu dans l'objet _ref_ ne doit pas être modifiée ou lu durant un rendu. _Il faut y accéder dans un gestionnaire d'événement_ ou _dans le hook `useEffect()`._**
- changer la valeur de la propriété _current_ de l'objet _ref_ **ne déclenche pas de nouveau rendu.**
- l'information est locale à chaque instance du composant.

### Premier exemple

Voici un exemple simple :

```ts
import { useRef } from "react";

export default function Compteur() {
  const ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert(`Vous avez cliqué ${ref.current} fois !`);
  }

  return <button onClick={handleClick}>Cliquez</button>;
}
```

Notez bien que nous accédons à la valeur uniquement dans un gestionnaire d'événement et non directement dans le corps de la fonction du composant qui doit rester pure !

Ici il y a un seul rendu initial (deux en mode strict) et plus aucun rendu pour chaque clic !

### Deuxième exemple

Voici un exemple utilisant également `useState()` :

```ts
import { useState, useRef } from "react";

export default function Chrono() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime !== null && now !== null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Temps passé : {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
    </>
  );
}
```

Le composant a un gestionnaire d'événement `handleStart()` qui va enregistrer dans l'état le temps de départ et le temps courant (qui sont les mêmes au départ). Il va démarrer un interval qui va définir le temps courant toutes les 10 millisecondes. Il va supprimer l'interval en cours s'il y en a un grâce à `clearInterval()`.

Le gestionnaire d'événement `handleStop()` va simplement stopper l'interval en cours qui met à jour _now_ toutes les 10 millisecondes.

Dans le corps de la fonction, on calcul le temps passé entre le temps initial (_startTime_) et maintenant (_now_) et on le convertit en secondes.

Pour le rendu, on affiche ce nombre en conservant 3 décimales.

Notez bien que _startTime_ et _now_ sont utilisés pour le rendu : ils permettent de calculer le nombre de secondes passées toutes les _10ms_. **Ils doivent donc être obligatoirement dans l'état et non dans une référence.**

En revanche, **l'id retourné par le `setInterval()` doit être fixe entre les rendus, et n'est pas utilisé pour les rendus.** Il doit donc être mis dans une référence.

### Comment fonctionne `useRef()` ?

En fait, `useRef()` est une version simplifiée de `useState()` vu qu'il n'y a pas de fonction _setter_.

On peut implémenter `useRef()` de cette manière :

```ts
function useRef(valeurInitiale) {
  const [ref] = useState({ current: valeurInitiale });
  return ref;
}
```

Notez que comme il n'y a pas de fonction _setter_, une modification de _ref.current_ est immédiate et n'est pas du tout liée aux rendus.

## useRef() et le DOM

### Manipuler le _DOM_ avec des références

_React_ met automatiquement à jour le _DOM_ pour correspondre au _JSX_ retourné par vos composants. Donc la plupart du temps vous n'avez pas besoin de manipuler directement le _DOM_.

Cependant, dans quelques cas, vous devez obligatoirement manipuler le _DOM_, par exemple pour focus un élément _HTML_, défiler jusqu'à l'affichage d'un élément, ou encore pour mesurer la taille ou la position d'un élément.

**Dans ces cas, vous devrez utiliser `useRef()` et l'attribut _JSX_ _ref_.**

Pour obtenir la référence d'un élément du _DOM_ il faut utiliser cette syntaxe :

```ts
import { useRef } from 'react';

const maRef = useRef(null);

<div ref={maRef}>
```

**Notez bien l'attribut _ref_ que nous lions à l'objet _ref_ retourné par le _hook_ `useRef()`.**

Le nœud du _DOM_ sera placé dans la propriété _current_.

Une fois l'accès à un élément du _DOM_ effectué, vous pouvez le manipuler, par exemple :

```ts
maRef.current.scrollIntoView();

maRef.current.focus();
```

### Donner le focus à un élément du _DOM_

Voici un exemple simple pour donner le focus à un élément :

```ts
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus le champ</button>
    </>
  );
}
```

### Utiliser une fonction de rappel pour manipuler les listes d'éléments

Parfois, vous voudrez obtenir des références à tous les éléments d'une liste, par exemple pour pouvoir défiler jusqu'à eux.

**Dans ce cas, il est possible de passer une fonction de rappel à l'attribut _ref_.**

Prenons un exemple :

```ts
import { useRef } from "react";

export default function CatFriends() {
  const itemsRef = useRef(new Map());

  function scrollToId(id) {
    const node = itemsRef.current.get(id);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  return (
    <>
      <nav>
        <button className="btn btn-primary m-20" onClick={() => scrollToId(0)}>
          Chat 1
        </button>
        <button className="btn btn-primary mr-15" onClick={() => scrollToId(5)}>
          Chat 6
        </button>
        <button className="btn btn-primary mr-15" onClick={() => scrollToId(9)}>
          Chat 10
        </button>
      </nav>
      <div>
        <ul className="d-flex" style={{ overflowX: "scroll" }}>
          {catList.map((cat) => (
            <li
              style={{ minWidth: "300px" }}
              key={cat.id}
              ref={(node) => {
                itemsRef.current.set(cat.id, node);
              }}
            >
              <img src={cat.imageUrl} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: "https://placekitten.com/250/200?image=" + i,
  });
}
```

**Nous allons détailler :**

`const itemsRef = useRef(new Map())` : nous mettons un _Map_ dans notre référence. Rappelez-vous qu'une référence peut contenir tout type de valeur.

La fonction `scrollToId(id)` permet de récupérer l'élément du _Map_ qui a pour clé _id_ et d'utiliser la valeur (qui est le nœud _HTML_) pour défiler jusqu'à lui.

La partie qui nous intéresse le plus est :

```ts
<ul className="d-flex" style={{ overflowX: "scroll" }}>
  {catList.map((cat) => (
    <li
      style={{ minWidth: "300px" }}
      key={cat.id}
      ref={(node) => {
        itemsRef.current.set(cat.id, node);
      }}
    >
      <img src={cat.imageUrl} />
    </li>
  ))}
</ul>
```

Pour chaque élément de la liste de chats qui contient des objets du type `{id: 1, imageUrl: 'url'}`, nous créons un élément de liste.

Notez bien la fonction de rappel passée à l'attribut _JSX_ _ref_. Cette fonction reçoit en argument le nœud _HTML_ de l'itération en cours.

Nous utilisons ensuite notre _Map_ contenu dans _itemsRef.current_ pour définir en clé l'_id_ du chat et en valeur le nœud (qui est l'élément de liste courant).

Grâce à cela nous pouvons ensuite accéder au nœud H*T*ML de chaque élément en utilisant `itemsRef.current.get(id)`.

## useRef() et les composants

### Utiliser une _ref_ sur un composant enfant

Par défaut, il n'est pas possible d'utiliser une _ref_ passée en propriété sur un composant enfant.

```ts
import { useRef } from "react";

function MonChamp(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MonChamp ref={inputRef} />
      <button onClick={handleClick}>Focus le champ</button>
    </>
  );
}
```

Le code précédent ne fonctionnera pas et vous aurez l'erreur suivante dans la console :

> _Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?_

**La raison est que par défaut, React interdit aux composants d'accéder aux nœuds _HTML_ d'autres composants.**

Ce comportement est justifié car cela rend le code plus difficilement compréhensible.

**Si vous voulez qu'un composant expose son _DOM_ aux autres composants, il faut utiliser une _API_ spécifique : `forwardRef()` :**

```ts
const MonChamp = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

**Cette _API_ permet d'accéder à la _ref_ passée par le composant parent en deuxième argument.**

Il est fréquent d’utiliser ce comportement pour les composants de bas niveau de type bouton ou champs.

Par contre, il est fortement recommandé de ne pas le faire avec les composants de plus haut niveau comme les listes, les formulaires ou des sections de pages.

### Limiter l'usage d'une ref sur un composant enfant

_Il s'agit d'un usage très avancé, utilisé le plus souvent pour développer des librairies d'éléments. Vous pouvez passer rapidement pour le moment._

Lorsque vous passez une _ref_ à un composant enfant, le composant parent peut ensuite tout faire avec cette _ref_ : comme par exemple modifier les styles de l'élément _HTML_.

Il existe une _API_ pour limiter les interactions possibles, `useImperativeHandle()` :

```ts
import { forwardRef, useRef, useImperativeHandle } from "react";

const MonChamp = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MonChamp ref={inputRef} />
      <button onClick={handleClick}>Focus le champ</button>
    </>
  );
}
```

Ici _realInputRef_ dans le composant enfant, contient la véritable référence au champ sur le _DOM_.

_ref_ ne contient qu'un objet que vous passez avec des méthodes définies à `useImperativeHandle()`.

Autrement dit ici, _ref_ ne contient qu'un objet avec une méthode `focus()`. et non la référence à l'élément du _DOM_. Cela permet de contrôler totalement les interactions possibles.

## Présentation du hook useEffect()

### Le hook `useEffect()`

`useEffect()` est un _hook_ natif permettant aux composants de se synchroniser avec des systèmes externes en permettant d'exécuter du code **_après le rendu_** d'un composant.

**Pour rappel, le corps d'un composant doit être pur car _React_ a absolument besoin que le rendu d'un composant soit pur.**

**Les effets permettent de créer des effets de bord qui sont causés par le rendu lui-même et non par des interactions utilisateurs.**

Nous verrons plus en détail dans le chapitre que `useEffect()` est utile :

- pour contrôler un composant non géré par _React_ (provenant par exemple d'une librairie tierce).
- pour effectuer une connexion à un serveur (par exemple une connexion _Websocket_ à un chat).
- pour effectuer des requêtes _HTTP_ de type _Ajax_ avec `fetch()` afin de récupérer des données.

### Déclarer un effet

La syntaxe pour utiliser un _hook_ `useEffect()` est la suivante :

```ts
import { useEffect } from "react";

function MonComposant() {
  useEffect(() => {
    // Le code ici sera exécuté après chaque rendu
  });
  return <div />;
}
```

Par défaut, un effet est exécuté après chaque rendu.

### Premier exemple d'effet

L'exemple interactif est disponible à la fin de la leçon :

```ts
import { useState, useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return (
    <video ref={ref} src={src} loop playsInline style={{ maxWidth: "600px" }} />
  );
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button
        className="btn btn-primary m-10"
        onClick={() => setIsPlaying(!isPlaying)}
      >
        {isPlaying ? "Pause" : "Play"}
      </button>
      <div>
        <VideoPlayer
          isPlaying={isPlaying}
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
        />
      </div>
    </>
  );
}
```

L’élément _HTML_ _video_ n'est pas un composant _React_, nous ne pouvons donc pas synchroniser une _prop_ _isPlaying_ avec l'état de lecture / pause du lecteur vidéo.

Le seul moyen de le faire est de synchroniser une variable d'état _isPlaying_ et le déclenchement de la lecture et de la pause avec une référence sur l'élément _HTML_ du lecteur.

Que se passerait-il si nous n'utilisions pas d'effet ? Comme ceci :

```ts
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();
  } else {
    ref.current.pause();
  }
  return (
    <video ref={ref} src={src} loop playsInline style={{ maxWidth: "600px" }} />
  );
}
```

Vous aurez immédiatement un crash car vous essayez d'accéder à la **_ref_ avant le premier rendu**. Or comme nous l'avons vu, il est interdit d'accéder à une _ref_ avant ou pendant le rendu.

Avant le premier rendu il n'y a même pas de _DOM_ donc impossible d'accéder à un élément de celui-ci !

Nous avons donc l'obligation d'utiliser le _hook_ `useEffect()` afin d’accéder au _DOM_ **_après_** le rendu.

### Les dépendances des effets

Par défaut, un effet est exécuté après chaque rendu ce qui n'est souvent pas le comportement souhaité.

Un code comme cela crée une boucle infinie :

```ts
const [compteur, setCompteur] = useState(0);
useEffect(() => {
  setCompteur(compteur + 1);
});
```

La raison est que lors du premier rendu il ne se passe rien, après le premier rendu l'effet est exécuté et la fonction _setter_ déclenche un nouveau rendu.

Le nouveau rendu déclenche l'effet, qui déclenche un nouveau rendu... à l'infini !

**Il est possible de préciser qu'un effet ne doit être ré-exécuté que si ses dépendances changent.**

Les dépendances se déclarent dans un tableau passé en deuxième argument à `useEffect(fonction, dépendances)`.

Par exemple :

```ts
useEffect(() => {
  // ...
}, [dependance1, dependance2]);
```

Ici l'effet ne sera ré-exécuté entre les rendus que si _dependance1_ ou _dependance2_ changent.

Donc pour résumer :

```ts
useEffect(() => {
  // Exécuté après chaque rendu
});

useEffect(() => {
  // Exécuté qu'une seule fois après le rendu initial
}, []);

useEffect(() => {
  // Exécuté après le rendu initial et à chaque fois que a ou b change
}, [a, b]);
```

**Notez que les références et les fonctions _setter_ n'ont pas besoin d'être déclarées comme dépendance car _React_ garantie l'identité stable de ces objets.**

C'est-à-dire que les références et les fonctions _setter_ sont toujours identiques -sont les mêmes objets- d'un rendu à l'autre.

## useEffect() en détail

### Nettoyage d'un effet

Parfois, il est nécessaire de nettoyer un effet.

**Par défaut un effet ne retourne rien, si vous retournez une fonction, cette fonction sera considérée comme la fonction de nettoyage de l'effet.**

**Elle sera appelée avant chaque exécution de l'effet (sauf à la première exécution bien sûr) et lorsque le composant est enlevé du _DOM_ (_unmount_).**

Prenons un exemple, considérons un effet dans un composant qui se connecte à un service externe, par exemple un chat :

```ts
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

Pour ne pas connecter le composant au chat à chaque rendu, il faut ajouter un tableau vide de dépendance pour ne se connecter qu'au premier rendu :

```ts
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

En mode développement, avec le mode strict, comme _React_ effectue un double rendu, vous aurez une double connection car l'effet s'exécutera deux fois.

**Cela vous permet de détecter ce genre de problème avec vos effets.** Imaginez que c'est une application avec plusieurs routes et que l'utilisateur retourne sur ce composant plusieurs fois : cela créera une connexion à chaque fois sans les couper et votre serveur sera rapidement saturé.

**Il faut retourner une fonction de nettoyage :**

```ts
useEffect(() => {
  const connection = createConnection();
  connection.connect();
  return () => {
    connection.disconnect();
  };
}, []);
```

Ici, _React_ appellera la fonction de nettoyage avant la seconde exécution, ce qui déconnectera le service avant de le reconnecter et il n'y aura donc qu'une seule connexion.

### Recommandations pour l'utilisation des effets

Voici les recommandations d'utilisation des effets :

- **N'utilisez des effets que pour du code qui est exécuté _parce que le composant est rendu_, pour tout ce qui est interaction utilisez des gestionnaires d'événement !**
- Si vous calculez quelque chose durant un rendu, n'utilisez pas un effet.
- Pour mettre en cache des informations gourmandes à calculer utilisez `useMemo()` et pas d'effet.
- Pour changer l'état d'une branche de composant si une valeur change n'utilisez pas d'effet mais une propriété _key_.
- Pour réinitialisez l'état après un changement de _prop_ n'utilisez pas un effet mais une fonction _setter_.
- N'oubliez pas de nettoyer vos effets pour éviter des fuites mémoires (`setInteval()` par exemple) ou des connexions multiples, ou encore des affichages multiples (ouverture de modale d'une librairie tierce par exemple).

## Présentation de useLayoutEffect()

### Le _hook_ `useLayoutEffect()`

Le _hook_ `useLayoutEffect()` est très similaire à `useEffect()`, **la différence est qu'il s'exécute de manière synchrone après que toutes les mutations du _DOM_ prévues par _React_ aient eu lieu.**

Il permet donc d'inspecter / modifier le _DOM_ juste avant que le navigateur ne l'affiche (phase _painting_).

Autrement dit, `useLayoutEffect()` **va bloquer l'affichage du rendu**, afin d'éventuellement faire des modifications, puis créer un nouveau rendu de manière synchrone.

Pour cette raison, il doit être utiliser de manière très parcimonieuse. Privilégiez `useEffect()` partout là où s'est possible.

Voici un exemple :

```ts
import { useState, useRef, useLayoutEffect } from "react";

export default function App() {
  const [width, setWidth] = useState(0);
  const buttonRef = useRef(null);

  useLayoutEffect(() => {
    const infos = buttonRef.current.getBoundingClientRect();
    setWidth(infos.width);
  }, []);

  return (
    <>
      <button ref={buttonRef} className="m-10">
        Je suis un bouton
      </button>
      <p className="m-10">{width}</p>
    </>
  );
}
```

## Mise en cache d'un composant avec memo

### Que signifie _Higher-Order_ ?

Le principe _Higher Order_ vient de la programmation fonctionnelle.

Dans ce paradigme de programmation nous utilisons des fonctions qui crÃ©e dâ€™autres fonctions.

Commençons par un exemple simple :

```ts
const tableau = [1, 2, 3];
const createurMultiplicateur = (n) => (arr) => arr.map((num) => num * n);
const doubler = createurMultiplicateur(2);
const triper = createurMultiplicateur(3);

doubler(tableau); // [2, 4, 6];
tripler(tableau); // [3, 6, 9];
```

Bien sûr il s’agit d’un exemple trivial, mais vous voyez le principe : cela permet d’économiser du code en créant des fonctions qui ont des fonctionnalités partagées.

Ici, nous utilisons une fonction qui retourne des fonctions permettant de multiplier tous les éléments d’un tableau par un nombre passé en paramètre.

## Qu’est-ce qu’un composant _HOC_ ?

**Un composant _HOC_ (pour _Higher Order Component_) est un composant qui retourne un nouveau composant avec des fonctionnalités supplémentaires.**

Les autres composants permettent d’utiliser des propriétés pour afficher des éléments, c’est-à-dire gérer l’UI. Ils peuvent utiliser des composants comme éléments d’affichage.

Autrement dit, un _HOC_ permet de créer des composants qui partagent des fonctionnalités pour éviter des réécritures de code.

Un _HOC_ est une fonction pure : c’est-à-dire qu’il ne doit pas contenir d’effets secondaires et modifier l’application. Il doit servir uniquement à créer des composants.

### Utilisation de _memo_

**Si vous avez un composant qui affiche toujours le même résultat pour les mêmes propriétés (_props_), vous pouvez le mettre en cache avec _memo_.**

**Cela permet à _React_ de ne pas recalculer le _DOM_ à afficher pour le composant en réutilisant son dernier rendu si les _props_ n'ont pas changé.**

_memo_ est un _HOC_ qui s’utilise de cette manière :

```ts
import { memo } from 'react';

const ComposantPur = memo(function Composant1(props) {
  ...
});
```

Une comparaison superficielle est effectuée sur les _props_. pour savoir si le composant mis en cache doit être ré-affiché ou non.

Si les _props_ ne changent pas, le composant ne sera pas re-rendu.

Vous pouvez passer également en second argument du _HOC_ `memo()` une fonction de comparaison personnalisée, elle doit retourner un booléen suivant que le composant doit être re-rendu ou non :

```ts
function ComposantPur(props) {
  /* rendu en utilisant props */
}

function fonctionDeComparaison(prevProps, nextProps) {
  /*
    Comparaison personnalisée avec les anciennes et nouvelles props
    Vous devez retourner false pour re-rendre le composant
  */
}

export default memo(ComposantPur, fonctionDeComparaison);
```

N'utilisez _memo_ que pour optimiser les performances de composants très gourmands en ressource. Il s'utiliser rarement et uniquement pour améliorer les performances de l'application.

## Mise en cache d'un calcul avec useMemo()

### Rappel sur la mémoïsation

La mémoïsation est une technique d'optimisation de code. Son objectif est de diminuer le temps d'exécution d'un programme informatique en mémorisant les valeurs retournées par une fonction.

### Le _hook_ `useMemo()`

**Le _hook_ `useMemo()` permet de mémoïser le résultat d’une fonction.**

Les fonctions mémoïsées ne seront pas ré-exécutées tant que les dépendances déclarées ne sont pas modifiées.

**Cette optimisation permet d’éviter des calculs coûteux à chaque rendu.**

Exemple :

```ts
const valeurMemoisee = useMemo(() => calculsComplexes(a, b), [a, b]);
```

Si les dépendances _a_ ou _b_ n’ont pas changé depuis la dernière invocation, `useMemo()` réutilise la dernière valeur renvoyée sans ré-exécuter la fonction.

**Vous pouvez également utiliser `useMemo()` pour ne rafraîchir un composant enfant que si une ou plusieurs valeurs changent.**

```ts
function Parent({ a, b }) {
  const enfant = useMemo(() => <Enfant a={a} />, [a]);
  return <>{enfant}</>;
}
```

## Mise en cache d'une référence avec useCallback()

### Le _hook_ `useCallback()`

**Le _hook_ `useCallback()` permet de mémoïser une fonction.**

Les fonctions mémoïsées ne seront pas modifiées tant que les dépendances déclarées ne changent pas.

**Ce _hook_ permet de passer des fonctions de rappel en _prop_ à des composants enfants afin qu'ils ne soient pas re-rendus si les fonctions ne changent pas.**

Voici la syntaxe :

```ts
const fonctionMemoisee = useCallback(() => {
  uneFonction(a, b);
}, [a, b]);
```
