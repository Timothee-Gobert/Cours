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
