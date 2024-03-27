# Mise en production avec Docker et Docker compose

## Introduction à la mise en production

### Objectifs du chapitre

Dans ce chapitre, nous allons voir comment mettre en production l'application que nous avons créée dans le chapitre précédent sur un serveur.

Nous allons pour ce faire devoir nous connecter au serveur en *SSH* et créer un certificat *TLS* pour pouvoir servir l'application de manière sécurisée.

Pour la partie *API*, nous allons voir comment utiliser *pm2* avec *Docker* afin de lancer plusieurs processus *Node.js* et augmenter les performances de notre microservice *API*.

Il s'agit d'un chapitre avancé qui demande d'avoir vu les cours *Linux*, *Git* et *Node.js*. Nous ne détaillerons pas toutes les explications ici car le chapitre serait bien trop long, aussi si vous ne comprenez pas une partie tournez vous vers le cours correspondant !

### Utilisation d'un *VPS*

Un serveur dédié virtuel (*VPS* en anglais) est une méthode de division d'un serveur en plusieurs serveurs virtuels indépendants. Cela permet aux hébergeurs de proposer des offres à bas coûts en louant des "bouts de serveur", plutôt que des serveurs dédiés, qui sont beaucoup plus onéreux.

Chaque serveur virtuel peut fonctionner avec un système d'exploitation différent et redémarrer indépendamment.

Les détails techniques de l'isolation sont totalement différents suivant les hébergeurs, ils reposent sur des technologies de virtualisation (machines virtuelles).

### Gestion du signal *SIGINT*

Nous allons modifier *api/src/index.js* pour mettre en place ce qu'on appelle un *graceful shutdown*.

Cela signifie que lorsque l'application *Node.js* reçoit un signal d'arrêt (*cf le cours Linux pour les signaux*), nous voulons couper le serveur pour ne plus qu'il accepte de requête entrante, puis une fois qu'il est coupé, couper la connexion à la base de données.

De cette manière, nous sommes sûr que lorsque nous ferons *docker compose stop* par exemple, notre application *Node.js* sera coupée proprement sans que des requêtes soient en cours de traitement, aussi bien au niveau du serveur que de la base de données.

```js
const express = require('express');
const { MongoClient } = require('mongodb');
const client = new MongoClient(uri);
let count;

const uri =
  process.env.NODE_ENV === 'production'
    ? `mongodb://${process.env.MONGO_USERNAME}:${process.env.MONGO_PWD}@db`
    : `mongodb://db`;

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
  count
    .findOneAndUpdate({}, { $inc: { count: 1 } }, { returnNewDocument: true, upsert: true })
    .then((doc) => {
      res.status(200).json(doc ? doc.count : 0);
    });
});

app.all('*', (req, res) => {
  res.status(404).end();
});

const server = app.listen(80);

// Graceful shutdown.
// On empêche les nouvelles connexions sur le serveur
// Ensuite on close proprement la connexion DB
process.on('SIGINT', () => {
  server.close((err) => {
    if (err) {
      process.exit(1);
    } else {
      if (client) {
        client.close((err) => {
          process.exit(err ? 1 : 0);
        });
      } else {
        process.exit(0);
      }
    }
  });
});
```

### Utilisation d'une version *LTS*

Pour la production il vaut mieux toujours utiliser les versions stables.

Pour l'image de *Node.js*, dans sa version *Alpine*, la version stable est *node:lts-alpine*.

Aussi, nous modifions le *Dockerfile* dans le dossier *api* :

```dockerfile
FROM node:lts-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
RUN npm install pm2 -g
COPY . .
CMD ["pm2-runtime", "ecosystem.config.js"]
EXPOSE 80
```