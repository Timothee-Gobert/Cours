## Persister des données avec Docker

### Introduction à la persistance

#### Le problème de la persistance

##### Les conteneurs sont éphémères

**Les conteneurs sont conçus pour être immuables et éphémères.**

Cela signifie qu'ils sont prévus pour être arrêtés, supprimés etc sans avoir à conserver d'état. A chaque fois que vous voulez faire une modification vous stopper le conteneur et vous en relancez un nouveau avec des options différentes ou vous modifiez l'image.

Par défaut, les fichiers d'un conteneur sont écrits dans une couche avec les droits d'écritures, mais **cette couche est supprimée si le conteneur n'existe plus.**

En outre, cette couche utilise *UnionFS* comme nous l'avons vu, ce qui réduit les performances d'écriture et de lecture. Ce n'est donc pas du tout adapté pour une base de données.

##### Comment gérer la persistance ?

Mais il y a des exceptions à ce principe : **les bases de données.**

Les bases de données servent justement à persister des données et il faut donc pouvoir ne pas perdre toutes les données à chaque fois que vous modifiez ou relancez un conteneur !

Pour ce faire, *Docker* propose l'utilisation de *bind mounts* et de *volumes* qui permettent de résoudre les problèmes de performance (en écrivant directement dans le système de fichiers de l'hôte) et les problèmes de persistance (ils sont conservés après la suppression du conteneur).

Quelle que soit la persistance choisie, cela ne **fait aucune différence du point de vue du conteneur** : les données sont exposées dans le système de fichiers du conteneur.

Pour visualiser la différence du **point de vue de la machine hôte**, rien de mieux que le schéma officiel :

![](/00-assets/images/Docker/image-6_28_1.png)

Dans le cas d'un *bind mount* : il s'agit tout simplement de fichiers et de dossiers n'importe où sur le système de fichiers de l'hôte. N'importe quel processus peut les modifier, y compris en dehors de *Docker*.

Dans le cas d'un *volume* : *Docker* utilise le système de fichiers de la machine hôte mais gère lui même cet espace et l'utilisateur doit passer par le *Docker CLI*. Sur *GNU/Linux* l'emplacement sera */var/lib/docker/volumes/* mais il ne faut jamais y toucher directement !

Dans le cas d'un *TMPFS* : il s'agit d'un stockage temporaire en mémoire vive (*RAM*).

#### Les *bind mounts*

Les *bind mounts* sont limités en fonctionnalités par rapport aux volumes. Ils ne sont pas recommandé sauf dans certains cas que nous étudierons.

**Lorsque vous utilisez un *bind mount* vous montez un fichier ou un dossier du système de fichiers de la machine hôte dans un conteneur.**

Il faut préciser le chemin absolu du fichier ou du dossier que vous voulez monter. Il peut être n'importe où sur l'hôte.

Si le fichier ou le dossier n'existe pas sur l'hôte, il sera créé par *Docker* à l'emplacement indiqué.

#### Les volumes

Les volumes sont créés et gérés entièrement par *Docker*.

Ils peuvent être créer en utilisant le *CLI* avec la commande :

```sh
docker volume create
```

Ils peuvent également être créé par *Docker* lors de la création d'un conteneur ou d'un service.

**Lorsque l'on crée un volume il est stocké dans un dossier sur l'hôte (*/var/lib/docker/volumes/* pour *GNU/Linux*).**

**Ce volume peut ensuite être monté (c'est-à-dire utilisé) dans un conteneur, le dossier est alors utilisé.**

C'est très similaire avec les *bind mounts*, mais la différence est que **les volumes sont entièrement gérés par *Docker* et qu'ils sont isolés de la machine hôte.**

D'ailleurs les permissions du dossier sont les suivantes : **0700/drwx------**. Ce qui signifie que seul l'utilisateur propriétaire qui est le *root* peut utiliser ce dossier.

**Un volume peut être utilisé par plusieurs conteneurs,** comme nous l'étudierons en détails dans une leçon.

#### Les tmpfs

Les *TMPFS* (pour *temporary file system*) permettent d'avoir un espace de stockage qui n'est pas persisté sur le disque.

Ils permettent de stocker de manière temporaire des données d'état ou des informations sensibles (comme des secrets - nous verrons cela avec *Docker swarm*).

Nous les étudierons brièvement dans une leçon. Ils ne sont disponibles pour le moment que sur les machines hôtes *GNU/Linux*. 

### Les bind mounts

#### Cas d'utilisations recommandés pour les *bind mounts*

**Il faut voir un *bind mount* comme une liaison dans les deux sens entre conteneur et hôte pour un dossier (et son contenu) ou un fichier.**

Les volumes sont la manière recommandée de stocker des données avec *Docker*, les *bind mounts* doivent donc être utilisés uniquement dans les cas suivants :
- partager des fichiers de configuration entre l'hôte et les conteneurs.
- partager le code source **lors du développement** pour permettre le *live reload*. 

#### Créer un *bind mount* avec `--mount`

**Le fichier ou le dossier qui est monté doit être désigné par un chemin absolu sur la machine hôte.**

Il existe deux manières de créer un *bind mount* sur un conteneur avec le *CLI*.

La première est avec `-v` ou `--volume`, nous ne la montrerons pas car elle est déconseillée par *Docker*. Pour plusieurs raisons : d'une part car elle porte à confusion avec le type de montage (*volume*, *bind mount* et *tmpfs*), d'autre part car elle est trop concise et porte à confusion.

La seconde est `--mount` qui est plus claire et plus sûre, c'est donc celle que nous étudierons.

L'option `--mount` accepte **une série de paires clé-valeur qui sont séparés par des virgules**, l'ordre des paires n'est pas important :
- **type** permet de spécifier le type de montage souhaité : *bind mount*, *volume* ou *tmpfs*. Les valeurs possibles sont donc *bind*, *volume* et *tmpfs*. Ici il faut donc mettre *bind*.
- **source** ou **src** permet de spécifier la source. Il s'agit du chemin absolu sur l'hôte du fichier ou du dossier qui doit être monté dans le conteneur.
- **destination**, **dst** ou **target** permet de spécifier la cible qui doit être le chemin où les fichiers et les dossiers seront montés dans le conteneur.
- **readonly** permet de rendre le montage en lecture seule. Le conteneur ne pourra alors pas écrire sur celui-ci. Ce cas d'utilisation est rare.

#### Premier exemple

Dans un dossier créez un autre dossier *data* :

```sh
mkdir data
```

Placez-y ensuite un fichier :

```sh
cd data
echo 123 > hello.txt
```

Créez ensuite un conteneur avec un *bind mount* sur le dossier que nous venons de créer :

```sh
docker container run -it --name alpine1 --mount type=bind,source="$(pwd)",target=/data alpine sh
```

Notez que `$(pwd)` est une substitution de commande qui va être remplacée par la valeur de la variable d'environnement *pwd*, qui contient le chemin absolu du répertoire courant. (*Pour en savoir plus voyez le cours Linux*).

Nous montons donc le dossier *data* de la machine hôte que nous avons créé dans le dossier */data* sur le conteneur.

Notez que ce dossier peut savoir le nom que vous voulez, il n'a pas à correspondre au nom sur la machine hôte.

*Sur Windows avec Git Bash uniquement*, il se peut que le montage ne fonctionne pas en mettant */data*. Il faut mettre *//data* (car Git Bash le transforme sinon en `C:\\data`).

Vérifiez que le dossier data est bien accessible dans le conteneur :

```sh
cat /data/hello.txt
```

Quittez ensuite le conteneur avec *exit*.

Vous pouvez vérifier le montage dans le conteneur :

```sh
docker container inspect alpine1
```

Cherchez dans toutes les informations de configuration sur le conteneur la partie *Mounts*. Vous aurez par exemple :

```js
Mounts": [
  {
    "Type": "bind",
    "Source": "/home/erwan/data",
    "Destination": "/data",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
    }
],
```

Remarquez bien que le volume *data* de l'hôte est bien monté sur */data* dans le conteneur.

*RW* signifie *Read Write*. La valeur serait *false* si nous avions monté le volume en lecture seul. Mais c'est très rare, la plupart du temps vous aurez *true*.

La propagation est un sujet très avancé que vous n'étudierons pas.

Supprimez maintenant les conteneurs :

```sh
docker container prune
```

Vous pouvez ensuite vérifier que les données sont toujours présentes sur l'hôte. Supprimer un conteneur n'entraîne pas la suppression des données sur l'hôte.

**Si le dossier cible sur le conteneur où vous montez le dossier de l'hôte, contient des données, alors ils seront cachés.** Ils ne seront pas perdus mais vous ne pourrez plus les voir tant que le montage existe.

### Utilisation d'un bind mount dans notre exemple

#### Le problème : les modifications des fichiers ne sont pas propagés au conteneur

Revenons à notre application d'exemple.

Si vous faites :

```sh
docker run -p 80:80 myapp
```

Puis que vous modifiez des fichiers dans *app.js*, la nouvelle version ne sera pas dans le conteneur.

Il faudra reconstruire une nouvelle fois l'image et créer un nouveau conteneur.

Cette situation ne permet pas de développer correctement, impossible de reconstruire l'image à chaque changement !

#### Solution : un bind mount !

Nous allons mettre en place une liaison entre les fichiers de notre application (ici uniquement *app.js*) et notre conteneur grâce à un *bind mount*.

Pour commencer créons un dossier *src* et déplaçons y le fichier *app.js*.

Ensuite, modifions le *Dockerfile* et plus spécifiquement l'instruction *CMD* pour prendre en compte le nouveau chemin :

```dockerfile
FROM node:alpine
WORKDIR /app
COPY ./package.json .
RUN npm install
COPY . .
ENV PATH=$PATH:/app/node_modules/.bin
CMD [ "nodemon", "src/app.js" ]
```

Nous pouvons reconstruire l'image :

```sh
docker build -t myapp .
```

Nous démarrons un conteneur avec la nouvelle version de l'image, sans oublier de publier le port 80 et avec le `--mount` dont nous avons parlé :

```sh
docker run -p 80:80 --mount type=bind,source="$(pwd)/src",target=/app/src myapp
```

Essayez de modifier *app.js* sur l'hôte, vous verrez *nodemon* redémarrer dans le terminal.

Vous pourrez constater à chaque fois les changements sur *localhost*, bien sûr en rafraichissant. 

