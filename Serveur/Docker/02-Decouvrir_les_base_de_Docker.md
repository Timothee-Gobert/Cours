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

