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

