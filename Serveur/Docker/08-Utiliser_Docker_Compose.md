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
