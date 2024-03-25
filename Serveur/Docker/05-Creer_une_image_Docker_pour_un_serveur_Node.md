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

### Création du Dockerfile

#### Mise en place du *Dockerfile*

Dans le *Dockerfile* nous mettons pour le moment :

```dockerfile
FROM node:alpine
WORKDIR /app
COPY . .
RUN npm install
CMD [ "nodemon", "/app/app.js" ]
```

La première instruction `FROM` permet de récupérer la dernière version de *Node.js* basée sur une image *alpine*. Comme nous l'avons vu, la distribution *alpine* est souvent recommandée car elle est très légère et sécurisée.

La seconde instruction `WORKDIR` permet de configurer le répertoire de travail pour les instructions suivantes. Elle permet également de créer le dossier s'il n'existe pas comme c'est le cas ici.

La troisième instruction `COPY`, permet de copier tout le contenu du contexte de *build* (contenu du dossier où est le *Dockerfile* par défaut) dans le **WORKDIR**.

La dernière instruction permet de lancer *nodemon*. C'est un outil de développement pour *Node.js* qui permet de recharger l'application à chaque changement.

*Sur Windows uniquement*, il se peut que le rechargement automatique ne fonctionne pas avec cette commande il faut donc remplacer par `CMD [ "nodemon", "src/app.js", "-L"]` . Cette option permet d'utiliser le *legacyWatch* qui peut être nécessaire pour Windows.

#### Construire l'image

Il ne nous reste plus qu'à construire l'image :

```sh
docker build -t myapp .
```

Nous pouvons essayer de lancer un conteneur sur l'image :

```sh
docker container run myapp
```

Mais cela ne fonctionne pas ! Il va falloir corriger plusieurs problèmes, ce qui va vous permettre de maîtriser les problèmes les plus communs lorsque l'on débute avec *Docker*. 

### Corriger le PATH

#### Le problème du *path*

Lorsque vous essaierez de lancer un conteneur basé sur votre image, vous aurez :

```sh
Error: Cannot find module '/app/nodemon'
```

Cette erreur signifie que *node* ne sait pas où se trouve le programme */app/nodemon* et qu'il ne peut donc pas le lancer.

Il y a beaucoup de manières de résoudre cette erreur. Nous allons en voir quelques unes.

#### Installer *nodemon* en global dans l'image

Une manière serait par exemple d'installer *nodemon* en global pour que la dépendance soit accessible dans le *PATH* par défaut :

```dockerfile
FROM node:alpine
WORKDIR /app
COPY . .
RUN npm install -g nodemon && npm install
CMD [ "nodemon", "/app/app.js" ]
```

#### Modifier la variable d'environnement *PATH*

Une autre manière, est d'ajouter le *PATH* contenant *nodemon* :

```dockerfile
FROM node:alpine
WORKDIR /app
COPY . .
RUN npm install
ENV PATH=$PATH:/app/node_modules/.bin
CMD [ "nodemon", "/app/app.js" ]
```

Dans ce cas, nous redéfinissons la variable d'environnement *PATH* du conteneur, en concaténant sa valeur actuelle (*$PATH*) avec le chemin vers les binaires du dossier des dépendances de *node*.

#### Utiliser *npm*

Encore une autre manière est d'utiliser le *CLI* de *npm* pour lancer *nodemon*. Il n'aura pas de problème de *PATH* dans ce cas.

Pour ce faire, il suffit d'ajouter un *script start* dans le *package.json* :

```json
{
  "scripts": {
    "start": "nodemon /app/app.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "nodemon": "^3.0.1"
  }
}
```

Et d'utiliser ensuite ce *script* dans le *Dockerfile* :

```dockerfile
FROM node:alpine
WORKDIR /app
COPY . .
RUN npm install
CMD [ "npm", "start" ]
```

Notez que la version à privilégier est la modification du *PATH* car en utilisant un *script* le processus qui lancera *nodemon* est *npm*. Lorsque vous quitterez le conteneur avec *CTRL + C* vous aurez une erreur car le signal d'interruption sera envoyé à *npm* et le processus enfant *nodemon* ne recevra pas le bon signal et renverra alors une erreur.

Nous avons réglé le problème du *path*. Nous reconstruisons donc notre image :

```sh
docker build -t myapp .
```

Mais cela ne fonctionne toujours pas :

```sh
docker run -it myapp
```

Lorsque nous allons sur *localhost* il ne se passe rien !

### Publier et exposer des ports

#### L'isolation par défaut du réseau d'un conteneur

Par défaut, le réseau d'un conteneur est isolé.

Il a une interface réseau avec une adresse *IP*, une passerelle, une table de routage, un service *DNS* etc. Autrement dit, c'est comme s'il s'agissait d'une machine avec un réseau propre.

Cela signifie que par défaut, lorsque l'on crée un conteneur avec `docker create`, ou qu'on l'exécute directement avec `docker run`, **tous les ports du conteneur sont fermés.**

**Pour rendre un ou plusieurs ports disponibles à un service extérieur à *Docker* il faut les publier.**

**Pour publier un port il faut utiliser l'option `--publish` ou `-p`.**

C**ela va ajouter une règle dans le pare-feu et va lier un port du conteneur à un port sur la machine hôte.**

La syntaxe est : `-p PORT_HOTE:PORT_CONTENEUR`.

Par exemple :

```sh
-p 8000:80
```

Permet de connecter le port *TCP 80* du conteneur au port *8000* de la machine hôte.

Dans notre exemple, si nous relançons le conteneur en faisant :

```sh
docker run -it -p 80:80 test
```

Nous pouvons maintenant afficher la page sur *localhost*.

Essayez de lancer le conteneur en modifiant la liaison des ports hôte / conteneur, par exemple :

```sh
docker run -it -p 8080:80 test
```

Cette fois le port sur la machine hôte sera *8080*, vous pourrez voir l'application sur *localhost:8080*.

#### L'instruction `EXPOSE`

**L'instruction `EXPOSE` permet d'informer *Docker* qu'un conteneur écoute sur les ports spécifiés lors de son exécution.**

Par exemple :

```sh
EXPOSE 80
```

Signifie que le conteneur écoute le port 80 en *TCP* (voir le cours *Node.js* pour les protocoles *TCP / UDP*).

Pour préciser *UDP* il faut faire :

```sh
EXPOSE 80/udp
```

Il y a très peu de chance que vous utilisiez *UDP* donc vous pouvez l'ignorer pour le moment.

**Attention !** L'instruction `EXPOSE` ne publie pas le port spécifié. Il faut toujours passer l'option `-p`.

Cela permet de documenter quels ports doivent être ouverts pour exécuter le conteneur avec `-p`.

Autrement dit, cela permet d'informer l'utilisateur du conteneur quels ports il doit publier. 

