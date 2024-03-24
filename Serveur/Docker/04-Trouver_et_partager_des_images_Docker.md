## Trouver et partager des images Docker

### Présentation de Docker Hub

#### Le service _Docker Hub_

_Docker Hub_ est la plateforme d'image officielle de _Docker_. Il s'agit d'un _registry_, c'est-à-dire d'un service d'hébergement qui contient un très grand nombre de _repositories_ d'images.

Un _repository_ est un ensemble d'images _Docker_ qui sont distinguées par des _tags_.

_Docker Hub_ permet de fournir toutes les **images officielles**, disponibles à [cette adresse](https://hub.docker.com/search?q=&type=image&image_filter=official). Il y en a plus de 160 aujourd'hui.

Une image officielle est une image qui a été contrôlé par les équipes de _Docker_ et dont le _Dockerfile_ utilise les meilleures recommandations. La sécurité des images est régulièrement contrôlée et des mises à jour fréquentes sont effectuées. Les images officielles sont réalisées avec un partenariat entre _Docker_ et les équipes de développeurs des librairies (_nginx_, _redis_, _mongoDB_ etc).

**Les images officielles sont les seules images sur _Docker Hub_ qui ont comme _repository_ juste leur nom et pas _utilisateur/nom_.** Par exemple, simplement _node_, _redis_ etc.

_Docker Hub_ permet également de distribuer ses propres images de manière publique ou privée (comme _Github_ et _Gitlab_ pour le code).

Rendez-vous à cette [adresse](https://hub.docker.com/signup) pour créer un compte sur _Docker Hub_.

#### Etude d'une image officielle

Nous allons voir ce que nous pouvons trouver lorsque nous nous rendons sur un _repository_ officiel, par exemple celui de _node_.

Vous aurez d'abord une partie de présentation :

![](/00-assets/images/Docker/image-4_19_1.png)

Vous pouvez retrouver le fait qu'il s'agit d'une image officielle (titre en bleu et _tag Official Image_ en gris).

Vous pouvez également voir le nombre de téléchargements, ici plus d'un milliard.

Vous avez la commande à taper pour télécharger l'image localement :

```bash
docker pull node
```

Vous avez ensuite tous les _tags_ disponibles pour l'image :

![](/00-assets/images/Docker/image-4_19_2.png)

Chaque ligne correspond à une image différente (avec un *ID* d'image différent).

Les images varient en fonction de l'image de base utilisée (la distribution *Linux* ou les librairies minimales) et la version de la librairie.

Par exemple ici, nous avons des images sur *Alpine*, sur des versions de *Debian*, des versions sur *slim*, sur *Ubuntu* etc de *Node.js* pour des versions de 10 à 15.

Sur la même ligne, vous avez tous les *tags* qui correspondent à la même image.

Autrement dit, *15.0.1-alpine3.10*, *15.0-alpine3.10*, *15-alpine3.10*, *alpine3.10*, *current-alpine3.10* désignent exactement la même image.

Vous pouvez le vérifier en faisant :

```bash
docker pull node:15.0.1-alpine3.10
docker pull node:15.0-alpine3.10
```

Puis :

```bash
docker image ls
```

Vous verrez les deux tags mais vous verrez aussi qu'ils correspondent au même *ID* d'image. En outre, le deuxième `docker pull` n'entraînera aucun téléchargement.

Si vous cliquez sur une des versions (sur une des lignes), vous serez redirigé sur le *Dockerfile* de l'image correspondante qui est la plupart du temps sur *Github*. Ces *Dockerfiles* sont complexes car ils doivent permettre de s'assurer de la sécurité de l'image.

Vous trouverez ensuite toujours des explications sur les versions des images, ici dans *Images Variants*.

Vous aurez accès à une documentation, ici qui est sur [Github](https://github.com/nodejs/docker-node/blob/main/README.md#how-to-use-this-image).
