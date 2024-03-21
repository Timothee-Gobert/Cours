## Modifier des commits

Introduction et git commit --amend
Rectifier un commit

Parfois vous souhaiterez modifier un commit que vous venez de faire : soit pour ajouter des fichiers que vous avez oublié, soit pour modifier le message de validation.

Dans ces deux cas il faudra utiliser :

git commit --amend

Attention ! Il ne faut pas utiliser cette commande si vous avez déjà push votre commit. Nous verrons dans le prochain chapitre ce que cela signifie.

Cette commande créera un nouvel objet commit avec un nouvel arbre et un nouveau message de validation.

Il aura donc un nouveau hash pour l'identifier, et c'est pour cette raison qu'il ne faut surtout pas modifier le commit de cette manière si vous l'avez déjà publié sur le répertoire distant.
Modifier le message de validation

Si vous voulez simplement modifier le message de validation, il suffit de faire :

git commit --amend

L'éditeur va s'ouvrir et vous pourrez alors modifier le message puis sauvegarder.
Ajouter un ou plusieurs fichiers oubliés

Imaginons que vous venez de réaliser un commit après des modifications :

git commit -am "Un message de validation"

Vous avez cependant oublié d'ajouter un fichier non suivi à l'index et donc au commit et vous ne voulez pas polluer l'historique du projet. Il faut alors taper les commandes suivantes :

git add fichier
git commit --amend

Si vous ne voulez pas modifier le message de validation mais seulement les fichiers oubliés :

git commit --amend --no-edit

git rebase
Git et le rebasage

Reprenons une situation que nous avons déjà vue.

Jusqu'à maintenant nous avons vu que nous pouvions faire une fusion à trois sources avec la stratégie récursive dans cette situation.

git checkout master
git merge auth

Dans ce cas, les trois sources seront les versions correspondant aux commits C (premier ancêtre commun), E et F (commits en violet sur le schéma ci-dessous).

Dans ce cas, un nouveau commit de fusion serait créé, et si des conflits de fusion apparaissaient il faudrait les résoudre. Nous aurions ensuite :

Il existe également une autre méthode, qui n'est pas une fusion : le rebasage.

Cette méthode permet de prendre tous les changements de chaque commit sur une branche et de les appliquer les uns après les autres à la fin d'une autre branche.

Il suffit de faire, à partir de la situation originelle :

git checkout auth
git rebase master

Notez bien que nous faisons le rebase depuis la branche auth avec pour cible la branche master.

Vous aurez par exemple dans votre terminal :

Rembobinage préalable de head pour pouvoir rejouer votre travail par-dessus...
Application de  Message de validation commit D
Application de  Message de validation commit F

Ce qui donne :

Notez bien l'ordre des commits ! Nous avons pris les commits de la branche auth et les avons ajoutés à la suite de la branche master.

Si des conflits apparaissent, il faudra bien sûr les résoudre.
Fonctionnement du rebasage

En fait, Git va rechercher le premier ancêtre commun aux deux branches (le commit C dans notre exemple).

Il va ensuite, prendre toutes les modifications de chaque commits sur la branche courante (ici auth et donc les commits D et F) et les enregistrer dans des fichiers temporaires.

Puis, la branche courante (toujours auth) est réinitialisée au commit de la branche de destination (commit E).

Enfin, les modifications correspondants aux anciens commits D et F sont appliquées. Les nouveaux commits contiennent les mêmes modifications mais ont des hashs différents (c'est pour cela que nous les notons comme étant D' et F').

Ensuite vous pouvez faire une fusion en avance rapide (fast-forward) ce qui revient à déplacer la branche master sur le même commit pointé par auth :

git checkout master
git merge auth

Vous aurez ensuite :

Le rebasage revient à supprimer la branche de l'historique et de faire comme si les changements avaient été appliqués à la suite de la branche ciblée.

Vous pouvez enfin supprimer la branche source (ici auth) :

git branch -d auth

Recommandations d'utilisation

Attention ! Il ne faut jamais rebaser des commits qui ont déjà été push.

Pourquoi ? Car comme nous l'avons vu en rebasant, les commits de la branche sont supprimés, et de nouveaux commits avec des hash différents seront créés.

De manière générale avec Git, et nous aurons l'occasion de le rappeler dans le chapitre suivant, il ne faut jamais modifier l'historique si il a été envoyé sur le répertoire distant, sinon vous casserez tout pour les autres.

Nous verrons en détails les recommandations de l'utilisation du rebasage avec les répertoires distants.

Mais pour le moment, retenez que vous ne devez rebaser que vos branches qui n'ont pas été push ou alors dont vous êtes certain à 100% que personne d'autre n'a travaillé dessus. 

Le rebasage interactif
Le rebasage interactif

Parfois, vous voudrez effectuer des modifications d'historique plus complexes : comme par exemple modifier ou rassembler plusieurs commits.

Le rebasage interactif permet de modifier plusieurs commits qui se suivent.

Il suffit de faire :

git rebase -i

Vous devez passer ensuite un commit de début et un commit de fin. Le commit de fin est le parent du dernier commit que vous voulez modifier.

Vous pouvez passer deux hash raccourcis, par exemple :

git rebase -i 902d5d1 6c9f6f4

Ou deux hash complets :

git rebase -i 902d5d16eec3decefbb8cb1c3976e0ab78a49776 6c9f6f44962cc8a023b16e27405ecee75cb6a894

Vous pouvez utiliser les raccourcis ^ ou ~. Par exemple :

git rebase -i HEAD~3

Le commit de début est alors celui pointé indirectement par HEAD, et le commit de fin est celui trois commits en amont de HEAD.

Nous avons donc :

Notez bien que nous modifierons ici les commits E, D et C.

Le commit B étant bien le parent du dernier commit que nous souhaitons modifier.

Chaque commit du rebasage interactif sera réécrit, que vous changiez le message de validation ou non.

Pour cette raison, il ne faut pas non plus rebaser interactivement les commits déjà push !
Fonctionnement de la commande

Si vous faites cette commande, l'éditeur s'ouvrira et vous pourrez entrer des options pour chaque commit :

git rebase -i HEAD~3

Donne par exemple :

Vous avez les commandes en anglais dans la vidéo et ici en français car suivant les environnements vous pouvez retrouver les deux.

Comme vous pouvez le lire, vous pouvez modifier simplement un ou plusieurs messages de validation avec reword, éditer un commit, le fusionner avec le commit précédent, en conservant (avec squash) ou en supprimant son message de validation (fixup), vous pouvez enfin en supprimer un avec drop.

Dans tous les cas les commits changeront de hash car de nouveaux objets commits seront créés !

Attention également que les commits sont dans l'ordre inversés par rapport à git log !

Les commits sont du plus ancien au plus récent !

C'est logique car Git va suivre vos commandes dans l'ordre du plus ancien au plus récent, pour modifier l'historique comme vous le souhaitez.
Renommer plusieurs messages de validation

Nous allons prendre un premier exemple où nous voulons simplement modifier les trois derniers messages de validation.

git rebase -i HEAD~3

Nous faisons ensuite :

Nous mettons à chaque ligne r pour reword.

Ensuite, nous sauvegardons (ctrl + o) et nous quittons (ctrl + x).

Git ouvre ensuite l'éditeur pour chacun des trois commits et nous laisse modifier les messages un par un.

A la fin des trois éditions il nous indique que le rebasage est terminé :

[HEAD détachée 359ccca] Quatrième sur auth
  Date: Sun Mar 15 18:32:02 2020 +0100
  1 file changed, 1 insertion(+)
  create mode 100644 index.js
[HEAD détachée 9ac6f97] Cinquième sur auth
  1 file changed, 1 insertion(+), 1 deletion(-)
[HEAD détachée eb396c1] Sixième
  1 file changed, 1 insertion(+), 1 deletion(-)
Successfully rebased and updated refs/heads/master.

Nous voyons que HEAD est déplacé en fait sur chaque commit pour le modifier. C'est pour cette raison que nous sommes en mode HEAD détaché. Cela revient à checkout sur chaque commit l'un après l'autre.

Si nous faisons :

git log --oneline

Nous remarquons :

eb396c1 (HEAD -> master) Sixième
9ac6f97 Cinquième sur auth
359ccca Quatrième sur auth

Nous avons bien les nouveaux messages de validation et les hash ont été modifiés !

Retenez donc bien que lors d'un rebasage, même interactif, de nouveaux objets commits sont créés !.
Changer l'ordre de commits

Grâce au rebasage interactif, vous pouvez très simplement modifier l'ordre d'une suite de commit.

Par exemple, pour inverser les deux derniers commits, nous pouvons faire :

git rebase -i HEAD~2

Il suffit ensuite de couper la deuxième ligne (ctrl + k avec nano) et de la coller au dessus de la première (ctl + u sur nano).

Deux cas peuvent ensuite survenir.
Vous n'avez pas de conflits

Dans ce cas, le rebasage est terminé.

Vous n'avez plus qu'à valider.
Vous avez des conflits

Si vous avez des conflits, allez dans VS Code.

Sélectionnez les changements à conserver.

Ajouter le ou les fichiers, une fois les conflits réglés, avec le bouton +.

Ensuite, faites dans le terminal :

git rebase --continue

Enfin, validez.

Git créera le nouveau commit puis passera au second, répétez les étapes si il y a encore des conflits avec le second commit.
Supprimer des commits

Pour supprimer un ou plusieurs commit vous pouvez commencer par faire :

git rebase -i HEAD~4

Le chiffre dépend bien sûr du nombre de commits que vous souhaitez modifier.

Ensuite vous avez deux choix, soit supprimer la ligne complètement (ctrl + k avec nano), soit ajouté d pour drop.

Enfin, vous n'avez plus qu'à sauvegarder et quitter.
Rassembler des commits

Parfois, vous voudrez rassembler plusieurs commits en un seul.

Par exemple, si vous voulez rassembler les trois derniers commits en un seul, vous pouvez faire :

git rebase -i HEAD~3

L'éditeur s'ouvrira comme d'habitude, et cette fois vous mettrez s pour squash :

Ensuite vous sauvegardez et Git va exécuter vos instructions. L'éditeur sera à nouveau ouvert pour vous inviter à choisir le message de validation pour le nouveau commit rassemblant les trois derniers commits :

Nous laissons les messages par défaut, c'est-à-dire que les messages de validation des trois commits sont mis dans le message de validation du nouveau commit de rebasage.

Après validation vous aurez par exemple :

[HEAD détachée 37b3e26] Un commit
  Date: Mon Mar 16 16:31:46 2020 +0100
  3 files changed, 3 insertions(+), 1 deletion(-)
  create mode 100644 index.js
  create mode 100644 test2.txt
Successfully rebased and updated refs/heads/master.

Si vous faites git log, vous aurez en dernier commit :

commit 37b3e26546047f852dc8f434a0435a058ea99287 (HEAD -> master)
Author: Erwan Vallée <contact@dyma.fr>
Date:   Mon Mar 16 16:31:46 2020 +0100

    Un commit

    Quatre sur auth

    Sixième

Nous avons donc bien rassemblé les trois derniers commits en un seul !
Diviser un commit en plusieurs commits

Parfois, vous ferez un commit avec des changements qui n'ont rien à voir et vous voudrez donc le diviser en plusieurs commits thématiques pour plus de cohérence et donc de lisibilité de l'historique de votre projet.

Il faut de nouveau utiliser le rebasage interactif si le commit à diviser n'est pas le dernier :

git rebase -i HEAD~3

Comme d'habitude, l'éditeur va s'ouvrir, mais cette fois-ci nous allons utiliser e ou edit sur le commit que nous souhaitons diviser.

Dans notre exemple, ce sera l'avant dernier commit :

Une fois que nous sauvegardons et quittons, nous avons cette fois :

arrêt à 12f0345... Six master
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue

C'est tout à fait logique ! Nous ne modifions pas que des messages de validation, et cela ne servirait à rien d'ouvrir l'éditeur de texte.

Ici nous sommes en cours de rebasage, arrêté sur le commit 12f0345, c'est-à-dire l'avant dernier commit que nous souhaitons modifier.

Si vous faîtes :

git status

Vous aurez d'ailleurs :

rebasage interactif en cours ; sur 8cb6127
Dernières commandes effectuées (2 commandes effectuées) :
    pick f6538e3 trois
    edit 12f0345 Six master
Prochaine commande à effectuer (1 commande restante) :
    pick 37b3e26 Un commit
  (utilisez "git rebase --edit-todo" pour voir et éditer)
Vous êtes actuellement en train d'éditer un commit pendant un rebasage de la branche 'master' sur '8cb6127'.
  (utilisez "git commit --amend" pour corriger le commit actuel)
  (utilisez "git rebase --continue" quand vous avez effectué toutes vos modifications)

rien à valider, la copie de travail est propre

Git nous indique bien que nous sommes en plein rebasage interactif.

Nous pouvons maintenant annuler le commit 12f0345 en gardant les modifications :

git reset HEAD^

Ce qui équivaut à se replacer au commit f6538e3 tout en conservant les modifications dans le répertoire de travail du commit que nous voulons diviser.

Vous aurez alors quelque chose comme :

Modifications non indexées après reset :
M	test.txt
M	fichier1.txt

Nous pouvons par exemple maintenant créer un commit avec le fichier test.txt :

git add test.txt
git commit -m 'Maj test pour corriger x'

Puis faire un second commit avec fichier1.txt :

git add fichier1.txt
git commit -m 'Maj fichier1 pour corriger y'

Enfin, il ne reste plus qu'à continuer le rebasage pour appliquer le dernier commit :

git rebase --continue

Vous aurez alors un message confirmant la fin du rebasage :

Successfully rebased and updated refs/heads/master.

Nous ne le dirons jamais assez, ne rebasez pas des commits publiés sur le répertoire distant ! Tous les hash des commits sont modifiés lors d'un rebasage et cela cassera l'historique de votre projet pour votre équipe !

Utilisation de git merge --squash
Rassembler plusieurs commits lors d'une fusion de branche

Parfois, typiquement pour réparer rapidement un bug, vous aurez une branche locale avec des messages de commits rapides (test x, fix y etc), ce qui nuit à la lisibilité de l'historique du projet.

Une fois le bug réparer vous voudrez donc créer un commit propre sans tous les tests ayant été effectués et avec un message de validation propre.

Pour ce faire, il faut utiliser l'option --squash lors de la fusion.

Vous pouvez faire :

git checkout master
git merge --squash branchefix
git commit -m "Réparation du bug X" -m "Liste des corrections apportées"

Vous aurez alors un nouveau commit correspondant à la version du dernier commit de la branche de fix avec un message de validation propre.

Après avoir bien vérifié que le projet est fonctionnel et que vous avez les correctifs apportés, vous pouvez supprimer la branche de fix :

git branch -D branchefix

Même chose ! A n'utiliser que si la branche de fix est locale et qu'aucune autre personne n'a travaillé dessus ! 

Le reflog
Le RefLog

Le RefLog est l'historique des références qui ont été pointées par HEAD et par vos branches.

Autrement dit, il contient toutes les références des commits sur lesquels ont été placés HEAD, y compris de manière indirecte lorsque HEAD pointe sur une branche qui pointe sur un commit.

Et ce, également y compris pour les commits dits orphelins, à savoir les commits qui ne sont accessibles depuis aucune branche.

Le ReFlog est uniquement local, cela signifie que ce sont les déplacements que vous avez fait localement uniquement.

Si vous faites :

git reflog

Vous aurez par exemple :

5b0b87f (HEAD -> master) HEAD@{0}: rebase -i (finish): returning to refs/heads/master
5b0b87f (HEAD -> master) HEAD@{1}: rebase -i (pick): Un commit
8ca8356 HEAD@{2}: commit: Maj test pour corriger x
f6538e3 HEAD@{3}: reset: moving to HEAD^
12f0345 HEAD@{4}: rebase -i : avance rapide
f6538e3 HEAD@{5}: rebase -i (start): checkout HEAD~3
37b3e26 HEAD@{6}: rebase -i (finish): returning to refs/heads/master
37b3e26 HEAD@{7}: rebase -i (squash): Un commit
9c25c71 HEAD@{8}: rebase -i (squash): # Ceci est la combinaison de 2 commits.
6c9f6f4 HEAD@{9}: rebase -i (start): checkout HEAD~3
1a0f18f HEAD@{10}: rebase -i (finish): returning to refs/heads/master
1a0f18f HEAD@{11}: rebase -i (start): checkout HEAD~2
9b2e9c4 HEAD@{12}: rebase -i (finish): returning to refs/heads/master
9b2e9c4 HEAD@{13}: rebase -i (continue): Cinquième sur auth
1a0f18f HEAD@{14}: rebase -i (continue): Sixième
9abf3d6 HEAD@{15}: rebase -i (start): checkout HEAD~2
c010523 HEAD@{16}: rebase -i (finish): returning to refs/heads/master
c010523 HEAD@{17}: rebase -i (pick): Sixième
0854787 HEAD@{18}: rebase -i (pick): Cinquième sur auth
9abf3d6 HEAD@{19}: rebase -i (reword): Quatre sur auth
359ccca HEAD@{20}: rebase -i : avance rapide

Vous avez le raccourci du hash de chaque commit permettant de l'identifier.

Vous avez ensuite ce qu'on appelle le raccourci Reflog, nous y reviendrons ensuite.

Viens ensuite la commande effectuée.
Raccourcis RefLog

Les raccourcis RefLog sont tout simplement un moyen plus rapide de raisonner pour la navigation lors de récupération de données.

HEAD@{n} désigne le commit à n déplacement avant la position de HEAD actuelle (qui est HEAD@{0}) .

Par exemple :

git show HEAD@{2}

Cela affichera les détails du commit sur lequel était placé HEAD en avant-dernier.
La récupération de données avec git reflog

Comme nous l'avons vu, vous pouvez perdre la référence d'un ou plusieurs commits soit en faisant git reset, soit en supprimant une branche.

Ces références et les objets correspondants sont conservés par défaut pendant 90 jours localement.

Vous pouvez donc récupérer des commits perdus durant cette période.

Mais n'en abusez pas ! C'est complexe et cela ne doit servir qu'en cas de mauvaise manipulation.

Nous allons prendre un exemple, nous avons :

git log --oneline

Qui donne :

5b0b87f (HEAD -> master) Un commit
8ca8356 Maj test pour corriger x
f6538e3 trois
8cb6127 Second commit de test
bfc88f2 Premier commit

Mettons que nous supprimons de notre branche les deux dernières références par inadvertance en faisant :

git reset --hard HEAD~2

Nous avons donc maintenant :

f6538e3 (HEAD -> master) trois
8cb6127 Second commit de test
bfc88f2 Premier commit

Mais nous avons supprimé par inadvertance un commit de trop ! Nous voulions conserver le commit 8ca8356 ! Comment le récupérer et le remettre sur notre branche ?

Commençons par voir la situation :

git reflog

Seules ces premières lignes nous intéresse :

f6538e3 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~2
5b0b87f HEAD@{1}: rebase -i (finish): returning to refs/heads/master
5b0b87f HEAD@{2}: rebase -i (pick): Un commit
8ca8356 HEAD@{3}: commit: Maj test pour corriger x

Nous savons que HEAD pointe sur master qui pointe sur f6538e3.

Et nous savons qu'avant cette commande HEAD était sur 5b0b87f.

Nous voyons également que le commit avec le message de validation qui nous intéresse a pour hash 8ca8356.

Nous pouvons donc créer une branche avec ce commit :

git branch recup 8ca8356

Voilà, le commit n'est plus orphelin car une branche pointe sur celui-ci, il est maintenant sauvé et ne sera pas détruit par le nettoyage automatique de Git au bout de quelques mois.

Nous pouvons faire une fusion en avance rapide sur master pour le récupérer sur la branche principale :

git merge recup
git branch -d recup

Nous aurions également pu ne pas passer par une branche et directement faire une fusion en avance rapide en utilisant le hash du commit à récupérer :

git merge 8ca8356

Dans les deux cas nous avons maintenant :

git log --oneline

Donnant :

8ca8356 (HEAD -> master) Maj test pour corriger x
f6538e3 trois
8cb6127 Second commit de test
bfc88f2 Premier commit

Notre commit est récupéré sur la branche master !
La récupération de données avec git fsck

Une autre option est de passer par la commande git fsck qui permet de vérifier la validité et l'intégrité des objets Git stockée dans la base de données locale de Git.

En faisant la commande :

git fsck --full

Vous obtiendrez la liste de tous les objets Git qui ne sont pas référencés par d'autres objets et qui sont donc orphelins :

Vérification des répertoires d'objet: 100% (256/256), fait.
dangling commit 419bd1b3112c583dc2fa59fd55699bf2f1c9cbe5
dangling blob 53dd7296d1e4dce32c962843fa0ff835dc3bc962
dangling commit a63a83939525aef5a46fdb1bd21615748bbefd56
dangling blob ac28f91b8c7314ce04f3f037948520dcc7a88ff7
dangling blob c05368e3ed4516b84b84980fc8e7927f32a90092
dangling commit c413b3309f58ddd7375f238969f858d069d9c0a7

Vous devez ensuite lire les commits pour retrouver celui que vous voulez en faisant :

git show 419bd1b3112c583dc2fa59fd55699bf2f1c9cbe5

Faites cette commande pour chaque objet commit qui sont orphelins jusqu'à retrouver le bon.

Ensuite, faites une fusion en avance rapide ou créez une branche sur le commit comme nous l'avons vu précédemment. 