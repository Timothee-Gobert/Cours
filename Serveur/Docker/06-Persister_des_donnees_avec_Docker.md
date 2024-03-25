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

### Les volumes

#### Cas d'utilisations recommandés pour les volumes

**Les volumes sont la manière recommandée de stocker des données avec *Docker*.**

Ils permettent :
- d'utiliser le *Docker CLI*.
- de partager des données entre plusieurs conteneurs en cours d'exécution.
- d'utiliser un espace de stockage totalement géré par *Docker* et donc de ne pas dépendre du système hôte. Ils fonctionneront quel que soit l'hôte et il n'y a pas à se préoccuper des chemins comme avec les *bind mounts*.
- ils permettent de stocker les données sur un hôte distant. Les volumes ne sont pas forcément stockés sur la machine hôte.
- ils permettent de facilement sauvegarder, restaurer ou migrer des données.
- ils ont des performances élevées et sont contrôlés par Docker ce qui est absolument nécessaires pour des bases de données.

#### Création des volumes

Les volumes peuvent être créer en utilisant le *CLI* avec la commande :

```sh
docker volume create
```

Ils peuvent également être créé par *Docker* lors de la création d'un conteneur ou d'un service.

Les volumes peuvent être anonyme ou vous pouvez définir un nom lors de la création.

Par exemple si vous faites :

```sh
docker volume create
```

Vous aurez en réponse seulement l'*ID* du volume, par exemple :

```
c496de105865487ab8f3b9136bbf191905c0f05c55a96add8e3560c23eb99350
```

Mais si vous faites :

```sh
docker volume create data
```

Le conteneur aura le nom data et il sera ensuite plus simple d'interagir avec celui-ci, et surtout de savoir à quoi il correspond.

#### Lister les volumes

Pour lister tous les volumes *Docker* existants, faites simplement :

```sh
docker volume ls
```

Vous aurez alors dans notre cas :

```sh
DRIVER    VOLUME NAME
local     c496de105865487ab8f3b9136bbf191905c0f05c55a96add8e3560c23eb99350
local     data
```

Ici nous avons un volume anonyme et un volume nommé.

#### Inspecter des volumes

Pour inspecter un volume il suffit de faire :

```sh
docker volume inspect ID_NOM
```

Par exemple :

```sh
docker volume inspect data
```

Le résultat sera alors par exemple :

```json
[
    {
        "CreatedAt": "2020-11-04T11:35:40+01:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/data/_data",
        "Name": "data",
        "Options": {},
        "Scope": "local"
    }
]
```

Nous retrouvons bien sur *GNU/Linux* l'emplacement pour les volumes dont nous avons parlé.

#### Monter des volumes

Il existe deux manières de monter un volume sur un conteneur avec le *CLI*.

La première est avec `-v` ou `--volume`, nous ne la montrerons pas car elle est déconseillée par *Docker*.

La seconde est `--mount` qui est plus claire et plus sûre.

##### Monter un volume avec `--mount`

L'option `--mount` accepte **une série de paires clé-valeur qui sont séparés par des virgules,** l'ordre des paires n'est pas important :
- **type** permet de spécifier le type de montage souhaité : *bind mount*, *volume* ou *tmpfs*. Les valeurs possibles sont donc *bind*, *volume* et *tmpfs*.
- **source** ou **src** permet de spécifier la source. Cela peut être le nom du volume. Si vous voulez créer un volume anonyme vous pouvez ne rien spécifier.
- **destination**, **dst** ou **target** permet de spécifier la cible qui doit être le chemin où les fichiers et les dossiers seront montés dans le conteneur.
- **readonly** permet de rendre le volume monté en lecture seule.

##### Comportement du montage suivant que le volume est vide ou non

**Si le volume que vous montez est vide** et si la cible dans le conteneur contient des dossiers et des fichiers, ils seront copiés dans le volume.

**Si le volume que vous montez contient des données** et si la cible dans le conteneur contient des dossiers et des fichiers, ils seront cachés par le volume. Ils ne seront pas perdus mais vous ne pourrez plus les voir tant que le volume est monté.

Si vous démarrez un conteneur et précisez un volume qui n'existe pas, un volume vide sera créé pour vous par *Docker*.

#### Supprimer des volumes

Les volumes sont prévus pour être persistés. Aussi, ils sont supprimés uniquement si vous le décidez en utilisant une des commandes que nous allons voir.

##### Supprimer un volume

Pour supprimer un seul volume il faut faire

```sh
docker volume rm ID_NOM
```

##### Supprimer tous les volumes

Faites très attention à cette commande car les volumes ne seront pas récupérables après suppression. Aussi vérifiez bien les volumes avant de décider de tous les supprimer :

```sh
docker volume prune
```

#### Exemples

Nous allons prendre quelques exemples pour vous familiariser avec les volumes.

Commençons par tout supprimer :

```sh
docker volume prune
```

##### Démarrer un conteneur avec un volume nommé

Nous allons commencer par démarrer un conteneur avec un volume qui n'existe pas :

```sh
docker container run -d --name nginx1 --mount source=data1,target=/app nginx
```

Nous lançons un conteneur en mode détaché avec pour nom *nginx1*.

Nous montons un volume qui a pour nom *data1* et nous ciblons */app* dans le conteneur.

*Docker* va créer un volume *data1* automatiquement, le monter sur le dossier */app* dans le conteneur. Comme */app* n'existe pas il va également le créer automatiquement.

Si vous ne précisez pas le *type* lors de l'utilisation de `--mount`, *Docker* va utiliser par défaut *type=volume*.

Vous pouvez vérifier tout cela :

```sh
docker volume ls
docker container exec nginx1 ls
```

Vérifiez également que le volume créé est bien monté au bon endroit sur le conteneur :

```sh
docker container inspect nginx1
```

Cherchez dans toutes les informations de configuration sur le conteneur la partie *Mounts*. Vous aurez par exemple :

```json
Mounts": [
  {
      "Type": "volume",
      "Name": "data1",
      "Source": "/var/lib/docker/volumes/data1/_data",
      "Destination": "/app",
      "Driver": "local",
      "Mode": "z",
      "RW": true,
      "Propagation": ""
  }
],
```
Remarquez bien que le volume *datat1* est bien monté sur */app*.

`RW` signifie *Read Write*. La valeur serait *false* si nous avions monté le volume en lecture seul. Mais c'est très rare, la plupart du temps vous aurez *true*.

Supprimez le conteneur et remarquez que le volume est toujours persisté :

```sh
docker container kill nginx1
docker container prune
docker volume ls
```

Vous pouvez essayer de le remonter sur un autre conteneur, cela fonctionnera sans problème :

```sh
docker container run -d --name nginx2 --mount source=data1,target=/app nginx
```

##### Démarrer un conteneur avec un volume anonyme

Vous pouvez aussi créer un conteneur avec un volume anonyme, par exemple pour faire rapidement quelques tests et vous pourrez le supprimer ensuite :

```sh
docker container run --mount target=/data -it alpine sh
```

Vérifiez que vous avez bien un dossier */data* :

```sh
ls
```

Magique ! *Docker* a également créé un volume automatiquement pour vous et l'a monté dans */data*.

Vous pouvez le voir en faisant :

```sh
docker volume ls
```

Stoppez le conteneur et supprimez le volume :

```sh
docker container kill nginx2
docker container rm nginx2
docker volume rm
```

### Partager des volumes entre des conteneurs et effectuer des sauvegardes

#### Partager un volume entre plusieurs conteneurs

Pour les applications avec des services répliqués (c'est-à-dire plusieurs instances de la même application permettant d'éviter les *downtimes*), il est courant de partager un *volume* entre eux.

Pour ce faire, il suffit de préciser le même volume entre les services.

Commençons par un peu de nettoyage :

```sh
docker container kill nginx1
docker container prune
docker volume prune
```

Nous allons créer un volume et le partager entre plusieurs instances :

```sh
docker volume create data
```

Ouvrez un premier terminal :

```sh
docker container run -it --rm --mount source=data,target=/data alpine sh
```

L'option `--rm` permet de supprimer le conteneur une fois que celui-ci est stoppé. Donc si vous quittez le conteneur il sera automatiquement supprimé. C'est très pratique pour ce genre de tests !

Créez un fichier dans */data* :

```sh
echo 123 > /data/test
```

Ouvrez un autre terminal ou quittez le conteneur et faites :

```sh
docker container run -it --rm --mount source=data,target=/data alpine sh
```

Constatez que le volume monté est bien partagé entre les deux conteneurs :

```sh
cat data/test
```

Notez qu'il est également possible de passer l'option `--volumes-from` pour monter les volumes d'un conteneur sur un autre conteneur.

Par exemple :

```sh
docker volume prune
docker volume create data
docker container run -d --name conteneur1 --mount source=data,target=/data nginx
```

Puis pour utiliser le même volume sur un autre conteneur :

```sh
docker container run -it --rm --volumes-from conteneur1 alpine sh
```

Si vous faites ensuite :

```sh
echo 123 > /data/test.txt
ls
```

Vous verrez bien */data* dans lequel est monté le même volume que pour *conteneur1*.

#### Effectuer une sauvegarde des volumes d'un conteneur

Pour sauvegarder tous les volumes utilisés par un conteneur en cours d'exécution, il suffit de créer un *bind mount* temporaire et de créer une sauvegarde du volume :

```sh
docker container run --rm --volumes-from conteneur1 --mount type=bind,src="$(pwd)",target=/backup alpine tar -cf /backup/backup.tar /data
```

`--volumes-from conteneur1` va monter tous les volumes montés dans le *conteneur1* sur le nouveau conteneur *alpine* que nous lançons.

`--mount type=bind,src="$(pwd)",target=/backup` va créer un *bind mount* entre le répertoire de travail (*$(pwd)* est une substitution de commande, cf cours Linux) et un dossier */backup* sur le conteneur.

*tar -cf /backup/backup.tar /data* va prendre le volume */data* monté sur notre nouveau conteneur, (et qui est le même que celui du *conteneur1*), et créer une archive dans */backup/backup.tar* sur le conteneur.

Or grâce au *bind mount*, le dossier *backup* est lié au répertoire de travail sur l'hôte. Après la suppression du conteneur (option `--rm`) les fichiers et dossiers situés dans */backup* sur le conteneur sont donc conservés sur l'hôte. En l'occurrence *backup.tar*.

Cela peut paraître très complexe au premier abord, aussi testez bien et revenez y plusieurs fois.

Pour tester ce qui a été archivé, vous pouvez faire sur l'hôte :

```sh
tar -tvf backup.tar
```

**Si vous ne voulez pas utiliser `--volume-from`, vous pouvez également opter pour la forme plus longue :**

```sh
docker container run --rm --mount source=data,target=/data --mount type=bind,source="$(pwd)",target=/backup -it alpine tar -czf /backup/backup.tar.gz /data
```

L'effet est le même.

A noter que cette fois-ci, en plus d'archiver le volume nous le compressons avec *gzip* (option `-z` de la commande tar, *cf cours Linux)*.

#### Effectuer une restauration d'un volume

Pour restaurer une archive compressée ou non compressée dans un nouveau volume, il suffit de faire :

```sh
docker run --mount type=volume,source=restore,target=/data --mount type=bind,source="$(pwd)",target=/backup -it alpine tar -xf /backup/backup.tar --strip-components 1 -C /data
```

`--mount type=volume,source=restore,target=/data` permet de créer un nouveau volume *restore* qui va être monté sur */data* dans le conteneur. Autrement dit, le volume *restore*, qui est au départ vide, se verra copier les fichiers et les dossiers contenus dans */data* sur le conteneur.

`--mount type=bind,source=$"(pwd)",target=/backup` permet de créer un *bind mount* qui va monter le répertoire courant de l'hôte avec le contenu de */backup* sur le conteneur. Autrement dit, cela va copier le contenu du répertoire courant dans le dossier */backup* du conteneur (en réalité ce n'est pas une copie, juste une liaison mais c'est plus facile de le voir comme une copie).

`tar -xf /backup/backup.tar -C /data` va décompresser */backup/backup.tar* dans le conteneur et mettre le résultat dans */data*. `--strip-components 1` permet de ne pas avoir un dossier *data* dans */data* mais directement son contenu.

Or comme */data* est lié au volume *restore*, cela revient à copier le résultat de la décompression dans le volume !

Notez que vous pouvez donner n'importe quel nom au volume *restore*. Ici c'est pour faciliter la compréhension mais le plus souvent on l'appellera par exemple *appdata*.

Maintenant que vous avez un volume avec les données restaurées, vous pouvez le monter sur n'importe quel conteneur :

```sh
docker container run -it --rm --mount source=restore,target=/data alpine sh
```

### Utiliser un volume pour une base de données

#### L'image officielle *mongo*

Allez chercher l'image officielle de *mongo* sur *Docker Hub*, ensuite trouvez la version *latest* et cliquez dessus.

Vous serez redirigé sur le *Dockerfile* de l'image *mongo* sur *Github*.

Nous allons simplement voir un extrait qui nous intéresse ici :

```dockerfile
# … extrait ...
VOLUME /data/db /data/configdb

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 27017
CMD ["mongod"]
```

`VOLUME /data/db /data/configdb` permet de créer deux volumes et de les monter sur */data/db* et */data/configdb* lors de la création d'un conteneur à partir de l'image.

`COPY docker-entrypoint.sh /usr/local/bin/` permet de copier le *script shell docker-entrypoint.sh* sur */usr/local/bin/* dans l'image.

`ENTRYPOINT ["docker-entrypoint.sh"]` permet d'exécuter le script *sh* copié au lancement d'un conteneur.

`EXPOSE 27017` permet de préciser que le service va être lancé sur le port *27017* du conteneur.

`CMD ["mongod"]` permet de lancer le démon *mongod* lors de l'exécution du conteneur.

Faisons un premier test :

```sh
docker container run -it mongo sh
```

Vérifiez bien la présence de */data/db* et */data/configdb* :

```sh
ls data
```

Quittez :

```sh
exit
```

Puis vérifiez la création de volumes :

```sh
docker volume ls
```

Vous verrez deux volumes anonymes qui ont été automatiquement créés.

Un peu de nettoyage :

```sh
docker container prune
docker volume prune
```

#### Lancer une instance *mongod* et s'y connecter

Nous allons lancer en mode détaché un conteneur *mongo* :

```sh
docker container run -d --name mongodb mongo
```

Comme nous l'avons vu cela va lancer *mongod* qui est le démon *MongoDB*.

Vous pouvez voir que la base de données est bien initialisée et tourne :

```sh
docker container logs mongodb
```

Vous pouvez maintenant utiliser le client sur le conteneur en cours d'exécution :

```sh
docker container exec -it mongodb mongosh
```

Les commandes suivantes ne sont pas importantes, elles nous permettent simplement d'ajouter des données dans la base *MongoDB* pour tester la persistance des volumes avec une base de données.

Commençons par créer et utiliser une nouvelle *db* :

```sh
use test
```

Insérons ensuite un *document* :

```sql
db.user.insertOne({name: "jean"})
```

Vérifions que le document est bien inséré :

```sql
db.user.findOne()
```

Quittons :

```sh
exit
```

Ensuite, supprimons le conteneur et les volumes anonymes :

```sh
docker container stop mongodb
docker container prune
docker volume prune
```

Les volumes anonymes contiennent les données mais ce n'est pas pratique, nous allons voir maintenant comment utiliser des volumes nommés ! Ce sera toujours le cas.

#### Persister les données avec un volume

Nous allons relancer un conteneur *mongo*, mais cette fois-ci en utilisant un volume nommé :

```sh
docker container run -d --name mongodb --mount source=mydb,target=/data/db mongo
```

Ici comme le volume *mydb* n'existe pas, il sera automatiquement créé par *Docker*. Il faut absolument le monter dans */data/db* et pas autre part car c'est ce chemin qui est utilisé par le démon *mongod*.

Nous pouvons vérifier maintenant que nous avons bien un volume nommé :

```sh
docker volume ls
```

Nous pouvons refaire notre test :

```sh
docker container exec -it mongodb mongosh
```

Commençons par créer et utiliser une nouvelle *db* :

```sh
use test
```

Insérons ensuite un *document* :

```sql
db.user.insertOne({name: "jean"})
```

Vérifions que le document est bien inséré :

```sql
db.user.findOne()
```

Quittons :

```sh
exit
```

Supprimons le conteneur :

```sh
docker container stop mongodb
docker container prune
```

Relançons, enfin un dernier conteneur en utilisant le même volume :

```sh
docker container run -d --name mongodb --mount source=mydb,target=/data/db mongo
```

Vérifions la persistance des données :

```sh
docker container exec -it mongodb mongosh
```

Puis allons sur la bonne base de données :

```sh
use test
```

Vérifions enfin la présence du document :

```sql
db.user.findOne()
```

Quittons :

```sh
exit
```

#### Utiliser *MongoDB Compass*

Si vous connaissez *MongoDB*, vous serez ravi d'apprendre qu'il est simple d'utiliser *mongo* avec *Compass* et *Docker*.

Créez un conteneur en publiant le port :

```sh
docker container run -d --name mongodb --mount source=mydb,target=/data/db -p 27017:27017 mongo
```

Vous pouvez ensuite aller dans *Compass*, faire *New connection* et cliquez simplement sur connecter sans rien rentrer. Comme c'est le port par défaut en *localhost* cela fonctionnera.

Si vous avez déjà *mongod* qui tourne sur votre machine locale, il suffit de modifier le port hôte sur lequel le conteneur sera lié :

```sh
docker container run -d --name mongodb --mount source=mydb,target=/data/db -p 27018:27017 mongo
```

Dans *Compass* entrez cette fois-ci :

```sh
mongodb://localhost:27018
```

### Utiliser TMPFS

#### Cas d'utilisations recommandés pour les *TMPFS*

Les *TMPFS* ne sont pas persistés. Ils permettent de garder des données en mémoire vive uniquement. Les principaux cas d'utilisation sont :
- données secrètes (mots de passe, secrets pour des paires de clés ou pour de l'encryption symétrique etc).
- données d'état qui seraient trop volumineuses pour être persistées ou trop coûteuse en performance pour être écrites sur disque.

#### Lancer un conteneur avec un *TMPFS* monté

Il suffit cette fois-ci de mettre *type=tmpfs* :

```sh
docker run --name tmp --mount type=tmpfs,target=/data -it alpine sh
```

Créer un fichier dans */data* :

```sh
echo 123 > /data/test
```

Quittez le conteneur :

```sh
exit
```

Relancez-le :

```sh
docker start -ai tmp
```

Il n'y aura plus de données dans */data* car le conteneur a été stoppé :

```sh
ls /data
```