# Trouver et partager des images Docker

## Présentation de Docker Hub

### Le service _Docker Hub_

_Docker Hub_ est la plateforme d'image officielle de _Docker_. Il s'agit d'un _registry_, c'est-à-dire d'un service d'hébergement qui contient un très grand nombre de _repositories_ d'images.

Un _repository_ est un ensemble d'images _Docker_ qui sont distinguées par des _tags_.

_Docker Hub_ permet de fournir toutes les **images officielles**, disponibles à [cette adresse](https://hub.docker.com/search?q=&type=image&image_filter=official). Il y en a plus de 160 aujourd'hui.

Une image officielle est une image qui a été contrôlé par les équipes de _Docker_ et dont le _Dockerfile_ utilise les meilleures recommandations. La sécurité des images est régulièrement contrôlée et des mises à jour fréquentes sont effectuées. Les images officielles sont réalisées avec un partenariat entre _Docker_ et les équipes de développeurs des librairies (_nginx_, _redis_, _mongoDB_ etc).

**Les images officielles sont les seules images sur _Docker Hub_ qui ont comme _repository_ juste leur nom et pas _utilisateur/nom_.** Par exemple, simplement _node_, _redis_ etc.

_Docker Hub_ permet également de distribuer ses propres images de manière publique ou privée (comme _Github_ et _Gitlab_ pour le code).

Rendez-vous à cette [adresse](https://hub.docker.com/signup) pour créer un compte sur _Docker Hub_.

### Etude d'une image officielle

Nous allons voir ce que nous pouvons trouver lorsque nous nous rendons sur un _repository_ officiel, par exemple celui de _node_.

Vous aurez d'abord une partie de présentation :

![img](../../assets/images/Docker/image-4_19_1.png)

Vous pouvez retrouver le fait qu'il s'agit d'une image officielle (titre en bleu et _tag Official Image_ en gris).

Vous pouvez également voir le nombre de téléchargements, ici plus d'un milliard.

Vous avez la commande à taper pour télécharger l'image localement :

```bash
docker pull node
```

Vous avez ensuite tous les _tags_ disponibles pour l'image :

![img](../../assets/images/Docker/image-4_19_2.png)

Chaque ligne correspond à une image différente (avec un _ID_ d'image différent).

Les images varient en fonction de l'image de base utilisée (la distribution _Linux_ ou les librairies minimales) et la version de la librairie.

Par exemple ici, nous avons des images sur _Alpine_, sur des versions de _Debian_, des versions sur _slim_, sur _Ubuntu_ etc de _Node.js_ pour des versions de 10 à 15.

Sur la même ligne, vous avez tous les _tags_ qui correspondent à la même image.

Autrement dit, _15.0.1-alpine3.10_, _15.0-alpine3.10_, _15-alpine3.10_, _alpine3.10_, _current-alpine3.10_ désignent exactement la même image.

Vous pouvez le vérifier en faisant :

```bash
docker pull node:15.0.1-alpine3.10
docker pull node:15.0-alpine3.10
```

Puis :

```bash
docker image ls
```

Vous verrez les deux tags mais vous verrez aussi qu'ils correspondent au même _ID_ d'image. En outre, le deuxième `docker pull` n'entraînera aucun téléchargement.

Si vous cliquez sur une des versions (sur une des lignes), vous serez redirigé sur le _Dockerfile_ de l'image correspondante qui est la plupart du temps sur _Github_. Ces _Dockerfiles_ sont complexes car ils doivent permettre de s'assurer de la sécurité de l'image.

Vous trouverez ensuite toujours des explications sur les versions des images, ici dans _Images Variants_.

Vous aurez accès à une documentation, ici qui est sur [Github](https://github.com/nodejs/docker-node/blob/main/README.md#how-to-use-this-image).

## Publier ses images sur Docker Hub

### Les _repositories_ publics ou privés

Sur _Docker Hub_, une fois que vous avez créé votre compte gratuit, rendez vous dans _Repositories_.

Vous avez le droit à un seul _repository_ privé gratuit, en revanche vous avez le droit à autant de _repository_ publics que vous souhaitez.

### Publier une image sur un _repository_

Pour publier une image, il faut commencer par se connecter :

```sh
docker login
```

Il faut entrer les identifiants que vous avez entrés lors de la création de votre compte.

Suivant votre environnement votre mot de passe sera stocké de manière chiffrée (par exemple dans _keychain_ sur _MacOS_) ou dans un fichier de configuration en _Base64_ dans _$HOME/.docker/config.json_.

Si vous vous connectez à _Docker Hub_ depuis une machine partagée ou un serveur, il est très important de se déconnecter une fois terminé :

```sh
docker logout
```

Dans le cas contraire vous vous exposez au vol de votre mot de passe.

Une fois connecté, il faut recréer un _tag_ adapté à votre _repository_. Il doit être de la forme _utilisateur/repository_ et éventuellement avec une version :_version_.

L'utilisateur doit être l'utilisateur que vous avez créé sur _Docker hub_.

Par exemple :

```sh
docker build -t jean/mon-app:1.0.0 .
```

Une fois votre image construite et correctement taguée, il suffit de la pusher, c'est-à-dire la publier sur _Docker Hub_ pour la rendre accessible :

```sh
docker image push jean/mon-app:1.0.0
```

Vous pouvez ensuite tout supprimer :

```sh
docker system prune -a
```

Puis télécharger localement l'image que vous avez publiée :

```sh
docker image pull jean/mon-app:1.0.0
```

### Chiffrer ses identifiants _GNU/Linux_

Si vous êtes sur une distribution Debian, comme Ubuntu par exemple, suivez ces étapes pour pouvoir chiffrer vos identifiants Docker :

```sh
docker logout
```

Puis :

```sh
sudo apt install pass
```

Puis :

```sh
gpg2 --gen-key
```

Entrez bien votre nom et votre email lorsqu'ils sont demandés. Puis faites :

```bash
wget https://github.com/docker/docker-credential-helpers/releases/download/v0.6.3/docker-credential-pass-v0.6.3-amd64.tar.gz && tar -xf docker-credential-pass-v0.6.3-amd64.tar.gz && chmod +x docker-credential-pass && sudo mv docker-credential-pass /usr/local/bin/
```

Dans la commande suivante, entrez le même nom passé à gpg2

```bash
pass init "VOTRE NOM
```

Puis :

```bash
nano ~/.docker/config.json
```

Puis :

```json
{
  "credsStore": "pass"
}
```

Puis :

```bash
docker login
```

## Les commandes save et load

### La commande `docker image save`

La commande `docker image save` permet de sauvegarder une ou plusieurs images dans une archive _tar_.

Les archives sont un moyen de partager facilement des images sans passer par un _repository_.

Par défaut, l'archive est envoyée sur le flux de sortie standard (_STDOUT_), il faut donc utiliser l'option `-o` ou `--output` pour la sauvegarder dans un fichier.

Par exemple :

```sh
docker image save -o mon_image.tar mon_image
```

A noter que vous pouvez bien sûr compresser l'archive, par exemple avec _gzip_ :

```sh
docker save mon_image | gzip > mon_image.tar.gz
```

### La commande `docker image load`

La commande `docker image load` permet de charger une image depuis une archive passée en entrée standard.

Vous pouvez utiliser une redirection :

```sh
docker image load < mon_image.tar
```

Et vous pouvez également utiliser l'option `-i` ou `--input` :

```sh
docker image load -i mon_image.tar
```

## Les commandes export et import

### La commande `docker container export`

La commande `docker container export` permet d'exporter le système de fichiers d'un conteneur dans une archive _tar_.

Attention ! Cette commande n'exporte pas les volumes associés au conteneur, que nous étudierons en détails dans un prochain chapitre.

Par défaut, l'archive est envoyée sur le flux de sortie standard (_STDOUT_), il faut donc utiliser l'option `-o` ou `--output` pour la sauvegarder dans un fichier :

```sh
docker container export -o conteneur_fs.tar mon_conteneur
```

Vous pouvez également utiliser une redirection :

```sh
docker container export mon_conteneur > conteneur_fs.tar
```

### La commande `docker image import`

La commande `docker image import` permet d'importer le système de fichiers d'une archive _tar_ pour créer une image.

Elle accepte également les URLs, et vous pouvez donc récupérer des archives distantes. Si l'archive est compressée, le démon va automatiquement la décompresser et l'extraire.

```sh
docker image import http://exemple.com/exampleimage.tgz
```

Vous pouvez également utiliser une archive locale :

```sh
docker image import conteneur_fs.tar
```
