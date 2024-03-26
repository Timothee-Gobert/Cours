## Utiliser Docker Compose

### Introduction à Docker compose

#### Qu'est-ce que *Docker Compose* ?

***Docker Compose* permet de faciliter la création d'application multi-conteneur.**

Dans une application vous avez au moins deux ou trois conteneurs : base de données, backend et application client.

Souvent viennent s'ajouter un reverse proxy pour permettre le routing vers différentes applications et de la répartition de charge (*load balancing*).

*Docker Compose* utilise des fichiers de configuration utilisant le langage *YAML*. Nous en verrons la syntaxe dans la leçon suivante.

***Docker Compose* ne remplace pas du tout les *Dockerfiles* !**

***Docker Compose* s'utilise de la manière suivante :**

1 - Votre application comporte plusieurs *Dockerfiles*, un par image que vous voulez utiliser.

2 - Votre application a un fichier *docker-compose.yml* qui définit les services que votre application a besoin pour fonctionner. Un service est par exemple une base de données, une *API Rest*, un *reverse proxy*, une application *SPA* etc.

3 - Vous lancez la commande `docker compose up` qui va démarrer tous les services suivant les options définies.

Nous reviendrons bien évidemment sur l'ensemble de ces étapes en détails.

#### Les avantages à utiliser Docker Compose

**Pour le développement,** plus besoin de guide d'installation pour votre équipe ! En une seule commande toute l'application s'installe et démarre avec la configuration adéquate. Plus besoin d'installer les dépendances pour chaque partie de votre application vous même, de mettre les bons ports, les bons réseaux, les bons volumes etc.

**Pour les test,** la possibilité de mettre en place votre application rapidement et de lancer des tests automatiquement accélère grandement vos mises en production.

`Docker Compose` ne recrée que les conteneurs dont la configuration a changé, sinon il les réutilise. Cela permet d'accélérer grandement l'arrêt et le redémarrage de votre application, ce qui est particulièrement utile en développement.

Pour vérifier que Docker Compose est bien installé :

```sh
docker compose version
```
### Première utilisation de Docker Compose

#### Le langage *YAML*

Le *YAML* (pour *YAML Ain't Markup Language* ou "YAML n'est pas un langage de balisage") est un langage permettant de représenter des informations élaborées tout en conservant une grande lisibilité. Il repose principalement sur l'indentation.

C'est un langage utilisé le plus souvent pour des fichiers de configuration.

Le langage *YAML* est extrêmement utilisé dans le monde du *devops* : *Docker*, tous les outils de *CI*, *ansible* et tant d'autres utilisent ce langage simple.

Nous allons voir les principaux éléments de la syntaxe de ce langage car tout ce que nous verrons à partir de maintenant utilisera du *YAML* :

`#` permet de commenter une ligne s'il est placé au début d'une ligne. S'il est avant une chaîne de caractère il signifie nombre littéral.

`~` signifie valeur nulle.

`-` permet de créer des listes. Chaque élément d'une liste doit être précédé par `-` puis un espace. Il doit y avoir un élément par ligne.

`clé: valeur` permet de déclarer un map clé / valeur.

L'indentation par des espaces permet de créer une hiérarchie, par exemple :

```yaml
paul:
nom: Paul Dupont
emploi: Developer
```

Une structure plus complexe :

```yaml
- paul:
  nom: Paul Dupont
  emploi: Developer
  languages:
    - javascript
    - css
    - html
```

#### Première utilisation : `docker compose up`

Commencez par créer un nouveau dossier, par exemple *compose* :

```sh
mkdir compose
cd compose
```

Dans ce dossier, créez un fichier *docker-compose.yml* ou *docker-compose.yaml* (les deux noms d'extensions sont possibles, par contre le nom du fichier est important) :

```yaml
version: "3.9"
services:
  myalpine:
    image: alpine
```

Ici nous définissons un service *myalpine*. Le nom des services est totalement libre. La recommandation est d'utiliser un nom descriptif de leur rôle, par exemple *api*, *db* etc.

La clé *image* permet de spécifier sur quelle image est basée le service. Ici, nous utilisons l'image *alpine*.

Sauvegardez et faites :

```sh
docker compose up
```

Vous aurez alors :

```sh
Creating network "compose_default" with the default driver
Pulling myalpine (alpine:)...
latest: Pulling from library/alpine
188c0c94c7c5: Pull complete
Digest: sha256:c0e9560cda118f9ec63ddefb4a173a2b2a0347082d7dff7dc14272e7841a5b5a
Status: Downloaded newer image for alpine:latest
Creating compose_myalpine_1 ... done
Attaching to compose_myalpine_1
compose_myalpine_1 exited with code 0
```

Chaque ligne est importante :

*Creating network "compose_default"* indique que *Docker* crée un réseau *bridge* par défaut (qui n'est propre à *Docker Compose* et n'est pas le réseau *bridge* que nous avions par défaut). Le nom du réseau est le nom du dossier où se trouve le fichier *docker-compose.yml* postfixé avec *_default*.

Vous pouvez retrouver ce nouveau réseau en faisant :

```sh
docker network ls
```

Ensuite, *Docker Compose* va cherche l'image indiquée pour le service *myalpine*.

Il va *pull* l'image sur *Docker Hub* comme `docker run` le ferait.

Il va ensuite démarrer un conteneur basé sur cette image *Creating compose_myalpine_1*. Le nom du conteneur est le nom du dossier, puis le nom du service et enfin un chiffre qui s'incrémente si nous lançons plusieurs conteneurs pour un service (nous verrons cela avec *swarm*).

Enfin, comme avec `docker run`, *Docker Compose* va attacher le conteneur lancé au terminal pour avoir les sorties d'erreur et standard.

Dans notre exemple il quitte immédiatement car aucune commande n'est passée.

#### Lancer une commande sur un service avec `docker compose run`

**La commande `docker compose run` permet d'exécuter une commande par un service.**

Par exemple :

```sh
docker compose run myalpine
```

Va démarrer un conteneur pour le service *myalpine* et exécuter la commande par défaut (qui est *sh* pour l'image *alpine*).

La commande va utiliser toute la configuration définie pour le service (pour le moment nous n'en avons pas) à l'exception des ports, pour éviter d'éventuelles collisions avec des ports déjà ouverts.

Vous pouvez remplacer la commande par défaut de l'image en passant une autre commande en argument :

```sh
docker compose run myalpine ls
```

Pour l'image *alpine*, la commande par défaut est *sh*. Vous remplacez donc *sh* par `ls`.

#### Lister les conteneurs lancés avec `Docker compose`

Vous pouvez lister tous les conteneurs en cours d'exécution en faisant :

```sh
docker container ls
```

Mais si vous voulez uniquement les conteneurs lancés par `Docker Compose` vous pouvez faire :

```sh
docker compose ps
```

Vous pouvez également lui passer l'option `-a` pour voir également les conteneurs lancés par `docker compose run`.

#### Stopper les conteneurs lancés avec `Docker compose`

Vous pouvez stopper tous les conteneurs lancés par `Docker compose` avec la commande :

```sh
docker compose down
```

La commande **va automatiquement supprimer tous les conteneurs stoppés,** comme si vous les aviez lancés avec l'option `--rm`.

Il va également supprimer les réseaux.

Pour supprimer également les volumes créés par `Docker Compose`, anonymes et nommés, il suffit de spécifier l'option `-v` :

```sh
docker compose down -v
```

A noter que les volumes anonymes ne sont jamais réutilisés par `Docker Compose` comme nous le verrons. Il en lance des nouveaux à chaque fois si vous en déclarez dans votre configuration.

#### Premières configurations

Les configurations des services sont appliquées à chaque démarrage de conteneur pour ces services.

Cela revient à exécuter `docker run` avec toutes les options définies en configuration, ce qui est un énorme gain de temps !

##### *command*

La configuration *command* permet de remplacer la commande par défaut de l'image.

Essayez en faisant :

```yaml
version: "3.9"
services:
myalpine:
  image: alpine
  command: ls
```

Refaites :

```sh
docker compose up
```

Pour les commandes de plus d'un mot, il vaut mieux utiliser l'autre syntaxe que nous connaissons : *command: ["npm", "run", "start"]*.

##### *entrypoint*

La configuration *entrypoint* permet de remplacer l'*entrypoint* par défaut de l'image. Nous n'allons pas revoir la distinction avec la commande que nous avons vu en détails dans un chapitre précédent.

Essayez en faisant :

```yaml
version: "3.9"
services:
myalpine:
  image: alpine
  entrypoint: ls
```

Refaites :

```sh
docker compose up
```

### Utilisation d'images personnalisées avec Docker Compose

Dans cette leçon nous allons voir comment `Docker Compose` s'utilise avec des images personnalisées. Autrement dit, comme il s'utilise avec des *Dockerfiles*.

#### Utilisation avec un *Dockerfile*

Commencez par créer un *Dockerfile* dans le même dossier :

```dockerfile
FROM alpine
CMD ["/bin/sh"]
```

Ensuite, modifiez le fichier *docker-compose.yml* pour l'utiliser :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build: .
```

Nous avons maintenant deux services : un premier, le service **a**, qui utilise l'image officielle *alpine*, et un second qui utilise notre *Dockerfile*.

En effet, l'option de configuration *build* permet de définir le chemin du contexte de *build*. Il n'y a rien besoin d'autre si le *Dockerfile* et dans le même dossier.

Si le *Dockerfile* **a** un autre nom et / ou si le contexte de *build* est dans un autre dossier, il faut utiliser la configuration plus détaillée, par exemple :

```yaml
build:
 context: ./dossier
 dockerfile: Dockerfile.dev
```

Créons un dossier *backend* et plaçons y notre *Dockerfile*. Nous devons maintenant écrire :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build:
      context: ./backend
      dockerfile: Dockerfile
```

#### Construire les images avec `docker compose build`

**La commande `docker compose build` permet de construire toutes les images spécifiées dans le fichier de configuration (dans les configurations *build*) et de les taguer pour une future utilisation.**

Essayez de faire :

```sh
docker compose build
```

Et listez ensuite les images :

```sh
docker image ls
```

L'image sera taguée comme ceci : **compose_b**, à savoir nom du dossier contenant le fichier *docker-compose.yml* puis le nom du service.

#### Passer des arguments et des labels durant le *build*

Nous avons déjà étudié l'instruction *ARG* pour passer des arguments et *LABEL* pour indiquer des labels dans des images personnalisées.

Il est possible de la même manière de passer des arguments ou des labels durant le *build* de l'image dans un fichier de configuration `Docker Compose`. En utilisant simplement *args* et *labels* dans la configuration du *build*.

Par exemple, si nous avons dans notre *Dockerfile* :

```dockerfile
FROM alpine
ARG FOLDER
RUN mkdir $FOLDER
CMD ["/bin/sh"]
```

Vous pouvez passer l'argument dans le fichier de configuration de `Docker Compose` :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build:
      context: ./backend
      dockerfile: Dockerfile
      args:
        - FOLDER=test
```

Vous pouvez tester :

```sh
docker compose build
```

Vous pouvez également passer des labels :

```yaml
version: '3.9'
services:
  a:
    image: alpine
    command: ['ls']
  b:
    build:
      context: ./backend
      dockerfile: Dockerfile
      args:
        - FOLDER=test
      labels:
        - email=jean@gmail.com
```

### Les ports et les volumes avec Docker Compose

#### Publier des ports

Vous pouvez définir quels ports doivent être publiés pour un service dans le fichier de configuration *docker-compose.yml*.

Pour ce faire, il suffit d'utiliser *ports*. Le format est toujours *HOTE:CONTENEUR* puis par défaut *TCP*. Vous pouvez préciser *UDP* avec */upd* :

```yaml
ports:
  - "80:80"
  - "8080:3000/udp"
```

Nous y reviendrons bien sûr en détails avec plusieurs exemples.

#### Créer des volumes

Il est possible de monter des chemins de l'hôte (*bind mount*) ou de créer des volumes anonymes ou nommés.

##### Créer des bind mounts

Reprenons notre exemple en créant un dossier */app* dans le *Dockerfile* :

```dockerfile
FROM alpine
ARG FOLDER
WORKDIR /app
RUN mkdir $FOLDER
CMD ["/bin/sh"]
```

Supposons que nous ayons un dossier *data* sur l'hôte que nous voulions *bind mount*, nous n'aurions qu'à faire :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
```

Il est bien sûr possible de préciser plusieurs volumes. Il faut simplement que chaque volume soit un élément de liste, donc précédé par un tiret puis un espace.

##### Créer des volumes anonymes

Pour créer un volume anonyme il faut simplement déclarer un volume de type *volume* et de spécifier uniquement une *target* (ou *destination* / *dst*) et pas de *source*. Dans ce cas `Docker Compose` créera automatiquement un volume anonyme :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
    - type: volume
      target: /app/data2
```

##### Créer des volumes nommés

Pour créer un volume nommé, il faut déclarer dans une configuration au même niveau que services une clé *volumes* : ce sont les volumes nommés qui seront automatiquement créés par `Docker Compose`.

Pour les utiliser, il suffit ensuite dans la configuration *volumes* d'un service, d'indiquer en *source* le nom du volume nommé créé :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
    - type: volume
      target: /app/data2
    - type: volume
      source: data3
      target: /app/data3
volumes:
  data3:
```

Ici *data3* est un volume nommé créé et qui est monté dans le service **b**.

Notez bien la syntaxe particulière de *data3*:. Si vous avez plusieurs volumes nommés un faut en mettre un par ligne et ne pas oublier les `:` :

```yaml
volumes:
  data1:
  data2:
  data3:
```

##### Utiliser des volumes nommés déjà créés

Si vous ne voulez pas que `Docker Compose` crée un nouveau volume nommé mais utilise un existant (que vous avez créé en dehors de `Docker Compose`), il faut préciser *external: true* dans la configuration, de cette manière :

```yaml
version: '3.9'
services:
a:
  image: alpine
  command: ['ls']
b:
  build:
    context: ./backend
    dockerfile: Dockerfile
    args:
      - FOLDER=test
    labels:
      - email=jean@gmail.com
  volumes:
    - type: bind
      source: ./data
      target: /app/data
    - type: volume
      target: /app/data2
    - type: volume
      source: data3
      target: /app/data3
volumes:
  data3:
    external: true
```

Ici *data3* ne sera pas créé par `Docker Compose`. Il utilisera le volume déjà créé. Si aucun volume avec ce nom n'existe, vous aurez l'erreur suivante :

```sh
ERROR: Volume data3 declared as external, but could not be found. Please create the volume manually using `docker volume create --name=data3` and try again.
```

