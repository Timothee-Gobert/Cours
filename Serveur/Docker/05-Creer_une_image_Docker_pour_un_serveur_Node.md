## Créer une image Docker pour un serveur Node

### Introduction au projet

#### Objectif du projet

L'objectif du projet va être de mettre en place une application *Express* (qui est un framework *Node.js*).

Cela n'a aucune importance de connaître *Node.js* pour le projet car l'objectif est d'étudier la mise en place d'un *Dockerfile* pour permettre l'utilisation d'une application.

Notre image devra comporter *Node.js* et son gestionnaire de paquet *npm*.

Elle devra comporter plusieurs dépendances : *nodemon* et *express*.

Et elle devra lancer l'application contenue dans *app.js* par défaut.

#### Mise en place du projet

Pour le projet, nous allons créer un nouveau *Dockerfile* dans un dossier.

Dans le même dossier nous allons également avoir un fichier *package.json* (qui permet de gérer les dépendances utilisées) et un fichier *app.js* (qui contiendra notre application).

Dans le fichier *package.json* avons donc nos deux dépendances :

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "nodemon": "^3.0.1"
  }
}
```

Dans le fichier *app.js* nous avons simplement une route *Express* qui enverra au format *JSON* la chaîne de caractère "Hello World!" pour toutes les routes :

```js
const express = require('express');
const app = express();

app.get('*', (req, res) => res.status(200).json('Hello World!'));

app.listen(80);
```
