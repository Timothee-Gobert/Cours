## La gestion de projets avec Git et Gitlab

Les tags
Qu'est-ce qu'un tag Git ?

Les tags sont des marqueurs pour des versions importantes. Ils sont également appelés étiquettes.

Vous pouvez marquer les versions avec un système semver (par exemple v1.0.0), vous pouvez marquer des étapes importantes : Nouvelle auth JWT etc.

Les tags sont très utilisés dans les projets importants car le nombre de commits est tel qu'il devient difficile de retrouver rapidement une version importante.
Créer un tag léger

Les tags légers sont des références statiques sur des commits.

Vous pouvez les voir comme des branches qui ne se déplacent jamais.

Pour créer une tag léger, il suffit de faire :

git tag nom

Cela créera un tag sur le commit pointé par HEAD.

Vous pouvez également créer un tag sur un commit spécifique plus ancien :

git tag nom hash

Les tags légers sont stockés dans .git/refs/tags/.

Le nom du fichier est le nom du tag est son contenu est le hash du commit sur lequel il pointe.
Créer un tag annoté

Les tags annotés sont des objets Git contenant des informations supplémentaires.

Ils contiennent le hash du commit, mais également le nom et l'adresse email du créateur du tag et un message d'étiquetage.

Pour créer un tag annoté il faut utiliser tag -a, par exemple :

git tag -a v1.0.1 -m 'Version 1.0.1 - auth fonctionnelle'

Le premier argument est le nom du tag. Ensuite, comme pour un commit, nous pouvons utiliser l'option -m pour spécifier directement le message du tag sans avoir à ouvrir l'éditeur de texte.
Visualiser les données d'un tag annoté

Pour visualiser toutes les informations d'un tag annoté, vous pouvez faire :

git show nomtag

Par exemple :

git show v1.0.1

Donnera les informations suivantes :

tag v1.0.1
Tagger: Jean Paul <jean@gmail.com>
Date:   Mon Mar 23 14:34:08 2020 +0100

Version 1.0.1 - auth fonctionnelle

commit 02d954056df1bbbb29668c8d8247104f416e3bce (tag: v1.0.1)
Author: Arthur <arthur@gmail.com>
Date:   Sat Mar 21 15:35:19 2020 +0100

    Le message de validation du commit s’affiche ici

diff --git a/test.js b/test.js
new file mode 100644
index 0000000..e69de29

Lister les tags

Pour lister les tags d'un projet, il suffit de faire :

git tag

Se positionner à la version référencée par un tag

Vous pouvez git checkout sur un tag comme vous le feriez sur un commit.

Vous serez dans ce cas en situation de HEAD détachée, servez-vous en uniquement pour vérifier l'état d'un projet à une certaine version.

git checkout nomtag

Par exemple :

git checkout v1.0.2

Supprimer un tag

Vous pouvez supprimer n'importe quel tag comme vous le feriez pour une branche :

git tag -d nomtag

Modifier un tag

Vous pouvez modifier un tag qui n'a pas encore été push en utilisant l'option --force ou -f.

Par exemple :

git tag -f v1.3 15027957251b64cf874c3557a0f3547bd83b3ff6

Cette commande va remplacer le tag v1.3 qui référençait un autre commit et référencer le commit avec le hash 15027957251b64cf874c3557a0f3547bd83b3ff6.

Attention ! Si vous modifiez un tag qui a été push, les autres utilisateurs n'auront pas la nouvelle référence en faisant git pull.

Ils conserveront l'ancienne référence pour le tag v1.3.

Il ne faut donc pas modifier un tag qui a été push. Trouvez un autre nom pour votre tag.

Si vous voulez absolument modifier le nom d'un tag il faut avertir tous les autres membres de votre équipe pour qu'ils fassent tous :

git tag -d v1.3
git fetch origin tag v1.3

De cette manière ils auront la nouvelle référence du tag, mais il vaut mieux changer de nom, c'est quand même plus simple !
Publier vos tags

Pour push vos tags, vous pouvez soit faire :

git push origin nomtag

Ou pour tous les push d'un coup :

git push origin --tags

Retrouver les tags sur Gitlab

Sur Gitlab, sélectionnez votre projet dans la liste des Projets en haut à gauche.

Une fois sur un projet, allez dans la colonne de gauche sur Dépôt puis sur Etiquettes.

Vous pouvez également utiliser le raccourci sur la page du projet montré dans la vidéo. 

Introduction aux fonctionnalités Gitlab
Le Project Overview

Lorsque vous sélectionnez un projet, vous arrivez sur le Project Overview, c'est-à-dire l'aperçu général du projet.

Vous retrouverez notamment :

Vous pouvez voir le nombre total de commits, et cliquer dessus pour arriver sur le détail des commits (comme un git log amélioré).

Vous avez également les branches, sur lesquelles vous pouvez également cliquer pour avoir les détails sur toutes les branches.

Et enfin, vous avez le raccourci pour les tags / étiquettes, que nous avons déjà montré.
Dépot > Commits

Si vous cliquez sur le raccourci commits vous arriverez dans le menu Dépôt > Commits.

Cette vue est très intéressante. Elle correspond à un git log avec des raccourcis très utiles.

Pour chaque commit vous avez la date, l'auteur et le message de validation.

En haut vous avez une liste sélectionnable (actuellement sur master) pour sélectionner la branche et visualiser les commits correspondants.

A droite de chaque commit, vous avez le hash réduit, que vous pouvez copier avec le bouton juste à droite Copy commit SHA.

Ensuite, plus à droite vous avez le bouton Parcourir les fichiers, qui vous permet de visualiser la version du répertoire à la version de ce commit. Cela équivaut à faire un git checkout sur le commit pour voir l'état du répertoire de travail.
Dépot > branches

L'onglet Dépot > branches sur lequel vous accédez également en cliquant sur le raccourci sur Project Overview, vous amène ici :

Vous retrouvez le nom des branches, le hash, la date et le début du message de validation du dernier commit sur chaque branche.

Vous pouvez les supprimer (sauf les branches protégées).

Vous pouvez créer une branche directement depuis Gitlab.

Vous pouvez également supprimer toutes les branches qui ont déjà été fusionnées.

Vous pouvez lancer l'outil de comparaison entre deux branches permettant de visualiser les commits divergents.

Vous pouvez également émettre une merge request ou demande de fusion que nous verrons plus tard. 

Les tickets Gitlab
Que sont les tickets (ou issues) ?

Les issues (ou tickets) sont extrêmement utiles pour la gestion de projets en équipe.

Nous vous les recommandons très fortement.

Elles permettent de créer des tickets pour des fonctionnalités, des bugs etc et de les assigner à un développeur.

Commencez par vous rendre dans le menu Tickets ou Issues si vous êtes resté en anglais.
La liste des tickets

Dans la liste vous pouvez voir tous les tickets ouverts et en créer de nouveaux.
Créer un ticket

Commencez par cliquer sur Créer un ticket :

L'écran de création d'un ticket est génial !

Commencez par créer un titre par exemple [feature] - Login Facebook.

Dans la description, vous pouvez créer une liste de tâches à effectuer pour ce ticket. Vous pouvez également ajouter des citations (par exemple pour citer de la doc), du code, des listes (numérotés ou non) et des tableaux.

Vous pouvez attacher des fichiers et notamment des images (ce qui est très utile pour rapporter des bugs).

Ensuite la liste sélectionnable Assignee permet de désigner le responsable du Ticket. Le ticket apparaîtra dans ses tickets et dans sa Todo list automatiquement :

Ici l'utilisateur a un ticket qui lui a été assigné et qu'il doit terminer. Il est également inclus dans sa liste de tâches à effectuer.

Nous verrons plus tard la partie Milestone, mais c'est un objectif à atteindre par exemple Authentification et vous pouvez ensuite créer les tickets correspondants pour atteindre l'objectif (par exemple [Back] Login, [Front] Login page, etc.

Due date permet de donner une date comme objectif pour compléter le ticket.

Labels permet de mettre une étiquette de couleur. Nous les reverrons.

Une fois que vous avez terminé, cliquez sur Submit ticket.
Détail d'un ticket

De retour sur la liste, vous pouvez sélectionner un ticket pour atteindre son écran détaillé :

Vous retrouver le titre et la description qui contient la liste de tâche.

Les membres de l'équipes peuvent y réagir avec des émoticônes, ils peuvent poster un message relatif au ticket en mentionnant des membres avec @pseudo etc.

Vous retrouvez ensuite l'historique du ticket : toutes les actions de tous les membres de votre équipe sur le ticket.

Close issue : permet de fermer le ticket, soit parce qu'il est terminé, soit parce qu'il est abandonné.

Dans la barre de droite, vous retrouvez les informations comme la date d'échéance (due date), les étiquettes et les participants.
Utiliser le numéro du ticket dans les messages de validation

Une bonne pratique est de systématiquement renseigner dans le corps du message de validation de vos commits, l'id du ticket correspondant.

L'id du ticket est #nombre, et vous le retrouvez en haut dans la navigation :

Par exemple :

git commit -m "Service Topbar" -m "#1"

Il faut également renseigner un court descriptif de ce que vous avez effectué dans le commit.

En renseignant #nombre dans le message de validation, Gitlab va automatiquement lié le commit au ticket, et vous pourrez le retrouver dans l'historique du ticket. C'est extrêmement pratique !
Ticket > Tableaux

Cette partie permet simplement d'afficher vos tickets sous forme de tableau Kanban.

Vous pouvez créer vos tableaux et vos listes.

Dans chaque tableau, vous pouvez ensuite déplacer vos tickets entre vos listes suivant votre progression.

Nous ne recommandons pas particulièrement l'utilisation des tableaux, mais certaines personnes aiment bien pouvoir visualiser leurs tâches de cette manière.

Cela donne par exemple :
Ticket > Étiquettes

Les étiquettes sont simplement des labels de couleurs que vous pouvez ajouter aux tickets.

Elles sont pratiques pour rapidement visualiser les tickets urgents, en cours, à faire etc.

Nous les recommandons et les utilisons extensivement.

Vous pouvez par exemple créer les étiquettes suivantes :

Urgent en rouge.

En cours en vert.

A faire en orange.

A tester en violet (ou A revoir, To review etc). 

Les Milestones Gitlab
Que sont les Milestones ?

Les Milestones ou Jalon sur Gitlab sont une manière de gérer un ensemble de tickets de manière très efficace.

Par exemple, vous souhaitez développer l'authentification d'une nouvelle application.

C'est votre objectif, votre Milestone.

Vous allez ensuite la découper en tickets : par exemple page de connexion, page d'inscription, route de connexion en back, sauvegarde des identifiants etc.
Créer un Milestone

Pour créer un milestone allez dans Tickets > Jalons et cliquer sur new milestone :

Il suffit d'entrer un titre, un descriptif et optionnellement des dates de début et de fin.

Une fois le milestone créé, il faut y associer tous les tickets correspondants.

Pour assigner un ticket à une milestone il suffit d'aller sur le ticket et dans la colonne de droite, aller dans Milestone puis Editer et sélectionnez le milestone.
Visualiser les milestones

Dans l'onglet Tickets > Jalons vous retrouverez tous les milestones actifs.

Vous pouvez cliquer sur une milestone pour afficher le détail :

Vous retrouvez tous les tickets associés et leur statut.

La première colonne contient les tickets associés mais qui n'ont pas encore été assignés à un développeur.

La seconde colonne contient les tickets associés et assignés qui sont donc en cours.

Et enfin, la troisième colonne contient les tickets terminés.

Comme pour les tickets, vous pouvez fermer les milestones, en faisant close milestone une fois tous les tickets complétés.

Dans la colonne de droite vous retrouvez les informations sur la date de début et de fin, ainsi que le nombre de tickets (issues) en cours (opened) et terminés (closed).

En haut, vous avez la barre de progression en fonction du nombre total de tickets et des tickets terminés. 

Les demandes de fusion (merge requests)
Que sont les demandes de fusion ? (merge request)

Cette fonctionnalité concerne les projets open source ou les très larges projets.

Dans ce type de projet, les développeurs ne pourront pas fusionner directement dans la branche master leur branche. Il faudra demander aux responsables de le faire.

Cette demande de vérification et de fusion s'appelle une demande de fusion (ou merge request).
Créer une demande de fusion

Vous devez vous rendre dans la colonne de droite sur Demandes de fusion.

Vous pouvez également le faire depuis une branche sur Gitlab.

Vous devez tout d'abord sélectionner la branche source (la branche dont la fusion est demandée) et la branche cible (target) qui est en général master, dans laquelle sera fusionnée la branche :

Vous arrivez ensuite sur l'écran permettant de rentrer le descriptif de la demande de fusion :

Vous devez ici mettre le descriptif de vos changements et assigner le reviewer, c'est-à-dire le ou les développeurs responsables des fusions.

Remarquez que vous avez une option permettant de supprimer automatiquement la branche lorsque la fusion est terminée (Delete source branch when merge request is accepted.).

Vous avez également une option pour squash tous les commits de la branche en un seul commit (Squash commits when merge request is accepted.).

C'est généralement déconseillé pour conserver l'historique entier du projet.

Cependant, dans les projets opensource, cela peut être utile si les messages de validation des commits de la branche ne respectent pas les normes du projet, ou qu'ils ne sont pas propres.

Une fois la demande de fusion créé, le développeur assigné va recevoir un email et va également voir qu'il a une demande de fusion dans sa barre de statut :
Le détail d'une demande de fusion

Celui qui va vérifier la demande de fusion et éventuellement l'accepter aura cette vue :

Il peut voir les commits et leur détails dans l'onglet commits.

Il peut visualiser les changements dans l'onglet changes.

Il peut faire des commentaires, vous demander des précisions etc.

Il peut bien sûr la clore, c'est-à-dire la refuser (avec close merge request).

Il peut résoudre les conflits directement dans Gitlab ou localement.
La résolution des conflits de fusion sur Gitlab

Dans Gitlab, l'outil de résolution des conflits est très similaire à VS Code :

Il doit sélectionner la version pour chaque conflit.

Ensuite il devra mettre le message du commit de fusion, ou garder celui par défaut.

Il doit enfin Commit to source branch quand il a terminé. Le commit de fusion sera ajouté.
Fusionner la branche

Une fois qu'il n'y a plus de conflits, le bouton vert Merge apparaît :

Une fois la fusion effectuée, vous aurez un message de confirmation :

Introduction au GitFlow

un flow Git est un ensemble de règles qu'une équipe s'impose sur un projet pour une gestion optimale d'un projet.

Nous allons vous proposer deux flow : un pour les petites et moyennes équipes (jusqu'à 20 développeurs) et un pour les larges projets (à partir de 20 développeurs).
Flow Git standard

Dans ce premier flow, vous avez une branche principale master qui correspond à la branche qui est en production sur des serveurs.

Vous avez une seconde branche develop qui est la branche de développement.

Les fonctionnalités doivent être réalisées sur des branches créées à partir de la branche develop. Ces branches sont fusionnées dans develop et supprimées une fois la fonctionnalité terminée.

Les fusions doivent être faites en git merge --no-ff pour conserver l'ensemble des branches de fonctionnalité en empêchant les fusions en avance rapide.

Tous les pulls doivent utiliser git pull --rebase pour conserver des historiques de branches propres.

Les fusions de la branche develop dans la branche master doivent toujours passer par une demande de fusion qui est validée par un autre développeur après relecture du code.

Une fois la fusion effectuée la version est mise en production.

Si un bug critique est présent en production, il est corrigé sur une branche hotfix créée depuis master et cette branche est ensuite fusionnée dans master et develop.
Le GitFlow

Le GitFlow est un flow plus exigeant que celui que nous avons vu. Il est utile pour les projets importants.

Ce flow a été créé en 2010 et est devenu très populaire.

Voici le schéma officiel de son créateur :

Nous allons d'abord expliquer les différentes branches et faire un exemple complet dans la leçon suivante.
Les branches principales

La branche master est la version en production. Elle a un tag pour chaque version.

La branche develop est la branche d'intégration avec les dernières fonctionnalités qui sont prêtes à être mises en production.

Ces deux branches sont les seules à être permanentes, elles sont appelées branches principales.
Les branches de support

Les branches de fonctionnalités (feature branches) sont des branches créées à partir de develop et doivent être fusionnées dans develop.

Le nom de la branche est par convention le nom de la fonctionnalité développée. Par exemple, authentification ou new-landing etc.

Elles sont utilisées pour chaque développement d'une fonctionnalité.

Les fusions dans develop doivent être faites sans avance rapide :

git merge --no-ff

Une fois la fusion effectuée, il suffit de la supprimer.

Les branches de mise en production (release branches) sont les branches qui sont utilisées pour le travail préparatoire à la mise en production d'une nouvelle version.

Elles sont créées depuis la branche develop et doivent être fusionnées dans develop et master une fois terminées.

Elles ne doivent contenir que des réparations de petits bugs avant la mise en production.

Leur nom doit commencer par release-* suivi du numéro de version. Ce sera le numéro de la version qui sera utilisée comme tag sur master.

Les fusions dans develop et master doivent être faites sans avance rapide également.

Après la fusion dans master, il ne faut pas oublier de créer le tag.

Une fois les fusions effectuées, il suffit de la supprimer.

Les branches de bugs critiques (hotfix) sont les branches utilisées pour effectuer des modifications critiques et imprévues sur la version en production.

Elles sont créées depuis master et sont fusionnées, sans avance rapide dans master et develop. Sauf si une branche release est en cours, dans ce cas il faut la fusionner dans master et cette branche release (qui sera elle-même fusionnée dans develop comme nous l'avons vu).

Leur nom doit commencer par hotfix-* puis le numéro de la nouvelle version qui résultera de la correction.

Il ne faut pas oublier de tag la nouvelle version sur master une fois la fusion effectuée avec le numéro de version correspondant. 

Exemple de GitFlow

Pour bien comprendre, nous allons dans cette leçon voir un exemple complet du GitFlow mais sans librairie pour nous aider.

Nous utiliserons la librairie dans la leçon suivante.

Pour ce test, nous partons d'un dépôt Git totalement vide.

Nous créons un nouveau projet sur Gitlab et nous le clonons en local :

git clone git@gitlab.com:erwanv/gitflow.git
cd gitflow

Initialisation

Nous créons un fichier et notre commit initial :

touch test.txt
git add test.txt
git commit -m "initial commit"

Nous créons le tag pour la version initiale v0.0.1.

Nous faisons un git push en veillant à préciser l'option -u pour créer la liaison avec la branche de suivi origin/master.

git tag v0.0.1
git push -u origin master --tags

Nous avons bien la confirmation : La branche 'master' est paramétrée pour suivre la branche distante 'master' depuis 'origin'..

Nous créons ensuite la branche develop :

git checkout -b develop
git push -u origin develop

Nous mettons ensuite en place la branche de suivi.
Cas d'un hotfix

Notre première version comporte malheureusement un bug que nous voulons vite corriger.

Nous allons donc créer une branche hotfix.

Comme nous étions à la version v0.0.1 et qu'il s'agit d'une correction de bug, la version sera hotfix-0.0.2 :

git checkout master
git checkout -b hotfix-0.0.2

Nous faisons notre correction et nos commits :

echo 42 > test.txt
git commit -am "fix: header réparé"

Nous n'avons plus qu'à le fusionner dans master :

git checkout master
git merge --no-ff hotfix-0.0.2
git tag -a 0.0.2

Nous n'oublions pas de créer le tag pour la nouvelle version. Nous décrivons dans la note la correction effectuée.

Nous fusionnons la branche dans develop :

git checkout develop
git merge --no-ff hotfix-0.0.2

Nous pouvons enfin la supprimer

git branch -d hotfix-0.0.2

Nous n'oublions pas de push toutes les branches et tous les tags :

git push --all
git push --tags

Vous pouvez regarder où nous en sommes :

git branch -avv

Qui nous donne :

develop                422c63d [origin/develop] Merge branch 'hotfix-0.0.2' into develop
* master                 4007bb8 [origin/master] Merge branch 'hotfix-0.0.2'
  remotes/origin/develop 422c63d Merge branch 'hotfix-0.0.2' into develop
  remotes/origin/master  4007bb8 Merge branch 'hotfix-0.0.2'

Développement d'une fonctionnalité

Nous allons développer l'authentification de notre application.

Nous commençons par créer la branche depuis develop :

git checkout develop
git checkout -b auth

Vous pouvez également utiliser le raccourci, strictement équivalent :

git checkout -b auth develop

Il signifie crée une branche auth depuis la branche develop et checkout dessus.

Nous faisons les modifications pour créer la fonctionnalité et nous créons un ou plusieurs commits.

touch connexion.js
touch inscription.js
git add .
git commit -m "feat: auth terminée"

Nous n'avons plus qu'à fusionner d'abord la branche dans develop :

git checkout develop
git merge --no-ff auth

Nous la supprimons et nous publions notre travail :

git branch -d auth
git push --all

Préparation d'une mise en production

Une fois le ou les fonctionnalités prêtes et fusionnées dans develop, nous allons créer une branche release pour réparer une mise en production.

Comme nous sortons de nouvelles fonctionnalités importantes, nous allons considérer qu'il s'agit d'une release majeure et le numéro de version sera 1.0.0 donc nous allons créer la branche release-1.0.0 depuis develop :

git checkout -b release-1.0.0 develop

Nous nous apercevons qu'il y a un petit bug, aucun problème nous le corrigeons directement sur la branche release :

echo 2 >> test.txt
git add .
git commit -m "fix: connexion réparée"

Après les tests de l'équipe, nous sommes prêts pour la mise en production.

Nous effectuons donc les fusions adéquates :

git checkout master
git merge --no-ff release-1.0.0
git tag -a 1.0.0

Dans le tag annoté nous décrivons les fonctionnalités de la nouvelle version, puis les correctifs apportés.

Nous fusionnons ensuite dans develop :

git checkout develop
git merge --no-ff release-1.0.0

Et nous supprimons enfin la branche :

git branch -d release-1.0.0

Comme vous pouvez le constater c'est assez imposant à mettre en oeuvre ! Il existe une librairie pour nous aider que nous allons voir dans la leçon suivante. 

La librairie GitFlow
La librairie git-flow

Il existe une librairie permettant de générer les actions Git à effectuer pour suivre le GitFlow de manière beaucoup plus rapide.

Sur MacOS, il suffit de faire brew install git-flow-avh.

Sur Linux / Windows (sur Git Bash), faites sudo apt-get install git-flow.

Nous repartons d'un nouveau projet Gitlab et d'un nouveau répertoire local :

git clone git@gitlab.com:erwanv/gitflow.git
cd gitflow

Initialiser git flow

Pour initialiser git flow dans un nouveau projet faites :

git flow init

Un ensemble de questions vous serons posé pour choisir les noms des branches :

Dépôt Git vide initialisé dans /home/erwan/projects/gitflow/.git/
No branches exist yet. Base branches must be created now.
Branch name for production releases: [master]
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Bugfix branches? [bugfix/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []
Hooks and filters directory? [/home/erwan/projects/gitflow/.git/hooks]

Vous pouvez faire entrée à chaque question pour choisir le nom par défaut qui convient très bien.

Pour le faire dès l'initialisation il suffit de faire :

git flow init -d

L'option -d étant pour default.

L'initialisation va créer un commit initial et la branche develop.

git branch -avv

Donne :

* develop 1e5c8e0 Initial commit
  master  1e5c8e0 Initial commit

N'oubliez pas de définir les relations branches de suivi et branches locales :

git checkout master
git push -u origin master
git checkout develop
git push -u origin develop

Développer une fonctionnalité

Pour développer une fonctionnalité il suffit de faire :

git flow feature start auth

La librairie va effectuer toutes les actions requises :

Basculement sur la nouvelle branche 'feature/auth'

Summary of actions:
- A new branch 'feature/auth' was created, based on 'develop'
- You are now on branch 'feature/auth'

Now, start committing on your feature. When done, use:

      git flow feature finish auth

Faisons un commit pour simuler le développement de la fonctionnalité :

touch test.txt
git add test.txt
git commit -m "feature: auth done"

Puis terminons la fonctionnalité :

git flow feature finish auth

Automatiquement la librairie réalise les actions suivantes :

Basculement sur la branche 'develop'
Mise à jour 1e5c8e0..a77a2c4
Fast-forward
  test.txt | 0
  1 file changed, 0 insertions(+), 0 deletions(-)
  create mode 100644 test.txt
Branche feature/auth supprimée (précédemment a77a2c4).

Summary of actions:
- The feature branch 'feature/auth' was merged into 'develop'
- Feature branch 'feature/auth' has been locally deleted
- You are now on branch 'develop'

Comme ce que nous avions fait manuellement à la leçon précédente, la branche de fonctionnalité a été fusionnée dans develop puis elle a été supprimée.
Développer une fonctionnalité à plusieurs

Simulons maintenant une nouvelle fonctionnalité sur laquelle nous allons développer à plusieurs :

git flow feature start new-landing

Simulons un commit :

touch test2.txt
git add test2.txt
git commit -m "feature: header et footer"

Partageons notre branche avec notre équipe :

git flow feature publish new-landing

Le résultat est le suivant :

Décompte des objets: 7, fait.
Delta compression using up to 12 threads.
Compression des objets: 100% (4/4), fait.
Écriture des objets: 100% (7/7), 622 bytes | 622.00 KiB/s, fait.
Total 7 (delta 0), reused 0 (delta 0)
To gitlab.com:erwanv/gitflow.git
  * [new branch]      feature/new-landing -> feature/new-landing
La branche 'feature/new-landing' est paramétrée pour suivre la branche distante 'feature/new-landing' depuis 'origin'.
Déjà sur 'feature/new-landing'
Votre branche est à jour avec 'origin/feature/new-landing'.

Summary of actions:
- The remote branch 'feature/new-landing' was created or updated
- The local branch 'feature/new-landing' was configured to track the remote branch
- You are now on branch 'feature/new-landing'

La librairie a commencé par pull la branche pour vérifier qu'elle n'existait pas déjà.

Ensuite elle a automatiquement créé la branche amont sur le dépôt distant et a créé le lien entre branche de suivi et branche locale.

Enfin, elle a push la branche.

Les autres membres de l'équipe n'ont plus qu'à récupérer les changements :

git flow feature track new-landing

Cette commande va récupérer la branche amont sur leur ordinateur.

Les membres de l'équipes peuvent développer la fonctionnalité sur la branche et une fois qu'elle est terminée faire :

git flow feature finish new-landing

La fusion sera effectuée et les branches locale et de suivi seront automatiquement supprimée :

Basculement sur la branche 'develop'
Mise à jour f20cb70..f58fe97
Fast-forward
  test2.txt | 1 +
  1 file changed, 1 insertion(+)
To gitlab.com:erwanv/gitflow.git
  - [deleted]         feature/new-landing
Branche feature/new-landing supprimée (précédemment f58fe97).

Summary of actions:
- The feature branch 'feature/new-landing' was merged into 'develop'
- Feature branch 'feature/new-landing' has been locally deleted; it has been remotely deleted from 'origin'
- You are now on branch 'develop'

Préparer une mise en production

Pour réaliser une mise en production, il faut passer par une branche release !

Très simple avec git-flow :

git flow release start v1.0.0

Nous vous conseillons de mettre le numéro de version comme nom de release. Car il sera utilisé notamment pour le tag.

Les actions suivantes seront effectuées :

Basculement sur la nouvelle branche 'release/v1.0.0'

Summary of actions:
- A new branch 'release/v1.0.0' was created, based on 'develop'
- You are now on branch 'release/v1.0.0'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

      git flow release finish 'v1.0.0'

Vous pouvez la publier pour que d'autres personnes puissent travailler dessus :

git flow release publish v1.0.0

Le résumé des actions sera le suivant :

Summary of actions:
- The remote branch 'release/v1.0.0' was created or updated
- The local branch 'release/v1.0.0' was configured to track the remote branch
- You are now on branch 'release/v1.0.0'

Les autres membres n'ont plus qu'à la récupérer et à faire des changements si besoin :

git flow release track v1.0.0

Lorsque la version est prête à être mise en production, il n'y a plus qu'à lancer :

git flow release finish v1.0.0

L'éditeur s'ouvrira trois fois : une première fois pour le message du commit de fusion dans master, une seconde fois pour le tag annoté sur master et une dernière fois pour le message du commit de fusion sur develop.

Basculement sur la branche 'master'
Merge made by the 'recursive' strategy.
  test.txt  | 0
  test2.txt | 1 +
  2 files changed, 1 insertion(+)
  create mode 100644 test.txt
  create mode 100644 test2.txt
Déjà sur 'master'
Votre branche est basée sur 'origin/master', mais la branche amont a disparu.
  (utilisez "git branch --unset-upstream" pour corriger)
Basculement sur la branche 'develop'
Déjà à jour !
Merge made by the 'recursive' strategy.
To gitlab.com:erwanv/gitflow.git
  - [deleted]         release/v1.0.0
Branche release/v1.0.0 supprimée (précédemment f58fe97).

Summary of actions:
- Release branch 'release/v1.0.0' has been merged into 'master'
- The release was tagged 'v1.0.0'
- Release tag 'v1.0.0' has been back-merged into 'develop'
- Release branch 'release/v1.0.0' has been locally deleted; it has been remotely deleted from 'origin'
- You are now on branch 'develop'

Il faut push le tag généré :

git push --tags

Il n'y a plus qu'à mettre en production la version !
Corriger un bug critique

Même chose, la correction d'un bug critique est facilité par la librairie.

Il suffit de passer la version qui sera mise en production après la correction. Dans notre exemple c'est la version v1.0.1 :

git flow hotfix start v1.0.1

Partagez le avec votre équipe :

git flow hotfix publish v1.0.1

Les membres de votre équipe peuvent travailler sur la branche en faisant :

git pull
git checkout hotfix/v1.0.1

Une fois les commits de correction effectué, plus qu'à faire :

git flow hotfix finish v1.0.1

L'éditeur de texte s'ouvrira 3 fois également.

Basculement sur la branche 'master'
Merge made by the 'recursive' strategy.
  test.txt | 1 +
  1 file changed, 1 insertion(+)
Basculement sur la branche 'develop'
Merge made by the 'recursive' strategy.
  test.txt | 1 +
  1 file changed, 1 insertion(+)
Branche hotfix/v1.0.1 supprimée (précédemment 5e0f0e7).

Summary of actions:
- Hotfix branch 'hotfix/v1.0.1' has been merged into 'master'
- The hotfix was tagged 'v1.0.1'
- Hotfix tag 'v1.0.1' has been back-merged into 'develop'
- Hotfix branch 'hotfix/v1.0.1' has been locally deleted
- You are now on branch 'develop'

N'oubliez pas de push le tag :

git push --tags

Supprimer une branche

Si vous abandonnez une branche de feature, release ou hotfix vous pouvez faire simplement :

git flow feature delete nom
git flow release delete nom
git flow hotfix delete nom