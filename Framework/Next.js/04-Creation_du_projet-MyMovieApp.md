# Création du projet - MyMovieApp

## Installation & configuration de l'application

Dans ce tout nouveau chapitre, nous allons nous concentrer sur la création d'un nouveau projet Next.js : MyMovieApp. Pour mettre en pratique toutes les notions comprises, je vous propose de créer ce projet de visualisation de films / séries.

Pour mener à bien ce projet, nous utiliserons une API : [TheMovieDB](https://www.themoviedb.org/).

### Installation du projet

Comme précédemment, pour créer un nouveau projet Next, nous utilisons le script : 

```sh
npx create-next-app@latest
```

Nous n'utiliserons toujours pas TypeScript pour le moment. 

L'option `--experimental-app` n'est plus à ajouter, elle permettait de définir que l'on souhait utiliser la version app router de Next quand elle n'était pas encore stable. Depuis juin 2023, cette version est stable.

#### Installation et configuration de SASS

SASS est un pré-processeur CSS qui vous permet entre autres de définir des variables dans votre projet, créer des mixins & fonctions, intégrer vos éléments de façon imbriquée.

Pour installer SASS dans notre projet, nous écrivons la commande suivante : 

```sh
npm install sass
```

Le packet s'ajoute au fichier package.json et est installé. On peut désormais l'utiliser pour intégrer notre projet.

Afin de pouvoir importer des fichiers SASS futurs rapidement depuis nos différents fichiers de style, il est possible d'ajouter une configuration dans le fichier next.config.js :

```jsx
sassOptions: {
    includePaths: [path.join(__dirname, "src/styles")],
},
```

Ceci nous permet désormais de créer un fichier *_variables.scss* à la racine de notre dossier src/styles. Désormais, dans l'ensemble de nos fichiers de style du projet, il sera possible d'importer nos variables facilement. 

Nous pourrions faire également la même chose pour des mixins ou des éléments de style que nous voudrions utiliser dans l'ensemble de notre code.

Enfin, l'installation de [FontAwesome](https://docs.fontawesome.com/web/use-with/react).

### Mise en place du projet

#### Composant Header

Pour commencer à mettre en place notre Header, nous créons le composant Header, inclus dans notre RootLayout de notre application.

Notre partie header contiendra une font Roboto. Comme évoqué ensemble lors des leçons précédentes, nous récupérons cette font depuis google fonts via Next.

```jsx
import { Roboto } from "next/font/google";
```

Cette font sera téléchargée directement vers le CDN et stockée statiquement dans notre application. Ceci évitera désormais à vos utilisateurs des requêtes extérieures et produira encore une fois une amélioration des performances de votre application.

![IMAGE Github](../../assets/images/github.png)[Code sur Github](https://github.com/AntoineBourin/my-movie-app/tree/v4.1.0)

## Mise en place des premiers syles SASS

Dans cette nouvelle partie, je vous propose d'intégrer la partie header de notre application et l'un des premiers composants : MediaCard.

### Amélioration de l'inclusion des fonts

Je vous propose de voir une autre manière d'inclure nos fonts dans notre application. 
Ici, nous aurons besoin de deux fonts différentes : Roboto & Montserrat.

Plutôt que de les intégrer en class de nos composants, il nous est possible lors de leur création de leur définir une variable, que l'on pourra ensuite utiliser dans nos styles : 

```jsx
export const roboto = Roboto({
  subsets: ["latin"],
  weight: ["100", "300"],
  variable: "--font-roboto",
});
```

Pour séparer les fonts au cas où nous voudrions les réutiliser pour un autre besoin, je les ai créées et exportées dans le fichier *fonts.js*.

Maintenant, il est possible de récupérer cette font et d'inclure la variable de manière globale, dans notre application, en l'incluant dans la balise `body`, de notre `RootLayout` : 

```jsx
import Header from "@/components/header/Header";
import "./globals.css";
import "@fortawesome/fontawesome-svg-core/styles.css";
import { roboto, montserrat } from "@/fonts";

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={`${roboto.variable} ${montserrat.variable}`}>
              ......
      </body>
    </html>
  );
}
```

Grâce à cette inclusion, désormais, dans n'importe quel style de notre application, on peut très facilement utiliser l'une ou l'autre des fonts créées : 

```jsx
font-family: var(--font-roboto);
```

### Le composant MediaCard

Comme nous allons bientôt afficher une liste de médias, films ou séries, il faut créer le rendu de la carte qui nous permettra d'afficher l'image du média, son titre et sa date de sortie notamment.

Comme pour le composant Header, j'ai également choisi de le créer dans le dossier **src/components**. De la même manière, nous utiliser un fichier *module.scss* pour l'intégration.

#### L'ajout du composant Image

Maintenant, il est question d'ajouter l'image de notre carte. Pour le moment, comme nous n'utilisons pas encore de données dynamiques, nous pouvons utiliser une image d'exemple récupérée. 

Ici, l'utilisation de ce composant nous permet notamment de procéder à une quantité d'optimisation sur les images utilisées. Parmi eux, une optimisation de taille, de formats... Pour éviter tout problème de sécurité sur l'optimisation d'images externes, Next nous demande de préciser, dans notre configuration *next.config.js*, une liste de patterns d'URL qui peuvent être inclus comme sources d'image.

Ici, nous créons la configuration de pattern pour l'inclusion de notre [image](https://image.tmdb.org/t/p/w500/hYQs5RPHiWctoYqvI8baHiIqdd8.jpg), provenant de *tmdb.org*.

```jsx
images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "image.tmdb.org",
        pathname: "/t/p/w500/**",
      },
    ],
  },
```

Il est possible d'utiliser le caractère `*` dans la partie `pathname` et `hostname` pour matcher un seul segment d'URL ou de sous-domaine et `**` pour une multitude de segments ou sous-domaines.

Il est maintenant possible d'inclure le composant Image avec la source correspondante.

```jsx
<Image
 src="https://image.tmdb.org/t/p/w500/hYQs5RPHiWctoYqvI8baHiIqdd8.jpg"
 alt="media title"
 fill
/>
```

[Code sur Github](https://github.com/AntoineBourin/my-movie-app/tree/4.2.0)

### Gestion des erreurs

Il est temps de s'occuper de la gestion des erreurs dans notre application ! Nous l'avions vu lorsque nous avions abordé les fichiers de routage, il est possible de créer des composants fallback qui constituent une interface de remplacement pour notre application dans le cas où des erreurs se produisent.

### Création du fichier *not-found.js*

Créé à la racine de notre dossier de routage *app*, une fois ce fichier créé, il nous permettra de créer une interface de remplacement pour nos pages 404, qu'elles soient : 

- générées lorsqu'un utilisateur tente d'accéder à une page non définie dans notre application
- générées par le biais d'un appel à la fonction `notFound`, dans l'une nos de pages

```jsx
const NotFound = () => {
  return (
    <div className="error">
      <h1>404</h1>
      <p>Oups, la page que vous demandez n&apos;existe pas.</p>
    </div>
  );
};

export default NotFound;
```

### Création du fichier *error.js*

Lorsque l'on vient simuler une nouvelle erreur javascript, notre erreur est pour le moment propagée sur l'ensemble de notre application. Impossible d'accéder à notre navigation par exemple via le RootLayout.

La création de ce fichier à la racine de notre dossier de routage permet de générer une interface utilisateur de remplacement pour l'ensemble des erreurs qui pourraient être créées par notre page d'accueil et jusqu'ici également nos pages présentes dans les segments **movies** et **series**.

```jsx
"use client";

const Error = () => {
  return (
    <div className="error">
      <h1>Erreur</h1>
      <p>Oups, une erreur s&apos;est produite.</p>
    </div>
  );
};

export default Error;
```

Rappelons ici que si l'on avait souhaité également gérer des erreurs qui seraient générées dans notre RootLayout, il aurait fallu créer un fichier *global-error.js*.

[Code sur Github](https://github.com/AntoineBourin/my-movie-app/tree/4.3.0)

## Préparation de la navigation

Comme nous l'avions vu ensemble, l'utilisation du composant Link de Next nous permet entre autres de profiter d'un système de *prefetch* des pages, éviter le re-rendu complet de nos composants (rerendu partiel)...

### Modification du composant `MediaCard`

```jsx
import Image from "next/image";
import styles from "./MediaCard.module.scss";
import Link from "next/link";

const MediaCard = ({ mediaId }) => {
  return (
    <div className={styles.card}>
      <Link href={`/movies/${mediaId}`}>
        <div className={styles.image}>
          <Image
            src="https://image.tmdb.org/t/p/w500/hYQs5RPHiWctoYqvI8baHiIqdd8.jpg"
            alt="media title"
            fill
          />
        </div>
        <div className={styles.content}>
          <h2>Creed III</h2>
          <p>Le 01/03/2023</p>
        </div>
      </Link>
    </div>
  );
};
```

### Ajout du segment dynamique `/movies/[id]/page.js`

C'est cette page qui viendra afficher le détail de notre film, lors du clic sur le composant MediaCard par un utilisateur ! Rappelez-vous, dans le cadre d'un segment dynamique, il nous est possible de récupérer son paramètre via le props *params*.

```jsx
const MovieIdPage = ({ params }) => {
  return (
    <div>
      <h1>Movie page with id : {params.id}</h1>
    </div>
  );
};

export default MovieIdPage;
```

### Modification du composant `Header`

Il convient maintenant d'ajouter les liens vers nos différentes pages dans notre header ! Egalement, l'icône pour accéder plus tard à la page utilisateur depuis [FontAwesome](https://docs.fontawesome.com/web/use-with/react).

```jsx
import styles from "./Header.module.scss";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faUser } from "@fortawesome/free-solid-svg-icons";
import Link from "next/link";

const Header = () => {
  return (
    <header className={styles.header}>
      <div className={styles.logo}>
        <p>
          <Link href="/">MyMovieApp</Link>
        </p>
      </div>
      <div className={styles.navigation}>
        <nav>
          <ul>
            <li>
              <Link href="/series">Séries</Link>
            </li>
            <li>
              <Link href="/movies">Films</Link>
            </li>
          </ul>
        </nav>
        <input type="text" placeholder="Rechercher un titre ..." />
        <div>
          <FontAwesomeIcon icon={faUser} />
        </div>
      </div>
    </header>
  );
};

export default Header;
```

[Code sur Github](https://github.com/AntoineBourin/my-movie-app/tree/4.4.0)

