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

