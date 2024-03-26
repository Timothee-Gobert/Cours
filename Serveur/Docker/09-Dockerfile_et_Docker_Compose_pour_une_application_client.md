## Dockerfile et Docker Compose pour une application client

### Mise en place du projet d'application cliente

#### Objectif du chapitre

L'objectif du chapitre est d'utiliser *Docker* avec une application client de type *SPA* (*Single Page Application*), comme *Angular*, *Vue*, *React* mais cela fonctionnerait de la même manière avec toute autre application client utilisant *Webpack*.

L'objectif est d'obtenir une environnement de développement et un environnement de production optimisés avec *Docker*.

Pour l'environnement de développement, il faut que le *live reload* fonctionne parfaitement et qu'il nécessite une seule commande pour que les autres membres de l'équipe puissent le lancer.

Pour l'environnement de production, il faut que l'image soit la plus petite / performante possible : idéalement qu'elle contienne le *build* de l'application mais pas les dépendances (*node_modules*) et pas tout le code source. Il faut ensuite que le *build* soit servi de manière optimale.

Nous allons donc voir dans ce chapitre comment faire pour obtenir cela. Nous avons choisi *React* car il n'y a rien à installer à par *Node.js* et c'est donc le plus simple, mais ce serait **exactement pareil avec *Angular* ou *Vue*** (les trois utilisant *Webpack*).

#### Installation de *Node.js*

Pour installer *Node.js* quel que soit votre OS c'est très simple. Vous pouvez utiliser un exécutable en le téléchargeant [ici](https://nodejs.org/en/download/).

Si vous êtes sur *GNU/Linux*, *MacOS* ou *Windows WSL* nous vous recommandons *Node Version Manager* (*NVM*). Il permet d'installer et d'utiliser plusieurs versions de *Node.js* en même temps.

Pour l'installer, rendez-vous [ici](https://github.com/nvm-sh/nvm#installing-and-updating) et copiez / collez la première commande dans un terminal.

Quittez ensuite le terminal et relancez en un nouveau pour mettre à jour le *shell*.

Installez ensuite la dernière version *LTS* :

```sh
nvm install --lts
```

Vous pourrez lister toutes les versions installées en faisant :

```sh
nvm ls
```

Pour changer de version faites simplement :

```sh
nvm use 18
```

#### Création de l'exemple

Nous allons utiliser le *create-react-app* qui est la librairie officielle pour créer une *SPA* *React*.

Nous allons utiliser *npx* est un outil inclus dans *npm* qui permet l'exécution de packages *Node.js*. Il permet d'installer temporairement une librairie pour l'exécuter, cela permet de ne pas polluer son environnement et d'exécuter toujours la dernière version :

```sh
npx create-react-app frontenddocker
```

*frontenddocker* est le nom du dossier où sera créée l'application *React*.

Vous n'avez plus qu'à ouvrir l'application dans *VS Code* :

```sh
code frontenddocker
```

#### Création du *Dockerfile*

Nous allons maintenant créer notre *Dockerfile*.

```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
CMD ["npm", "start"]
```

Nous partons d'une image *Node.js* car *Webpack* a besoin de *Node.js* pour s'exécuter.

Nous allons créer un dossier */app* dans l'image et y copier notre *package.json*.

Nous installons nos dépendances et nous copions enfin tout le code présent dans le dossier courant sur l'hôte.

Nous exécutons la commande `npm run start` qui permet de lancer le script de développement pour *React*.

Nous pouvons ensuite créer notre image :

```sh
docker build -t myreact .
```

Nous lançons ensuite notre image sans oublier de publier le port *3000* qui est **le port par défaut sur lequel le serveur de développement de Webpack sert l'application :**

```sh
docker run -it -p 3000:3000 --name react myreact
```

### Mise en place du live reload

#### Création d'un *bind mount* pour le *live reload*

Comme nous l'avions vu pour notre petite application *Node.js*, il est nécessaire d'avoir un *bind mount* avec l'hôte pour que le *live reload* puisse fonctionner.

En effet, sans *bind mount*, les fichiers dans le conteneur ne sont pas mis à jour lorsque nous modifions les fichiers sur l'hôte : rappelez-vous que le système de fichiers du conteneur est isolé.

Pour y remédier il faut donc une liaison entre le système de fichiers de l'hôte et celui du conteneur.

Nous créons donc notre *bind mount* :

```sh
docker run -it -p 3000:3000 --mount type=bind,src="$(pwd)",dst=/app myreact
```

Cela fonctionne si on a le dossier *node_modules* sur l'hôte mais si on ne l'a plus cela ne fonctionne pas : cela prouve que ce sont les dépendances de l'hôte et non pas du conteneur qui sont exécutées dans le conteneur.

Nous vous invitons à supprimer le dossier *node_modules* et à retester : cela ne fonctionne plus.

La raison est simple : lorsque nous faisons le *bind mount* nous lions le système de fichiers de l'hôte à celui du conteneur. Or, sur la machine hôte il n'y a plus de dossier *node_modules* et celui-ci est donc supprimé dans le conteneur.

Or, lorsque les autres développeurs de votre équipe utiliseront votre image, ils n'auront pas *node_modules* installés, cela ne fonctionnera donc pas pour eux.

En outre, si jamais ils installent les dépendances localement (avec `npm install`), ce seront leurs dépendances qui seront copiées et exécutées dans le conteneur ce qui peut poser des problèmes de compatibilité. Prenons quelques exemples :

- si la machine hôte est sur *Windows*, comme le conteneur est sous *Linux* certaines dépendances ne fonctionneront tout simplement pas, car elles auront été *build* pour *Windows* lors de l'installation.

- si la version de *Node.js* de l'hôte est la version 17 et qu'il installe les dépendances localement, les *build* seront prévus pour la version 17. Or, lorsque les dépendances seront copiées par le *bind mount* dans le conteneur, et si le conteneur a la version 18 de *Node.js*, certaines dépendances ne fonctionneront pas car les *builds* seront à la mauvaise version.

**Pour être certain que ce soient les dépendances installées dans le conteneur et non pas les dépendances installées sur l'hôte qui soient exécutées**, il va falloir ruser !

C'est absolument nécessaire si vous voulez une image qui fonctionne sur n'importe quel environnement et ne jamais avoir de problèmes d'incompatibilité.

#### Utilisation d'un double montage : *bind mount* et *volume anonyme*

La ruse est d'utiliser un volume anonyme :

```sh
docker container run --rm -it -p 3000:3000 --mount dst=/app/node_modules --mount type=bind,src="$(pwd)",dst=/app myreact
```

Ici, nous montons un volume anonyme sur */app/node_modules*. Comme la destination n'est pas vide, le comportement de *Docker* est de copier le contenu de la destination dans le volume anonyme. **Il contiendra alors les *node_modules* installés dans le conteneur.**

Le *bind mount* ne va pas supprimer le contenu du volume anonyme et le conteneur utilisera donc bien les dépendances installées dans ce dernier.

**Il faut utiliser `--rm` pour que le volume anonyme soit automatiquement supprimé lorsque nous quittons le conteneur** (lorsque nous avons fini de développer).

Nous avons maintenant un environnement de développement parfaitement fonctionnel ! Mais la commande est longue et laborieuse, nous allons donc mettre en place *Docker Compose* pour l'automatiser.

#### Live reload sur Windows

Si vous êtes sur *Windows*, il est possible que le *live reload* ne fonctionne pas. En effet, il y a un problème de synchronisation entre le système de fichiers de *Windows* et celui de *Linux*.

Pour y remédier, il faut ajouter l'option `WATCHPACK_POLLING` à la commande. Cela va forcer le *live reload* à utiliser un système de polling pour détecter les changements de fichiers.

```sh
docker run -it -p 3000:3000 -e WATCHPACK_POLLING=true --mount type=bind,src="$(pwd)",dst=/app myreact
```

### Mise en place de Docker Compose

#### Création du fichier *docker-compose.yml*

Nous créons le fichier pour *Docker Compose* :

```yaml
version: '3.9'
services:
  client:
    build: .
    ports:
      - '3000:3000'
    volumes:
      - type: bind
        source: ./
        target: /home/node
      - type: volume
        target: /home/node/node_modules
```

Nous pouvons ensuite simplement lancer notre application en développement :

```sh
docker compose up
```

#### Version pour Windows

Pour Windows, il faut modifier le fichier *docker-compose.yml* :

```yaml
version: "3.9"
services:
  client:
    build: .
    ports:
      - "3000:3000"
    environment:
      WATCHPACK_POLLING: "true"
    volumes:
      - type: bind
        source: ./
        target: /home/node
      - type: volume
        target: /home/node/node_modules
```

### Lancer les tests pendant le développement

#### Lancer les tests pendant le développement

Dans certaines équipes, l'approche *TDD* (*Test Driven Development*) est de mise : il faut d'abord écrire les tests pour chaque fonctionnalité et ensuite écrire le code source pour les valider.

Dans une telle approche nous voulons lancer continuellement tous ou certains tests à chaque sauvegarde lors du développement.

Pour ce faire, nous allons créer un second service pour les tests !

Nous modifions notre fichier *docker-compose.yml* :

```yaml
version: "3.9"
services:
  client:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - type: bind
        source: ./
        target: /home/node
      - type: volume
        target: /home/node/node_modules
  test:
    build: .
    volumes:
      - type: bind
        source: ./
        target: /home/node
      - type: volume
        target: /home/node/node_modules
    command: ["npm", "run", "test"]
```

Nous avons un second service avec les mêmes montages, ce qui va permettre d'avoir les mises à jour à chaque sauvegarde et le relancement automatique des tests.

Nous testons :

```sh
docker compose up --build
```

Les tests sont biens relancés automatiquement si nous modifions le code, nous pouvons ajouter des tests dans *App.test.js* et ils sont bien pris en compte et lancés automatiquement.

Il reste un dernier problème : les services sont attachés à notre terminal mais ils n'ont pas de *TTY* et le *STDIN* (entrée standard) n'est pas liée à notre terminal. Cela fait que nous pouvons voir les sorties standard et d'erreur (*STDERR*, *STDOUT*), mais nous ne pouvons rien taper. Nous n'avons pas de *shell* interactif.

La solution et d'activer le *TTY* et la liaison avec l'entrée standard, exactement comme si nous lancions un conteneur avec `docker run -it`.

Pour cela, nous devons activer les options *tty* et *stdin_open* :

```yaml
version: "3.9"
services:
  client:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - type: bind
        source: ./
        target: /home/node
      - type: volume
        target: /home/node/node_modules
  test:
    build: .
    volumes:
      - type: bind
        source: ./
        target: /home/node
      - type: volume
        target: /home/node/node_modules
    tty: true
    stdin_open: true
    command: ["npm", "run", "test"]
```

Nous coupons et nous relançons :

```sh
docker compose down -v
docker compose up --build
```

Nous ouvrons un autre terminal.

Nous récupérons le nom ou l'*ID* du conteneur lancé pour le service *test* :

```sh
docker container ls
```

Nous attachons notre second terminal :

```sh
docker container attach NOM_OU_ID
```

Nous avons maintenant un terminal qui est connecté à un *shell interactif* ! Nous pouvons donc taper des commandes relatives aux tests sans soucis.

Nous pouvons enfin couper proprement, en supprimant le volume anonyme :

```sh
docker compose down -v
```

Notre environnement de développement est maintenant parfaitement opérationnel : nous avons le *live reload* et les tests, nous pouvons partager l'image à toute notre équipe et elle fonctionnera sans problème pour eux. 

### Mise en place de l'environnement de production

#### Utilisation de *NGINX*

Nous allons utiliser *NGINX* (prononcez *engine-x*) qui est le serveur Web le plus utilisé au monde.

Il s'agit d'un serveur *opensource* développé en *C* extrêmement performant. Comme *Node.js*, il s'agit d'un environnement asynchrone où chaque requête n'est pas traitée par un *thread* dédié. Cela lui permet de gérer un grand nombre de connexions de manière optimale.

En deux mots, *nginx* est un serveur très performant et extrêmement optimisé (il consomme peu de mémoire par requête) contrairement à des serveurs plus anciens comme *Apache*.

Notez que *nginx* n'est pas un serveur permettant de faire des *APIs*, il n'est donc pas à comparer à *Node.js*, *Java* etc.

**On l'utilise comme point d'entrée** qui va ensuite diriger les requêtes sur les bons serveurs / services (*reverse proxy*), qui va répartir la charge entre plusieurs serveurs (*load balancing*), ou qui va permettre d'optimiser les performances (terminaison *TLS*, mise en cache, compression, limitation des requêtes (*rate limiting*), persistance des sessions etc).

#### Création d'un *Dockerfile* pour la production

Nous allons créer un fichier *Dockerfile.prod* que nous allons utiliser pour créer l'image de production de notre application : à savoir l'image contenant le *build* et qui est servie de manière optimisée :

```dockerfile
FROM node:alpine as build
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN ["npm", "run", "build"]

FROM nginx
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
```

Ici nous utilisons un *multi-stage build*, c'est-à-dire une construction d'image en plusieurs étapes.

Dans un *multi-stage build* vous pouvez utiliser plusieurs instructions `FROM`. Chaque instruction `FROM` est une étape de la construction (*build stage*).

Il est possible de facilement copier le résultat d'un *build* entre des étapes différentes de la construction.

C'est ce que nous faisons ici : nous récupérons le *build* de l'application *React* depuis */app/build*, qui a été généré par la première étape, et nous le copions dans la deuxième étape, à l'emplacement nécessaire pour que *nginx* serve le *build*.

Cela permet ici de passer à une image de plus de 430 Mo à seulement 130 Mo et quelques : nous n'avons pas *Node.js*, tout le code source et tous les modules. Dans une application "classique", la différence serait encore plus flagrante, avec une consommation souvent de l'ordre de 10 fois moins d'espace disque.

Nous construisons ensuite l'image en précisant que nous utilisons le fichier *Dockerfile* de production avec l'option `-f` :

```sh
docker build -t reactprod -f Dockerfile.prod  .
```

Nous n'avons plus qu'à lancer l'image dans un conteneur :

```sh
docker run -p 80:80 --name nginx -d --restart always reactprod
```

Vous pouvez maintenant voir votre application dans un navigateur sur http://localhost/.

Sur votre serveur vous auriez simplement à mettre l'image et à lancer la commande précédente.

#### Création d'un fichier pour *Docker Compose*

Nous allons créer un fichier *docker-compose.prod.yml* :

```yaml
version: "3.9"
services:
  reactprod:
    build:
      context: .
      dockerfile: Dockerfile.prod
    ports:
      - "80:80"
```
Il n'y a plus qu'à lancer l'application avec *Docker Compose* :

```sh
docker compose -f docker-compose.prod.yml up
```
