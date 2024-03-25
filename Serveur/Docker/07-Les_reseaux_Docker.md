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