# Utiliser Docker Compose

## Introduction à Docker compose

### Qu'est-ce que *Docker Compose* ?

***Docker Compose* permet de faciliter la création d'application multi-conteneur.**

Dans une application vous avez au moins deux ou trois conteneurs : base de données, backend et application client.

Souvent viennent s'ajouter un reverse proxy pour permettre le routing vers différentes applications et de la répartition de charge (*load balancing*).

*Docker Compose* utilise des fichiers de configuration utilisant le langage *YAML*. Nous en verrons la syntaxe dans la leçon suivante.

***Docker Compose* ne remplace pas du tout les *Dockerfiles* !**

***Docker Compose* s'utilise de la manière suivante :**

1 - Votre application comporte plusieurs *Dockerfiles*, un par image que vous voulez utiliser.

2 - Votre application a un fichier *docker-compose.yml* qui définit les services que votre application a besoin pour fonctionner. Un service est par exemple une base de données, une *API Rest*, un *reverse proxy*, une application *SPA* etc.

3 - Vous lancez la commande `docker compose up` qui va démarrer tous les services suivant les options définies.

Nous reviendrons bien évidemment sur l'ensemble de ces étapes en détails.

### Les avantages à utiliser Docker Compose

**Pour le développement,** plus besoin de guide d'installation pour votre équipe ! En une seule commande toute l'application s'installe et démarre avec la configuration adéquate. Plus besoin d'installer les dépendances pour chaque partie de votre application vous même, de mettre les bons ports, les bons réseaux, les bons volumes etc.

**Pour les test,** la possibilité de mettre en place votre application rapidement et de lancer des tests automatiquement accélère grandement vos mises en production.

`Docker Compose` ne recrée que les conteneurs dont la configuration a changé, sinon il les réutilise. Cela permet d'accélérer grandement l'arrêt et le redémarrage de votre application, ce qui est particulièrement utile en développement.

Pour vérifier que Docker Compose est bien installé :

```sh
docker compose version
```
## Première utilisation de Docker Compose

### Le langage *YAML*

Le *YAML* (pour *YAML Ain't Markup Language* ou "YAML n'est pas un langage de balisage") est un langage permettant de représenter des informations élaborées tout en conservant une grande lisibilité. Il repose principalement sur l'indentation.

C'est un langage utilisé le plus souvent pour des fichiers de configuration.

Le langage *YAML* est extrêmement utilisé dans le monde du *devops* : *Docker*, tous les outils de *CI*, *ansible* et tant d'autres utilisent ce langage simple.

Nous allons voir les principaux éléments de la syntaxe de ce langage car tout ce que nous verrons à partir de maintenant utilisera du *YAML* :

`#` permet de commenter une ligne s'il est placé au début d'une ligne. S'il est avant une chaîne de caractère il signifie nombre littéral.

`~` signifie valeur nulle.

`-` permet de créer des listes. Chaque élément d'une liste doit être précédé par `-` puis un espace. Il doit y avoir un élément par ligne.

`clé: valeur` permet de déclarer un map clé / valeur.

L'indentation par des espaces permet de créer une hiérarchie, par exemple :

```yaml
paul:
nom: Paul Dupont
emploi: Developer
```

Une structure plus complexe :

```yaml
- paul:
  nom: Paul Dupont
  emploi: Developer
  languages:
    - javascript
    - css
    - html
```

### Première utilisation : `docker compose up`

Commencez par créer un nouveau dossier, par exemple *compose* :

```sh
mkdir compose
cd compose
```

Dans ce dossier, créez un fichier *docker-compose.yml* ou *docker-compose.yaml* (les deux noms d'extensions sont possibles, par contre le nom du fichier est important) :

```yaml
version: "3.9"
services:
  myalpine:
    image: alpine
```

Ici nous définissons un service *myalpine*. Le nom des services est totalement libre. La recommandation est d'utiliser un nom descriptif de leur rôle, par exemple *api*, *db* etc.

La clé *image* permet de spécifier sur quelle image est basée le service. Ici, nous utilisons l'image *alpine*.

Sauvegardez et faites :

```sh
docker compose up
```

Vous aurez alors :

```sh
Creating network "compose_default" with the default driver
Pulling myalpine (alpine:)...
latest: Pulling from library/alpine
188c0c94c7c5: Pull complete
Digest: sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e7841a5b5a
Status: Downloaded newer image for alpine:latest
Creating compose_myalpine_1 ... done
Attaching to compose_myalpine_1
compose_myalpine_1 exited with code 0
```

Chaque ligne est importante :

*Creating network "compose_default"* indique que *Docker* crée un réseau *bridge* par défaut (qui n'est propre à *Docker Compose* et n'est pas le réseau *bridge* que nous avions par défaut). Le nom du réseau est le nom du dossier où se trouve le fichier *docker-compose.yml* postfixé avec *_default*.

Vous pouvez retrouver ce nouveau réseau en faisant :

```sh
docker network ls
```

Ensuite, *Docker Compose* va cherche l'image indiquée pour le service *myalpine*.

Il va *pull* l'image sur *Docker Hub* comme `docker run` le ferait.

Il va ensuite démarrer un conteneur basé sur cette image *Creating compose_myalpine_1*. Le nom du conteneur est le nom du dossier, puis le nom du service et enfin un chiffre qui s'incrémente si nous lançons plusieurs conteneurs pour un service (nous verrons cela avec *swarm*).

Enfin, comme avec `docker run`, *Docker Compose* va attacher le conteneur lancé au terminal pour avoir les sorties d'erreur et standard.

Dans notre exemple il quitte immédiatement car aucune commande n'est passée.

### Lancer une commande sur un service avec `docker compose run`

**La commande `docker compose run` permet d'exécuter une commande par un service.**

Par exemple :

```sh
docker compose run myalpine
```

Va démarrer un conteneur pour le service *myalpine* et exécuter la commande par défaut (qui est *sh* pour l'image *alpine*).

La commande va utiliser toute la configuration définie pour le service (pour le moment nous n'en avons pas) à l'exception des ports, pour éviter d'éventuelles collisions avec des ports déjà ouverts.

Vous pouvez remplacer la commande par défaut de l'image en passant une autre commande en argument :

```sh
docker compose run myalpine ls
```

Pour l'image *alpine*, la commande par défaut est *sh*. Vous remplacez donc *sh* par `ls`.

### Lister les conteneurs lancés avec `Docker compose`

Vous pouvez lister tous les conteneurs en cours d'exécution en faisant :

```sh
docker container ls
```

Mais si vous voulez uniquement les conteneurs lancés par `Docker Compose` vous pouvez faire :

```sh
docker compose ps
```

Vous pouvez également lui passer l'option `-a` pour voir également les conteneurs lancés par `docker compose run`.

### Stopper les conteneurs lancés avec `Docker compose`

Vous pouvez stopper tous les conteneurs lancés par `Docker compose` avec la commande :

```sh
docker compose down
```

La commande **va automatiquement supprimer tous les conteneurs stoppés,** comme si vous les aviez lancés avec l'option `--rm`.

Il va également supprimer les réseaux.

Pour supprimer également les volumes créés par `Docker Compose`, anonymes et nommés, il suffit de spécifier l'option `-v` :

```sh
docker compose down -v
```

A noter que les volumes anonymes ne sont jamais réutilisés par `Docker Compose` comme nous le verrons. Il en lance des nouveaux à chaque fois si vous en déclarez dans votre configuration.

### Premières configurations

Les configurations des services sont appliquées à chaque démarrage de conteneur pour ces services.

Cela revient à exécuter `docker run` avec toutes les options définies en configuration, ce qui est un énorme gain de temps !

#### *command*

La configuration *command* permet de remplacer la commande par défaut de l'image.

Essayez en faisant :

```yaml
version: "3.9"
services:
myalpine:
  image: alpine
  command: ls
```

Refaites :

```sh
docker compose up
```

Pour les commandes de plus d'un mot, il vaut mieux utiliser l'autre syntaxe que nous connaissons : *command: ["npm", "run", "start"]*.

#### *entrypoint*

La configuration *entrypoint* permet de remplacer l'*entrypoint* par défaut de l'image. Nous n'allons pas revoir la distinction avec la commande que nous avons vu en détails dans un chapitre précédent.

Essayez en faisant :

```yaml
version: "3.9"
services:
myalpine:
  image: alpine
  entrypoint: ls
```

Refaites :

```sh
docker compose up
```

## Utilisation d'images personnalisées avec Docker Compose

Dans cette leçon nous allons voir comment `Docker Compose` s'utilise avec des images personnalisées. Autrement dit, comme il s'utilise avec des *Dockerfiles*.

### Utilisation avec un *Dockerfile*

Commencez par créer un *Dockerfile* dans le même dossier :

```dockerfile
FROM alpine
CMD ["/bin/sh"]
```

Ensuite, modifiez le fichier *docker-compose.yml* pour l'utiliser :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build: .
```

Nous avons maintenant deux services : un premier, le service **a**, qui utilise l'image officielle *alpine*, et un second qui utilise notre *Dockerfile*.

En effet, l'option de configuration *build* permet de définir le chemin du contexte de *build*. Il n'y a rien besoin d'autre si le *Dockerfile* et dans le même dossier.

Si le *Dockerfile* **a** un autre nom et / ou si le contexte de *build* est dans un autre dossier, il faut utiliser la configuration plus détaillée, par exemple :

```yaml
build:
 context: ./dossier
 dockerfile: Dockerfile.dev
```

Créons un dossier *backend* et plaçons y notre *Dockerfile*. Nous devons maintenant écrire :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build:
      context: ./backend
      dockerfile: Dockerfile
```

### Construire les images avec `docker compose build`

**La commande `docker compose build` permet de construire toutes les images spécifiées dans le fichier de configuration (dans les configurations *build*) et de les taguer pour une future utilisation.**

Essayez de faire :

```sh
docker compose build
```

Et listez ensuite les images :

```sh
docker image ls
```

L'image sera taguée comme ceci : **compose_b**, à savoir nom du dossier contenant le fichier *docker-compose.yml* puis le nom du service.

### Passer des arguments et des labels durant le *build*

Nous avons déjà étudié l'instruction *ARG* pour passer des arguments et *LABEL* pour indiquer des labels dans des images personnalisées.

Il est possible de la même manière de passer des arguments ou des labels durant le *build* de l'image dans un fichier de configuration `Docker Compose`. En utilisant simplement *args* et *labels* dans la configuration du *build*.

Par exemple, si nous avons dans notre *Dockerfile* :

```dockerfile
FROM alpine
ARG FOLDER
RUN mkdir $FOLDER
CMD ["/bin/sh"]
```

Vous pouvez passer l'argument dans le fichier de configuration de `Docker Compose` :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build:
      context: ./backend
      dockerfile: Dockerfile
      args:
        - FOLDER=test
```

Vous pouvez tester :

```sh
docker compose build
```

Vous pouvez également passer des labels :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build:
      context: ./backend
      dockerfile: Dockerfile
      args:
        - FOLDER=test
      labels:
        - email=jean@gmail.com
```

## Les ports et les volumes avec Docker Compose

### Publier des ports

Vous pouvez définir quels ports doivent être publiés pour un service dans le fichier de configuration *docker-compose.yml*.

Pour ce faire, il suffit d'utiliser *ports*. Le format est toujours *HOTE:CONTENEUR* puis par défaut *TCP*. Vous pouvez préciser *UDP* avec */upd* :

```yaml
ports:
  - "80:80"
  - "8080:3000/udp"
```

Nous y reviendrons bien sûr en détails avec plusieurs exemples.

### Créer des volumes

Il est possible de monter des chemins de l'hôte (*bind mount*) ou de créer des volumes anonymes ou nommés.

#### Créer des bind mounts

Reprenons notre exemple en créant un dossier */app* dans le *Dockerfile* :

```dockerfile
FROM alpine
ARG FOLDER
WORKDIR /app
RUN mkdir $FOLDER
CMD ["/bin/sh"]
```

Supposons que nous ayons un dossier *data* sur l'hôte que nous voulions *bind mount*, nous n'aurions qu'à faire :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
```

Il est bien sûr possible de préciser plusieurs volumes. Il faut simplement que chaque volume soit un élément de liste, donc précédé par un tiret puis un espace.

#### Créer des volumes anonymes

Pour créer un volume anonyme il faut simplement déclarer un volume de type *volume* et de spécifier uniquement une *target* (ou *destination* / *dst*) et pas de *source*. Dans ce cas `Docker Compose` créera automatiquement un volume anonyme :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
    - type: volume
      target: /app/data2
```

#### Créer des volumes nommés

Pour créer un volume nommé, il faut déclarer dans une configuration au même niveau que services une clé *volumes* : ce sont les volumes nommés qui seront automatiquement créés par `Docker Compose`.

Pour les utiliser, il suffit ensuite dans la configuration *volumes* d'un service, d'indiquer en *source* le nom du volume nommé créé :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
    - type: volume
      target: /app/data2
    - type: volume
      source: data3
      target: /app/data3
volumes:
  data3:
```

Ici *data3* est un volume nommé créé et qui est monté dans le service **b**.

Notez bien la syntaxe particulière de *data3*:. Si vous avez plusieurs volumes nommés un faut en mettre un par ligne et ne pas oublier les `:` :

```yaml
volumes:
  data1:
  data2:
  data3:
```

#### Utiliser des volumes nommés déjà créés

Si vous ne voulez pas que `Docker Compose` crée un nouveau volume nommé mais utilise un existant (que vous avez créé en dehors de `Docker Compose`), il faut préciser *external: true* dans la configuration, de cette manière :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
    - type: volume
      target: /app/data2
    - type: volume
      source: data3
      target: /app/data3
volumes:
  data3:
    external: true
```

Ici *data3* ne sera pas créé par `Docker Compose`. Il utilisera le volume déjà créé. Si aucun volume avec ce nom n'existe, vous aurez l'erreur suivante :

```sh
ERROR: Volume data3 declared as external, but could not be found. Please create the volume manually using `docker volume create --name=data3` and try again.
```

## Utiliser des variables d'environnement avec Docker Compose

### Utiliser des variables d'environnement

Vous pouvez utiliser des variables d'environnement dans votre *Dockerfile* ou votre fichier *docker-compose.yml*, par exemple pour définir une version d'image :

```yaml
backend:
  image: "node-app:${NODE_APP_VERSION}"
```

En plus de vos propres variables, de nombreuses images officielles requièrent de passer des valeurs pour des variables d'environnement. Nous l'avons déjà vu dans un exemple pour définir le mot de passe pour l'image officielle de *postgres*. Nous le verrons dans les leçons suivantes avec l'image officielle *mongo*.

### Définir la valeur des variables d'environnement avec *environment*

Le premier moyen pour définir des variables d'environnement est d'utiliser la clé de configuration *environment* :

```yaml
backend:
  environment:
    - NODE_APP_VERSION=2.2.3
```

### Utiliser un fichier externe avec env_file et .env

Par défaut, *Docker Compose* va lire les variables d'environnement dans le fichier *.env* situé au même niveau que le *docker-compose.yml*.

Vous pouvez ainsi avoir un fichier *.env* avec par exemple :

```sh
NODE_APP_VERSION=2.2.3
NODE_ENV=dev
DEBUG=1
```

Si vous voulez spécifier un fichier différent il faut utiliser la clé de configuration *env_file*. Par exemple :

```yaml
backend:
  env_file:
    - config/env.dev
```

### Définir la valeur des variables d'environnement avec *Docker CLI*

Vous pouvez déclarer des variables d'environnement lors du lancement avec `docker compose` en utilisant l'option `-e` :

```sh
docker compose run -e USER=paul up
```

Vous pouvez ne pas définir de valeur s'il s'agit d'une variable d'environnement qui a déjà une valeur disponible dans votre *env* (*cf cours **Linux***) :

```sh
docker compose run -e USER up
```

Vous pouvez enfin directement préciser l'emplacement du fichier pour les variables d'environnement à utiliser :

```sh
docker compose --env-file .config/.env.dev up
```

Cela vous permet d'avoir par exemple un fichier de variables d'environnement pour la production, par exemple dans *config/env.prod* et un fichier pour le développement *config/env.dev* par exemple.

### L'ordre de priorité

Normalement vous utiliserez soit *environment* soit un fichier externe et vous ne devrez pas rencontrer de problème, mais ayez en tête l'ordre de priorité de résolution de la valeur des variables d'environnement par *Docker Compose*.

Lorsque vous utilisez une variable d'environnement, *Docker Compose* va utiliser cet ordre :
1. Le fichier *docker-compose.yml*.
2. Les variables d'environnement de votre *shell*.
3. Le fichier des variables d'environnement défini, par exemple *.env.dev*.
4. Le fichier *Dockerfile* (si vous avez défini des valeurs dans des instructions `ENV`).

S'il ne trouve pas la valeur de la variable à tous ces endroits, et dans cet ordre, la variable sera non définie.

### Définir le nom de son projet pour *Docker Compose*

Nous avons vu que *Docker Compose* préfixe tout (les réseaux, les volumes nommés, les conteneurs lancés etc) avec le nom du dossier contenant le fichier *docker-compose.yml*.

*Docker* considère qu'il s'agit du nom du projet par défaut. Maintenant que vous connaissez les variables d'environnement sachez que vous pouvez définir le nom de votre projet avec la variable `COMPOSE_PROJECT_NAME`.

Par exemple, vous pouvez mettre dans votre fichier *.env* :

```sh
COMPOSE_PROJECT_NAME=monprojet
```

## Utiliser des réseaux avec Docker Compose

### Création d'un réseau par défaut

Nous avons vu que *Docker Compose* créait un réseau *bridge* par défaut et qu'il le nommait *nomduprojet_default*.

**Tous les services définis dans le *docker-compose.yml* rejoignent ce réseau avec leur nom définis dans ce fichier.** Par exemple :

```yaml
version: '3.9'
services:
  api:
    image: node
  db:
    image: mongo:7
```

Ici, sur le réseau créé par défaut, *api* et *db* rejoindront le réseau avec les noms *api* et *db*, ce qui est extrêmement pratique pour bénéficier de la résolution des noms automatiques en *IP*.

### Utiliser des alias avec `--link`

Vous pouvez définir des alias pour que vos services soient également accessible sur les réseaux *Docker* sous d'autres noms.

Par exemple :

```yaml
version: '3.9'
services:
  api:
    image: node
  db:
    image: mongo:7
    links:
      - 'db:database'
      - 'db:mongo'
```

Ici, le service *mongo* pourra être accessible sur le réseau par défaut sous les noms *db*, mais aussi *database* et *mongo*.

### Créer d'autres réseaux

En plus du réseau par défaut, il est possible de définir d'autres réseaux. Dans ce cas,il faut les déclarer au plus au niveau comme pour les volumes nommés, puis les utiliser dans les services.

Prenons un exemple :

```yaml
version: '3.9'
services:
  proxy:
    image: nginx
    networks:
      - frontend
  app:
    image: nginx
    networks:
      - frontend
      - backend
  api:
    image: node
    networks:
      - backend
  db:
    image: mongo:7
    networks:
      - backend
networks:
  frontend:
  backend:
```

Ici nous avons créé deux réseaux : *frontend* et *backend* qui sont de type *bridge* par défaut avec *Docker Compose*.

Nos services sont connectés à l'un, à l'autre, ou aux deux grâce à la clé de configuration *networks*.

### Changer le nom du réseau par défaut

Comme vous le savez, le réseau par défaut est préfixé par le nom de l'application.

Il est possible de changer son nom par défaut en faisant :

```yaml
version: '3.9'
services:
  api:
    image: node
  db:
    image: mongo:7
networks:
  default:
    name: monreseau
```

Il est possible de configurer également les réseaux créés de la même manière :

```yaml
version: '3.9'
services:
  proxy:
    image: nginx
    networks:
      - frontend
  app:
    image: nginx
    networks:
      - frontend
      - backend
  api:
    image: node
    networks:
      - backend
  db:
    image: mongo:7
    networks:
      - backend
networks:
  frontend:
    name: frontend
  backend:
    name: backend
```

## Utiliser Docker Compose avec notre exemple

### Application d'exemple

Nous allons reprendre notre application d'exemple et utiliser *Docker Compose*.

Pour rappel nous avons un dossier *src* dans lequel nous avons un fichier *app.js* :

```js
const express = require('express');
const { MongoClient } = require('mongodb');
const app = express();

const client = new MongoClient('mongodb://db');
let count;

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

app.get('/', (req, res) => {
  count
    .findOneAndUpdate({}, { $inc: { count: 1 } }, { returnNewDocument: true, upsert: true })
    .then((doc) => {
      res.status(200).json(doc ? doc.count : 0);
    });
});

app.listen(80);
```

Nous avons à la racine un fichier *Dockerfile*:

```dockerfile
FROM node:alpine
WORKDIR /app
COPY ./package.json .
RUN npm install
COPY . .
ENV PATH=$PATH:/app/node_modules/.bin
CMD [ "nodemon", "src/app.js" ]
```

*Sur Windows uniquement*, il se peut que le rechargement automatique ne fonctionne pas avec cette commande il faut donc remplacer par *CMD [ "nodemon", "src/app.js", "-L"]* . Cette option permet d'utiliser le *legacyWatch* qui peut être nécessaire pour Windows.

Et un fichier *package.json* :

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^6.1.0",
    "nodemon": "^3.0.1"
  }
}
```

### Création de notre service pour notre base de données

Nous allons commencer par créer un fichier *docker-compose.yml*.

Puis nous allons y définir notre service de base de données :

```yaml
version: '3.9'
services:
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: mydb
        target: /data/db

volumes:
  mydb:
    external: true
```

Ici notre service s'appelle *db*. Il utilise l'image *mongo* officielle.

Nous avons un volume nommé qui est externe (c'est-à-dire non créé par *Docker Compose*) appelé *mydb*.

Nous le montons dans le chemin nécessaire pour l'image *mongo* : */data/db*.

Nous devons donc créer le volume nous-mêmes :

```sh
docker volume create mydb
```

Nous pouvons ensuite lancer notre service en mode détaché :

```sh
docker compose run -d db
```

*Docker Compose* retourne alors l'*ID* du conteneur, par exemple :

```
docker-test_db_run_2031d0db2c39
```

Vous pouvez vous connecter dessus :

```sh
docker exec -it docker-test_db_run_2031d0db2c39 mongosh
```

Une fois sur le *CLI mongo* nous pouvons utiliser la base de données de test :

```sql
use test
```

Nous pouvons ensuite recréer notre collection et notre document :

```sql
db.count.insertOne({count: 0});
db.count.findOne()
```

Nous pouvons quitter :

```sh
exit
docker compose down
```

*Docker Compose* stoppe et supprime le conteneur lancé pour le service et supprime le réseau par défaut :

```sh
Stopping docker-test_db_run_4af54c203c46 ... done
Removing docker-test_db_run_4af54c203c46 ... done
Removing network docker-test_default
```

## Mettre en place le service pour le serveur

### Création du service pour notre serveur

Nous modifions notre fichier *docker-compose.yml* pour créer le service pour notre serveur :

```yaml
version: '3.9'
services:
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: mydb
        target: /data/db
  server:
    build: .
    ports:
      - 80:80
    volumes:
      - type: bind
        source: ./src
        target: /app/src

volumes:
  mydb:
    external: true
```

Nous pouvons ensuite lancer notre application :

```sh
docker compose up
```

Nous pouvons ensuite nous rendre sur *localhost* dans un navigateur.

### Mise en place d'une authentification pour *MongoDB*

Nous devons supprimer notre volume pour le recréer avec l'authentification activée :

```sh
docker compose down
docker volume rm mydb
```

Nous le recréons :

```sh
docker volume create mydb
```

#### Initialisation de la base de données avec l'authentification activée

Nous modifions notre fichier *docker-compose.yml* pour utiliser les variables d'environnement spécifiées par l'image officielle *mongo* pour activer l'authentification :

```yaml
version: '3.9'
services:
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: mydb
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
  server:
    build: .
    ports:
      - 80:80
    volumes:
      - type: bind
        source: ./src
        target: /app/src

volumes:
  mydb:
    external: true
```

Pour trouver les configurations possibles il suffit de se rendre sur *Docker Hub* sur la page de l'image. Vous aurez toujours une documentation précisant les options et les cas d'utilisation possibles.

Nous créons ensuite un fichier *.env* pour passer les valeurs :

```sh
MONGO_INITDB_ROOT_USERNAME=jean
MONGO_INITDB_ROOT_PASSWORD=123456
```

Nous lançons enfin notre application

```sh
docker compose up -d
```

Nous nous connectons au client *mongo* :

```sh
docker compose exec db mongosh
```

Nous passons sur la base de données *test* :

```sh
use test
```

Nous essayons d'insérer un document :

```sql
db.count.insertOne({count: 0});
```

Nous avons cette fois bien une erreur :

```json
WriteCommandError({
        "ok" : 0,
        "errmsg" : "command insert requires authentication",
        "code" : 13,
        "codeName" : "Unauthorized"
})
```

Il faut nécessairement s'authentifier pour exécuter des commandes.

Nous pouvons alors nous authentifier dans la console :

```sql
use admin
db.auth({user: 'jean', pwd: '123456'})
```

La console retourne alors `1`, ce qui signifie que l'authentification s'est bien déroulée.

Nous pouvons maintenant créer notre document :

```sql
use test
db.count.insertOne({count: 0});
```

#### Création d'un utilisateur dans la base de données *MongoDB* pour notre application

Nous allons ensuite créer un utilisateur pour notre application serveur. Cet utilisateur *MongoDB* pourra uniquement lire et écrire dans la base de données test mais il ne pourra pas gérer les bases de données et les utilisateurs.

```sql
use admin
db.createUser({user: 'nodeapp', pwd: '123456', roles:[{role: 'readWrite', db: 'test'}]})
```

Nous avons en retour la confirmation :

```json
Successfully added user: {
        "user" : "nodeapp",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "test"
                }
        ]
}
```

Maintenant que l'authentification est activée, l'application ne pourra plus se connecter :

```sh
docker compose up
```

Puis rendez vous sur *localhost* dans un navigateur.

Dans votre terminal vous aurez alors l'erreur suivante :

```sh
MongoError: command findAndModify requires authentication
```

Il faut que nous authentifions notre application serveur.

```sh
docker compose down
```

#### Authentification du serveur

Nous allons utiliser des variables d'environnement pour passer au serveur les identifiants de la base de données lors du lancement.

Pour ce faire, nous allons modifier *src/app.js* :

```js
const express = require('express');
const { MongoClient } = require('mongodb');
const app = express();

const client = new MongoClient(`mongodb://${process.env.MONGO_USERNAME}:${process.env.MONGO_PWD}@db`);

let count;

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

app.get('/', (req, res) => {
  count
    .findOneAndUpdate({}, { $inc: { count: 1 } }, { returnNewDocument: true, upsert: true })
    .then((doc) => {
      res.status(200).json(doc ? doc.count : 0);
    });
});

app.listen(80);
```

Dans le fichier *.env*, nous ajoutons deux nouvelles variables :

```sh
MONGO_INITDB_ROOT_USERNAME='nodeapp'
MONGO_INITDB_ROOT_PASSWORD=123456
MONGO_USERNAME='nodeapp'
MONGO_PWD=123456
```
Il ne nous reste plus qu'à passer ces variables lors du lancement en modifiant *docker-compose.yml* :

```yaml
version: '3.9'
services:
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: mydb
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
  server:
    build: .
    ports:
      - 80:80
    volumes:
      - type: bind
        source: ./src
        target: /app/src
    environment:
      - MONGO_PWD
      - MONGO_USERNAME
volumes:
  mydb:
    external: true
```

L'application est maintenant authentifiée :

```sh
docker compose up
```

## Les configurations depends_on et restart

### La configuration *depends_on*

**La configuration *depends_on* permet de spécifier qu'un service dépend d'autres services.**

Cela entraîne les comportements suivants :

`docker compose up` va démarrer les services qui sont des dépendances d'autres services avant.

`docker compose up SERVICE` va inclure automatiquement le démarrage de toutes les dépendances du service.

`docker compose down` va d'abord stopper les dépendances avant les autres services.

Prenons un exemple :

```yaml
version: "3.9"
services:
  node:
    image: node
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: mongo:7
```

Ici, les bases de données *redis* et *mongo* seront démarrées avant node car node les a déclarés en dépendances.

**Attention ! *depends_on* n'attend pas que les services soient prêts, il ne fait aucun test.** Il garantit uniquement que les conteneurs soient en cours d'exécution avant que le conteneur qui a des dépendances ne soit lancé.

### La configuration *restart*

Par défaut, les conteneurs ne sont pas redémarrés si ils sont stoppés : que ce soit à cause d'une erreur, manuellement ou si le démon *dockerd* est arrêté (redémarrage de la machine physique, erreur du démon etc).

Il est possible de changer ce comportement en spécifiant pour chaque service, quelle est la politique de redémarrage pour celui-ci. Pour cela, il faut utiliser la clé de configuration *restart*.

Les valeurs possibles sont :
- **none :** celle par défaut. Les conteneurs ne sont pas redémarrés, y compris en cas d'erreur.
- **on-failure :** permet de redémarrer le conteneur en cas d'erreur.
- **always :** redémarre tout le temps le conteneur. S'il est stoppé manuellement (`docker container stop`), il n'est redémarré **que si le démon *dockerd* est redémarré.**
- **unless-stopped :** redémarre tout le temps le conteneur. S'il est stoppé manuellement (`docker container stop`), il n'est jamais redémarré.

En reprenant notre exemple, nous pourrions faire :

```yaml
version: '3.9'
services:
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: mydb
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
    restart: 'unless-stopped'
  server:
    build: .
    ports:
      - 80:80
    volumes:
      - type: bind
        source: ./src
        target: /app/src
    environment:
      - MONGO_PWD
      - MONGO_USERNAME
    depends_on:
      - db
    restart: 'unless-stopped'
volumes:
  mydb:
    external: true
```

Notez que vous pouvez modifier la configuration du redémarrage d'un conteneur déjà en cours d'exécution en faisant :

```sh
docker container update --restart unless-stopped ID
```

## Les autres commandes de docker-compose

### Afficher les logs

Pour afficher les logs de tous les services lancés par *Docker Compose* vous pouvez faire :

```sh
docker compose logs
```

Si vous voulez suivre le fichier de log, et donc afficher les logs en temps réel, il faut utiliser l'option `-f` ou `--follow` :

```sh
docker compose logs -f
```

### Afficher les services en cours d'exécution

Pour afficher l'ensemble des conteneurs lancés pour les services par *Docker Compose*, il suffit de faire :

```sh
docker compose top
```

### Supprimer les conteneurs stoppés

Pour supprimer les conteneurs stoppés qui avaient été lancés par *Docker Compose*, il suffit de faire :

```sh
docker compose rm
```

Pour ne pas avoir à confirmer, vous pouvez passer l'option `-f` ou `--force` :

```sh
docker compose rm -f
```

Pour stopper et supprimer les conteneurs, vous pouvez faire :

```sh
docker compose rm -s
```

Et pour n'avoir aucune confirmation à faire :

```sh
docker compose rm -sf
```

Si vous voulez aussi supprimer les volumes anonymes attachés aux conteneurs :

```sh
docker compose rm -v
```

Donc pour stopper les conteneurs, supprimer les volumes anonymes attacher et ne pas avoir à entrer de confirmation :

```sh
docker compose rm -svf
```

### Vérifier le mapping des ports d'un service

Pour vérifier qu'un port est bien publié sur un service vous pouvez faire :

```sh
docker compose port SERVICE PORT
```

Par exemple :

```sh
docker compose port server 80
```

Qui vous affichera `0.0.0.0:80` s'il est bien publié, sinon rien.

### Afficher la configuration de *Docker Compose*

Pour afficher la configuration qui sera lancée, notamment avec les valeurs des variables d'environnement, vous pouvez faire :

```sh
docker compose config
```

### Télécharger les dernières versions des images

Pour télécharger l'ensemble des images utilisées par vos services dans votre *docker-compose.yml*, vous pouvez faire :

```sh
docker compose pull
```

Cela peut être utile pour mettre à jour les images *latest* qui sont disponibles localement. 