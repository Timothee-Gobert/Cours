## Créer une image avec un Dockerfile

### Introduction au Dockerfile

#### Qu'est-ce qu'un Dockerfile ?

Jusqu'à maintenant nous avons utilisé des images officielles, mais la plupart du temps vous utiliserez vos propres images qui sont configurées pour contenir votre application et toute la configuration dont elle a besoin.

Dans ce chapitre nous allons donc voir comment créer vos images, en commençant par les _Dockerfiles_.

**Un _Dockerfile_ contient les instructions permettant à _Docker_ de construire (_build_) une image automatiquement.**

Ce fichier contient donc toutes les commandes nécessaires à la construction de l'image que vous souhaitez.

Comme nous le verrons un _Dockerfile_ est constituée de trois parties :

**1 - Une image de base.** C'est l'image à partir de laquelle vous allez effectuer les modifications pour créer votre image. Vous n'allez en effet pas créer un système d'exploitation tout seul !

**2 - Les instructions.** Elles sont des commandes Docker permettant de détailler toutes les modifications à apporter à l'image de base pour aboutir à votre image finale, par exemple votre application backend.

**3 - L'action.** C'est la commande qui est exécutée par défaut lorsque l'image sera lancée dans un conteneur.

#### Installer l'extension VS Code Docker

**L'extension _VS Code Remote Docker_ est une extension officielle de _Microsoft_ permettant de gérer très simplement les conteneurs depuis _VS Code_ :**

![](/00-assets/images/Docker/image-3_13_1.png)

Nous verrons au fur et à mesure du cours ses fonctionnalités. Mais notez déjà l'onglet *Docker* (baleine remplie de conteneurs) à gauche après l'installation.

Pour l'installer rapidement faites *CTRL + P* puis entrez :

```bash
ext install ms-azuretools.vscode-docker
```

Vous pouvez également l'installer manuellement en allant dans l'onglet Extensions et en recherchant *docker*.

#### Créer notre premier *Dockerfile*

Dans un nouveau dossier, créez un fichier *Dockerfile*. Respectez exactement la casse et n'ajoutez aucune extension.

Dans ce fichier placez le code suivant :

```bash
FROM alpine

RUN apk add --update nodejs

COPY ./app.js /app/

CMD [ "node", "/app/app.js" ]
```

Nous détaillerons bien sûr toutes les instructions dans les leçons suivantes, pour l'instant contentez vous de les copier pour construire votre première image.

Notez que *apk* est le gestionnaire de paquets de *Alpine*, c'est l'équivalent de *APT* pour les distributions *Debian* et donc notamment pour *Ubuntu*.

`apk add` permet d'installer un paquet comme `apt install`.

`apk update` permet de mettre à jour les paquets disponibles, comme `apt update`.

`apk add --update` permet en fait de d'abord faire apk update puis `apk add`. Cela revient à faire `apt update && apt install` sur *Debian*.

Créez également un fichier dans le même dossier, *app.js*, dans lequel vous mettez simplement :

```js
console.log("Bonjour !");
```

Attention à bien mettre le chemin */app/* avec un *slash* à la fin, sinon *Docker* considérera qu'il s'agit du fichier *app* et non le dossier *app* et ne placera pas *app.js* dans un nouveau dossier */app* mais renommera le fichier *app.js* en *app*.

Attention également à bien utiliser les guillemets doubles et non simples dans *CMD* car ils permettent d'échapper les slashs (cf formation *Linux*).

#### Construire une image

Pour créer une image à partir d'un *Dockerfile* il faut la *build*. Pour ce faire, il suffit d'utiliser `docker image build CHEMIN`.

Le processus de construction d'une image (*build*) utilise **un *Dockerfile* et un contexte**.

**Le contexte est l'ensemble des fichiers contenus dans le chemin passé à la commande `docker image build`.** Vous aurez ainsi par exemple lors du *build* :

```bash
Sending build context to Docker daemon 3.51 MB
```

Cela signifie que les fichiers et les dossier contenus dans le chemin passé en argument à la commande font *3,51 Mo*.

C'est pour cette raison qu'il faut créer le *Dockerfile* dans le dossier de votre application, ou ici pour tester, dans un dossier à part. Si vous créez votre *Dockerfile* directement dans le répertoire racine */*, l'ensemble de votre disque dur est envoyé comme contexte au démon !

**Le *build* est effectué par le démon *Docker*.**

A supposer que vous êtes dans le dossier qui contient le *Dockerfile*, vous pouvez donc faire :

```bash
docker image build .
```

`.` étant le dossier courant.

Vous aurez alors le résultat suivant :

```bash
Sending build context to Docker daemon 3.072kB
Step 1/4 : FROM alpine
latest: Pulling from library/alpine
188c0c94c7c5: Pull complete
Digest: sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e7841a5b5a
Status: Downloaded newer image for alpine:latest
---> d6e46aa2470d
Step 2/4 : RUN apk add --update nodejs
---> Running in bfa6e6159c03
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/8) Installing ca-certificates (20191127-r4)
(2/8) Installing brotli-libs (1.0.9-r1)
(3/8) Installing c-ares (1.16.1-r0)
(4/8) Installing libgcc (9.3.0-r2)
(5/8) Installing nghttp2-libs (1.41.0-r0)
(6/8) Installing libstdc++ (9.3.0-r2)
(7/8) Installing libuv (1.38.1-r0)
(8/8) Installing nodejs (12.18.4-r0)
Executing busybox-1.31.1-r19.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 37 MiB in 22 packages
Removing intermediate container bfa6e6159c03
---> 00f5133d389e
Step 3/4 : COPY ./app.js /app
---> 5985a238bb14
Step 4/4 : CMD [ "node", '/app/app.js' ]
---> Running in 8f3fb48afa08
Removing intermediate container 8f3fb48afa08
---> 526fb4a033d6
Successfully built 526fb4a033d6
```

#### Taguer une image

Pour donner un nom à une image, c'est ce qu'on appelle un *taguer* ou donner un *tag*, il faut utiliser l'option `-t` de la commande *build*.

Nous pouvons par exemple faire :

```bash
docker build -t node:latest .
```

Vous obtiendrez cette fois (après avoir tout nettoyé avec `docker system prune -a`) :

```bash
Sending build context to Docker daemon 3.072kB
Step 1/4 : FROM alpine
latest: Pulling from library/alpine
188c0c94c7c5: Pull complete
Digest: sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e7841a5b5a
Status: Downloaded newer image for alpine:latest
---> d6e46aa2470d
Step 2/4 : RUN apk add --update nodejs
---> Running in bc5c9a4cd53e
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/8) Installing ca-certificates (20191127-r4)
(2/8) Installing brotli-libs (1.0.9-r1)
(3/8) Installing c-ares (1.16.1-r0)
(4/8) Installing libgcc (9.3.0-r2)
(5/8) Installing nghttp2-libs (1.41.0-r0)
(6/8) Installing libstdc++ (9.3.0-r2)
(7/8) Installing libuv (1.38.1-r0)
(8/8) Installing nodejs (12.18.4-r0)
Executing busybox-1.31.1-r19.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 37 MiB in 22 packages
Removing intermediate container bc5c9a4cd53e
---> d87483fefe3e
Step 3/4 : COPY ./app.js /app
---> 1f8aef922d23
Step 4/4 : CMD [ "node", '/app/app.js' ]
---> Running in 6af9582dba62
Removing intermediate container 6af9582dba62
---> ae97a4b96431
Successfully built ae97a4b96431
Successfully tagged node:latest
```

#### Lister notre image et l'exécuter

Maintenant que nous avons *build* et tagué notre image, nous pouvons l'afficher en faisant :

```bash
docker image ls
```

Vous verrez le nom de l'image dans la colonne *REPOSITORY* et le tag dans la colonne *TAG*

Vous pouvez maintenant démarrer un conteneur à partir de votre image en faisant :

```bash
docker run node
```

