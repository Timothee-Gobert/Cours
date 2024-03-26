## Utiliser Docker Compose avec plusieurs services

### Introduction à l'architecture du projet

L'objectif du chapitre est d'arriver à mettre en place des environnements réalistes d'une application complète pour le développement et la production.

#### Environnement de développement

Voici un petit récapitulatif de notre architecture en développement :

**Application client :** c'est l'application *SPA* servie aux utilisateurs. Nous utilisons *React*, mais cela fonctionnerait exactement pareil pour *Angular* et *Vue*. L'application client utilise *Webpack dev server* en développement. Il s'agit d'un serveur *Node.js* qui permet de détecter les changements dans le code source, de recompiler le code et de rafraichir la page du navigateur. Cela permet d'obtenir une fonctionnalité essentielle lorsqu'on développe : *le live-reload*.

**Reverse-proxy :** il s'agit du point d'entrée de l'application qui va se charger ici uniquement de répartir les requêtes entrantes entre nos services. Pour ce fait, nous allons utiliser *nginx* avec une configuration très basique. Il va rediriger :
- les requêtes commençant par */api* vers notre *API* qui sera un serveur *Node.js*.
- les requêtes commençant par */sockjs-node* vers le serveur de développement *Webpack*. La petite difficulté, que nous étudierons en détails, est de gérer les *Websockets* qui sont nécessaires pour *Webpack dev server* afin d'avertir le navigateur qu'il doit recharger la page après une recompilation.
- les autres requêtes vers *Webpack dev server* qui sert notre application *React* en développement.

**Base de données :** nous aurons une base de données *MongoDB* auquelle notre serveur *Node.js* se connectera.

#### Environnement de production

Voici un petit récapitulatif de notre architecture en production :

**Reverse-proxy :** il s'agit du point d'entrée de l'application qui va se charger ici uniquement de répartir les requêtes entrantes entre nos services. Pour ce faire, nous allons utiliser *nginx* avec une configuration très basique. Il va rediriger :
- les requêtes commençant par */api* vers notre *API* qui sera un serveur *Node.js*.

- les autres requêtes vers un autre service *nginx* qui va uniquement servir le *build* de notre application *React* de manière optimisée.

**Base de données :** nous aurons une base de données *MongoDB* auquelle notre serveur *Node.js* se connectera.

#### Création du projet

Nous allons créer un dossier *fullstack*.

Nous l'ouvrons dans *VS Code* :

```sh
code fullstack
```

Nous créons ensuite un dossier *api* qui contiendra notre serveur *Node.js*, un dossier *client*, pour notre application *React*, un dossier *db* pour la configuration de la *base de données* et enfin un dossier *reverseproxy* pour le point d'entrée *nginx*.

Nous créons ensuite un fichier *docker-compose.dev.yml* et un fichier *docker-compose.prod.yml*. 

### Mise en place de la configuration client

#### Création de l'application client

Nous supprimons le dossier *client* pour permettre l'utilisation de `create-react-app` (en effet, la librairie va se charger de créer ce dossier) :

```sh
npx create-react-app client
```

Dans le dossier client, nous créons ensuite un fichier *Dockerfile.dev* :

```dockerfile
FROM node:alpine
WORKDIR /home/node
COPY package.json .
RUN npm install
COPY . .
CMD [ "npm", "start" ]
```

Nous passons à la modification de *docker-compose.dev.yml* :

```yaml
version: '3.9'
services:
  client:
    stdin_open: true
    tty: true
    ports:
      - '3000:3000'
    build:
      dockerfile: Dockerfile.dev
      context: ./client
    volumes:
      - type: bind
        source: ./client
        target: /home/node
      - type: volume
        target: /home/node/node_modules
```

Nous avons la même configuration que précédemment pour l'environnement de développement de notre application client.

Nous pouvons tester en faisant :

```sh
docker compose -f docker-compose.dev.yml up --build
```

#### Modification du code source de l'application

Pour pouvoir tester l'*API* que nous allons mettre en place, nous allons modifier le code source de notre application *React*.

Nous modifions *client/src/App.js* :

```js
import logo from './logo.svg';
import './App.css';
import { useEffect, useState } from 'react';

function App() {
  const [count, setCount] = useState();
  useEffect(() => {
    async function fetchCount() {
      try {
        const response = await fetch('/api/count');
        if (response.ok) {
          setCount(await response.json());
        }
      } catch (error) {
        console.log(error);
      }
    }
    fetchCount();
  }, []);

  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <p>Count: { count }</p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```

Le code précédent fait simplement une requête en utilisant l'API fetch sur notre API que nous mettrons en place juste après.

#### Problème hot reload Windows

Si vous rencontrez un problème pour faire fonctionner le hot reload sur Windows. Ajoutez ceci au *service client* dans le fichier *docker-compose.dev.yml* :

```yaml
environment:
  - WATCHPACK_POLLING=true
```

### Mise en place de l'API Node.js

#### Initialisation de l'application

Dans *api*, nous initialisons notre *package.json* :

```sh
npm init -y
```

Nous installons ensuite les dépendances nécessaires :

```sh
npm i express nodemon mongodb
```

Nous créons ensuite un dossier *src* dans lequel nous plaçons un fichier *index.js* :

```js
const express = require('express');
const { MongoClient } = require('mongodb');
let count;
const uri = process.env.NODE_ENV === 'production' ?
`mongodb://${ process.env.MONGO_USERNAME }:${ process.env.MONGO_PWD }@db` :
`mongodb://db`;

const client = new MongoClient(uri);
async function run() {
  try {
    await client.connect();
    await client.db('admin').command({ ping: 1 });
    console.log('CONNEXION DB OK !');
    count = client.db('test').collection('count');
  } catch (err) {
    console.log(err.stack);
  }
}
run().catch(console.dir);

const app = express();

app.get('/api/count', (req, res) => {
  count.findOneAndUpdate({}, { $inc: { count: 1 } }, { returnNewDocument: true, upsert: true }).then((doc) => {
    res.status(200).json(doc ? doc.count : 0);
  })
})

app.all('*', (req, res) => {
  res.status(404).end();
})

app.listen(80);
```

Nous réutilisons le même code que pour le chapitre précédent, en créant simplement la route */api/count*.

#### Création du *Dockerfile*

Nous créons le fichier *api/Dockerfile* :

```dockerfile
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 80
CMD [ "npm", "start" ]
```

Nous créons le script correspondant dans *package.json*:

```json
"scripts": {
  "start": "nodemon ./src/index.js",
}
```

#### Modification de *docker-compose.dev.yml*

Nous modifions notre configuration pour *Docker Compose* :

```yaml
version: "3.9"
services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.dev
    volumes:
      - type: bind
        source: ./client
        target: /home/node
      - type: volume
        target: /home/node/node_modules
    ports:
      - 3000:3000
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    volumes:
      - type: bind
        source: ./api/src
        target: /app/src
    ports:
      - 3001:80
```

Nous devons avant de pouvoir tester, mettre en place le service pour la base de données.

### Mise en place du service pour la base de données

#### Modification de *docker-compose.dev.yml*

Nous modifions notre configuration pour *Docker Compose* pour ajouter notre service de base de données *MongoDB* :

```yaml
version: "3.9"
services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.dev
    volumes:
      - type: bind
        source: ./client
        target: /home/node
      - type: volume
        target: /home/node/node_modules
    ports:
      - 3000:3000
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    volumes:
      - type: bind
        source: ./api/src
        target: /app/src
    ports:
      - 3001:80
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbtest
        target: /data/db
volumes:
  dbtest:
```

#### Création de la collection et du document

Nous lançons la base de données :

```sh
docker compose -f  docker-compose.dev.yml run db
```

Puis nous nous connectons au client *MongoDB* :

```sh
docker container exec -it ID mongosh
```

Nous utilisons la bonne *db* :

```sql
use test
```

Nous insérons un document :

```sql
db.count.insertOne({count: 0})
```

Puis nous quittons :

```sql
exit
```

Pour éviter de le faire à chaque fois et pour éviter que les membres de votre équipe ne doivent utiliser un *README*, vous pouvez aussi simplement modifier *api/src/index.js* et plus particulièrement la méthode de connexion :

```js
MongoClient.connect(MongUrl, async (err, client) => {
  if (err) {
    console.log(err);
  } else {
    console.log('CONNEXION DB OK !');
    count = client.db('test').collection("count");
    if ((await count.countDocuments()) === 0) {
      count.insertOne({ count: 0 });
    }
  }
});
```

Nous verrons plus tard comment utiliser des scripts d'initialisation pour la base de données.

Nous pouvons maintenant vérifier que nos services fonctionnent bien :

```sh
docker compose -f docker-compose.dev.yml up
```

Nous testons ensuite notre *API* en ouvrant un navigateur et en allant sur *localhost:3001/api/count*.

Nous testons également que l'application *React* est disponible : *localhost:3000*. 

