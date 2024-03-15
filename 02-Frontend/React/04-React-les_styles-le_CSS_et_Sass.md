# React, les styles, le CSS et Sass

## JSX et attribut style

### Utilisation des doubles accolades pour le style en ligne

**Avec *JSX* et *React*, il est obligatoire de passer un objet à l'attribut *style*.**

Nous avions vu qu'il est obligatoire d'utiliser les doubles accolades pour passer un objet comme *props* et c'est la même règle avec l'attribut *style*.

Nous devons donc écrire :

```jsx
<p style={{ backgroundColor: 'black', color: 'pink' }}></p>
```
 
### Utilisation du *camelCase* pour le style en ligne

Comme tout est transformé en objet *JavaScript* avec *JSX*, nous ne pouvons pas utiliser des tirets dans les noms de propriétés *CSS*. Il est obligatoire d'utiliser le *camelCase*.

Nous ne pouvons donc pas écrire :

```jsx
<p style={{ background-color: 'black'  }}></p>
```

Vous obtiendrez une erreur.

## Utilisation de CSS avec React

### Ajouter des classes aux éléments JSX

Nous avons déjà vu qu'il fallait obligatoirement utiliser l'attribut *className* et non *class* pour ajouter une ou plusieurs classes à un élément *JSX* :

```jsx
<span className="menu navigation-menu">Menu</span>
```

Vous pouvez bien sûr utiliser une variable, ce qui est très courant pour ajouter ou modifier des classes suivant des conditions :

```jsx
render() {
  let className = 'menu';
  if (condition) {
    className += ' menu-active';
  }
  return <span className={className}>Menu</span>
}
```
 
### Importer du *CSS* grâce à *Vite*

Grâce à *Vite* vous pouvez directement créer un fichier `Composant.css` puis l'importer dans le fichier *JavaScript* `Composant.jsx` de cette manière :

```jsx
import './Composant.css';
```

En production, tout le *CSS* est mis dans un seul fichier par *Vite* lorsque vous construisez l'application avec :

```bash
npm run build
```

**Notez donc que le *CSS* n'est pas du tout isolé par défaut en procédant de cette manière : tout le style défini est global à toute l'application.** 

Il est donc conseillé de n'utiliser cette méthode que pour le style global de l'application qui est importé dans le fichier `main.jsx` par :

```jsx
import './index.css';
```

Nous allons voir dans la prochaine leçon comment écrire du *CSS* isolé grâce aux modules.

## React et les modules CSS

### Qu’est-ce qu’un *module CSS* ?

**Un *module CSS* est un fichier dont toutes les classes et les animations sont isolées localement par défaut.**

Comme nous l’avons vu, le *CSS* est normalement global par défaut. Les modules *CSS* permettent donc de résoudre cette problématique.

**Les modules *CSS* ne sont pas une spécification *CSS* ou des navigateurs, il s’agit d’une [librairie](https://github.com/css-modules/css-modules) utilisée notamment par *Vite* qui permet de changer automatiquement le nom des classes pour permettre cette isolation.**

### *React* et les *modules CSS*

*Vite* permet d’utiliser les *modules CSS* par défaut.

Il suffit de respecter certaines conventions de nommage afin de respecter la configuration de *Vite*.

Il est obligatoire que les *modules CSS* aient comme nom : `[nomDuModule].module.css`

Ou comme nous le verrons dans la leçon suivante avec l’utilisation de *Sass* : `[nomDuModule].module.scss`

Par exemple, imaginons un module *Message.module.css* pour styliser un composant :

```scss
.theme {
  color: green;
}

.message {
  composes: theme;
  font-size: 20px;
}
```

**Notez l’utilisation de *composes* qui permet de fusionner des classes entre elles grâce aux *modules CSS*.**

La classe *.message* aura donc color de la classe *.theme*.

Pour utilise ce style dans des composants, il suffit de l’importer et de les utiliser de cette manière :

```jsx
import styles from './Message.module.css'

// ...

<li className={styles.message}> {message} </li>
```

Vous noterez qu’il suffit d’importer le module et de lui donner le nom que vous souhaitez, (*styles* par convention), puis d’utiliser *className* et de lui passer la classe de votre module ; en l'occurrence, *styles.message*.

Grâce à *Vite* la classe sera transformé en quelque chose qui ressemble à :

```jsx
class="Message_message_tp5lz"
```

### Utilisation de plusieurs classes sur un élément

Pour utiliser plusieurs classes simultanément sur un élément, il suffit d’utiliser les littéraux de chaîne de caractères permis par *JavaScript* :

```jsx
<div className={`${styles.classe1} ${styles.classe2}`}>
```

## React et Sass

### Qu’est-ce que *Sass* ?

*Sass* est un préprocesseur *CSS*. C’est-à-dire que c’est un langage de description de styles qui est transpilé en *CSS*.

En résumé, il s’agit de *CSS* avec plein de fonctionnalités additionnelles. 

Avec *Sass* vous pouvez par exemple utiliser l’imbrication :

```scss
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }
}
```

Vous pouvez également utiliser des opérateurs :

```scss
.article {
  width: 600px / 960px * 100%;
}
```

Il existe d’autres fonctionnalités telles les *mixins*, *@extends* etc.

>*Vous pouvez aller voir le chapitre Sass de la formation HTML & CSS pour en apprendre plus.*

### Sass et React

Il n'y a rien de plus à faire avec *Vite* pour utiliser *Sass* avec *React*.

Pour l’utiliser il vous suffit de faire :

```bash
npm i -D sass
```

Et… c’est tout !

Vous n’avez plus qu’à nommer vos modules avec `[nom].module.scss` et le tour est joué. 

**Vous bénéficiez de la puissance de *Sass* et de l’isolation permise par les *modules CSS*, c’est-à-dire de tout ce que vous avez besoin pour créer une application importante.**
