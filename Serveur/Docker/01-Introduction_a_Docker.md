## Introduction à Docker

## A quoi sert Docker ?

### _Docker_ en deux mots

_Docker_ est un outil open-source écrit en langage _Go_ et créé en 2012 par trois ingénieurs français. Il résulte de développements internes de l'entreprise _dotCloud_ co-fondés par deux français d'Epitech.

Il s'agit aujourd'hui d'une entreprise américaine qui a été scindée en deux : une partie open-source et une partie entreprise qui a été revendue à _Mirantis_.

Nous allons dans ce chapitre uniquement nous intéresser à _Docker Community Edition_, la partie entreprise étant réservée aux très très grosses entreprises.

Une bonne première définition de _Docker_ est **un outil qui peut empaqueter une application et ses dépendances dans un conteneur isolé, qui pourra être exécuté sur n'importe quel environnement.**

Appréhender _Docker_ n'est pas une chose aisée. Aussi il est normal d'être perdu au début et il faut avoir bien avancé pour comprendre l'utilité et la puissance de cet outil !

Nous allons quand même lister les principaux avantages dès maintenant.

### Les principaux avantages à utiliser _Docker_

#### 1 - Utilisation plus efficiente des ressources système

_Docker_ permet une utilisation plus efficiente des ressources d'un système tout en permettant les mêmes avantages qu'une machine virtuelle : c'est-à-dire principalement l'isolation et la reproductibilité.

Les applications conteneurisées utilisent beaucoup moins de ressources que des machines virtuelles car elles utilisent le même noyau _Linux_.

Elles démarrent et s'arrêtent en quelques millisecondes, alors qu'il faut quelques secondes ou minutes pour une machine virtuelle.

#### 2 - Reproductibilité

Un conteneur _Docker_ est garanti d'être identique quel que soit le système.

Il garantit que la bonne version de chaque dépendance soit installée. Ainsi, chaque membre d'une équipe est certain d'avoir le ou les applications qui fonctionnent identiquement sur des environnements différents (pas les mêmes _OS_, pas les même configurations locales, pas les mêmes dépendances globales etc).

Avec _Docker_, plus jamais vous entendrez le : "Moi, cela fonctionne sur ma machine".

Cela permet aux développeurs de se soucier uniquement du code et pas de l'environnement où celui-ci sera finalement exécuté.

#### 3 - Isolation

Les dépendances ou les configurations d'un conteneur n'affectera aucune dépendance ou configuration des autres conteneurs ou de la machine hôte.

Vous pouvez ainsi avoir 5 versions de _Node.js_ ou de _MongoDB_ différentes localement sans aucune peine ! C'est un énorme plus lorsque l'on travaille sur plusieurs projets avec des versions de dépendances qui ne sont pas les mêmes. En effet, elles présentent souvent des incompatibilités ou des _breaking changes_ se qui se révèlent être un véritable casse-tête !

#### 4 - Mises à jour et tests

Pour ceux qui ont à gérer plusieurs serveurs, vous savez combien il est complexe de mettre à jour correctement des environnements complexes.

Il faut bien sûr mettre à jour l'_OS_ sur chaque serveur, puis les environnements serveurs et les bases de données : le tout dans le bon ordre et sans oublier de serveurs.

Avec _Docker_, la mise à jour d'un environnement dans le _cloud_, même sur de nombreux serveurs est très simple. Vous gagnerez un temps précieux.

Même chose si vous devez gérer plusieurs environnement : par exemple le classique _development / tests_, _staging_ et _production_. Il faut être sûr que tous les environnements aient les mêmes dépendances, les mêmes versions d'_OS_ etc pour que les tests soient fiables. Avec _Docker_ toute cette gestion d'environnements est simplifiée !

#### 5 - Bonnes pratiques grâce aux images open source

Comme nous le verrons, _Docker_ est si populaire que tous les mainteneurs de librairies ou de logiciels divers (environnement serveur, serveur _Web_, bases de données etc) maintiennent des images _Docker_.

Ces images _Docker_ officielles, disponibles sur une plateforme que nous étudierons, _Docker Hub_, permettent de lancer des conteneurs fiables très rapidement et sans avoir à les configurer soit même.

### Les grands principes des conteneurs Docker

_Docker_ est une plateforme pour les développeurs et les _devops_ qui permet de _build_, de lancer et de partager des applications en utilisant des conteneurs.

Les conteneurs, comme nous allons l'apprendre, permettent d'isoler les applications pour **assurer une parfaite réplicabilité des environnement.**

Autrement dit, ils permettent de s'assurer que vous exécuter une ou plusieurs applications sur le même environnement quel que soit la machine physique et l'_OS_ utilisé.

**Les principes des _conteneurs Docker_ sont les suivants :**

- **ils sont flexibles :** n'importe quelle applications, même les plus complexes, peuvent être containerisée (on dit aussi "dockerisée").
- **ils sont légers :** le fait qu'ils partagent les ressources et le noyau _Linux_ du système d'exploitation de la machine hôte font qu'ils sont beaucoup plus légers que des machines virtuelles (nous l'étudierons en détails dans le chapitre).
- **ils sont portables :** ils sont utilisables sur n'importe quel environnement (tout _OS_, aussi bien pour le développement local que sur des centaines de serveurs dans le _cloud_).
- **ils ont un couplage faible :** cela signifie que les conteneurs sont autonomes car ils sont isolés. Il est facile de les supprimer, les ajouter, les modifier (mise à jour par exemple) sans perturber les autres conteneurs tournant sur une machine.
- **ils sont scalables :** des outils permettent de déployer des centaines voir des milliers de conteneurs en quelques minutes voir secondes sur un très grande nombre de serveurs. Il est même possible d'automatiser le déploiement de nouveaux conteneurs (_auto-scaling_).
- **ils sont sécurisés :** les contraintes et l'isolation par défaut des conteneurs apporte une grande sécurité des environnements déployés.

## Comment fonctionne Docker

### Qu'est-ce qu'une machine virtuelle ? ( _Virtual Machine_ ou _VM_)

Une machine virtuelle est une machine créée par un logiciel et qui n'a pas donc d'existence physique.

Elle permet de simuler des ressources matérielles (disque dur, _RAM_, _CPU_ etc) et logicielles afin qu'un ou plusieurs autres systèmes d'exploitation puissent être utiliser sur la même machine physique.

L'avantage d'une machine virtuelle est la possibilité de lancer plusieurs environnements isolés, avec des systèmes d'exploitation pouvant être différents, en même temps, sur la même machine.

Cela est particulièrement utile pour des serveurs, souvent très puissants, qui peuvent ainsi lancer des dizaines voir des centaines d'environnements isolés sur les mêmes machines physiques (c'est l'une des technologie du _Cloud_).

C'est aussi utile pour pouvoir utiliser différents systèmes d'exploitation sans avoir à redémarrer : c'est parfait pour le développement par exemple. Vous pouvez avoir une machine virtuelle avec tous les logiciels que vous souhaitez sur _GNU/Linux_ et la lancer depuis _Windows_ en deux clics par exemple.

L'inconvénient est bien sûr une diminution des performances, car il faut diviser l'allocation des ressources physiques entre machine hôte et machine virtuelle, et il y a à exécuter la couche système hôte, l'hyperviseur et la couche système invité comme nous allons le voir.

![](/00-assets/images/Docker/image-1_03_1.png)

### Différences avec _Docker_

**Il faut bien comprendre que _Docker_ utilise le noyau _Linux_ et l'environnement _GNU_ de votre système.**

Si vous êtes sur n'importe quelle distribution de *GNU/Linux*, il pourra utiliser directement cet environnement.

Si vous êtes sur *MacOS* ou *Windows*, il utilisera l'hyperviseur installé sur le système (de type 2) et lancera une machine virtuelle *GNU/Linux* qu'il utilisera.

Dans tous les cas, *Docker* utilise **un seul système d'exploitation *GNU/Linux*.**

Sur ce système, il va lancer un **conteneur** qui est un processus (c'est-à-dire un programme en cours d'exécution, *c.f. cours Linux / Bash*), avec des fonctionnalités d'encapsulation lui permettant d'être isolé de l'hôte et des autres conteneurs.

Chaque conteneur a son propre système de fichiers isolé qui est fourni par une image *Docker*. Ce système de fichiers contient le code, les binaires, les fichier exécutables et toutes les dépendances requises pour faire fonctionner une application.

Nous reviendrons sur tous ces aspects en détails, mais cela permet d'avoir une idée du fonctionnement par rapport à une machine virtuelle.

![](/00-assets/images/Docker/image-1_03_2.png)

Autrement dit, *Docker* partage le système d'exploitation entre tous les conteneurs qu'il lance, qui sont des simples processus légers. Alors qu'une machine virtuelle est un système d'exploitation entier nécessitant plusieurs gigaoctets de mémoire vive, beaucoup d'espace disque et une utilisation importante du *CPU*.

### Technologies utilisées par Docker

*Docker* est un environnement beaucoup plus facile d'accès et déjà paramétré de technologies existantes. En fait, il s'agit d'une surcouche et d'une *API* permettant de contrôler des fonctionnalités bas niveau du noyau *Linux*.

*Docker* est écrit en langage *Go* et utilise toutes les fonctionnalités *Linux* que nous allons voir.

#### Les namespaces

*Docker* utilise les *namespaces Linux* qui permettent de créer des espaces isolés sur un système d'exploitation *Linux*.

C'est cette fonctionnalité de *Linux* qui confère aux conteneurs leur isolation. Chaque aspect d'un containeur est exécuté dans un *namespace* qui lui est propre et auquel seul ce dernier peut accéder.

*Docker* utilise ainsi des *namespaces* pour :
- **l'isolation des processus du conteneur (*PID namespace*)**
- **l'isolation des interfaces réseaux du conteneur (*NET namespace*)**
- **l'isolation des ressources de communications inter-processus (*IPC namespace*)**
- **l'isolation du système de fichiers (*MNP namespace*)**
- **l'isolation des identifiants du noyau et des versions (*UTS namespace*)** qui permet notamment d'avoir un *hostname* différent pour les conteneurs comme nous le verrons.

#### Les Cgroups

*Docker* utilise également les *control groups* (*cgroups*).

Cette technologie *Linux* permet de limiter l'accès aux ressources à des processus. Grâce à cela, *Docker* partage les ressources système disponibles entre tous les conteneurs (*RAM*, *CPU*, accès réseaux et lecture / écriture disques etc).

Il est également possible, grâce à cette technologie, de limiter les ressources allouées à chaque conteneur comme nous le verrons plus tard.

![](/00-assets/images/Docker/image-1_03_3.png)

#### *UnionFS*

*UnionFS* (pour *Union File System*) est un type de système de fichiers pour Linux.

Suivant la distribution *Linux*, *Docker* utilise l'une des implémentations disponibles : *AUFS*, *btrfs*, *vfs* ou *DeviceMapper*.

Ces systèmes de fichiers fonctionnent en créant des couches (*layers*). Ils sont au coeur des images *Docker* et leur permettent d'être légères et très rapides.

Nous étudierons en détails le fonctionnement de ces systèmes de fichiers lorsque nous verrons les images.

