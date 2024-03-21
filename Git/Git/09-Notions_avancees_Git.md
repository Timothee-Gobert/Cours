## Notion avancées de Git

Introduction aux hooks
Qu'est-ce qu'un hook ?

Un hook en programmation permet d'exécuter du code à un moment précis lorsque certains événements surviennent.

Git permet l'utilisation de hooks pour exécuter des script avant et après quasiment toutes les actions possibles.

Par exemple, grâce au hooks vous pouvez exécuter un script avant ou après un commit.

Ces hooks sont stockés dans le dossier caché de Git et plus précisément dans .git/hooks.

Par défaut, Git place quelques scripts mais qui ne sont pas activés :

ls .git/hooks

Donne :

applypatch-msg.sample
fsmonitor-watchman.sample
pre-applypatch.sample
prepare-commit-msg.sample
pre-rebase.sample
update.sample
commit-msg.sample
post-update.sample
pre-commit.sample
pre-push.sample
pre-receive.sample

Les scripts sont par défaut des scripts shell exécutables donc par bash par exemple.

Vous pouvez cependant les écrire dans n'importe quel langage, par exemple en Node.js.

L'extension .sample permet de ne pas les lancer par défaut. Il faut retirer l'extension pour les activer.
Exemple de hook pre-commit

Un hook pre-commit s'exécute avant d'effectuer le commit, avant même l'ouverture de l'éditeur de texte pour le message de validation si vous n'utilisez pas l'option -m.

Si vous ne retournez pas null ou 0 dans le script exécuté dans ce hook, le commit sera annulé.

Voici un exemple de hook pre-commit que nous utilisons chez Dyma.

Son objectif est d'empêcher d'effectuer un commit contenant un console.log :

#!/bin/bash

exec < /dev/tty
if test $(git diff --cached | grep ^+.*console.log | wc -l) != 0
then
    read -p "Au moins un console.log est ajouté dans votre commit ! Êtes-vous certain de continuer ? [o/n]: " choix
    if [[ $choix == 'o' || $choix == 'O' ]]
    then
        exit 0;
    else
        exit 1;
    fi
fi

Il est utile pour ne pas se retrouver en production avec des console.log utilisée parfois en développement.

Vous pourrez retrouver notre cours dédié sur les shells mais voici quelques explications.

#!/bin/bash permet de dire que c'est un script shell devant être exécuté avec bash.

exec < /dev/tty permet d'activer la possibilité pour les utilisateurs de rentrer du texte.

test permet de tester une expression et de renvoyer 0 si elle est vraie et 1 si elle est fausse.

git diff --cached permet de lister les changements qui sont indexés par rapport au dernier commit.

| est un pipe : c'est-à-dire qu'il permet de rediriger la sortie de l'expression précédente en entrée de l'expression suivante.

grep permet de filtrer. Ici nous utilisons une expression régulière (^+.*console\.log) qui signifie : commence par un + (pour un ajout dans l'index) suivi de tout caractère un nombre indéfini de fois, suivi de console.log.

wc -l permet d'afficher le nombre de lignes d'une expression.

if test $(git diff --cached | grep ^+.*console\.log | wc -l) != 0 signifie donc : si il y a au moins une ligne qui ajoute un console.log et qui est indexée alors passe au then.

read -p permet d'afficher du texte et de permettre à l'utilisateur de rentrer du texte directement après celui-ci.

"Au moins un console.log est ajouté dans votre commit ! Êtes-vous certain de continuer ? [o/n]: " choix : permet d'afficher la phrase et de stocker ce que rentre l'utilisateur dans la variable choix.

if [[ $choix == 'o' || $choix == 'O' ]] : Les doubles crochets permettent l'utilisation de plusieurs conditions en bash. Ici, si la variable choix vaut o ou O nous retournons 0 sinon 1.

Si nous retournons 0, le commit est effectué, si nous retournons 1, le commit n'est pas fait.

Nous verrons dans la leçon suivante qu'il existe des librairies facilitant l'utilisation des hooks. 

Librairies pour les hooks

Plusieurs difficultés à utiliser les hooks comme nous l'avons vu dans la leçon précédente se présentent.

La première est qu'il est difficile d'utiliser des librairies effectuant des actions dans certains hooks.

La seconde est que le dossier hook est dans le dossier local caché .git, il n'est donc pas suivi par Git et n'est pas partagé avec votre équipe. Cela peut poser problème si vous utilisez des hooks pour forcer des bonnes pratiques sur votre dépôt.

Heureusement, il existe des librairies qui vont nous aider à manipuler les hooks Git !
La librairie husky

Si vous avez fait n'importe quel cours JavaScript sur Dyma vous savez ce qu'est npm.

Il vous suffit d'installer Node.js pour l'obtenir.

Dans un projet, il faut faire :

npm init -y

Initialisez le suivi Git :

git init

Pour créer le package.json et permettre l'installation de packages.

Pous pouvez ensuite faire :

npx husky-init && npm install

Cela installera la librairie husky comme dépendance de développement.

Husky permet d'utiliser tous les hooks Git très facilement.
Utiliser vos scripts avec Husky

Comment faire pour utiliser vos scripts avec Husky ?

Nous allons reprendre notre exemple de la leçon précédente en le modifisant pour qu'il soit utilisable avec sh.

Modifiez le fichier .husky/pre-commit créé par Husky et mettez :

#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

exec < /dev/tty
if test $(git diff --cached | grep ^+.*console.log | wc -l) != 0
then
    echo "Au moins un console.log est ajouté dans votre commit !"
    echo "Êtes-vous certain de continuer ?"
    read -p "[o/n]>" choix
    if test $choix != "o"
    then
        echo "Abandon du commit"
        exit 1
    fi
fi

Le script s'exécutera maintenant avant chaque commit !

A noter que ce n'est pas utilisable avec la création de commit dans VS Code car celui-ci n'utilise pas de shell et n'ouvre pas de terminal. Il ne peut donc pas demander d'input à l'utilisateur.

Une version compatible, qui ne demande pas d'input aux utilisateurs serait :

# Check
if test $(git diff --cached | grep ^+.*console.log | wc -l) != 0
then
    echo "Console.log détecté dans votre commit"
    exit 1
fi

Utiliser la librairie commitlint

La librairie commitlint est une librairie forçant l'utilisation de conventions pour la rédaction des messages de validation pour les commits.

Nous vous conseillons de conserver la convention par défaut qui est utilisée par de très nombreuses équipes et a été créée par l'équipe Angular de Google.

1 - Elle oblige à que les messages soient de la forme type: titre.

Les types possibles sont :

build
chore
ci
docs
feat
fix
perf
refactor
revert
style
test

build : build de l'application pour la mise en production.

chore : ne touche pas au code de production mais des configurations d'outiles : Webpack etc.

ci : relatif à l'intégration continu / déploiement continu.

docs : mise à jour de la documentation.

feat : nouvelle fonctionnalité (feature).

fix : correction de bug.

perf : optimisation des performances.

refactor : refactorisation, pas de nouvelle fonctionnalité.

revert : retour en arrière (avec git revert).

style : modifications uniquement de mise en page.

test : ajout / modification de tests.

2 - Elle oblige à mettre les message de validation en minuscules uniquement.

3 - Elle oblige a mettre un message de titre de 100 caractères maximum (texte passé à la première utilisation de l'option -m).

Pour installer la librairie, il suffit de faire :

npm install --save-dev @commitlint/{cli,config-conventional}

Ensuite ouvrez un terminal et faites :

echo "module.exports = { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js

Cela va créer le fichier de configuration pour la convention utilisée par commitlint.

Enfin installez le hook avec Husky en faisant :

npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'

Utiliser prettier dans un hook

Nous avons déjà vu dans tous les cours la librairie de mise en forme du code Prettier.

Vous pouvez utiliser un hook afin de reformater le code systématiquement avant un commit.

npm install --save-dev prettier pretty-quick

Enfin installez le hook avec Husky en faisant :

npx husky set .husky/pre-commit "npx pretty-quick --staged"

Vous avez maintenant une bonne base pour utiliser les hooks Git dans votre équipe ! 

Les alias Git
Les alias

Les alias sont utilisés partout en programmation. Ils permettent simplement de créer des raccourcis pour une ou plusieurs commandes.

Git possède également son système d'alias et vous pouvez donc écrire facilement tous les raccourcis que vous souhaitez pour vos commandes Git.

Créer un alias est très simple :

git config --global alias.raccourci commande

Exemples communs

Voici quelques exemples d'alias proposés par Git :

git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status

Voici un exemple pour désindexer en utilisant git unstage :

git config --global alias.unstage 'reset HEAD --'

Vous pouvez ainsi faire :

git unstage fichier

Un autre alias très utilisé, permet d'afficher le dernier commit :

git config --global alias.last 'log -1 HEAD'

Vous pouvez juste faire :

git last

Vous êtes totalement libre de créer vos propres alias, selon vos cas d'utilisation. 

Utilisation de git cherry-pick
La commande git cherry-pick

La commande git cherry-pick permet de sélectionner un commit par référence et de l'appliquer sur la branche pointée par HEAD.

Elle permet d'appliquer un commit d'une branche sur une autre branche.

Nous allons maintenant voir des cas d'utilisation de cette commande.

Nous verrons ensuite des cas où il vaut mieux utiliser une autre commande.
Cas d'utilisation de cherry-pick

Prenons un exemple de cas d'utilisation de la commande git cherry-pick.

Prenons la situation suivante :

Nous voulons récupérer les modifications du commit D sur master et ne pas conserver le commit E. Nous pourrions faire un rebasage interactif, mais un moyen plus rapide est :

git cherry-pick D

Où D est le hash du commit D.

Un nouveau commit contenant les changements du commit D, que nous appellerons D' est créé sur master.

Nous pouvons ensuite abandonner la branche :

git branch -D branche

Nous avons enfin :

git cherry-pick peut donc être utilisé lorsque vous voulez effectuer un rebasage d'un commit unique.
Autre cas d'utilisation de cherry-pick

Prenons la situation suivante :

Vous ne souhaitez pas immédiatement fusionner la branche dans master car la fonctionnalité est loin d'être terminée, mais vous avez besoin absolument de modifications apportées par le commit E.

Ce n'est pas idéal, mais pour vous en sortir, vous pouvez faire :

git cherry-pick E

E étant bien sûr le hash entier ou raccourci du commit E.

Ce qui vous donne :

Le commit E' contient toutes les modifications correspondant à la version du commit E mais il n'a pas le même hash car la date de commit est différente, un autre objet commit est donc créé.

A noter que vous pouvez cherry-pick plusieurs commits d'un coup :

git cherry-pick hash1 hash2


Git LFS
Qu'est-ce que Git LFS ?

Git LFS (Large File Storage) est une extension open source résolvant un problème essentiel : le traitement des fichiers volumineux.

Dans votre projet Web vous aurez sûrement des images, des vidéos etc et vous voudrez les suivre avec Git pour pouvoir revenir à une version antérieure : par exemple si vous voulez revoir comment était votre landing il y a quelques mois il faut que les images aient été suivis dans Git, sinon lorsque vous les remplacerez au fur et à mesure de l'avancement de votre projet par de nouvelles images, Git ne pourra pas les récupérer.

Mais il y a un problème : suivre des fichiers volumineux va rapidement faire grossir la taille de votre dépôt Git.

A terme si votre projet a utilisé plusieurs centaines d'images mais n'en a que quelques dizaines en production, votre dépôt Git fera plusieurs gigaoctets.

Heureusement Git LFS va vous permettre de résoudre définitivement cette problématique !

Cette extension va permettre deux choses :

Premièrement, de stocker les fichiers volumineux autre part que dans votre répertoire (sur des serveurs distants le plus souvent gérés par Github, Gitlab ou Bitbucket).

Deuxièmement, elle va ne télécharger les fichiers volumineux que lorsque vous allez checkout sur une version qui en a besoin.

Ainsi tous les fichiers ne seront pas récupérés lors d'un git clone ou lors d'un git fetch, votre dépôt sera donc rapidement téléchargeable et ne prendra pas trop de place sur votre disque.

En fait, Git LFS va remplacer les larges fichiers par des pointeurs vers le lieu de sauvegarde de ces fichiers : de cette manière Git ne stockera que des pointeurs faisant quelques octets et non pas les fichiers volumineux qui feront rapidement plusieurs gigaoctets.
Fonctionnement détaillé

Nous utilisons les images de Atlassian Bitbucket le principal contributeur de l'extension car elles sont claires.

Première étape, lorsque vous ajoutez un fichier qui correspond au filtre (voir détails plus bas), Git LFS va remplacer le contenu du fichier par un pointeur et va sauvegarder le contenu du fichier en cache.

Deuxième étape, lorsque vous faites git push, tous les fichiers qui sont référencés par un pointeur vont être transférés du cache vers un serveur contenant le store Git LFS. Dans les dépôts local et distant Git il n'y a donc plus que les pointeurs vers les fichiers.

Troisième étape, lorsque vous faites git checkout sur un commit dont la version contient des fichiers référencés, Git LFS va charger ces fichiers depuis le store Git LFS automatiquement !

Tout ceci fonctionne en fait avec des hooks Git ! A chaque fois que vous effectuez des actions Git, des hooks vont effectuer les opérations adéquates pour la gestion des fichiers avec Git LFS.

Ce qui est génial avec cette extension, c'est que mis à part le paramétrage du filtre, vous n'avez rien à faire ! Tout est fait automatiquement ! C'est pourquoi nous recommandons très fortement son utilisation.
Installation de la librairie Git LFS

Pour installer Git LFS sur MacOS :

brew install git-lfs

Sur GNU/Linux ou Windows :

sudo apt-get install git-lfs

Ensuite, vous n'avez plus qu'à faire dans un projet Git dans lequel vous voulez utiliser Git LFS :

git lfs install

Vous aurez le message de confirmation suivant :

Updated pre-push hook.
Git LFS initialized.

Comme expliqué précédemment le hook pre-push est utilisé pour envoyer les fichiers traités par Git LFS sur les serveurs distants.
Cloner un dépôt distant utilisant Git LFS

Il faut que vous aillez Git LFS installé ensuite vous n'avez plus qu'à faire :

git lfs clone lien

Un simple git clone fonctionne aussi, mais les performances sont meilleures en utilisant git lfs clone pour la récupération des fichiers gérés par LFS.

En fait, cette commande permet la parallélisation des téléchargements et de diminuer drastiquement le nombre de requêtes HTTP nécessaires.
Récupération des objets distants

Même chose pour la commande git pull, vous pouvez faire un simple git pull et les fichiers seront automatiquement téléchargés.

Si vous voulez paralléliser les téléchargements des fichiers gérés par Git LFS car votre projet en contient de nombreux, il faut créer l'alias suivant :

git config --global alias.plfs "!git -c filter.lfs.smudge= -c filter.lfs.required=false pull && git lfs pull"

Ensuite vous n'aurez plus qu'à faire :

git plfs

Paramétrer les types de fichiers et les dossiers gérés par Git LFS

Par défaut Git LFS ne fait rien, il faut lui indiquer quels types de fichier il doit prendre en charge.

Pour ce faire, il faut utiliser la commande git lfs track "regex" (attention ! Les guillemets sont importants).

Comme par exemple :

git lfs track "*.png"

Ici, tous les fichiers dont l'extension est .png seront pris en charge automatiquement.

Vous pouvez aussi dire à Git LFS de s'occuper de tous les fichiers placés dans un dossier, par exemple le plus courant est de faire :

git lfs track "assets/"

Il faut lancer cette commande à la racine de votre projet, car les liens sont créés à partir du dossier où vous êtes situé.

Git LFS va créer un fichier de configuration Git .gitattributes permettant de lier des actions à certains fichiers en utilisant des regex (aussi appelés pattern pour les fichiers).
Lister les fichiers et dossiers gérés par Git LFS

Vous pouvez simplement afficher les fichiers et dossiers gérés par Git LFS avec la commande :

git lfs track

Qui donnera par exemple :

Listing tracked patterns
    *.png (.gitattributes)
    assets (.gitattributes)

Arrêter de suivre des fichiers ou des dossiers

Il suffit d'utiliser git lfs untrack pour arrêter le suivi par Git LFS.

Vous pouvez par exemple faire :

git lfs untrack "assets/"

Récupérer des fichiers Git LFS récents

Par défaut, Git LFS ne charge que les fichiers utilisés par la version correspondant au commit sur lequel vous pointez.

Pour charger tous les fichiers utilisés par le dernier commit des branches récentes, vous pouvez faire :

git lfs fetch --recent

Une branche est considérée comme récente si il y a eu au moins un commit pendant les 7 derniers jours.

Cela peut être utile par exemple si vous voulez faire une session de review de plusieurs branches avec votre équipe.

Vous pouvez également télécharger tous les fichiers gérés par LFS :

git lfs fetch --all

Cela risque de prendre du temps si votre projet utilise de nombreux fichiers !
Supprimer des fichiers en cache pour gagner de la place

Si vous manquez de place en local, vous pouvez supprimer tous les fichiers localement suivis par Git LFS qui ne sont pas utilisés par le commit pointé, ou par des commits pas encore push ou par des commits créé pendant les 10 derniers jours.

Il suffit de faire :

git lfs prune

Lancer cette commande régulièrement est une bonne idée pour garder une taille du dépôt local modérée.

En effet, par défaut, les fichiers ne sont pas automatiquement supprimés localement par Git LFS.

Avant de lancer cette commande vous pouvez voir les suppressions qui seront effectuées en faisant :

git lfs prune --verbose --dry-run

Mais quoiqu'il arrive vous pourrez les re-télécharger facilement depuis les serveurs distants comme nous l'avons vu avec les commandes précédentes.

Pour être certain que vos serveurs distants ont bien une copie des fichiers qui seront supprimés vous pouvez faire :

git lfs prune --verify-remote

La commande va vérifier sur les serveurs distants la présence des fichiers avant de les supprimer.

Pour toujours utiliser cette option pour les lfs prune vous pouvez utiliser cette configuration :

git config --global lfs.pruneverifyremotealways true