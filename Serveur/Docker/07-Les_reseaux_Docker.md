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

