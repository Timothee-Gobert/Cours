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

### Optimisation et .dockerignore

#### Modifier l'ordre des instructions

Pour le moment, à chaque fois que nous reconstruisons l'image suite à un changement, l'instruction `RUN npm install` n'utilise pas le cache, ce qui entraîne une installation complète.

Dans une application classique vous aurez quelques dizaines de dépendances et il faut donc absolument que le cache soit utilisé pour ne pas avoir à réinstaller toutes les dépendances à chaque nouveau *build* !

Pourquoi le cache est invalidé ? Tout simplement car rappelez-vous : `COPY` compare les *checksums* des fichiers. Comme nous modifions un fichier l'instruction ne peut utiliser le cache et une nouvelle image intermédiaire est créée.

Ensuite, comme il n'y a pas d'image fille avec l'instruction `RUN npm install` pour cette nouvelle image, le cache n'est pas non plus utilisé pour l'instruction `RUN`.

Il faut donc faire en sorte que l'instruction `COPY` avant l'instruction `RUN` ne soit pas invalidée à chaque changement dans les fichiers de notre application !

Pour ce faire, nous allons faire :

```dockerfile
FROM node:alpine
WORKDIR /app
COPY ./package.json .
RUN npm install
COPY . .
ENV PATH=$PATH:/app/node_modules/.bin
CMD [ "nodemon", "/app/app.js" ]
```

Notez que nous faisons deux instructions `COPY`.

Dans la première, **nous copions uniquement le fichier *package.json*.**

Cela signifie que le cache ne sera invalidé **que si le fichier *package.json* est modifié.**

Ensuite, nous exécutons les installations de dépendances et après seulement nous copions le reste de l'application !

En définitive, les 4 premières instructions pourront maintenant utiliser le cache même si nous modifions les fichiers de l'application, et c'est ce que nous voulions.

Rendez-vous dans *app.js* et modifiez le message de retour. Faites un *build* de l'image à chaque fois :

```sh
docker build -t test .
```

Vous verrez maintenant que les quatres premières instructions utilisent bien le cache :

```sh
Sending build context to Docker daemon  4.786MB
Step 1/7 : FROM node:alpine
  ---> 1e8b781248bb
Step 2/7 : WORKDIR /app
  ---> Using cache
  ---> 783a9c60e848
Step 3/7 : COPY ./package.json .
  ---> Using cache
  ---> 3ef039395cd6
Step 4/7 : RUN npm install
  ---> Using cache
  ---> 31a48b7c6698
```

Notez que nous n'en avons pas du tout terminé ! Nous avons résolu quasiment tous les problèmes mais il en reste un de taille : à chaque fois que nous modifions l'application il faut reconstruire l'image.

*nodemon*, qui permet de détecter les changements et relancer l'application avec les modifications pendant le développement ne sert donc pour le moment à rien.

Il nous faut des connaissances supplémentaires pour palier ce problème que nous apprendrons justement dans le chapitre suivant.

#### Le fichier *.dockerignore*

Si vous connaissez *Git*, le fonctionnement du fichier *.dockerignore* est très similaire.

Ce fichier est lu par le *Docker CLI* avant qu'il envoie le contexte au démon.

**Il faut que ce fichier soit dans le dossier racine du contexte.** La plupart du temps au même niveau que le *Dockerfile*.

Ce fichier permet de spécifier les fichiers et les dossiers qui doivent être ignorés par *Docker*. Ils ne seront pas envoyés au démon et *Docker* n'en aura aucune connaissance.

Cela permet d'enlever des fichiers volumineux (par exemple *node_modules*) ou sensibles (des clés, des secrets etc) pour les instructions `ADD` et `COPY`.

La syntaxe est la suivante :

`#` permet d'insérer un commentaire.

`*/temp*` permet d'exclure les fichiers et les dossiers qui commencent par *temp* dans les dossiers enfants du répertoire racine du contexte. Par exemple, */app/temp_fichier*, */temp/temporaire/* etc seront exclus. Par contre */app/dossier/temp* ne le sera pas.

`*/*/temp*` permet d'exclure les fichiers et les dossiers qui commencent par *temp* dans les dossiers qui sont à deux niveaux du répertoire racine du contexte. Donc */dossierA/dossierB/temp* sera exclus mais pas *app/temp*.

`fichier?` permet d'exclure tous les dossiers et les fichiers dans le dossier racine du contexte qui commence par *fichier* et sont suivis d'un caractère alphanumérique. Par exemple */fichierA* ou *fichier1* seront exclus.

`**` signifie n'importe quel niveau de dossier. Donc par exemple `**/*.txt` permet d'exclure tous les fichiers terminant par *.txt* dans n'importe quel dossier du contexte.

`!` signifie exception. Cela permet par exemple de ne pas exclure un fichier qui serait normalement exclus suivant les règles précédentes :

```
**/*.txt
!README.txt
```

Ici tous les fichiers *.txt* sont exclus par la première règle. Mais la deuxième ligne permet d'ajouter une exception pour le fichier *README.txt* qui sera alors inclus dans le contexte.

**L'ordre des règles est très important dans le *.dockerignore*. C'est toujours la dernière règle qui l'emporte pour chaque *match*.** Donc, ceci ne fonctionnerait pas :

```
!README.txt
**/*.txt
```

#### Quelques rappels sur les commandes

Pour lancer notre application en mode détaché (pour reprendre la main dans le terminal) :

```sh
docker run -d --name appnode -p 80:80 myapp
```

Pour obtenir un terminal sur notre conteneur en cours d'exécution :

```sh
docker exec -it appnode sh
```

Pour voir la consommation des ressources de nos conteneurs en cours d'exécution :

```sh
docker container stats
```
