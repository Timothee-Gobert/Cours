# La librairie React Router

## Introduction à la navigation et à React Router

### La navigation et les *SPA*

Sur une application qui n'est pas une *SPA* (*Single Page Application*), chaque requête entraîne le rechargement de la page qui est générée puis retournée par le serveur.

Sur une *SPA*, il serait très peu performant de devoir recharger toutes les librairies à chaque navigation sur une nouvelle page, il vaut mieux pouvoir générer les composants nécessaires en récupérant éventuellement quelques données sur le serveur.

Pour cette raison, tous les *frameworks frontend* utilisent un système de navigation utilisant du *JavaScript* dans le navigateur.

Cela permet de ne jamais recharger la page lors des navigations de l'utilisateur, tout en lui permettant d'utiliser la navigation de l'application comme une navigation "classique", à savoir conserver l'historique de navigation, pouvoir utiliser les boutons précédent et suivant du navigateur etc.

### Introduction à React Router

Un router sur une *SPA* permet de simuler une application avec plusieurs pages. 

Il va déclencher le chargement de composants sur certaines URLs tout en restant sur la même page.

Un router est donc essentiel à une application complexe car il permet de faciliter le développement de la navigation des utilisateurs dans votre application.

*React* n'a pas de librairie officielle pour le router, nous allons utiliser la plus populaire qui est *React Router*.

### Application de départ

Créez une nouvelle application et copiez-y les styles du projet (dossier *src/assets/styles*).

#### Fichier *mainx.jsx*

Nous avons le code de base pour le fichier racine :

```jsx
import React, { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './assets/styles/index.scss';
import App from './App';

const rootElement = document.getElementById('root');
const root = createRoot(rootElement);

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

#### Fichier *App.jsx*

Pour le composant racine nous avons simplement :

```jsx
import Footer from './components/Footer/Footer';
import Header from './components/Header/Header';
import styles from './App.module.scss';

function App() {
  return (
    <div className={`d-flex flex-column ${styles.appContainer}`}>
      <Header />
      <div className="flex-fill">
        <h1>App</h1>
      </div>
      <Footer />
    </div>
  );
}
export default App;
```

Créez un dossier **components** dans **src** et créez-y un dossier **Header** et un dossier **Footer**.

#### Composant *components/Footer/Footer.jsx*

Pour le pied de page nous avons :

```jsx
import styles from './Footer.module.scss';

function Footer() {
  return (
    <footer
      className={`${styles.footer} d-flex flex-row align-items-center justify-content-center p-20`}
    >
      <p>Copyright © 2025 Dyma, Inc.</p>
    </footer>
  );
}

export default Footer;
```

#### Composant *components/Header/Header.jsx*

Pour l'en-tête nous avons :

```jsx
import styles from './Header.module.scss';

function Header() {
  return (
    <header className={`${styles.header} d-flex flex-row align-items-center`}>
      <div className="flex-fill">
        <strong> React-router </strong>
      </div>
      <ul className={styles.headerList}></ul>
    </header>
  );
}

export default Header;
```

#### Module *components/Header/Header.module.scss*

Pour le module du composant nous avons le même que pour le projet :

```css
@use '../../assets/styles/mixins' as mixin;

.header {
  position: relative;
  padding: 10px 20px;
  height: 58px;
  background-color: white;
  box-shadow: var(--box-shadow-nav);
  color: var(--primary);
  img {
    width: 130px;
    cursor: pointer;
  }
}

.headerList {
  @include mixin.xs {
    display: none;
  }
}

.headerXs {
  display: none !important;
  cursor: pointer;
  transition: opacity 0.2s;
  &:hover {
    opacity: 0.8;
  }
  @include mixin.xs {
    display: block !important;
  }
}
```

#### Module *components/Footer/Footer.module.scss*

Même chose pour le module Sass du pied de page :

```css
.footer {
  background-color: var(--gray-1);
  color: white;
}
```

#### Installation de React Router

Installez enfin la librairie pour le Router :

```sh
npm i react-router-dom
```

## Aperçu du système de navigation de React Router

Dans cette leçon nous allons voir la base du fonctionnement de *React Router* sans trop rentrer dans les détails. Nous approfondirons ensuite dans le reste du chapitre.

Créez un dossier **pages** dans **src**. Dans ce nouveau dossier, créez les dossiers **Homepage** et **Profile**.

### Composant *pages/Homepage/Homepage.jsx*

Créez le composant et placez-y pour le moment :

```jsx
export default function Homepage() {
  return <h2>Homepage</h2>;
}
```

### Composant *pages/Profile/Profile.jsx*

Créez le composant et placez-y pour le moment :

```jsx
export default function Profile() {
  return <h2>Profile</h2>;
}
```

### Fichier *main.jsx*

Pour utiliser le *Router*, il faut utiliser le *RouterProvider* mis à disposition par la librairie.

**Il est nécessaire de lui passer les routes de l'application que le *Router* doit rendre grâce à la propriété *router*.**

**Une route est constituée a minima d'un chemin (*path*) et d'un composant associé.**

```jsx
import React, { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './assets/styles/index.scss';
import { RouterProvider } from 'react-router-dom';
import { router } from './router';

const rootElement = document.getElementById('root');
const root = createRoot(rootElement);

root.render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>
);
```

### Fichier *router.jsx*

Au même niveau que le fichier *main.jsx*, créez un fichier *router.jsx.*

```jsx
import { createBrowserRouter } from 'react-router-dom';
import App from './App';
import Homepage from './pages/Homepage/Homepage';
import Profile from './pages/Profile/Profile';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      {
        path: '/',
        element: <Homepage />,
      },
      {
        path: '/profile',
        element: <Profile />,
      },
    ],
  },
]);
```

*createBrowserRouter* permet de créer un *Router* qui utilise l'*API* du *DOM History* permettant de gérer les URLs et l'historique de navigation. C'est le Router recommandé pour une application Web. 

L'*API History* est une *Web API* officielle compatible tous navigateurs.

**Le *router* doit contenir un tableau de routes, qui peuvent contenir comme dans notre exemple des routes imbriquées, grâce à la propriété *children* qui peut elle-même prendre un tableau de routes.**

**Une route a au minima une propriété *path* et une propriété *element*. Le chemin est simplement la route associée à l'élément *React* devant être rendu par le *Router*.**

### Composant *App.jsx*

Dans le composant racine, qui est rendu par le *Router* quel que soit le chemin, nous ajoutons un composant *Outlet* :

```jsx
import Footer from './components/Footer/Footer';
import Header from './components/Header/Header';
import { Outlet } from 'react-router-dom';
import styles from './App.module.scss';

function App() {
  return (
    <div className={`d-flex flex-column ${styles.appContainer}`}>
      <Header />
      <div className="flex-fill">
        <Outlet />
      </div>
      <Footer />
    </div>
  );
}
export default App;
```

**Un composant *Outlet* doit être utilisé dans un élément parent rendu par le *Router* (ici notre composant App) pour définir où doivent être affichées ses routes enfants (déclarées dans la propriété *children*).**

Cela permet aux composants enfants d'être affichés lorsque leur chemin correspond à l'URL visité.

### Composant *Header.jsx*

Nous utilisons des composants *Link* pour créer des liens :

```jsx
import { Link } from 'react-router-dom';
import styles from './Header.module.scss';

function Header() {
  return (
    <header className={`${styles.header} d-flex flex-row align-items-center`}>
      <div className="flex-fill">
        <strong> React-router </strong>
      </div>
      <ul className={styles.headerList}>
        <Link to="/">Homepage</Link>
        <Link to="/profile">Profile</Link>
      </ul>
    </header>
  );
}

export default Header;
```

**Le composant *Link* fourni par le *Router*, permettant à l'utilisateur de naviguer sur une autre page en cliquant dessus.**

Les composants *Link* sont rendus comme des éléments *HTML* `<a>` avec une propriété *href*.

**Le composant prend une propriété *to* qui permet de définir le chemin cible pour la navigation.**

## Options index et caseSensitive et gestion des erreurs

### Propriétés des routes

Les routes peuvent prendre un grand nombre de propriétés :

```jsx
interface RouteObject {
  path?: string;
  index?: boolean;
  children?: React.ReactNode;
  caseSensitive?: boolean;
  id?: string;
  loader?: LoaderFunction;
  action?: ActionFunction;
  element?: React.ReactNode | null;
  errorElement?: React.ReactNode | null;
  handle?: RouteObject["handle"];
  shouldRevalidate?: ShouldRevalidateFunction;
}
```

### La propriété *index*

Cette propriété prend un booléen. **Si la propriété vaut *true* alors la route est une route index.**

**Les routes index sont affichées au niveau du composant *Outlet* de leur composant parent lorsque l'URL correspond à l'URL de la route parente.**

Autrement dit, c'est comme si c'était une route enfant par défaut.

Reprenons notre exemple, et modifions le fichier *router.jsx* :

```jsx
export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      {
        index: true,
        element: <Homepage />,
      },
      {
        path: '/profile',
        element: <Profile />,
      },
    ],
  },
]);
```

**La route index sera affichée sur le chemin `/` qui est le chemin du composant parent du composant *Homepage*.**

Il s'agit donc bien de la route enfant par défaut pour composant parent *App*.

*Homepage* sera affiché au niveau du composant *Outlet* dans le composant *App*.

#### La propriété caseSensitive

**Cette propriété permet de définir si la correspondance de l'URL à une route doit respecter la casse (minuscule / majuscule) ou non.**

Par exemple :

```jsx
export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        element: <Homepage />,
      },
      {
        path: '/profile',
        element: <Profile />,
        caseSensitive: true,
      },
    ],
  },
]);
```

**Dans notre exemple la route sera activée si l'*URL* est exactement */profile* et non si elle est par exemple */PROFILE* ou */proFILE*.**

### Gestion des erreurs

**La propriété errorElement permet de rendre un composant différent lorsqu'une erreur survient lors du rendu du composant passé à element ou si aucune route ne correspond.**

Modifiez le composant `Header` pour ajouter un composant *Link* pointant vers une route qui n'existe pas, pour déclencher une erreur :

```jsx
import { Link } from 'react-router-dom';
import styles from './Header.module.scss';

function Header() {
  return (
    <header className={`${styles.header} d-flex flex-row align-items-center`}>
      <div className="flex-fill">
        <strong> React-router </strong>
      </div>
      <ul className={styles.headerList}>
        <Link to="/">Homepage</Link>
        <Link to="/profile">Profile</Link>
        <Link to="/efzef">???</Link>
      </ul>
    </header>
  );
}

export default Header;
```

Modifions maintenant le fichier *router.jsx* pour utiliser la propriété que nous avons vu :

```jsx
import { createBrowserRouter } from 'react-router-dom';
import App from './App';
import Homepage from './pages/Homepage/Homepage';
import Profile from './pages/Profile/Profile';
import ErrorPage from './pages/ErrorPage/ErrorPage';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <Homepage />,
      },
      {
        path: '/profile',
        element: <Profile />,
        caseSensitive: true,
      },
    ],
  },
]);
```

Vous pouvez mettre la propriété *errorElement* au sommet de l'arbre des composants pour gérer toutes les erreurs de la branche au même endroit. Il est également possible de le mettre plus bas dans l'arbre pour gérer différemment les erreurs suivants les composants.

Créez enfin le composant *ErrorPage* dans le dossier **pages/ErrorPage** :

```jsx
import React from 'react';
import { useRouteError } from 'react-router-dom';

export default function ErrorPage() {
  const error = useRouteError();
  console.log(error);
  return (
    <>
      <h2>ErrorPage</h2>
      <p>{error.message || error.statusText}</p>
    </>
  );
}
```

Le *hook* `useRouteError()` permet d'accéder à l'erreur dans le composant passé à *errorElement*.

### Gestionnaire d'erreur global

Grâce aux éléments fournis par la librairie, vous pouvez créer un gestionnaire d'erreur global :

```jsx
function GestionnaireGlobal() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    if (error.status === 404) {
      return <div>Cette page n'existe pas !</div>;
    }

    if (error.status === 401) {
      return <div>Vous n'êtes pas autorisé à voir cette page.</div>;
    }

    if (error.status === 503) {
      return <div>Le service ne fonctionne pas. Contactez-nous !</div>;
    }
  }

  return <div>Erreur générique...</div>;
}
```

`isRouteErrorResponse()` permet simplement de vérifier qu'il s'agit d'une erreur provenant de la navigation.

## useMatch, Link et NavLink

### Le composant NavLink

**Le composant *NavLink* est un composant *Link* qui sait s'il est actif.**

C'est très utile pour signaler à l'utilisateur sur quel lien il est actuellement.

**Par défaut, la classe *active* est ajoutée sur le composant *NavLink* lorsqu'il est actif (c'est-à-dire si l'*URL* correspond à la route *to* du lien).**

**Il est également possible de personnalisé le nom des classes ou les styles appliqués au composant suivant qu'il est actif ou non. Pour cela, il faut utiliser une fonction qui reçoit *isActive* en argument.**

Voici un exemple du fonctionnement :

```jsx
import { NavLink } from "react-router-dom";

function Menu() {
  let activeStyle = {
    textDecoration: "underline",
  };

  let activeClassName = "underline";

  return (
    <nav>
      <ul>
        <li>
          <NavLink
            to="/articles"
            style={({ isActive }) =>
              isActive ? activeStyle : undefined
            }
          >
            Articles
          </NavLink>
        </li>
        <li>
          <NavLink
            to="/profile"
            className={({ isActive }) =>
              isActive ? activeClassName : undefined
            }
          >
            Profil
          </NavLink>
        </li>
      </ul>
    </nav>
  );
}
```

**Il est possible d'utiliser la propriété *end* sur le composant *NavLink* afin que le composant ne soit considéré comme actif que si son chemin est actif et non celui de ses descendants ou routes du même niveau.**

### Le *hook* `useMatch()`

**Le *hook* `useMatch()` permet d'obtenir des données sur une route avec le chemin spécifié en fonction de l'URL actuelle.**

Le *hook* retourne *null* si l'URL ne correspond pas au chemin de la route passée en argument.

Si le chemin correspond, elle retourne un objet de cette forme :

```jsx
{
  params: {}
  pathname: "/profile",
  pathnameBase: "/profile",
  pattern: {
    caseSensitive: false
    end: true
    path: "/profile"
  }
}
```

### Modification du composant `Header`

Reprenons notre exemple, et modifions le composant *Header.jsx* :

```jsx
import { NavLink, useMatch } from 'react-router-dom';
import styles from './Header.module.scss';

function Header() {
  const matchProfile = useMatch('/profile');
  const matchHomepage = useMatch('/');

  console.log(matchProfile);

  return (
    <header className={`${styles.header} d-flex flex-row align-items-center`}>
      <div className="flex-fill">
        <strong> React-router </strong>
      </div>
      <ul className={styles.headerList}>
        <NavLink end to="/">
          Homepage
        </NavLink>
        <NavLink to="/profile">Profile</NavLink>
      </ul>
    </header>
  );
}

export default Header;
```

### Modification du *partial _base.scss*

Ajoutez le style suivant :

```css
a {
  color: var(--text-color);
  text-decoration: none;
  margin: 0 5px;
}
```

### Modification du *partial _theme.scss*

Ajoutez le style suivant :

```css
.active {
  font-weight: 500;
  color: var(--primary);
}
```

## Les routes imbriquées

### Chemins relatifs et absolus

**Un lien *commençant par* `/` est un lien *absolu* et doit comporter tout le chemin après exemple.com.**

**Un lien *ne commençant pas par* `/` est un lien *relatif* qui dépend du contexte où il est utilisé pour résoudre l'URL.**

#### Chemins relatifs et absolus pour les liens

Considérons par exemple la route *https://exemple.com/home/articles/123*.

Si nous sommes dans le composant correspondant à cette route par exemple *ArticleDetails* et que nous déclarons des *NavLink* avec des liens relatifs :

- `<NavLink to="edit">` : va se résoudre au chemin absolu */home/articles/123/edit*.
- `<NavLink to=".">` : va se résoudre au chemin absolu */home/articles/123*.
- `<NavLink to="..">` : va se résoudre au chemin absolu */home*.
- `<NavLink to=".." relative="path">` : va se résoudre au chemin absolu */home/articles*.

**Notez bien que le premier `..` sans la propriété *relative* remonte d'un cran dans la hiérarchie des routes et non dans les segments de l'*URL*.**

Autrement dit, par défaut, les routes relatives sont en fonction de la hiérarchie des routes et non des segments d'URL.

I**l faut ajouter `relative="path"` pour que le chemin relatif soit résolu en fonction de l'URL.**

#### Chemins relatifs et absolus pour le *router*

Faites également attention aux routes déclarées dans votre fichier *router.jsx*.

Il faut bien distinguer les chemins relatifs et absolus. **Les chemins relatifs sont résolus en chemin absolu en fonction des chemins parents.**

Dans notre exemple, `path: 'data'` est résolue en fonction de la route parente qui est `path: '/profile'` et correspond donc à la route absolue */profile/data*.

### Création des composants *ProfileOverview* et *ProfileData*

Dans le dossier **Profile**, créez un dossier **pages**. Dans ce dernier dossier, créez 2 dossiers pour 2 nouveaux composants : **ProfileOverview** et **ProfileData**.

Dans le composant *ProfileOverview.jsx* mettez :

```jsx
export default function ProfileOverview() {
  return <h3>ProfileOverview</h3>;
}
```

Dans le composant *ProfileData.jsx* mettez :

```jsx
export default function ProfileData() {
  return <h3>ProfileData</h3>;
}
```

### Modification du composant *Profile*

Dans le composant *Profile*, nous allons créer des liens vers nos nouveaux composants en utilisant le composant *NavLink*.

**Il ne faut surtout pas oublier de placer le composant *Outlet* pour que *React Router* sache où afficher les composants imbriqués rendus par le *Router* :**

```jsx
import { NavLink, Outlet } from 'react-router-dom';

export default function Profile() {
  return (
    <>
      <ul className="d-flex p-20">
        <li>
          <NavLink end to="">
            Overview
          </NavLink>
        </li>
        <li>
          <NavLink to="data">Data</NavLink>
        </li>
      </ul>
      <div className="p-20">
        <h2 className="mb-20">Profile</h2>
        <Outlet />
      </div>
    </>
  );
}
```

N'oubliez pas la propriété end sur le composant NavLink pour Overview afin que le lien ne soit actif que sur la route /profile et pas sur la route /profile/data.

### Modification du fichier *router.jsx*

Il faut bien sûr créer nos nouvelles routes afin que le *Router* sache quels composants affichés sur les liens */profile* et */profile/data* :

```jsx
import React from 'react';
import { createBrowserRouter } from 'react-router-dom';
import App from './App';
import Homepage from './pages/Homepage/Homepage';
import Profile from './pages/Profile/Profile';
import ErrorPage from './pages/ErrorPage/ErrorPage';
import ProfileOverview from './pages/Profile/pages/ProfileOverview/ProfileOverview';
import ProfileData from './pages/Profile/pages/ProfileData/ProfileData';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <Homepage />,
      },
      {
        path: '/profile',
        element: <Profile />,
        caseSensitive: true,
        children: [
          {
            index: true,
            element: <ProfileOverview />,
          },
          {
            path: 'data',
            element: <ProfileData />,
          },
        ],
      },
    ],
  },
]);
```

N'oubliez pas la propriété *index* pour que le composant *ProfileOverview* soit rendu par défaut sur */profile*.

## Segments dynamiques et queryParams

### Les segments dynamiques

Très souvent, vous souhaiterez ajouter une partie dynamique à une route. Par exemple, pour pouvoir récupérer dynamiquement un *id* d'un élément à afficher.

**Pour indiquer au *Router* qu'une partie de la route est dynamique il suffit d'utiliser *:param* où *param* est le nom du paramètre qui sera récupéré.**

Par exemple : `path: '/profile/:id'`. Ici le paramètre a pour nom *id*.

Il est possible d'ajouter autant de segment dynamique que nécessaire :

Par exemple : `path: '/articles/:articleId/review/:reviewId'`. Ici les paramètres dynamiques auront pour noms *articleId* et *reviewId*.

***React Router* est suffisamment intelligent pour faire correspondre la route la plus spécifique lorsque plusieurs routes correspondent à l'URL.**

Prenons l'exemple, où nous avons une route `path: '/profile/:id'` et une route `path: '/profile/edit'`.

Supposons que nous sommes sur l'URL `'/profile/edit'`. 

Les deux routes correspondent à l'URL mais comme la seconde est plus spécifique, **car un segment dynamique est considérée comme moins spécifique qu'un segment fixe**, c'est uniquement la seconde qui sera sélectionnée.

### Récupérer les valeurs des segments dynamiques

**Le *Router* met à disposition le *hook* `useParams()` qui permet d'accéder aux paramètres utilisés.**

Il retourne un objet qui contient des paires clé / valeur avec les paramètres dynamiques de l'URL courante qui correspondent au *path* d'une route.

**Notez bien que les routes enfants peuvent accéder à tous les paramètres dynamiques de leurs routes parentes.**

On utilise l'affectation par décomposition permise par JavaScript pour accéder directement aux paramètres :

```jsx
import { useParams } from 'react-router-dom';

const { userId } = useParams();
```

### Les paramètres de recherche

Le Router supporte également parfaitement les paramètres de recherche (*searchParams*).

**Pour les utiliser il suffit d'ajouter un `?` à la fin de l'URL puis des paramètres de la recherche avec la syntaxe `param1=valeur1&param2=valeur2&param3=valeur3`.**

### Accéder aux paramètres de recherche

**Le *Router* met à disposition le *hook* `useParams()` qui permet d'accéder aux paramètres de recherche.**

`useSearchParams()` retourne un tableau de deux valeurs : la première valeur contient les paramètres de recherche et la seconde contient une fonction qui peut être utilisée pour modifier les paramètres de recherche.

Voici la syntaxe :

```jsx
import { useSearchParams } from "react-router-dom";

const [searchParams, setSearchParams] = useSearchParams();
```

### Les routes étoiles

**Les routes jokers ou étoiles (en anglais *splats*, *catchall* ou *star* routes) sont déclenchées pour tous les caractères à partir de l'étoile.**

Par exemple, `/profile/*` va correspondre à toutes les routes commençant par `/profile`.

Notez que la règle de correspondance la plus spécifique, fait que les segments fixes ou dynamiques seront prioritaires sur les routes étoiles. Vous pouvez donc les utiliser comme des routes par défaut si aucune autre route correspond à l'URL.

### Retour à l'exemple 

#### modification de *router.jsx*

```jsx
import { createBrowserRouter } from 'react-router-dom';
import App from './App';
import Homepage from './pages/Homepage/Homepage';
import Profile from './pages/Profile/Profile';
import ErrorPage from './pages/ErrorPage/ErrorPage';
import ProfileOverview from './pages/Profile/pages/ProfileOverview/ProfileOverview';
import ProfileData from './pages/Profile/pages/ProfileData/ProfileData';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <Homepage />,
      },
      {
        path: '/profile/:id',
        element: <Profile />,
        caseSensitive: true,
        children: [
          {
            index: true,
            element: <ProfileOverview />,
          },
          {
            path: 'data',
            element: <ProfileData />,
          },
        ],
      },
      {
        path: '/profile/*',
        element: <Profile />,
      },
    ],
  },
]);
```

#### Modification du composant *Header.jsx*

Voici l'état du composant *Header* :

```jsx
import { NavLink, useMatch } from 'react-router-dom';
import styles from './Header.module.scss';

function Header() {
  const matchProfile = useMatch('/profile/:id');
  const matchHomepage = useMatch('/');

  console.log(matchProfile);

  return (
    <header className={`${styles.header} d-flex flex-row align-items-center`}>
      <div className="flex-fill">
        <strong> React-router </strong>
      </div>
      <ul className={styles.headerList}>
        <NavLink end to="/">
          Homepage
        </NavLink>
        <NavLink to="/profile/123?age=12">Profile</NavLink>
      </ul>
    </header>
  );
}

export default Header;
```

#### Modification du composant *Profile.jsx*

Voici l'état du composant *Profile* :

```jsx
import { NavLink, Outlet, useParams, useSearchParams } from 'react-router-dom';

export default function Profile() {
  const { id } = useParams();
  const [queryParams, setQueryParams] = useSearchParams();

  console.log(id);
  console.log(queryParams.get('age'));

  return (
    <>
      <ul className="d-flex p-20">
        <li>
          <NavLink end to="">
            Overview
          </NavLink>
        </li>
        <li>
          <NavLink to="data">Data</NavLink>
        </li>
      </ul>
      <div className="p-20">
        <h2 className="mb-20">Profile</h2>
        <Outlet />
      </div>
    </>
  );
}
```

useNavigate, useLocation et scroll
Le hook useLocation()

Le hook useLocation() permet de retourner l'objet location (correspondant à la page visitée), qui est de la forme :

{
  hash: ""
  key: "g0gd63e6"
  pathname: "/profile/123"
  search: "?age=12"
  state: null
}

A chaque fois que l'utilisateur clique sur un lien, ou plus généralement navigue, la location change.

Les propriétés de l'objet sont :

    pathname : propriété qui contient le chemin absolu sans les paramètres de recherche ou le hash.
    search : contient la partie contenant les paramètres de la recherche mais ils ne sont pas parsés. Il s'agit de la chaîne de caractères brute avec le ?.
    hash : contient le hash de l'URL en incluant le #. S'il n'y a pas de hash ce sera une chaîne de caractères vide.
    state : il est possible de passer un objet d'état pour l'associer à l'objet location. C'est utile pour sauvegarder toute information que vous voulez sauvegarder dans l'URL. Par exemple des données de session.
    key : clé unique qui est associée à la location donnée. Pour la location de départ ce sera la chaîne default et pour toutes les locations suivantes ce sera un identifiant unique généré par le Router.

Un exemple d'utilisation peut être d'envoyer les pages visitées par vos utilisateurs à Google Analytics :

import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();

  useEffect(() => {
    ga('send', 'pageview');
  }, [location]);

  return (
    // ...
  );
}

 
Le hook useNavigate()

Le hook useNavigate() permet de déclencher une navigation depuis le code de vos composants (et non pas par un clic sur un lien de l'utilisateur).

Par exemple :

import { useNavigate } from "react-router-dom";

const navigate = useNavigate('/profile');

Vous pouvez passer une URL absolue ou relative, mais aussi un chiffre (par exemple -1). 

Si vous passez un chiffre cela permet d'utiliser l'historique du navigateur (-2 correspond à cliquer sur deux fois précédent sur le navigateur).

 
Le composant ScrollRestoration

Le composant ScrollRestoration permet la restauration du défilement lors de la navigation. Il s'utilise au plus haut niveau de l'application, une seule fois :

import { ScrollRestoration } from "react-router-dom";

function App() {
  return (
    <div>
      {/* ... */}
      <ScrollRestoration />
    </div>
  );
}

 
La propriété preventScrollReset

La propriété preventScrollReset s'utilise sur un lien (composants Link ou NavLink).

Grâce à cette propriété, Si vous utilisez ScrollRestoration, cela empêche la restauration du défilement en haut de la fenêtre lorsque le lien est cliqué.

Cela n'empêche pas la restauration du défilement en utilisant les boutons suivant et précédent du navigateur, mais uniquement lorsque l'utilisateur clique sur un lien qui a la propriété preventScrollReset.

Un exemple est lorsque vous avez des onglets permettant de naviguer qui ne sont pas en haut de la page et que vous voulez les laisser visible à l'utilisateur lors d'une navigation.

 
Modification du composant ProfileOverview

Voici où nous en sommes :

import { useNavigate } from 'react-router-dom';

export default function ProfileOverview() {
  const navigate = useNavigate();

  function navigateToData() {
    navigate('data', { preventScrollReset: true });
  }

  return (
    <>
      <h3>ProfileOverview</h3>
      <button onClick={navigateToData}>data</button>
      <ul>
        <li>
          OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW
          OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW
          OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW
          OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW
          OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW
          OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW
          OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW OVERVIEW{' '}
        </li>
        /* ... */
      </ul>
    </>
  );
}

 
Modification du composant ProfileData

Voici où nous en sommes :

import { NavLink } from 'react-router-dom';

export default function ProfileData() {
  return (
    <>
      <h3>ProfileData</h3>
      <ul>
        <li>
          DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA
          DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA
          DATA DATA DATA DATA DATA DATA DATA DATA DATA DATA{' '}
        </li>
        /* ... */
        <NavLink preventScrollReset to="..">
          TO OVERVIEW
        </NavLink>
        /* ... */
      </ul>
    </>
  );
}

 
Modification du composant App

Voici où nous en sommes :

import { useEffect } from 'react';
import Footer from './components/Footer/Footer';
import Header from './components/Header/Header';
import { Outlet, ScrollRestoration, useLocation } from 'react-router-dom';
import styles from './App.module.scss';

function App() {
  const navigation = useLocation();

  useEffect(() => {
    console.log(navigation);
  }, [navigation]);
  return (
    <div className={`d-flex flex-column ${styles.appContainer}`}>
      <Header />
      <div className="flex-fill">
        <Outlet />
      </div>
      <Footer />
      <ScrollRestoration />
    </div>
  );
}
export default App;

Introduction aux loaders et useLoaderData
Les chargeurs de données (data loaders)

Il est possible d'utiliser des chargeurs qui permettent de charger des données depuis une API pendant la navigation.

C'est un système très puissant permettant de charger toutes les données nécessaires pour chaque niveau de route.

Vous pouvez ainsi par exemple utiliser un chargeur de données sur la route / pour récupérer les données indispensables au fonctionnement de toute l'application, et charger des données plus spécifiques pour certaines routes imbriquées.

Pour utiliser un chargeur il faut :

    Utiliser la propriété loader sur une route donnée :

{
  index: true,
  element: <Homepage />,
  loader: homepageLoader,
},

Le loader peut être déclaré directement sur la route de cette manière :

{
  index: true,
  element: <Homepage />,
  loader: async ({ request }) => {
    const res = await fetch("/api/user");
    const user = await res.json();
    return user;
  }}
},

Mais le plus souvent il vaut mieux utiliser un fichier séparé pour ne pas surcharger le fichier des routes.

Il est à noté que le loader reçoit les paramètres dynamiques dans la propriété params, vous pouvez donc les utiliser facilement :

{
  path: '/users/:userId',
  element: <UserDetails />,
  loader: async ({ params }) => {
    const res = await fetch(`/api/users/${params.userId}`);
    const user = await res.json();
    return user;
  }}
},

    Utiliser le hook useLoaderData() sur le composant rendu par le Router qui utilise un chargeur :

function UserDetails() {
  const user = useLoaderData();
}

 
Retour à notre exemple
Fichier homepageLoader.jsx

Dans le dossier src créer un dossier loaders. Créez dans ce dernier le fichier homepageLoader.js :

export async function homepageLoader() {
  const response = await fetch('https://restapi.fr/api/recipes');
  if (response.ok) {
    return response.json();
  }
}

Il se peut que lorsque vous fassiez cet exemple il y ait une seule ou plus de recette. Dans ce cas, reprenez le dernier chapitre du projet et effectuez un seed comme vu précédemment.
Fichier router.jsx

Dans le fichier contenant nos routes il ne faut pas oublier d'ajouter le loader :

import { createBrowserRouter } from 'react-router-dom';
import App from './App';
import Homepage from './pages/Homepage/Homepage';
import Profile from './pages/Profile/Profile';
import ErrorPage from './pages/ErrorPage/ErrorPage';
import ProfileOverview from './pages/Profile/pages/ProfileOverview/ProfileOverview';
import ProfileData from './pages/Profile/pages/ProfileData/ProfileData';
import { homepageLoader } from './loaders/homepageLoader';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <Homepage />,
        loader: homepageLoader,
      },
      {
        path: '/profile/:id',
        element: <Profile />,
        caseSensitive: true,
        children: [
          {
            index: true,
            element: <ProfileOverview />,
          },
          {
            path: 'data',
            element: <ProfileData />,
          },
        ],
      },
      {
        path: '/profile/*',
        element: <Profile />,
      },
    ],
  },
]);

Composant Homepage.jsx

Dans le composant nous utilisons le hook que nous avons vu :

import { useLoaderData } from 'react-router-dom';

export default function Homepage() {
  const recipes = useLoaderData();

  return (
    <>
      <h2>Homepage</h2>
      <ul>{recipes && recipes.map((r) => <li key={r._id}>{r.title}</li>)} </ul>
    </>
  );
}