# Introduction

### Prérequis

Pour faire cette formation il faut impérativement maîtriser HTML, CSS et JavaScript.

## Qu'est-ce que React ?

React est une librairie (également appelée bibliothèque) *JavaScript* permettant de créer des interfaces utilisateurs pour des applications.

React a été créé par Jordan Walke, un ingénieur de Facebook en 2011 et a été utilisé dans le fil d’actualité dès 2011 puis dans Instagram dès 2012.

Il permet de créer de très grandes applications qui peuvent charger des données sans avoir à recharger la page.

### Le framework le plus utilisé et apprécié

Il s’agit aujourd’hui du *framework JavaScript* le plus utilisé au monde.

*React* est utilisé par Netflix, Yahoo, Airbnb, Sony, Atlassian, Facebook, Instagram, WhatsApp, Microsoft, Paypal et tant d’autres entreprises.

Sur *NPM* il s’agit du *framework front* le plus téléchargé mensuellement devant Angular et Vue :

Selon un sondage de Stackoverflow réalisé en 2022, il s’agit du framework Web que les développeurs souhaitent le plus apprendre :

### Avantages de React

#### Des performances incroyables

React utilise un *DOM virtuel* (que nous étudierons en détails) pour obtenir des performances très supérieures à du *JavaScript* natif.

En effet, au lieu de modifier directement le *DOM*, qui est l’arbre des éléments HTML, ce qui est coûteux en performance, React va calculer à l’aide d’un *DOM virtuel* tous les changements optimaux à effectuer sur le *DOM* avant de les effectuer.

Cette stratégie permet des gains de performance très importants.

#### Utilisation de JSX et philosophie de la librairie

Avec React au lieu d’utiliser du *HTML*, nous utilisons du *JSX*.

Ce n’est pas obligatoire mais fortement recommandé.

Le *JSX* permet d’écrire du code *HTML* dans du code *JavaScript*.

Ce code sera ensuite transpilé en *JavaScript*.

Nous verrons tous les avantages d’utiliser ce *JavaScript* amélioré, en même temps que nous l’apprendrons, dans le chapitre suivant.

React a pour philosophie de ne pas séparer la structure *HTML*, le style *CSS* et la logique *JavaScript*, mais de séparer l’application en composants qui sont complets et construits autour d’une fonctionnalité.

Cette approche permet de créer des applications qui soient plus facilement maintenables car elles sont mieux structurées. 

Elle permet également une réutilisation des composants dans plusieurs endroits de votre application.

#### Une communauté énorme

La communauté de développeurs React est la plus importante de tous les frameworks front.

Il vous sera facile de trouver des réponses sur Stackoverflow si vous bloquez ! Mais nous sommes là pour vous aider et le cours répondra à toutes vos questions !

 #### Un écosystème fabuleux

React possède un écosystème incroyable, il existe en effet plusieurs milliers de librairies qui sont construites pour être utilisées spécifiquement avec React.

Dans ce cours nous étudierons les plus importantes qui vous permettront de réaliser des applications très complexes avec notamment *React Router* et *Recoil*.

Pour tout ce que vous voudrez faire, vous pouvez être certain d’une chose : il y aura une librairie de l'écosystème React pour vous y aider !

#### Le mobile avec React Native 

Grâce à *React Native* vous pouvez facilement porter votre application React sur mobile.

*React Native* permet de développer des composants comme sur React pour une application mobile hybride (disponible sur iOS et Android).

 #### Les Single Page Applications

React est un *framework* permettant de construire des *Single Page Applications (SPA)*. 

Vous connaissez forcément les *SPA* : Gmail, Google Analytics, Trello, Dropbox en sont des exemples parmi tant d’autres.

De nombreux *frameworks front* ont adopté cette architecture (Vue, Angular, React, Ember, Meteor etc). Mais de quoi s’agit-il exactement ?

**Une *SPA* est une application qui fonctionne dans un navigateur sans que l’utilisateur n’ait besoin de recharger la page**.

Le principe est de simuler une application hors ligne : pas de rechargement des pages, de la rapidité, pas d’attente supplémentaire due au réseau etc. 

Les principales caractéristiques de la *SPA* sont :

- **le rendu est effectué côté client** (quand un élément change, la page est modifiée grâce aux scripts de l’application chargée côté client).
- pour fonctionner **elle charge en principe une seule fois l’application** (*HTML*, *CSS* et *scripts*).
- **seules les données sont transmises**, si nécessaire, entre le serveur et l’application client (le plus souvent au format *JSON*).
- le développement mobile est simplifié car **le code backend peut être utilisé que l’application soit Web ou native** (iOS, Android).
- elle est particulièrement adaptée pour stocker les données localement et de n’envoyer des requêtes au serveur lorsque c’est nécessaire.

![spa.png](/00-assets/images/React/spa.png)

## vite, swc et JSX

### Installation de Node.js

Dans le cours vous aurez besoin d’installer des *packages*, c’est-à-dire des librairies qui deviendront des dépendances de votre projet *React*. Il vous faut donc le gestionnaire de dépendances pour les environnements *JavaScript* : *npm*.

*npm* est conçu pour simplifier l’installation, la mise à jour et la désinstallation des librairies utilisées dans votre application. Il est inclus dans *Node.js*.

Installez la version stable (*LTS*) de *Node.js* en vous rendant [ici](https://nodejs.org/en/download/).  

En plus de la gestion des dépendances, *Node.js* vous sera indispensable pour le serveur de développement de *React*. 

Ouvrez un terminal et vérifiez que l'installation est bien effectuée :

```bash
node -v
```
 
### vite

#### Qu'est-ce que Vite ?

*Vite* (qui vient du mot français) est un outil de *build* qui vise à fournir une expérience de développement plus rapide et plus épurée pour les projets web modernes. Il se compose de deux parties principales :

- Un serveur de développement qui offre des améliorations riches en fonctionnalités par rapport aux modules *ES* natifs, par exemple un remplacement de module à chaud (*HMR*) extrêmement rapide.
- Une commande de *build* qui *bundle* votre code avec *Rollup*, pré-configuré pour générer des ressources statiques hautement optimisées pour la production.

*Vite* est un outil aux opinions tranchées et vient avec des paramètres par défaut judicieux dès le départ.

#### Le problème avant Vite

Avant la disponibilité des modules ES natifs dans les navigateurs, les développeurs n'avaient pas de mécanisme natif pour écrire du *JavaScript* de manière modulaire, ce qui a conduit à **la pratique du "*bundling*" : l'utilisation d'outils pour traiter et concaténer les modules source en fichiers exécutables dans le navigateur**.

Des outils tels que *webpack*, *Rollup* et *Parcel* ont amélioré l'expérience de développement, mais avec l'augmentation du volume de *JavaScript* dans les applications ambitieuses, les performances des outils basés sur *JavaScript* ont commencé à poser problème.

Le temps de démarrage du serveur de développement et le temps de réflexion des modifications de fichiers dans le navigateur sont devenus trop longs, affectant la productivité et le moral des développeurs.

Pour résoudre ces problèmes, *Vite* tire parti des avancées récentes, notamment la disponibilité des modules *ES* natifs dans tous les navigateurs et l'émergence d'outils *JavaScript* écrits dans des langages compilés en natif. 

#### Performances excellentes en développement

Contrairement aux configurations de *build* basées sur des *bundlers* qui doivent traiter toute l'application avant de pouvoir la servir, *Vite* améliore le temps de démarrage du serveur de développement en divisant les modules en deux catégories : les dépendances et le code source.

![Vite avec ES Modules](/00-assets/images/React/vite-bundle-new.svg)

Les dépendances, qui changent rarement, sont pré-bundlées rapidement avec *esbuild* un outil écrit en *Go* ou *swc* un outil écrit en *Rust*. Le code source, qui contient souvent du *JavaScript* non standard (par exemples du *TypeScript*) nécessitant une transformation et est fréquemment modifié, est servi via des modules *ES* natifs, permettant au navigateur de demander et de traiter le code à la demande.

Dans les configurations basées sur des *bundlers*, éditer un fichier peut nécessiter de reconstruire et de recharger l'ensemble du *bundle*, ce qui est inefficace, surtout pour les grandes applications. *Vite*, en revanche, effectue des mises à jour *HMR* (*Hot Module Replacement*) sur des modules *ES* natifs, invalidant et mettant à jour uniquement le nécessaire, ce qui accélère considérablement le processus.

![Vite et bundle à l'ancienne](/00-assets/images/React/vite-bundle-old.svg)

#### Le bundling reste nécessaire pour la production

Il reste nécessaire de *bundler* l'application pour la production en raison de l'inefficacité de servir des modules *ES* non *bundlés*. En effet, dans ce cas de très nombreux allers-retours réseau sont nécessaires.

*Vite* propose une commande de *build* préconfigurée pour gérer cela efficacement.

>*Vite* n'utilise pas *esbuild* pour le *bundling* en production, malgré sa vitesse, en raison de l'incompatibilité avec l'*API* de *plugin* de *Vite* et de la flexibilité offerte par *Rollup*, l'outil actuellement utilisé par *Vite* pour le *bundling* en production.

 
### Le langage JSX

#### React recommande l’utilisation de JSX. 

*JSX* est une extension syntaxique à *JavaScript* qui permet d’utiliser du *HTML* amélioré dans du *JavaScript*.

#### Quels sont les avantages à utiliser JSX ?

1. Premièrement, cela fait beaucoup de code en moins.
1. Deuxièmement, il s’agit de la philosophie même de *React*. *React* prône en effet la séparation des préoccupations (*separation of concerns*) plutôt que la séparation des langages.

Autrement dit dans la logique *React*, il ne faut pas séparer le *template HTML* de la logique *JavaScript* car les deux participent au même objectif : l’affichage d’une UI à l’utilisateur.

En effet, la logique du rendu est trop couplée avec toutes les logiques côté *JavaScript* : comment les événements sont gérés, comment le rendu change au cours du temps et comment les données sont préparées pour être affichées.

 
### Aller plus loin sur les différents outils et le bundling de production

#### Rollup

*Vite* utilise *Rollup* pour le processus de *bundling* lors de la mise en production.

*Rollup* est un outil de module *bundler* qui permet de regrouper différents fichiers *JavaScript* en un seul *bundle*, optimisant ainsi le chargement des *scripts* dans le navigateur.

*Rollup* prend en charge la création du *bundle* final, la découpe du code (*code-splitting*), et l'inclusion des *assets* nécessaires. Il s'occupe également de l'optimisation des bundles pour une performance optimale.

#### esbuild et swc

Ces deux outils sont alternatifs : soit on utilise *esbuild* soit *swc*. Nous utiliserons *swc* dans la formation.

Ces outils sont utilisés par *Vite* pour la pré-bundling des dépendances et la minification du code.

Ils sont utilisés pour transformer et minifier rapidement le code *JavaScript* et *CSS*. La minification consiste à réduire la taille des fichiers en supprimant les espaces inutiles, en renommant les variables pour réduire leur taille, etc.

Ils permettent également de transpiler le *Sass* en *CSS* ou le *TypeScript* en *JavaScript*.

#### Étapes de Vite pour la mise en production

- **Pré-bundling des dépendances :** avant de démarrer le serveur de développement, *Vite* pré-bundle les dépendances du projet avec *esbuild* ou *swc* pour accélérer les temps de chargement initiaux.
- **Transpilation :** *Vite* transpile le code source en code compatible avec les navigateurs cibles. Cela peut inclure la transpilation de *JSX*, *TypeScript*, ou d'autres syntaxes modernes.
- **Bundling :** *Vite* utilise *Rollup* pour créer un ou plusieurs *bundles* à partir du code source et des dépendances. Cela inclut également le découpage du code pour le chargement différé des modules.
- **Optimisations :** *Vite* applique diverses optimisations pour réduire la taille des fichiers et améliorer les performances, y compris la minification (avec *esbuild* ou un autre outil), l'élimination du code mort (*tree-shaking*), et la séparation des fichiers communs (*chunk splitting*).
- **Génération des assets :** *Vite* génère également les fichiers *CSS*, les images optimisées, les polices et d'autres *assets* nécessaires pour l'application.

## Environnement Windows

### Installation de Node.js

Comme nous l'avons vu, nous devons installer des dépendances *JavaScript*, appelées *packages*, nous vous conseillons donc d’installer *npm* qui est le gestionnaire de dépendances de *Node.js*.

Si vous n’avez pas *npm*, installez *Node.js* en vous rendant [ici](https://nodejs.org/en/download/).

 
### Installation de Visual Studio Code

Pour installer l’éditeur *VS Code*, il suffit de le télécharger et de l’installer. Il fonctionne sur tous les environnements : *Linux*, *Windows* et *MacOS*.

Vous pouvez le télécharger [ici](https://code.visualstudio.com/).

### Ouvrir un terminal dans VS Code

**Pour ouvrir le terminal sur *VS Code* vous pouvez au choix :**

- Utiliser `Ctrl+ ù`. Si le raccourci ne fonctionne pas. Allez dans le menu `File > Preferences > Keymaps`. Ensuite recherchez *view toggle terminal*. Et choisissez un raccourci puis faites `Entrée`. Nous recommandons `Ctrl` et la touche la plus à gauche juste sous la touche d'échappement. 
- Aller dans le menu `Affichage > Terminal`.
- Aller dans le menu `Terminal > Nouveau terminal`.
- Tapez `Ctrl+Shift+P` puis tapez `Toggle terminal` puis `entrée`.

### Installation d’extensions VS Code

Allez dans l’onglet extension sur *VS Code*.

Recherchez et installez *Prettier*, *ESlint* et *Material Icon Theme*.

Allez ensuite dans `Fichier > Préférences > Paramètres`. Recherchez *format on save* et cochez la case.

Cela permet d'activer le formatage de *Prettier* à chaque sauvegarde.

## Environnement GNU/Linux

### Installation de Node.js avec Node Version Manager :

*nvm* permet d’installer et d’utiliser plusieurs versions de *Node.js* localement. C’est intéressant si vous avez plusieurs projets à des versions différentes.

Installez nvm :

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

Collez les commandes suivantes :

```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"

[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```
 
### Installez Node.js LTS :

```bash
nvm install --lts
```
 
### Installation de Node sans nvm

Ouvrez un terminal et entrez la commande suivante qui va installer la version *LTS* de *Node.js* :

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

sudo apt-get install -y nodejs
```

Vérifiez l’installation :

```bash
node -v
```
 
### Installation de VS Code avec snap sur Ubuntu

Pour installer l’éditeur *VS Code* sur *Ubuntu*, nous vous recommandons d’utiliser snap.

Ouvrez un terminal et faites simplement :

```bash
snap install code
```
 
### Installation de VS Code avec l’installeur

Si vous êtes sur une autre distribution ou que vous ne souhaitez pas utiliser snap, vous pouvez également utiliser l’installeur.

Vous pouvez le télécharger [ici](https://code.visualstudio.com/).

### Ouvrir un terminal dans VS Code

**Pour ouvrir le terminal sur *VS Code* vous pouvez au choix :**

- Utiliser `Ctrl+ ù`. Si le raccourci ne fonctionne pas. Allez dans le menu `File > Preferences > Keymaps`. Ensuite recherchez *view toggle terminal*. Et choisissez un raccourci puis faites `Entrée`. Nous recommandons `Ctrl` et la touche la plus à gauche juste sous la touche d'échappement. 
- Aller dans le menu `Affichage > Terminal`.
- Aller dans le menu `Terminal > Nouveau terminal`.
- Tapez `Ctrl+Shift+P` puis tapez `Toggle terminal` puis `entrée`.

### Installation d’extensions VS Code

Allez dans l’onglet extension sur *VS Code*.

Recherchez et installez *Prettier*, *ESlint* et *Material Icon Theme*.

Allez ensuite dans `Fichier > Préférences > Paramètres`. Recherchez *format on save* et cochez la case.

Cela permet d'activer le formatage de *Prettier* à chaque sauvegarde.

## Environnement MacOS

### Installation de Node.js

Comme nous l'avons vu, nous devons installer des dépendances *JavaScript*, appelées *packages*, nous vous conseillons donc d’installer *npm* qui est le gestionnaire de dépendances de *Node.js*.

Si vous n’avez pas *npm*, installez *Node.js* en vous rendant [ici](https://nodejs.org/en/download/).

 
### Installation de Visual Studio Code

Pour installer l’éditeur *VS Code*, il suffit de le télécharger et de l’installer. Il fonctionne sur tous les environnements : *Linux*, *Windows* et *MacOS*.

Vous pouvez le télécharger [ici](https://code.visualstudio.com/).

### Ouvrir un terminal dans VS Code

**Pour ouvrir le terminal sur *VS Code* vous pouvez au choix :**

- Utiliser `Ctrl+ ù`. Si le raccourci ne fonctionne pas. Allez dans le menu `File > Preferences > Keymaps`. Ensuite recherchez *view toggle terminal*. Et choisissez un raccourci puis faites `Entrée`. Nous recommandons `Ctrl` et la touche la plus à gauche juste sous la touche d'échappement. 
- Aller dans le menu `Affichage > Terminal`.
- Aller dans le menu `Terminal > Nouveau terminal`.
- Tapez `Ctrl+Shift+P` puis tapez `Toggle terminal` puis `entrée`.

### Installation d’extensions VS Code

Allez dans l’onglet extension sur *VS Code*.

Recherchez et installez *Prettier*, *ESlint* et *Material Icon Theme*.

Allez ensuite dans `Fichier > Préférences > Paramètres`. Recherchez *format on save* et cochez la case.

Cela permet d'activer le formatage de *Prettier* à chaque sauvegarde.