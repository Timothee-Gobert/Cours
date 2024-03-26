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

