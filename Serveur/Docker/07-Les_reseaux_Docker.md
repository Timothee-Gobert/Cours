## Les réseaux Docker

### Introduction aux réseaux

#### Connecter les conteneurs entre eux avec les réseaux

L'une des grandes puissances de *Docker* est la possibilité de connecter les conteneurs entre eux très facilement.

Vous pouvez finement paramétrer comment ils se connectent grâce aux réseaux *Docker* c'est ce que nous allons voir dans ce chapitre.

#### Les *drivers* des réseaux *Docker*

Le système de réseaux *Docker* utilisent des *drivers*. Il en existe plusieurs par défaut.

##### Le *driver bridge*

**Le *bridge* est le *driver* par défaut pour les réseaux *Docker*.**

Nous les verrons en détails, mais ces réseaux sont utiles lorsque vous essayez de faire communiquer des conteneurs sur une même machine.

##### Le *driver host*

**Le *host* est le *driver* permettant de supp rimer l'isolation réseau d'un ou plusieurs conteneurs avec l'hôte.**

Il permet de complètement supprimer la gestion du réseau par *Docker*, c'est comme si les conteneurs tournaient directement sur l'hôte. Attention, uniquement pour le réseau ! (Par pour l'isolation des processus, du système de fichiers etc).

##### Le *driver overlay*

**L'*overlay* est le *driver* permettant de connecter plusieurs démons *Docker*.**

Il permet en fait de déployer des conteneurs **sur plusieurs machines distantes** et des les faire communiquer. Nous verrons tout cela lorsque nous étudierons *Swarm*.

Les autres *drivers* possibles sont *none* (pour utiliser un *plugin* pour la gestion des réseaux qui ne sont pas gérés par *Docker*) et *macvlan* (qui permet d'utiliser des adresses *MAC* aux conteneurs). Ces autres possibilités sont vraiment pour des cas très rares et nous ne les étudierons donc pas. 

### Le réseau bridge

#### Les réseaux *bridge*

Un *bridge network* ou pont réseau permet d'interconnecter deux segments de réseaux distincts.

**Dans *Docker*, un réseau *bridge* utilise un pont logiciel qui permet à plusieurs conteneurs connectés à ce réseau de communiquer.**

Ce réseau permet de maintenir une isolation avec tous les autres conteneurs qui ne sont pas connectés au réseau *bridge*.

**Les réseaux *bridge* ne sont utilisables que pour les conteneurs qui sont exécutés sur le même hôte.**

Pour faire communiquer des conteneurs qui sont situés sur des hôtes différents il faut utiliser un réseau *overlay* comme nous le verrons.

**Lorsque vous lancez *Docker* sur une machine, un réseau *bridge* par défaut est créé automatiquement.**

Tous les conteneurs qui sont ensuite créés rejoignent ce réseau par défaut. Vous pouvez bien sûr empêcher cela en spécifiant un autre réseau.

Le problème de ce réseau par défaut est que les conteneurs ne peuvent communiquer qu'en utilisant leur adresse *IP*, ce qui est très peu flexible. Sans compter sur le fait que les adresses *IP* ne sont pas fixes pour chaque conteneur : sur différents hôtes elles ne seront pas les mêmes.

Nous allons donc étudier ce réseau par défaut, puis voir la bonne manière d'utiliser les réseaux *bridge* : en créant ses propres réseaux qui ont de nombreux avantages comme nous le verrons.

#### Lister les réseaux existants

Vous pouvez lister les réseaux existants avec la commande suivante :

```sh
docker network ls
```

Vous verrez que vous avez notamment un réseau *bridge* par défaut, comme par exemple :

![](/00-assets/images/Docker/image-7_36_1.png)

#### Inspecter les réseaux

Vous pouvez voir les détails d'un réseau avec :

```sh
docker network inspect
```

Pour inspecter le réseau par défaut, faites :

```sh
docker network inspect bridge
```

#### Les conteneurs se connectent au réseau *bridge* par défaut

Nous allons partir d'une nouvelle base :

Commencez par couper tous les conteneurs :

```sh
docker stop $(docker container ps -aq)
```

Supprimez tous les conteneurs stoppés et les volumes :

```sh
docker container prune
docker volume prune
```

Lançons ensuite un premier conteneur :

```sh
docker container run -it alpine sh
```

Dans un nouveau terminal vérifiez que ce nouveau conteneur est bien connecté au réseau *bridge* :

```sh
docker network inspect bridge
```

Intéressez-vous à la partie *Containers*, vous aurez par exemple :

```json
"Containers": {
      "f3d3c575a149c8a484554e8a2204af5d5e4a38d839a389f284895f7c287b2b9d": {
    "Name": "mystifying_ardinghelli",
    "EndpointID": "faf31c95fc48a6e5a7abdf885b76ee62589e05f5a2637aa49df2badcdc64bac6",
    "MacAddress": "02:42:ac:11:00:02",
    "IPv4Address": "172.17.0.2/16",
    "IPv6Address": ""
      }
},
```

Notez son adresse *IP* : *172.17.0.2*.

Lancez ensuite un autre conteneur :

```sh
docker container run -it alpine sh
```

Vous devez donc avoir deux terminaux avec deux conteneurs en cours d'exécution.

Sur le terminal où vous venez de lancer le second conteneur faites :

```sh
ifconfig
```

Cherchez la partie *inet addr* pour la carte *eth0*, vous aurez par exemple :

![](/00-assets/images/Docker/image-7_36_2.png)

```sh
inet addr:172.17.0.3
```

Vous avez maintenant les deux *IP* de vos conteneurs.

Vous pouvez dans le premier conteneur faire :

```sh
ping 172.17.0.3
```

Et dans le second :

```sh
ping 172.17.0.2
```

Vous aurez par exemple :

Cela montre que le conteneur a pu envoyer des paquets de 64 octets sans problème à l'autre conteneur.

Si les deux conteneurs n'étaient pas sur le même réseau ils ne pourraient pas communiquer, et donc ne pourraient pas s'envoyer de paquets.

Vous voyez à quel point l'utilisation du réseau *bridge* par défaut est laborieux : il faut récupérer pour chaque conteneur son *IP* attribuée par *Docker*.

Ce n'est pas possible pour gérer convenablement un environnement *Docker*, imaginez si vous deviez aller sur votre serveur et récupérer les *IPs* pour pouvoir configurer la connexion entre votre *backend*, votre base de données etc.

Heureusement il existe une solution très simple à mettre en oeuvre et qui est prévue par *Docker* !

#### Connecter deux conteneurs avec link

Il est possible de connecter des conteneurs sans avoir à utiliser leur *IP* en utilisant l'option `--link`.

**Attention ! Cette option est dépréciée et sera à terme supprimée.** Nous vous la montrons uniquement car vous pouvez encore la rencontrer et il faut donc la comprendre. Mais il ne faut plus l'utiliser, il faut créer ses réseaux, comme nous le verrons dans la leçon suivante.

Quittez les deux conteneurs s'ils étaient encore en cours d'exécution.

Créez un premier conteneur en lui donnant un nom :

```sh
docker container run --name alpine1 -it alpine sh
```

Créez ensuite un second conteneur dans un deuxième terminal, en utilisant l'option *link* :

```sh
docker container run  --name alpine2 --link alpine1 -it alpine sh
```

Remarquez que dans le conteneur *alpine2* vous pouvez envoyer des paquets à *alpine1* :

```sh
ping alpine1
```

Par contre dans *alpine1*, vous ne pouvez pas envoyer de paquets à *alpine2* :

```sh
ping alpine2
```

La raison est qu'un `--link` est unidirectionnel. Il faudrait créer un `--link` également pour le premier conteneur. C'est pour ces complexités que *Docker* a décidé d'abandonner `--link` et de passer à des réseaux personnalisés. 

### Créer ses réseaux bridge

#### Les réseaux *bridges* créés par l'utilisateur

Nous avons vu que *Docker* fournit un réseau *bridge* par défaut auquel tous les conteneurs se connectent.

**Ce réseau *bridge* par défaut ne doit pas être utilisé.** *Docker* signale en effet qu'il devrait être déprécié, mais qu'il est nécessaire de le conserver pour des raisons de compatibilité.

Nous pouvons également créer nos propres réseaux *bridge*, ces réseaux sont bien meilleurs pour les raisons suivantes :
- ils permettent **une résolution *DNS* automatique entre les conteneurs**. En utilisant le réseau *bridge* par défaut, les conteneurs ne peuvent interagir qu'en utilisant leur adresse *IP* (comme nous l'avons vu dans la leçon précédente). Sur un réseau *bridge* que nous créons, les conteneurs peuvent utiliser leur nom pour se trouver sur le réseau.
- ils permettent **une meilleure isolation**. Si vous ne spécifiez pas un réseau pour un conteneur il se connecte au réseau *bridge* par défaut (comme nous l'avons vu également). Cela diminue la sécurité, car des conteneurs qui n'appartiennent pas aux mêmes applications ou services peuvent communiquer à travers ce réseau. Avec des réseaux personnalisés nous spécifions quels conteneurs sont sur quels réseaux et donc pouvons définir exactement quels conteneurs peuvent communiquer entre eux.
- ils permettent **de connecter ou déconnecter des conteneurs lors de l'exécution**. Ce n'est pas possible avec le réseau par défaut.
- ils permettent une configuration différente pour chaque réseau. Ce que ne permet pas le réseau par défaut, car il y en a qu'un seul.

Aussi, n'utilisez que des réseaux *bridge* personnalisés ! Nous verrons d'ailleurs qu'avec *Docker compose* ils sont encore plus simples à configurer.

#### Créer un réseau *bridge* personnalisé

Pour créer un réseau, il suffit d'utiliser :

```sh
docker network create
```

Par défaut, un réseau créé de cette manière sera de type *bridge*.

Nous allons pouvoir montrer la résolution *DNS* automatique avec un réseau personnalisé.

Commençons par créer un réseau :

```sh
docker network create mynet
```

Vérifions que nous le voyons bien :

```sh
docker network ls
```

Passons ensuite à la création de nouveaux conteneurs, en spécifiant cette fois-ci qu'ils sont connectés au réseau que nous venons de créer **grâce à l'option** `--network`.

```sh
docker container run --network mynet --name alpine1 -d alpine ping google.fr
```

Ce premier conteneur va simplement envoyer des requêtes en continu à *google.fr*.

Démarrons maintenant un second conteneur :

```sh
docker container run --network mynet --name alpine2 -it alpine sh
```

Essayons de contacter *alpine1* :

```sh
ping alpine1
```

Génial ! Grâce à la résolution *DNS* automatique de *Docker*, le nom *alpine1* est résolu automatiquement en une adresse *IP* et permet la communication avec l'autre conteneur.

Vérifions maintenant dans un nouveau terminal le réseau par défaut :

```sh
docker network inspect bridge
```

Vous verrez que nos deux conteneurs n'y sont plus connectés par défaut : *"Containers": {},*.

En effet, du moment que nous spécifions un autre réseau, les conteneurs ne se connectent plus au réseau *bridge* par défaut.

Par contre, ils sont bien connectés sur notre réseau *bridge* personnalisé :

```sh
docker network inspect mynet
```

Vous retrouverez bien nos deux conteneurs dans la clé *Containers*, vous retrouverez d'ailleurs les noms des conteneurs :

![](/00-assets/images/Docker/image-7_37_1.png)

Nous pouvons enfin vérifier, dans le terminal où *alpine2* n'est pas en cours d'exécution qu'*alpine1* peut également communiquer avec *alpine2* :

```sh
docker exec -it alpine1 ping alpine2
```

#### Supprimer un ou plusieurs réseaux

Pour supprimer un réseau, sur lequel aucun conteneur en cours d'exécution n'est connecté, il suffit de faire :

```sh
docker network rm
```

Par exemple, pour supprimer le réseau que nous venons de créer il faut faire :

```sh
docker container stop alpine1 alpine2
docker network rm mynet
```

Vous pouvez supprimer tous les réseaux d'un coup :

```sh
docker network prune
```

### Connecter un serveur Node.js et une bdd MongoDB

#### Objectif

Notre objectif va être de connecter deux conteneurs : un premier conteneur utilisera l'image officiel *MongoDB* avec un volume nommé, comme nous l'avons appris, un second conteneur utilisera notre image de serveur *Node.js* que nous allons modifier.

L'application sera très triviale. Elle va simplement enregistrer en base de données la valeur d'un compteur. Ce compteur sera incrémenté à chaque fois que l'utilisateur fait une requête *HTTP GET* sur le port 80 (en se rendant sur *localhost* dans un navigateur).

Voici le schéma de ce que nous voulons faire :

![](/00-assets/images/Docker/image-7_38_1.png)

#### Création du conteneur avec la base de données

Avant de créer le conteneur, nous devons créer le volume nommé pour notre base de données :

```sh
docker volume create mydb
```

Ensuite, nous pouvons lancer notre conteneur en mode détaché.

Nous n'oublions pas de lui donner un nom (pour que nous puissions le connecter plus tard au réseau que nous créerons, et pour pouvoir utiliser la résolution *DNS* de *Docker*).

Nous faisons un *mount* de type *volume* car il s'agit d'une base de données :

```sh
docker run --name db --mount src=mybd,target=/data/db -d mongo
```

Nous pouvons vérifier que le conteneur est bien en cours d'exécution :

```sh
docker container ls
```

#### Création de notre réseau *bridge* personnalisé

Nous commençons par créer notre réseau :

```sh
docker network create mynet
```

Nous connectons ensuite notre base de données à celle-ci :

```sh
docker network connect mynet db
```

Petit bonus, déconnectons notre *db* du réseau *bridge* par défaut :

```sh
docker network disconnect bridge db
```

Inspectons ensuite les réseaux :

```sh
docker network inspect mynet
docker network inspect bridge
```

Vous devez avoir maintenant 0 conteneur sur le réseau *bridge* par défaut et 1 conteneur connecté sur le réseau *mynet*.

#### Initialisation compteur dans la base de données

Connectons nous maintenant à la base de données avec le client *MongoDB* :

```sh
docker exec -it db mongosh
```

Utilisons la base de données *test* :

```sh
use test
```

Insérons un document dans une collection *count* que nous créons :

```sql
db.count.insertOne({count: 0})
```

Vérifions que le document est bien présent :

```sql
db.count.findOne()
```

Quittons :

```sh
exit
```

#### Serveur *Node.js*

Nous allons modifier brièvement notre image pour pouvoir connecter notre application *Node.js* à notre base de données *MongoDB*.

Pour ce faire, nous allons simplement installer le *driver* officiel de *MongoDB*.

Modifiez le fichier *package.json* pour ajouter la ligne : `"mongodb": "^6.1.0"`.

Vous aurez donc :

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^6.1.0",
    "nodemon": "^3.0.1"
  }
}
```

Reconstruisez l'image :

```sh
docker build -t node-server .
```

Lancez enfin le conteneur à partir de notre image en lui donnant un nom, en créant un *bind mount* pour pouvoir obtenir le *live reload* de *nodemon*, en publiant bien le port 80 et le connectant au réseau que nous avons créé :

```sh
docker container run --name server --mount type=bind,src="$(pwd)"/src,target=/app/src -p 80:80 --network mynet node-server
```

Il faut que vous soyez dans le dossier où se trouve l'image car sinon le chemin passé en *src* pour le *bind mount* ne sera pas le bon.

Vous pouvez vérifier que le port *80* est bien publié en faisant :

```sh
docker container port server
```

Vous aurez comme retour :

```sh
80/tcp -> 0.0.0.0:80
```

### Configuration du serveur

#### Modification du code du serveur

Modifiez le fichier *src/app.js* :

```js
const express = require('express');
const { MongoClient } = require('mongodb');
const app = express();
let count;

const client = new MongoClient('mongodb://db');
async function run() {
  try {
    await client.connect();
    await client.db('admin').command({ ping: 1 });
    console.log('CONNEXION DB OK !');
    count = client.db('test').collection('count');
  } catch (err) {
    console.log(err.stack);
  }
}
run().catch(console.dir);

app.get('/', (req, res) => {
  count
    .findOneAndUpdate({}, { $inc: { count: 1 } }, { returnNewDocument: true, upsert: true })
    .then((doc) => {
      res.status(200).json(doc ? doc.count : 0);
    });
});

app.listen(80);
```

Le code précédent permet simplement de connecter le client *MongoDB* lors du lancement du serveur *Node.js* qui utilise le *framework Express*.

Sur les requêtes *HTTP GET* sur la route `/` il incrémente le compteur dans la base de données et retourne le résultat au format *JSON* pour que le navigateur puisse l'afficher.

Vous pouvez maintenant sauvegarder et vous rendre sur *localhost* dans un navigateur et rafraîchir pour voir le compteur s'incrémenter. 