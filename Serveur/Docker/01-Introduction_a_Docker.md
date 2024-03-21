## Introduction à Docker

## A quoi sert Docker ?

### *Docker* en deux mots

*Docker* est un outil open-source écrit en langage *Go* et créé en 2012 par trois ingénieurs français. Il résulte de développements internes de l'entreprise *dotCloud* co-fondés par deux français d'Epitech.

Il s'agit aujourd'hui d'une entreprise américaine qui a été scindée en deux : une partie open-source et une partie entreprise qui a été revendue à *Mirantis*.

Nous allons dans ce chapitre uniquement nous intéresser à *Docker Community Edition*, la partie entreprise étant réservée aux très très grosses entreprises.

Une bonne première définition de *Docker* est **un outil qui peut empaqueter une application et ses dépendances dans un conteneur isolé, qui pourra être exécuté sur n'importe quel environnement.**

Appréhender *Docker* n'est pas une chose aisée. Aussi il est normal d'être perdu au début et il faut avoir bien avancé pour comprendre l'utilité et la puissance de cet outil !

Nous allons quand même lister les principaux avantages dès maintenant.

### Les principaux avantages à utiliser *Docker*

#### 1 - Utilisation plus efficiente des ressources système

*Docker* permet une utilisation plus efficiente des ressources d'un système tout en permettant les mêmes avantages qu'une machine virtuelle : c'est-à-dire principalement l'isolation et la reproductibilité.

Les applications conteneurisées utilisent beaucoup moins de ressources que des machines virtuelles car elles utilisent le même noyau *Linux*.

Elles démarrent et s'arrêtent en quelques millisecondes, alors qu'il faut quelques secondes ou minutes pour une machine virtuelle.

#### 2 - Reproductibilité

Un conteneur *Docker* est garanti d'être identique quel que soit le système.

Il garantit que la bonne version de chaque dépendance soit installée. Ainsi, chaque membre d'une équipe est certain d'avoir le ou les applications qui fonctionnent identiquement sur des environnements différents (pas les mêmes *OS*, pas les même configurations locales, pas les mêmes dépendances globales etc).

Avec *Docker*, plus jamais vous entendrez le : "Moi, cela fonctionne sur ma machine".

Cela permet aux développeurs de se soucier uniquement du code et pas de l'environnement où celui-ci sera finalement exécuté.

#### 3 - Isolation

Les dépendances ou les configurations d'un conteneur n'affectera aucune dépendance ou configuration des autres conteneurs ou de la machine hôte.

Vous pouvez ainsi avoir 5 versions de *Node.js* ou de *MongoDB* différentes localement sans aucune peine ! C'est un énorme plus lorsque l'on travaille sur plusieurs projets avec des versions de dépendances qui ne sont pas les mêmes. En effet, elles présentent souvent des incompatibilités ou des *breaking changes* se qui se révèlent être un véritable casse-tête !

#### 4 - Mises à jour et tests

Pour ceux qui ont à gérer plusieurs serveurs, vous savez combien il est complexe de mettre à jour correctement des environnements complexes.

Il faut bien sûr mettre à jour l'*OS* sur chaque serveur, puis les environnements serveurs et les bases de données : le tout dans le bon ordre et sans oublier de serveurs.

Avec *Docker*, la mise à jour d'un environnement dans le *cloud*, même sur de nombreux serveurs est très simple. Vous gagnerez un temps précieux.

Même chose si vous devez gérer plusieurs environnement : par exemple le classique *development / tests*, *staging* et *production*. Il faut être sûr que tous les environnements aient les mêmes dépendances, les mêmes versions d'*OS* etc pour que les tests soient fiables. Avec *Docker* toute cette gestion d'environnements est simplifiée !

#### 5 - Bonnes pratiques grâce aux images open source

Comme nous le verrons, *Docker* est si populaire que tous les mainteneurs de librairies ou de logiciels divers (environnement serveur, serveur *Web*, bases de données etc) maintiennent des images *Docker*.

Ces images *Docker* officielles, disponibles sur une plateforme que nous étudierons, *Docker Hub*, permettent de lancer des conteneurs fiables très rapidement et sans avoir à les configurer soit même.

### Les grands principes des conteneurs Docker

*Docker* est une plateforme pour les développeurs et les *devops* qui permet de *build*, de lancer et de partager des applications en utilisant des conteneurs.

Les conteneurs, comme nous allons l'apprendre, permettent d'isoler les applications pour **assurer une parfaite réplicabilité des environnement.**

Autrement dit, ils permettent de s'assurer que vous exécuter une ou plusieurs applications sur le même environnement quel que soit la machine physique et l'*OS* utilisé.

**Les principes des *conteneurs Docker* sont les suivants :**

- **ils sont flexibles :** n'importe quelle applications, même les plus complexes, peuvent être containerisée (on dit aussi "dockerisée").
- **ils sont légers :** le fait qu'ils partagent les ressources et le noyau *Linux* du système d'exploitation de la machine hôte font qu'ils sont beaucoup plus légers que des machines virtuelles (nous l'étudierons en détails dans le chapitre).
- **ils sont portables :** ils sont utilisables sur n'importe quel environnement (tout *OS*, aussi bien pour le développement local que sur des centaines de serveurs dans le *cloud*).
- **ils ont un couplage faible :** cela signifie que les conteneurs sont autonomes car ils sont isolés. Il est facile de les supprimer, les ajouter, les modifier (mise à jour par exemple) sans perturber les autres conteneurs tournant sur une machine.
- **ils sont scalables :** des outils permettent de déployer des centaines voir des milliers de conteneurs en quelques minutes voir secondes sur un très grande nombre de serveurs. Il est même possible d'automatiser le déploiement de nouveaux conteneurs (*auto-scaling*).
- **ils sont sécurisés :** les contraintes et l'isolation par défaut des conteneurs apporte une grande sécurité des environnements déployés.

-------

