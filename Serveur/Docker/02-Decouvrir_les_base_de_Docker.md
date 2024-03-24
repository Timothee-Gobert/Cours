## Découvrir les bases de Docker

### Premier conteneur

#### Lancer son premier conteneur

*Docker* a un conteneur d'exemple qui permet de voir comment fonctionne le lancement d'un conteneur.

Faites dans un terminal :

```bash
docker run hello-world
```

Il s'agit de notre première commande avec le *CLI Docker* ! Grâce à cette commande, le client *Docker* demande au démon *Docker* d'exécuter un nouveau conteneur à partir de l'image spécifiée.

Vous aurez :

```bash


Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: 3
sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
  1. The Docker client contacted the Docker daemon.
  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
  3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
  4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
  $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
  https://hub.docker.com/

For more examples and ideas, visit:
  https://docs.docker.com/get-started/
```

Nous allons analyser chaque ligne et l'expliquer.

```bash
Unable to find image 'hello-world:latest' locally
```

*Docker* commence par vérifier si l'image depuis laquelle on veut démarrer un conteneur avec *docker run* est disponible localement.

Comme c'est la première fois que vous cherchez à utiliser cette image et que *Docker* ne l'a pas téléchargé depuis *Docker Hub* il indique qu'il n'arrive pas trouver sur votre machine l'image *hello-world:latest*.

Le *:latest* est le *tag* de l'image, c'est-à-dire sa version. Nous y reviendrons en détails.

Sachez que par défaut, si vous ne précisez pas de *tag* pour une image, *Docker* téléchargera la dernière version disponible sur le *registry* (*Docker Hub*), c'est-à-dire la version de l'image correspondant au *tag latest*.

```bash
latest: Pulling from library/hello-world
```

*Docker* va alors récupérer l'image depuis le *registry* configuré (qui est *Docker Hub* par défaut, comme nous l'avons déjà expliqué). Il exécute en fait *docker pull hello-world* pour vous.

```bash
0e03bdcc26d7: Pull complete
Digest: 3
sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0
```

*Docker* indique qu'il a fini le téléchargement de l'image (appelé *pull*).

Il fournit ensuite **le *digest* de l'image téléchargée, qui est son identifiant unique** (il s'agit en fait d'un *hash SHA256* de la configuration *JSON* de l'image).

```bash
Status: Downloaded newer image for hello-world:latest
```

*Docker* indique qu'il a fini le téléchargement et l'extraction de la dernière version (*tag latest*) de l'image *hello-world*.

Enfin, tout le reste est un message affiché par le conteneur et redirigé vers le terminal de la machine hôte. Pour afficher ce message, le démon a créé un conteneur depuis l'image téléchargée et l'a exécuté.

Lors de la création du conteneur, le démon *dockerd* fait plein de choses : il lui assigne une *IP* virtuelle sur un réseau privé, il lui assigne un nom aléatoire, il lui crée une couche dans son système de fichiers isolé avec les droits d'écriture etc. Nous y reviendrons plus en détails.

Une fois le conteneur créé, le démon va exécuter la commande précisée dans le fichier de configuration (*Dockerfile*) que nous étudierons dans le prochain chapitre.

Pour ce conteneur, après l'exécution de la commande, le conteneur n'est plus exécuté car il se coupe après avoir affiché le message.

Vous pouvez le vérifier en faisant :

```bash
docker container ls -a
```

Vous verrez dans la colonne *STATUS*, *Exited* ce qui signifie qu'il n'est plus en cours d'exécution.

Nous étudierons bien sûr en détails cette commande dans le formation.

Vous pouvez également voir que l'image est maintenant disponible localement :

```bash
docker images
```

Vous verrez que vous aurez bien l'image *hello-world*.

### Relancer le conteneur

Que se passe t-il si vous refaites :

```bash
docker run hello-world
```

Vous aurez cette fois-ci directement :

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
  1. The Docker client contacted the Docker daemon.
  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
  3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
  4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.



To try something more ambitious, you can run an Ubuntu container with:
  $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
  https://hub.docker.com/

For more examples and ideas, visit:
  https://docs.docker.com/get-started/
```

Cela signifie en fait que *Docker* n'a pas eu besoin de télécharger l'image depuis *Docker Hub*. Il a vu que l'image était disponible localement et a donc pu créer immédiatement un nouveau conteneur à partir de celle-ci.

Il s'agit bien d'un nouveau conteneur et pas du même, vous pouvez le vérifier avec *docker container ls -a*.

Vous aurez ainsi :

![](/00-assets/images/Docker/image-2_06_1.png)

## Lancer un conteneur

### La commande `docker run`

La commande `docker run` est en réalité un raccourci notamment pour plusieurs commandes *Docker* que nous étudierons plus tard : `docker pull image`, `docker container create -d image`, `docker container start ID`, `docker container -a ID`.

Comme nous l'avons vu dans le premier chapitre, un conteneur est un processus en cours d'exécution sur la machine hôte. L'hôte peut être votre machine locale ou un serveur distant, comme nous le verrons plus tard.

#### Comportement par défaut de `docker run image`

**La commande `docker run` va commencer par créer un conteneur,** qui revient à :
- créer un système de fichiers isolé pour le conteneur, ainsi qu'un réseau privé et un arbre des processus isolé.
- créer une couche avec les droits d'écriture activés (*writable container layer*) au-dessus de la couche de l'image en lecture seule.
- exécuter la commande prévue dans l'image en utilisant la configuration par défaut spécifiée. La configuration peut être relative à l'exécution en premier plan ou en arrière plan, à l'identification du conteneur, aux paramètres réseaux et à des limites d'utilisation des ressources (*CPU*, mémoire etc).

Par défaut également, la sortie d'erreur (*STDERR*) et la sortie standard (*STDOUT*) sont attachées à votre terminal pour que vous puissiez voir les flux de sortie de votre conteneur dans votre terminal directement.

Par défaut, c'est comme si vous faisiez en fait :

```bash
docker run -a stdin -a stdout image
```

Nous allons commencer par démarrer un conteneur en tâche de fond, par exemple l'image officielle de *redis* qui est la base de données clé / valeur la plus utilisée :

```bash
docker run redis
```

Ici l'image officielle prévoit notamment que par défaut le processus *redis-server* tourne en premier plan sur le port *6379*.

Vous pouvez constater que nous avons :

```bash
1:C 22 Oct 2020 12:36:12.762 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 22 Oct 2020 12:36:12.762 # Redis version=6.0.8, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 22 Oct 2020 12:36:12.762 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 22 Oct 2020 12:36:12.765 * Running mode=standalone, port=6379.
1:M 22 Oct 2020 12:36:12.765 # Server initialized
1:M 22 Oct 2020 12:36:12.765 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 22 Oct 2020 12:36:12.765 * Ready to accept connections
```

Puis que nous gardons la main car le processus est lancé par défaut en mode interactif.

Vous pouvez ainsi quitter avec *Ctrl + C*.

Relancez maintenant un nouveau conteneur mais cette fois-ci avec une option qui va permettre de lancer le processus en arrière-plan :

```bash
docker run -d redis
```

Nous verrons bien sûr cette option dans une prochaine leçon. Mais ce qu'il est important de retenir c'est que **les images ont des configurations par défaut qu'il est possible de modifier en passant des option.**

Ici, le processus n'est plus interactif mais en arrière plan :

```
9e38f6e910342adb6dd91ba916cef79e5b71252d509818f1e86a615d5705b751
```

#### L'option -i de docker run

Nous allons voir une première option `--interactive` ou `-i` qui permet de garder l'entrée standard ouverte même si le processus est lancé par défaut en arrière plan ou qu'il ne démarre pas de service et donc quitte immédiatement.

Pour ce faire, nous allons utiliser l'image officielle d'*Alpine*.

La distribution *Alpine Linux* est une distribution *GNU/Linux* extrêmement légère. Il n'y a que très peu de programmes installés. Par exemple, la distribution n'a pas *bash* comme *shell* installé par défaut mais uniquement *sh*.

Grâce à cela, l'image officielle d'*Alpine* disponible sur *Docker Hub* fait uniquement *5.57 MB* décompressée (à comparer avec environ *80 MB* pour *Ubuntu*).

Pour rappel, *Docker Hub* est le *registry* officiel *Docker*, il s'agit d'une bibliothèque contenant des images officielles publiques, ainsi que des projets publics ou privés. Vous pouvez le voir comme le "Github / Gitlab des images".

Lançons d'abord *alpine* en mode par défaut donc non interactif :

```bash
docker run alpine
```

Si vous faites *docker container ls* vous pouvez voir que le conteneur n'est pas en cours d'exécution. La raison est que l'image n'exécute aucun service par défaut, aussi il n'y a rien à exécuter et le conteneur, qui rappelons-le encore est un processus, s'arrête.

Faites maintenant :

```bash
docker run -i alpine
```

Cette fois vous êtes en mode interactif et vous pouvez entrer des commandes :

```bash
ls
```

La commande listera les fichiers et les dossiers du conteneur.

Quittez ensuite le conteneur avec *CTRL + C* car vous êtes en mode interactif.

#### L'option `-t` de `docker run`

**L'option `--tty` ou `-t` permet de démarrer un *pseudo-TTY* (c'est-à-dire un terminal) et de l'allouer au conteneur.**

Essayez de faire :

```bash
docker run -it alpine
```

Cette fois-ci vous avez un terminal connecté à un *shell sh* en mode interactif.

Vous avez maintenant un prompt :

```bash
/ #
```

Nous verrons plus tard que la commande lancée par défaut par l'image *alpine* est */bin/sh* et c'est pour cette raison que vous avez *sh* en mode interactif.

Cela revient en fait à faire :

```bash
docker run -it alpine sh
```

#### Petit exercice

Nous allons maintenant faire un petit test du mode interactif. Notre but va être d'installer *curl* et *bash* dans un conteneur basé sur une image *alpine* en mode interactif.

Nous commençons donc par lancer le conteneur avec un *TTY* en mode interactif :

```bash
docker run -it alpine
```

Nous allons ensuite mettre à jour les paquets avec le gestionnaire de paquets d'*Alpine*, *APK* (équivalent à *APT* pour les systèmes *Debian*) :

```bash
apk update
```

Nous installons ensuite *bash* :

```bash
apk add bash
```

Nous pouvons maintenant le lancer :

```bash
bash
```

Nous vérifions que nous utilisons bien le *shell bash* :

```bash
echo $0
```

Nous avons bien */bin/bash*.

### Démarrer en mode *background* ou *foreground*

Vous pouvez créer et démarrer un conteneur au premier plan (*foreground*) ou un conteneur en mode détaché, en arrière plan (*background*).

#### Démarrer en mode *foreground*

Par défaut, avec la commande `run` le conteneur est exécuté au premier plan. Par exemple, faites :

```bash
docker run alpine ping google.fr
```

Ici cela va créer et démarrer un conteneur *alpine* et lui demander d'exécuter la commande `ping google.fr`.

Cette commande permet d'envoyer des paquets à intervalle de temps régulier à http://www.google.fr pour obtenir des statistiques réseau.

Vous verrez que dans ce cas, **les sorties standard et d'erreur et l'entrée standard du conteneur sont liées à votre terminal :**

![](/00-assets/images/Docker/image-2_07_1.png)

Vous remarquez que la sortie standard du conteneur est liée à votre terminal car les retours sont affichés dans votre terminal (sur la machine hôte).

Vous remarquez également que vous pouvez envoyer un *SIGINT* avec *Ctrl + C* et que cela fonctionne car l'entrée standard du conteneur est liée à celle de votre terminal également.

#### Démarrer en mode *background*

Maintenant, nous allons démarrer un conteneur à partir de la même image mais en mode *background*.

Il faut alors utiliser l'option `-d` ou `--detach`.

```bash
docker run -d alpine ping google.fr
```

Cette fois-ci le conteneur est exécuté en arrière plan, vous ne pouvez plus rien faire car vous ne l'avez pas lancé en mode interactif.

Le conteneur continue à envoyer toutes les requêtes mais en arrière plan.

Vous obtenez uniquement l'*ID* du conteneur dont vous avez lancé l'exécution en arrière-plan, par exemple :

```
35934c147b56911427c4dd9f6e7e3a08a0a822bd938033df3e489620e2f2f1e8
```

Nous apprendrons bien sûr dans les leçons suivantes à arrêter et supprimer les conteneurs. Ne vous inquiétez pas !

Mais vous pouvez déjà afficher toute la sortie standard de votre conteneur avec la commande `docker logs`, que nous reverrons.

Il faut passer à la commande le nom ou l'*ID* d'un conteneur, par exemple :

```bash
docker logs 35934c147b56911427c4dd9f6e7e3a08a0a822bd938033df3e489620e2f2f1e8
```

Vous aurez alors :

```bash
PING google.fr (216.58.213.67): 56 data bytes
64 bytes from 216.58.213.67: seq=0 ttl=116 time=18.703 ms
64 bytes from 216.58.213.67: seq=1 ttl=116 time=17.098 ms
64 bytes from 216.58.213.67: seq=2 ttl=116 time=17.410 ms
64 bytes from 216.58.213.67: seq=3 ttl=116 time=17.941 ms
64 bytes from 216.58.213.67: seq=4 ttl=116 time=17.801 ms
64 bytes from 216.58.213.67: seq=5 ttl=116 time=17.795 ms
64 bytes from 216.58.213.67: seq=6 ttl=116 time=18.211 ms
64 bytes from 216.58.213.67: seq=7 ttl=116 time=18.084 ms
64 bytes from 216.58.213.67: seq=8 ttl=116 time=18.547 ms
64 bytes from 216.58.213.67: seq=9 ttl=116 time=18.518 ms
64 bytes from 216.58.213.67: seq=10 ttl=116 time=18.346 ms
64 bytes from 216.58.213.67: seq=11 ttl=116 time=16.750 ms
64 bytes from 216.58.213.67: seq=12 ttl=116 time=17.499 ms
64 bytes from 216.58.213.67: seq=13 ttl=116 time=18.773 ms
64 bytes from 216.58.213.67: seq=14 ttl=116 time=22.429 ms
64 bytes from 216.58.213.67: seq=15 ttl=116 time=21.544 ms
64 bytes from 216.58.213.67: seq=16 ttl=116 time=18.390 ms
64 bytes from 216.58.213.67: seq=17 ttl=116 time=20.201 ms
```
## Obtenir de l'aide, lister et supprimer des images et des conteneurs

### Structure des commandes et obtenir de l'aide

Entrez simplement :

```bash
docker
```

Qui lance en fait `docker help`.

Vous aurez alors la liste des commandes de base sous *Commands*, qui ont cette syntaxe :

```bash
docker commands [...options]
```

Un exemple est `docker ps -a` qui permet de lister tous les conteneurs.

Vous aurez également la liste des commandes de gestion par sujet qui sont appelées *Management Commands*. Leur syntaxe est :

```bash
docker commande sous-command [...options]
```

Un exemple est `docker container ls -a` qui permet de lister tous les conteneurs.

**Vous remarquerez que plusieurs commandes sont en doublons, comme par exemple *docker ps* et *docker container ls*.**

La raison est historique, à mesure que *Docker* s'est complexifié et que le nombre de commandes du *CLI* a explosé, les commandes ont étés divisées en groupes pour plus de clarté.

Certaines commandes, parmi les plus utilisées sont maintenues (du moins pour l'instant) en commande de base car plus rapides.

Nous vous recommandons d'utiliser d'abord les commandes au format long pour vous y habituer avant de passer aux commandes raccourcies.

**Pour obtenir de l'aide sur une commande spécifique , il suffit d'utiliser l'option `--help` :**

```bash
docker container ls --help
```

Vous aurez alors le détail de la commande avec toutes les options :

```bash
Usage:	docker container ls [OPTIONS]

List containers

Aliases:
  ls, ps, list

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all
                        states) (default -1)
  -l, --latest          Show the latest created container (includes all
                        states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
```

### Lister les conteneurs et les images

Il est très utile de pouvoir lister les conteneurs et les images afin de savoir l'état de votre environnement *Docker*.

#### Lister les conteneurs

Pour lister les conteneurs **en cours d'exécution** vous pouvez utiliser deux commandes :

```bash
docker container ls
```

Et un alias qui est :

```bash
docker ps
```

Pour lister tous les conteneurs, y compris ceux qui sont seulement créés ou qui ne sont plus en cours d'exécution (*exited*). Il faut utiliser l'option `-a` ou `--all`.

```bash
docker container ls -a
```

Ou :

```bash
docker ps -a
```

Vous obtiendrez alors quelque chose comme :

![](/00-assets/images/Docker/image-2_08_1.png)

Vous retrouverez le début de l'*ID* des conteneurs, l'image utilisée, la commande lancée lors de l'exécution du conteneur, la date de création du conteneur, son statut, les ports (que nous verrons dans un prochain chapitre) et enfin un nom aléatoire généré par *Docker* pour pouvoir faciliter l'utilisation des commandes du *CLI*.

#### Lister les images

Pour lister les images locales vous pouvez utiliser deux commandes :

```bash
docker images
```

Ou :

```bash
docker image ls
```

Ce qui peut donner par exemple :

![](/00-assets/images/Docker/image-2_08_2.png)

Vous retrouverez le nom de l'image, son *tag* (version), son *ID*, la date de création de l'image et la taille décompressée.

Le plus souvent, vous listerez les images suivant les versions comme nous le verrons, par exemple vous pourrez lister toutes les versions des images de *MongoDB* que vous avez localement en faisant :

```bash
docker image ls mongo
```

### Supprimer un ou plusieurs containers
#### Supprimer un conteneur arrêté

Pour supprimer un conteneur qui n'est pas en cours d'exécution (avec le statut *UP*) vous pouvez faire :

```bash
docker container rm NOM_OU_ID
```

Mais si le conteneur est en cours d'exécution vous aurez un message d'erreur.

Lancez un conteneur en arrière-plan avec le nom *redis* :

```bash
docker run --name redis -d redis
```

Essayez de le supprimer :

```bash
docker container rm redis
```

Vous aurez le message d'erreur suivant :

```bash
Error response from daemon: You cannot remove a running container 15e7ada9de7cc51ad6656f3f245672d9a18059083264165b9a60078ef941791c. Stop the container before attempting removal or force remove
```

Le démon *Docker* indique qu'il faut couper le conteneur avant de le supprimer.

#### Supprimer un conteneur en cours d'exécution

Vous pouvez **forcer la suppression avec l'option `--force` ou `-f` :**

```bash
docker container rm -f redis
```

**Vous pouvez aussi supprimer plusieurs conteneurs d'un coup en passant leur ID ou leur nom en arguments.**

Par exemple, lancez trois conteneurs :

```bash
docker run --name redis1 -d redis
docker run --name redis2 -d redis
docker run --name redis3 -d redis
```

Puis supprimez les :

```bash
docker container rm -f redis1 redis2 redis3
```

Vérifiez que les conteneurs sont bien supprimés en faisant :

```bash
docker container ls
```

#### Supprimer tous les conteneurs arrêtés

Pour supprimer tous les conteneurs arrêtés vous pouvez directement faire :

```bash
docker container prune
```

Vous aurez alors un message de confirmation :

```bash
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N]
```

Il faut entrer `y` puis la touche `Entrée` pour confirmer.

Vous aurez ensuite la liste de tous les *ID* des conteneurs supprimés ainsi que la place récupérée.

La mémoire disque récupérée est souvent faible grâce au mécanisme des couches superposées (système de fichiers *UnionFS* que nous avons vu dans le premier chapitre).

En effet, un conteneur utilise la couche de l'image en lecture seule et ajoute une couche en écriture par dessus, seule cette couche est supprimée lorsque vous supprimez le conteneur.

### Supprimer une ou plusieurs images

#### Supprimer une ou plusieurs images sans conteneur en cours d'exécution

Vous pouvez supprimer une image avec la commande :

```bash
docker image rm NOM_OU_ID
```

Par exemple :

```bash
docker image rm alpine
```

Supprimera les images *alpine* présentes.

Vous pouvez bien sûr passer plusieurs noms ou *ID* d'images à supprimer :

```bash
docker image rm alpine nginx
```

#### Supprimer une ou plusieurs images avec un conteneur en cours d'exécution

Lancez un conteneur en arrière plan :

```bash
docker run --name redis -d redis
```

Essayez ensuite de le supprimer :

```bash
docker image rm redis
```

Vous aurez l'erreur suivante :

```bash
Error response from daemon: conflict: unable to remove repository reference "redis" (must force) - container d6f1d731d932 is using its referenced image bd571e6529f3
```

**Il n'est pas possible de supprimer une image qui est utilisée par un conteneur en cours d'exécution.**

Vous pouvez **forcer la suppression avec l'option `--force` ou `-f` :**

```bash
docker image rm -f redis
```

Dans ce cas, l'image n'est pas supprimée mais elle n'est plus suivie (sa version, *tag*, est supprimé, et elle n'est plus liée au *repository*, c'est-à-dire au serveur dont elle provient). Autrement dit, elle devient *dangling*.

Vous pouvez le voir en faisant :

```bash
docker images
```

Vous aurez :

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              bd571e6529f3        8 days ago          104MB
```

#### Supprimer une image *dangling*

Les images *dangling* sont les images qui ne sont pas versionnées par un *tag*.

Pour supprimer les images *dangling* et **qui ne sont pas utilisées par un conteneur en cours d'exécution**, il suffit de faire :

```bash
docker image prune
```

#### Supprimer toutes les images locales

**Pour supprimer l'ensemble des images locales qui n'ont pas de conteneur en cours d'exécution associé il faut utiliser l'option `-a` :**

```bash
docker image prune -a
```

Vous aurez le message de confirmation suivant :

```bash
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N]
```

### Tout supprimer

Pour supprimer :
- tous les conteneurs qui ne sont pas en cours d'exécution
- tous les réseaux qui ne sont pas utilisés par au moins un conteneur en cours d'exécution
- toutes les images *dangling* qui ne sont pas taguées et qui ne sont pas utilisées par au moins un conteneur en cours d'exécution
- tous les caches utilisés pour la création d'images Docker.

Il suffit de faire :

```bash
docker system prune
```

Pour en plus supprimer les images qui sont *taguées*, donc toutes les images sauf celles utilisées par un conteneur en cours d'exécution, vous pouvez utiliser l'option `--all` ou `-a` :

```bash
docker system prune -a
```

## Le cycle de vie d'un conteneur

### Créer un conteneur sans l'exécuter

Nous avons vu que `docker run image` permettait de réaliser un ensemble d'actions et notamment de créer un conteneur en mode détaché (arrière-plan).

**La commande `docker container create` permet de créer un conteneur mais de ne pas l'exécuter.**

C'est utile lorsque vous souhaitez créer un conteneur avec une configuration spécifique et le démarrer lorsque vous en aurez besoin.

Le statut initial d'un conteneur créé avec cette commande est *CREATED*.

Faites par exemple :

```bash
docker container create alpine
```

Puis faites `docker container ls -a`. Vous verrez que le conteneur a le statut *Created*.

Lorsque vous faites cette commande, l'*ID* du conteneur est affiché pour que vous puissiez le démarrer lorsque vous en avez besoin.

La commande vous affichera ainsi quelque chose comme :

```
25ce86f0edead3d3eeece53ae8f31225648b7fbbe94d8a7d539988bfbf6e386f
```

Par exemple, pour une configuration basique, vous pouvez créer un conteneur en mode interactif auquel un *TTY* sera rattaché et avec le nom "alpine1" :

```bash
docker container create --name alpine1 -it  alpine
```

### Exécuter un conteneur avec `docker container start`

Vous pouvez ensuite démarrer un conteneur créé quand vous le souhaitez avec `docker container start`.

Si le conteneur n'est pas interactif et n'est pas relié à un terminal, le containeur quittera comme si vous aviez fait `docker run alpine`.

Par contre, si vous aviez créer le conteneur avec les options `-it`, le conteneur est alors interactif, vous pouvez le lancer avec l'option `-a` pour l'attacher à votre terminal :

```bash
docker container start -a alpine1
```

Quelques explications s'imposent.

Premièrement, *start* ne peut pas changer les options ou la commande définies lors de la création. Si le conteneur n'est pas relié à un *TTY* et qu'il n'est pas interactif vous ne pourrez pas le changer.

Ainsi si vous faites :

```bash
docker container create --name alpine2  alpine
```

Et que vous essayez d'avoir un terminal interactif :

```bash
docker container start -ai alpine2
```

Cela ne fonctionnera pas !

Par contre, si vous faites la même chose avec un conteneur qui avait été créé avec les options `-it` vous pourrez faire :

```bash
docker container start -ai alpine1
```

**L'option `-a` pour `--attach` permet d'attacher la sortie standard (*STDOUT*) et la sortie d'erreur (*STDERR*) à votre terminal.** Autrement dit, cela permet d'afficher les flux de sortie de votre conteneur dans votre terminal comme si vous étiez dans le conteneur.

**L'option `-i` pour `--interactive` permet d'attacher l'entrée standard (*STDIN*) de votre terminal au conteneur.** Autrement dit, cela permet de taper des commandes dans votre terminal comme si vous étiez dans le conteneur.

A noter que `docker start -ai` revient à utiliser les commandes suivantes :

```bash
docker start alpine1
docker attach alpine1
```

En effet, cette commande permet d'attacher les trois flux d'un conteneur (*STDIN*, *STDOUT* et *STDERR*) au terminal.

### Stopper un conteneur en cours d'exécution avec `docker container stop`

Nous allons recréer un conteneur en utilisant l'image *alpine* et qui va exécuter la commande `ping google.fr` :

```bash
docker container create alpine ping google.fr
```

Nous allons ensuite le démarrer en utilisant son *ID* :

```bash
docker container start 22c829bbb7990ccab66efaaf56bbdf3bd79ef90854a2ff6cce8db5f62a7b9922
```

Il faut bien sûr remplacer par l'*ID* qui vous a été fourni.

Ensuite, nous allons lister tous les conteneurs en cours d'exécutions :

```bash
docker container ls
```

Vous aurez alors :

![](/00-assets/images/Docker/image-2-09-1.png)

*Vous pourriez en voir deux si vous aviez démarrer l'autre conteneur dans une leçon précédente.*

Vous voyez que le statut du conteneur est *UP* donc en cours d'exécution.

Vous pouvez d'ailleurs vérifier les logs du conteneur :

```bash
docker container logs stupefied_blackwell
```

#### Stopper un conteneur avec un *SIGTERM*

Nous allons le stopper en utilisant la commande **`docker container stop` ou l'alias `docker stop`**.

La commande accepte soit l'*ID* soit le nom du conteneur :

```bash
docker container stop stupefied_blackwell
```

Notez que vous pouvez utiliser la touche tabulation pour obtenir l'autocomplétion après avoir entré le début du nom du conteneur, c'est beaucoup plus rapide !

Le conteneur va mettre plusieurs secondes à se couper et il vous sera retourné son nom, dans notre exemple :

```bash
stupefied_blackwell
```

En fait, `docker container stop` envoie un *SIGTERM* pour laisser au conteneur le temps d'effectuer des éventuelles tâches de nettoyage / sauvegarde avant de se couper.

Si au bout de quelques secondes le conteneur est toujours en cours d'exécution, le démon envoie alors un *SIGKILL* au conteneur pour forcer son interruption.

Ici c'est le cas car la commande *ping* n'a pas de code permettant de répondre à un *SIGTERM*.

>*Si vous n'êtes pas à l'aise avec ces notions, nous vous conseillons de vous tourner vers le cours **Linux**.*

#### Stopper un conteneur avec un *SIGKILL*

Nous pouvons également forcer l'exécution du conteneur en cours d'exécution immédiatement en envoyant un *SIGKILL* avec la commande `docker container kill` ou l'alias `docker kill`.

Nous allons recréer un conteneur en arrière plan :

```bash
docker run -d alpine ping google.fr
```

Puis nous allons le *kill* immédiatement avec la commande `docker container kill` :

```bash
docker container kill 7d99
```

Notez que vous pouvez taper que le début de l'*ID* du conteneur tant que celui-ci est discriminant (à savoir qu'aucun autre objet *Docker* commence par le même début).

Ne vous inquiétez pas, si le début de l'*ID* n'est pas discriminant vous aurez une erreur qui vous préviendra :

```bash
Error response from daemon: Cannot kill container: 4: Multiple IDs found with provided prefix: 4
```

Il suffira alors de rentrer quelques caractères de plus.

#### Stopper tous les conteneurs en cours d'exécution

Pour stopper tous les conteneurs en cours d'exécution vous pouvez faire :

```bash
docker stop $(docker ps -aq)
```

Si vous avez fait le cours *Linux* cette commande est triviale. Sinon voici quelques explications :

**$()** est une substitution de commande qui va exécuter la commande entre les parenthèses et la remplacer avec le résultat avant d'exécuter la `commande docker stop`.

`docker ps -aq` ou `docker container ls -aq` permettent de lister tous les *ID* des conteneurs.

Nous passons donc la liste des *ID* de tous les conteneurs à la commande `docker stop` qui va pouvoir stopper tous les conteneurs en cours d'exécution.

### Petit exercice

Maintenant que vous connaissez pas mal de choses nous allons faire un petit exercice.

L'objectif est de démarrer trois conteneurs basés respectivement sur les images officielles de *MongoDB*, *redis* et *postgreSQL*.

Ces conteneurs doivent être démarrés en arrière-plan et avec les noms *mongo*, *redis* et *postgre*.

Pour commencer, recherchez le nom des images sur *Docker Hub*.

Une fois toutes les images lancées, vérifiez bien que les conteneurs sont en cours d'exécution.

Ensuite coupez tous les conteneurs et supprimez toutes les images.

Voici la solution. Nous commençons par lancer toutes les images en arrière plan :

```bash
docker run -d --name mongo mongo
docker run -d --name redis redis
docker run -d --name postgres postgres
```

Ensuite, nous allons vérifier que tous les conteneurs sont bien en cours d'exécution :

```bash
docker container ls
```

Nous remarquons que nous n'avons que *mongo* et *redis* qui sont en cours d'exécution. Il faut alors comprendre pourquoi *postgres* ne fonctionne pas. C'est le moment d'utiliser `docker logs` !

```bash
docker logs postgres
```

Nous avons l'erreur suivante :

```bash
Error: Database is uninitialized and superuser password is not specified.
        You must specify POSTGRES_PASSWORD to a non-empty value for the
        superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

        You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
        connections without a password. This is *not* recommended.

        See PostgreSQL documentation about "trust":
        https://www.postgresql.org/docs/current/auth-trust.html
```

L'erreur nous indique que sommes obligé de passer une option lors de la création pour définir un mot de passe ou accepter les connexions sans mot de passe à la base de données.

Comme c'est un exercice nous allons utiliser le mode non recommandé sans mot de passe. Nous allons donc supprimer l'ancien conteneur arrêté et en redémarrer un avec l'option :

```bash
docker container prune
```

Nous validons, puis :

```bash
docker container run --name postgres -d -e POSTGRES_HOST_AUTH_METHOD=trust postgres
```

L'option `-e` que nous n'avions pas encore vue permet de définir **une variable d'environnement pour le conteneur.** Nous reverrons souvent cette option.

Nous vérifions que cette fois-ci les conteneurs sont bien lancés :

```bash
docker container ls
```

Il ne nous reste plus qu'à les stopper :

```bash
docker container stop $(docker container ls -aq)
```

Puis à tout supprimer :

```bash
docker system prune -a
```

## Docker pause, unpause, rename et exec

### Renommer un conteneur

Nous avons vu qu'il est possible de passer un nom lors de la création d'un conteneur avec `docker container create` ou `docker container run` en utilisant l'option `--name`.

Après la création d'un conteneur, il reste possible de changer de nom en utilisant la commande `docker container rename`.

```bash
docker container rename CONTENEUR NOUVEAU_NOM
```

Pour cibler le conteneur à renommer vous pouvez comme d'habitude utiliser le nom ou l'*ID* de celui-ci.

### Mettre en pause et reprendre les processus dans d'un conteneur

La commande `docker container pause` permet de suspendre tous les processus d'un ou de plusieurs conteneurs spécifiés.

Il utilise en fait la fonctionnalité du noyau *Linux* des *freezer cgroup* qui permet de suspendre les processus sans envoyer de signaux (*SIGSTOP*). Cela permet en fait que les processus ne puissent pas réagir car aucun signal n'est envoyé et de les "geler".

Ainsi, il est possible de mettre en pause et de reprendre n'importe quel processus.

Pour reprendre les processus d'un ou plusieurs conteneurs *Docker* il suffit ensuite d'utiliser `docker container unpause`.

Mettre en pause des conteneurs est utile dans de rares cas : par exemple si vous voulez mettre en pause des conteneurs trop gourmands en ressource sur certains serveurs le temps d'un pic de charge.

### Exécuter des commandes dans des conteneurs en cours d'exécution

**Pour exécuter des commandes dans des conteneurs en cours d'exécution il faut utiliser la commande `docker container exec`.**

La commande `docker container exec` n'exécute la commande passée en argument que tant que le premier processus du conteneur (avec le *PID 1*) est en cours d'exécution.

Vous pouvez passez également l'option `--detach` ou `-d` pour exécuter la nouvelle commande en arrière plan.

Vous pouvez également utiliser les options `--interactive` ou `-i` et `--tty` ou `-t` que nous avons déjà vues.

Prenons un exemple : nous allons commencer par démarrer un conteneur *redis* en arrière-plan avec le nom *redis* :

```bash
docker run -d --name redis redis
```

Nous allons ensuite exécuter le *CLI* de *redis* à l'intérieur du conteneur :

```bash
docker exec -it redis redis-cli
```

Nous pouvons ensuite taper une ou deux commandes redis qui fonctionneront :

```bash
set cle 42
get cle
```

### Obtenir un *shell* dans n'importe quel conteneur en cours d'exécution

Pour obtenir un terminal connecté à un *shell* dans n'importe quel conteneur vous pouvez utiliser :

```bash
docker exec -it redis bash
```

Si jamais *bash* n'est pas installé dans le conteneur, par exemple avec *alpine* vous pouvez bien sûr utiliser *sh* à la place :

```bash
docker exec -it redis sh
```

### Chemins sur Windows avec GitBash

Si vous utilisez *GitBash* sur *Windows*, *Git Bash* tente de convertir les chemins de fichiers *POSIX* (comme */app*) en chemins de fichiers *Windows*.

Ainsi, dans l'exemple de la vidéo, si vous utilisez */app*, *Git Bash* va essayer de convertir */app* en *C:/Program Files/Git/app *ce qui ne fonctionnera pas. Il faut donc faire :

```bash
docker container exec test mkdir //app
``` 

Pensez-y si vous rencontrez d'autres problèmes de chemins.