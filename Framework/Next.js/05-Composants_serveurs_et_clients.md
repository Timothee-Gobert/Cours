# Composants serveurs et clients

## Composants serveurs et clients

Il est maintenant temps de parler des deux types de composants que vous permet d'utiliser Next.js ! 

Avant React 18, **vous n'aviez en fait la possibilité de créer que des composants clients.** Désormais, vous allez pouvoir choisir individuellement, pour chacun des composants de votre application, si vous souhaitez les voir rendus côté serveur (avec des composants serveurs) ou côté client.

### Les composants serveurs

L'utilisation de ces composants vont vous permettre d'écrire du code comme si vous étiez en train de travailler sur une application serveur : rappelez-vous, nous avions vu en exemple une application comme *Symfony* par exemple. 

Ce composant est le **type de composant par défaut** lorsque vous créez un nouveau composant à la racine de votre dossier **app**.

En clair, toute la résolution des dépendances de votre composant, les appels aux données que vous pourriez effectuer etc. vont être rendus côté serveur. Votre client, le consommateur de l'application, ne recevra lui qu'un **contenu statique HTML**.

Cela vous permet entre autre de considérablement **réduire le bundle javascript** que vous allez distribuer à vos clients et éviter l'un des inconvénients que l'on avait vu lors de l'utilisation d'une SPA : à mesure que votre application grossit, ce bundle grossit également, votre application est de moins en moins performante.

Egalement, vous pouvez directement dans ces composants **accéder à des ressources backend** tel que des bases de données, **garder des informations secrètes** côté serveur comme des API key.

Pour tous ces avantages, nous tenterons toujours d'**utiliser au maximum les composants serveurs** et de réduire l'utilisation des composants clients qu'au moment où c'est nécessaire.

#### Création du composant `PostList.js`

```jsx
const PostList = async () => {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const posts = await response.json();

  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <p>{post.title}</p>
        </div>
      ))}
    </div>
  );
};

export default PostList;
```

La totalité de ce composant va pouvoir être rendu côté serveur. Comme vu ensemble, l'écriture de ce code ne produit, dans le navigateur, aucune requête dans le network, côté client. En sortie, réellement, le client ne recevra qu'un document HTML statique qui ressemblerait par exemple à ça :

```jsx
<div>
  <div><p>sunt aut facere repellat provident occaecati excepturi optio reprehenderit</p></div>
  <div><p>qui est esse</p></div>
  <div><p>ea molestias quasi exercitationem repellat qui ipsa sit aut</p></div>
</div>
```

Egalement, et nous verrons ce point en détail ensemble, vous profiterez de tout un système de mise en cache mis en place par le framework.

### Les composants clients

Les composants clients, ne donnent plus vraiment envie d'être utilisés. Et pourtant, comme les composants serveurs ne servent que du contenu HTML statique en sortie, ils vont être nécessaires pour **permettre à nos clients d'interagir avec notre application**. 

Lorsque vous souhaitez gérer des évènements (`onClick`, `onChange`...) ou utiliser des *hooks* (`useState`, `useEffect`), ou encore **intéragir avec le DOM**, utiliser des **API navigateur**, l'utilisation du composant client sera obligatoire.

Pour pouvoir utiliser un composant en tant que client, il faudra ajouter en première ligne du fichier de votre composant la directive suivante : `"use client"`

Vous rencontrerez également des cas où vous souhaiterez récupérer des données dans votre composant client, par exemple dans le cadre d'une recherche dans votre application ou lors d'une validation de formulaire : ce sera toujours possible.

Pour maximiser les performances de votre application Next, il sera nécessaire bien entendu de découper votre code de manière à ce que **vos composants clients soient le plus possible réduits à leur strict nécessité.**

Nous avons vu en exemple, dans le cadre de l'affichage de la liste de nos posts, l'ajout d'un bouton sur lequel notre utilisateur souhaiterait venir interagir :

#### Création du composant `Like.js`

```jsx
"use client";
import { useState } from "react";

const Like = () => {
  const [likes, setLikes] = useState(0);
  return (
    <div>
      <button onClick={() => setLikes((likes) => likes + 1)}>
        Liked {likes} times
      </button>
    </div>
  );
};

export default Like;
```

### Jusqu'à l'inclusion des deux...

Maintenant, nous pouvons inclure notre composant client dans notre composant serveur. Et nous verrons dans cette prochaine leçon les petites spécificités à l'inclusion de ces deux éléments.

```jsx
import Like from "./Like";

const PostList = async () => {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const posts = await response.json();

  return (
    <div>
      {posts.map((post) => (
        <div key={post.id}>
          <p>{post.title}</p>
          <Like />
        </div>
      ))}
    </div>
  );
};

export default PostList;
```

[Code sur Github](https://github.com/AntoineBourin/nextjs-demo/tree/v5.1.0)

