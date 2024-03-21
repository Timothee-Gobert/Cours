## Les branches

Introduction aux branches
Qu'est-ce qu'une branche ?

Nous avons vu que chaque commit (sauf le premier, appelé commit racine) a une référence à son commit parent, formant ainsi une ligne ou une chaîne de commits :

Parfois vous avez cependant besoin de travailler sans impacter cette ligne principale de développement, appelée branche master.
Les cas d'utilisation des branches

Des cas d'utilisation typique des branches sont principalement :

Premièrement, le développement d'une nouvelle fonctionnalité.

Deuxièmement, effectuer des tests de fonctionnalité sans risquer d'interférer avec la version en production.

Grâce aux branches, la ligne principale des commits n'est pas du tout impactée, ce qui permet d'avoir une branche propre généralement réservée à la mise en production : master.

Nous verrons en détails dans un chapitre dédié quel est le processus de développement classique avec une équipe qui utilise Git comme système de contrôle de versions.
Recommandations sur les branches

Les branches sont une des fonctionnalités principales de Git, comme elles n'ont aucun coût d'espace mémoire (il s'agit simplement d'un pointeur sur un commit), elles sont très recommandées.

En effet, elles ne prennent que 41 caractères (40 caractères du hash et un retour à la ligne), on dit donc que les branches en Git sont "gratuites", c'est-à-dire qu'elles ne coûtent quasiment aucune mémoire.

Ce qui prend de la mémoire en Git, comme vous le savez, c'est de sauvegarder un instantané du projet, et donc un commit. Il vaut donc mieux éviter de commit trop fréquemment : les commits sont conçus pour être des sauvegardes propres.

N'hésitez donc pas en revanche à vous servir de branches ! Sur un projet en équipe, il n'est pas rare d'avoir une dizaine de branches actives sur un projet.

Nous allons apprendre à bien gérer ses branches.

Lister et créer des branches avec git branch
Créer une branche

Créer une branche en Git est extrêmement simple, il suffit de faire : git branch nom.

Prenons notre exemple et expliquons le fonctionnement :

Nous sommes à la version correspondant au commit C et nous souhaitons démarrer un nouveau développement qui va prendre du temps, par exemple développer un système d'authentification.

Nous allons donc créer une branche en faisant :

git branch auth

Nous avons pour le moment simplement créé un nouveau pointeur sur le commit C, qui est notre nouvelle branche.

Nous pouvons vérifier simplement :

cat .git/refs/heads/auth
cat .git/refs/heads/master

Qui donneront toutes les deux le même hash, qui est celui du commit C dans notre exemple.

Ces deux références pointent donc exactement sur le même commit.

Attention ! Pour le moment HEAD pointe toujours sur master, nous ne nous sommes pas positionné sur la nouvelle branche auth, nous l'avons simplement créée.

D'ailleurs si vous faites :

git log

Vous verrez bien :

commit 9e633d56381d9f0335320f22ccf3f1562d1f80d8 (HEAD -> master, auth)

Il est bien indiqué que HEAD pointe sur master, et que master et auth pointe sur le commit.
Lister les branches

Avec Git il est très simple de visualiser toutes les branches locales d'un projet en une commande :

git branch

Si vous ne passer pas de nom, git branchva simplement lister tous les noms des branches. Dans notre exemple, nous aurons :

auth
* master

L'étoile indique que HEAD pointe actuellement sur la branche master.

Cela signifie que si vous faites des modifications et que vous les sauvegardées dans un commit, ce dernier se placera sur la branche master.

Vous pouvez également obtenir plus d'informations avec git branch -v.

git branch -v

Cette option vous donnera notamment le début du hash du commit sur lequel pointe chaque branche, ainsi que le début du message de validation, par exemple :

auth   d9d259c nouveau commit sur auth
* master 9e633d5 Premier commi

Nous allons voir comment basculer entre les branches dans la prochaine leçon !
Modifier le nom d'une branche

Modifier le nom d'une branche est très simple puisque c'est simplement le nom d'un fichier situé dans .git/refs/heads.

Il suffit de faire :

git branch -m “nouveauNom”

Supprimer une branche

Pour supprimer une branche il suffit de faire :

git branch -d branche

Si la branche contient des modifications qui n'ont pas été fusionnées (nous verrons en détails la fusion dans les prochaines leçons), il faudra forcer la suppression (mais attention tous les changements et tous les commits de la branche seront perdus) :

git branch -D branche

L'option -D est un raccourci pour --delete --force et signifie "forcer la suppression de la branche". 

Basculer entre les branches avec git checkout
Basculer sur une branche

Pour basculer sur une branche il faut utiliser la commande git checkout branche que nous avions déjà vue.

Ainsi en reprenant l'exemple précédent et en faisant :

git checkout auth

Nous basculons sur la branche auth et HEAD pointe alors sur la nouvelle branche :

Nous pouvons vérifier en faisant :

cat .git/HEAD

Qui nous donne dans notre exemple :

ref: refs/heads/auth

Même chose, si nous tapons git branch, nous aurons l'étoile qui indique que HEAD pointe bien sur la branche auth :

* auth
  master

Maintenant si nous faisons un nouveau commit que va t-il se passer ?

echo "Start auth" > auth.js
git commit -am "nouveau commit sur auth"

Nous avons simplement créé un nouveau fichier de test et avons indexé et sauvegardé en une seule commande grâce aux options -am de git commit.

Nous arrivons à ce point :

Nous avons effectué un nouveau commit mais sur la branche auth sur laquelle nous nous sommes placé avec git checkout auth.

Mettons maintenant que nous voulons retourner sur la branche principale pour faire un commit apportant des modifications réparant un bug.

Aucun problème ! Nous faisons :

git checkout master

Ensuite, nous effectuons les modifications de réparations, puis nous faisons :

git commit -am "fix: un problème x"

Nous en sommes là :

Nous avons donc fait un commit de réparation, E dans notre exemple, qui est sur la ligne principale.

Une fois notre branche master réparée, il nous suffit de retourner sur notre branche auth pour continuer à travailler sur notre fonctionnalité :

git checkout auth

Modification du répertoire de travail

Lorsque nous basculons sur une branche avec git checkout branche, nous ne faisons pas que déplacer HEAD !

Nous chargeons la version qui est pointée par la branche dans le répertoire de travail.

Nous allons voir ce que cela signifie exactement.
Cas de fichiers non suivis

Si nous sommes sur une branche et que nous avons des modifications non suivis par Git (par exemple des nouveaux fichiers), et que nous utilisons git checkout autrebranche, il ne se passera rien.

Les fichiers non suivis resteront présents dans le répertoire de travail.

C'est tout à fait logique : les fichiers étant non suivis, Git ne s'en préoccupent pas et les conserve donc en changeant de branche.
Cas de modifications sur des fichiers suivis

Si vous essayez en revanche de basculer sur une autre branche avec git checkoutalors que vous avez des modifications sur des fichiers suivis, vous aurez cette erreur :

error: Your local changes to the following files would be overwritten by checkout:

Please commit your changes or stash them before you switch branches.
Aborting

Git vous renvoie cette erreur car les modifications seront définitivement perdues si vous basculez sur l'autre branche. En effet, Git va charger la version du fichier correspondant au commit sur lequel point l'autre branche et va donc écraser la version de votre répertoire de travail contenant vos modifications !

Il faut donc sauvegarder ou annuler vos modifications ou alors comme nous le verrons les remiser avec git stash.
Créer une branche et y basculer en une commande

Il existe une commande qui permet de faire en une fois :

git branch feat1
git checkout feat1

Autrement, dit de créer une branche et d'y basculer HEAD immédiatement.

Cette commande est git checkout -b nomBranche :

git checkout -b feat1

Fusionner des branches avec git merge
Fusion avance rapide (fast forward merge)

Reprenons notre exemple de toute à l'heure, et imaginons que vous avez trois autres commits pour terminer la fonctionnalité d'authentification.

Elle est maintenant fonctionnelle et vous souhaitez fusionner vos changements dans la branche principale master afin de mettre cette nouvelle fonctionnalité en production.

Voici notre exemple :

Dans cet exemple nous n'avons pas fait de commit sur master depuis C et nous n'avons que des commits sur la branche auth.

Nous allons fusionner la branche auth dans la branche master pour récupérer les commits D, E et F sur la branche master.

Nous commençons par basculer HEAD sur master :

git checkout master

Ensuite nous demandons à Git de fusionner notre branche auth dans notre branche master :

git merge auth

Nous obtenons :

Updating 9e633d5..d9d259c
Fast-forward

Suivi de la liste des fichiers modifiés (qui revient dans notre exemple à un git diff entre le commit C et le commit F).

Nous obtenons ainsi :

Autrement dit nous avons simplement déplacé la branche master sur le commit F !.

Si la branche master n'a pas de commit en plus par rapport à la branche fusionnée dans celle-ci on appelle cela une fusion fast-forward car aucun commit de fusion n'est créé.

Autrement dit, retenez que si aucun commit n'est divergent entre les deux branches à fusionner, alors il s'agit d'une fusion avance rapide (ou fusion fast forward).

Le seul effet de la fusion dans ce cas est de déplacer master (et donc HEAD indirectement qui pointe sur master).

Une fois la fusion terminée, vous pouvez supprimer la branche pour garder une liste de branches propres :

git branch -d auth

Fusion à trois sources (three-way merge)

Reprenons notre exemple avant la fusion mais cette fois-ci, nous avons dû ajouter un correctif sur la branche principal pour vite le mettre en production.

Ce correctif ne modifie pas des fichiers modifiés également par les commits situés sur la branche auth, nous verrons dans la leçon suivante pourquoi cela importe.

Nous avons donc un commit E sur la branche master qui n'est pas sur la branche auth.

Nous avons donc des modifications divergentes entre les deux branches.

Voici la situation :

Nous faisons comme précédemment, nous commençons par basculer HEAD sur master :

git checkout master

Ensuite nous demandons à Git de fusionner notre branche auth dans notre branche master :

git merge auth

Mais cette fois-ci le résultat est différent !

Vous avez l'éditeur par défaut qui se lance et qui vous invite à spécifier le message de validation pour un nouveau commit. Celui-ci est pré-rempli avec :

Merge branch 'auth'
# Please enter a commit message to explain why this merge is necessary,
# especially if it merges an updated upstream into a topic branch.

Git vous demande de renseigner un message de validation pour expliquer la fusion.

Si vous validez, vous avez :

Merge made by the 'recursive' strategy.

La situation est maintenant la suivante :

Cela peut paraître complexe au premier abord mais nous allons tout détailler.

Dans cette situation il y a des commits divergents entre les branches auth et master. Git ne peut donc pas simplement déplacer la branche master sur le commit pointé par la branche auth comme dans le cas d'une fusion fast-forward.

En effet, dans ce cas, le commit H serait complètement perdu -et ses modifications aussi-.

Dans une fusion à trois sources, Git va prendre les deux commits pointés par les branches à fusionner ainsi que le commit qui est leur plus proche ancêtre commun.

Dans notre exemple, Git va donc prendre le commit G pour la branche auth et E pour la branche master, et leur plus proche ancêtre commun, le commit C. Ce sont les commits en violet dans notre exemple.

Git va utiliser ces trois commits pour créer un nouveau commit de fusion, le commit H dans notre exemple.

Les commits de fusion sont les seuls commits qui ont plusieurs commit parents.

Si vous faisiez :

git cat-file H

Avec H le hash du commit, vous auriez quelque chose comme :

tree 213bb9eebc6aad69424d62cd3c04bc9165c9be6c
parent fe6fc75fa3b625a90301ad21936448ca23d0421a
parent b9ef0fc3fa4f5450cdc5d9320330a8e41b4e3773
author dymafr <jean@gmail.com> 1584120243 +0100
committer dymafr <jean@gmail.com> 1584120243 +0100

Vous retrouveriez bien les deux références à deux commits parents.

Vous pouvez ensuite supprimer la branche fusionnée :

git branch -d auth

Nous serions alors dans cette situation :

Remarquez que les commits D, F et G restent sauvegardées !

En effet, ici nous pourrons donc toujours retourner sur les versions correspondants à ces commits qui sont définitivement sauvegardés !
Lister les branches fusionnées et non fusionnées

Pour lister les branches qui ont été fusionnées et peuvent être donc supprimées sans risque car les changements sont intégrés dans une autre branche vous pouvez faire :

git branch --merged

Il faut ignorer la branche avec * qui permet simplement de vous rappeler où est HEAD.

Vous pouvez également lister les branches qui vous restent à fusionner :

git branch --no-merged

Ces branches ont des modifications qui n'ont été fusionnées dans aucune branche. Elles ne peuvent être supprimées qu'avec git branch -D car leurs changements seront perdus définitivement. 

Résoudre des conflits de fusion
Les conflits de fusion

Reprenons exactement la même situation que tout à l'heure, mais cette fois-ci, le commit E et l'un des commits D, F ou G ont modifié les mêmes fichiers.

Autrement dit nous avons non seulement des commits divergents entre les branches, mais également des modifications conflictuelles sur les mêmes fichiers !

Git ne peut pas les résoudre seul ! Il a besoin que vous lui spécifiez pour chaque modification quelle version prendre : c'est ce qu'on appelle des conflits de fusion (ou merge conflicts).

Lorsque vous ferez :

git checkout master

Puis

git merge auth

Cette fois-ci, le résultat sera différent. Vous aurez par exemple :

Auto-merging fichier1.txt
CONFLICT (content): Merge conflict in fichier1.txt

Vous aurez une ligne par conflit indiquant dans quels fichiers se situent les conflits.

Il faudra alors manuellement résoudre les conflits un par un.

Si vous faites git status, Git vous signalera quels fichiers n'ont pas été fusionnés à cause des conflits, par exemple :

On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:      fichier1.txt

You have unmerged paths signifie que vous avez des fichiers non fusionnés et qu'il faut fix conflicts, c'est-à-dire traiter les conflits de fusion.

Une fois que les conflits sont résolus, Git vous signale qu'il faudra utiliser git add pour mark resolution, c'est-à-dire indexer la version où les conflits ont été résolus puis terminer la fusion en tapant git commit.

Heureusement, l'éditeur VS Code a un outil de résolution des conflits très efficaces !

Il suffit de cliquer sur les fichiers situés dans la catégorie MERGE CHANGES qui ont une petite icône C pour conflicts.

Ensuite, une fenêtre s'ouvre où vous voyez les conflits avec la version du fichier en vert correspondant à la branche sur laquelle pointe HEAD, appelée Current Change (dans notre exemple ce sera la version de la branche master) et la version du fichier en bleu correspondant à la version du commit de la branche fusionnée dans l'autre branche appelée Incoming Change :

Les signes < et = et > sont ajoutés par Git pour marquer respectivement : le début de la version HEAD, la fin de la version HEAD et le début de la version correspondant à l'autre branche fusionnée, et la fin de cette version.

Vous pouvez soit sélectionner la version de la branche sur laquelle pointe HEAD en appuyant sur Accept Current Change. (Dans notre exemple la branche master).

Soit sélectionner la version de la branche fusionnée dans la branche sur laquelle pointe HEAD en appuyant sur Accept Incoming Change. (Dans notre exemple la branche auth).

Soit accepter les deux changements (c'est très rarement le cas).

Une fois tous les conflits résolus, vous devrez indexer les fichiers, ce qui indique à Git que les conflits sont résolus.

Si vous faites maintenant git status vous aurez par exemple :

On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Git vous signale que vous êtes sur la branche master, que vous avez bien résolu tous les conflits mais que la fusion n'est pas terminée.

Enfin, vous devez git commit pour terminer la fusion.

Vous aurez alors l'éditeur qui s'ouvre :

Merge branch 'auth'

Conflicts:
    fichier1.txt
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
#	.git/MERGE_HEAD
# and try again.

Git détecte les conflits qui ont été résolus et prérempli le message de fusion.

Vous pouvez détailler les raisons des conflits et comment vous les avez résolus si besoin. Dans la plupart des cas, vous n'aurez qu'à valider le message par défaut.

Un nouveau commit de fusion est créé comme précédemment (commit H dans notre exemple).
Récapitulatif

Si vous avez des conflits lors d'une fusion :

1 - Ouvrez l'onglet Source Control de VS Code : cliquez sur les fichiers dans MERGE CHANGES et résolvez fichier par fichier les conflits.

2 - Indexez tous les fichiers sous MERGE CHANGES une fois les résolutions des conflits terminées : vous pouvez soit cliquer sur le bouton + dans VS Code soit utiliser git add fichier pour chaque fichier.

3 - Terminez la fusion en faisant git commit : en tapant git commit, la fusion est sauvegardée et un nouveau commit de fusion est créé comme précédemment (commit H dans notre exemple).
Abandonner une fusion

Parfois, vous rencontrerez des conflits sans vous y attendre, par exemple lorsque vous travaillerez avec des dépôts distants comme nous allons l'apprendre.

Vous ne voudrez peut être par résoudre les conflits immédiatement et fusionner plus tard. Dans ce cas, il est tout à fait possible d'abandonner la fusion et de la remettre à plus tard :

git merge --abort

abort signifie abandonner.

Les parties suivantes sont très avancées et nous vous conseillons d'y revenir plus tard.
Ignorer les conflits dû aux espaces

Imaginons que vous travaillez en équipe et qu'un utilisateur utilise les tabulations et un autre des espaces dans VS Code pour le formatage du code.

Lorsque vous aurez une fusion, vous rencontrerez de nombreux conflits à cause de ces divergences entre espaces et tabulations.

Dans ce cas vous pouvez utiliser une option spécifique permettant d'ignorer ces différences pour la fusion :

git merge -X ignore-all-space

L'option -X permet de passer une option de fusion pour la stratégie de fusion à adopter. Ici nous lui passons la stratégie de fusion ignore-all-space.

Cela permettra de fusionner sans avoir de conflits concernant les espaces ! Il ne restera plus qu'à s'entendre dans l'équipe sur quoi utiliser en les tabulations et les espaces une bonne fois pour toute !
Visualisation des versions pour les fusions à trois sources

Comment fait Git pour vérifier les différences entre les trois sources ? Pour rappel les trois sources sont : la version du commit pointé par la branche où l'on se trouve, la version du commit pointé par la branche que l'on souhaite fusionner, et la version du commit qui est le premier ancêtre commun entre les deux branches.

Nous pouvons bien visualiser cela en faisant :

git ls-files -u

Cette commande permet d'afficher des informations sur les fichiers dans l'index et dans le répertoire de travail.

L'option -u pour --unmerged permet de montrer les fichiers non fusionnés, qui sont donc en conflit.

Dans notre exemple nous aurions :

100644 58c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c 1	fichier1.txt
100644 b4de3947675361a7770d29b8982c407b0ec6b2a0 2	fichier1.txt
100644 98717958718606fc4a305ac962ee80a352539601 3	fichier1.txt

Remarquez les chiffres 1, 2 et 3 : nous avions expliquer que ce chiffre servez pour les fusions ! Nous y voici !

1 signifie qu'il s'agit de la version du fichier correspondant au commit qui est le premier ancêtre commun entre les deux branches. Elle a pour nom base.

2 signifie qu'il s'agit de la version du fichier correspondant au commit pointé par la branche où l'on se trouve (correspondant à HEAD). Elle a pour nom ours.

3 signifie qu'il s'agit de la version du fichier correspondant au commit pointé par la branche que l'on souhaite fusionner (correspondant au fichier .git/MERGE_HEAD). Elle a pour nom theirs.

Nous pouvons facilement visualiser les différences entre chaque avec les commandes :

git diff --ours
git diff --theirs
git diff --base

Nous ne détaillerons pas car il est bien mieux de visualiser ces différences en utilisant le gestionnaire de conflits de fusion de VS Code.
Défaire une fusion

Que se passe t-il dans notre exemple, si nous avons effectué la fusion par erreur et que nous ne voulions pas fusionner toute suite ?

Autrement dit nous voulons annuler le commit H de notre exemple et restaurer l'état avant la fusion à trois sources.

Il faut qu'il n'y ait pas eu de commit après H sinon cette solution ne fonctionnera pas.

Il faut considérer deux cas : soit vous avez push (nous étudierons en détails le travail avec un répertoire distant), soit vous avez la fusion uniquement localement.
La fusion n'est présente que localement

Si vous avez la fusion uniquement localement, c'est très simple.

Il suffit de faire :

git reset --hard HEAD~

Cela signifie déplace la branche pointée par HEAD sur le commit parent du commit actuel (ici H) et charge la version de ce commit dans le répertoire de travail et l'index.

Autrement dit nous arrivons à :

Mais comment Git sait quel parent sélectionner ? En effet H a deux commits parents : E et G. Comment sait-il qu'il faut déplacer la branche master sur E plutôt que sur G ?

C'est le rôle d'un fichier spécial : ORIG_HEAD.

Vous pouvez le voir en faisant :

cat .git/ORIG_HEAD

Il contient la référence vers le commit avant le démarrage de la fusion, soit le commit E. Git utilise ce fichier pour savoir où aller !
La fusion a été partagée

Dans ce cas, comme vu précédemment, reset est une très mauvaise option. Comme nous le verrons vous ne pourrez pas push sans force ce qui est une extrêmement mauvaise pratique et peut générer des catastrophes.

Dans ce cas il faut faire :

git revert -m 1 HEAD

Cette commande signifie défait les changements dans un nouveau commit de revert.

L'option -m signifie --mainline et permet de spécifier quelle branche sélectionner pour déterminer le commit parent (de H dans notre exemple) dont nous souhaitons récupérer la version des fichiers (commit E dans notre cas).

1 signifie la branche sur laquelle nous sommes, autrement dit la branche master.

Nous passons à revert le commit à défaire, HEAD, qui est le commit actuel ici H.

H^ est le commit de revert qui va défaire tous les changements apportés par la branche auth et retourner donc dans la version des fichiers telle qu'elle était au commit E.

Attention dans ce cas, la branche auth est quand même considérée comme ayant été fusionnée avec master et donc si vous essayiez de la fusionner plus tard dans master vous aurez :

Already up-to-date.

Il faudra revert le commit H^ pour refusionner plus tard auth dans master en faisant :

git revert H^

Où H^ est bien sûr le hash du commit.

Cela revient à défaire le commit qui avait défait la fusion. Si vous avez fait d'autres commit sur la branche auth de notre exemple, il faudra en plus faire :

git merge auth

Cela fusionnera tous les commits de la branche auth après G qui est le dernier commit considéré par Git comme ayant déjà été fusionné.

Morale de l'histoire : il faut faire attention lorsque vous faites un push !
Fusionner par préférence

Nous avons vu deux types de fusions jusqu'à maintenant : la fusion avance rapide (fast forward) et la fusion à trois source avec stratégie récursive (three-way merge by recursive strategy).

Il existe d'autres options de fusions avancées avec Git, et nous allons en voir une autre ensemble : la fusion par préférence. Il s'agit d'une option passée à une fusion à trois source avec stratégie récursive.

La fusion par préférence permet de dire à Git de préférer toujours une branche donnée lorsqu'il rencontre des conflits de fusion. Lorsque des conflits de fusion ne sont pas rencontrées, les changements apportées par l'autre branche sont fusionnés .

Vous pouvez ainsi privilégier la branche sur laquelle vous êtes appelée ours ou la branche que vous fusionnez dans la branche actuelle appelée theirs.

Pour privilégier toujours la branche à fusionner dans la branche actuelle, auth dans notre exemple, il faut faire :

git merge auth -X theirs

Si nous voulons au contraire privilégiez la version des fichiers de master lors des conflits, il faut faire :

git merge auth -X ours

Attention ! Il ne faut surtout pas confondre -X avec -s !!

Avec -X vous passez une option de fusion à la stratégie de fusion utilisée par défaut, ici la stratégie trois sources récursives.

Avec -s vous spécifiez la stratégie à adopter, autrement dit vous n'utilisez plus la stratégie récursive.

Par exemple si vous faites :

git merge auth -s ours

Vous utilisez la stratégie de fusion ours qui a un effet totalement différent ! Elle permet de ne garder que les changements de la branche sur laquelle pointe HEAD (ici master). Il n'y a donc aucun conflit, mais cela revient à écarter totalement les modifications de la branche à fusionner.

Autrement dit n'utilisez jamais -s car changer la stratégie de fusion ne vous sera jamais utile.

En revanche, si vous êtes sûr que vous voulez garder la version de HEAD ou au contraire la version de l'autre branche en cas de conflit, vous pouvez utiliser -X ours ou -X theirs. 

Remiser ses changements avec git stash
Remettre des changements à plus tard

Parfois, vous êtes en plein travail sur une branche dans un état complètement intermédiaire mais vous devez aller sur une autre branche pour faire une réparation rapide puis revenir à votre travail.

D'autre fois, vous vous serez tromper de branches et vous voudrez conserver vos changements mais les appliquer sur une autre branche.

Bien sûr dans les deux cas, vous ne devez pas avoir fait de commit. Il faut alors utiliser git stash.

git stash permet d'enregistrer les modifications présentes dans votre répertoire et travail et votre index pour les réappliquer plus tard.

Prenons un exemple où vous avez des changements sur votre répertoire de travail et / ou votre index et que vous tapez :

git stash

Vous aurez quelque chose comme :

Copie de travail et état de l'index sauvegardés dans WIP on master: 8cb6127 Second commit de test

WIP signifie work in progress, ou travail en cours.

Si vous avez Git en anglais, vous aurez :

Saved working directory and index state WIP on master: 8cb6127 Second commit de test

Si vous faites un git status vous aurez maintenant :

Sur la branche master
rien à valider, la copie de travail est propre

Cela ressemble à la situation après un commit, sauf qu'il n'y a aucune sauvegarde et donc aucun commit de fait.
Sauvegarder temporairement les fichiers non suivis

Par défaut, git stash ne va pas sauvegarder temporairement les fichiers non suivis par Git, comme pour les autres commandes en Git. Ces fichiers sont ignorés.

Si vous voulez cependant les inclure dans la sauvegarde temporaire, il suffit de faire :

git stash -u

L'option -u est un raccourci pour --include-untracked. Autrement dit, inclure les fichiers non suivis.
Ne pas sauvegarder temporairement les fichiers indexés

Parfois vous voudrez sauvegarder temporairement les modifications dans votre répertoire de travail mais pas les modifications indexées.

Par exemple si vous voulez faire un commit avec les modifications propres (que vous avez donc indexées) et ne sauvegarder temporairement que les modifications qui sont dans votre répertoire de travail pour terminer votre travail plus tard. Dans ce cas il faut faire :

git stash --keep-index

Visualiser la liste des stash

Il est possible de sauvegarder temporairement autant de stash que nécessaire.

Vous pouvez lister toutes ces sauvegardes temporaires en faisant :

git stash list

Vous aurez quelque chose comme :

stash@{0}: WIP on master: 8cb6127 Second commit de test
stash@{1}: WIP on master: 8cb6127 Second commit de test

Ici dans notre exemple, nous avons fait deux sauvegardes temporaires, c'est-à-dire deux git stash sur la même branche et avec la version correspondant au même commit.

Mais vous pouvez avoir des stash sur n'importe quelle branche et n'importe quelle version.
Appliquer un stash pour restaurer les changements de la sauvegarde temporaire

Vous pouvez appliquer un stash sur n'importe quelle branche, y compris si il y a des modifications dans le répertoire de travail ou l'index.

Dans ce dernier cas, Git vous invitera à résoudre les conflits éventuels si des modifications concernent les mêmes lignes.

Par défaut, Git n'appliquera pas l'index.
Appliquer un stash en le conservant

Pour appliquer la dernière sauvegarde temporaire sur la branche et sur le commit sur lequel vous vous trouvez (sur lequel HEAD est positionné), c'est-à-dire la 0 dans notre exemple, il suffit de faire :

git stash apply

Les modifications enregistrées dans la stash seront appliquées sur le répertoire de travail ainsi que sur l'index.

Pour appliquer une sauvegarde spécifique vous pouvez utiliser le numéro du stash :

git stash apply stash@{1}

Appliquer un stash en le supprimant

Vous pouvez également appliquer une sauvegarde temporaire puis la supprimer immédiatement avec :

git stash pop stash@{1}

Cela peut être utile si vous utilisez souvent les stash et que vous ne voulez pas en conserver trop au risque de vous y perdre.
Appliquer également l'index d'un stash

Comme nous l'avons vu, par défaut lors de l'application d'une sauvegarde temporaire, l'index n'est pas appliqué.

Pour l'appliquer il faut utiliser l'option --index avec l'une des deux commandes vu précédemment.

Par exemple :

git stash apply --index

Supprimer un stash

Il est possible de supprimer un stash lorsque vous l'avez appliqué avec git apply ou si vous n'avez plus besoin de cette sauvegarde temporaire.

Dans ce cas il suffit de faire par exemple :

git stash drop stash@{1}

Visualiser les changements contenus dans un stash

Parfois vous ne vous souviendrez plus des modifications contenues dans une sauvegarde temporaire, vous pourrez faire la commande suivante pour les visualiser :

git stash show -p stash@{0}

En remplaçant 0 par le numéro du stash dont vous souhaitez visualiser les modifications.
Créer une branche depuis une sauvegarde temporaire

Parfois vous voudrez récupérer le travail sur une sauvegarde temporaire et le continuer sur une branche.

Par exemple si vous avez fait un test, êtes parti sur autre chose entre temps, et voulez finalement faire de votre test une fonctionnalité.

Dans ce cas, il suffit de faire :

git stash branch nom

Où nom est le nom de la branche que vous créez. 

Visualiser les branches
Utilisation de git log

Il existe une option de git log permettant de visualiser les branches : --graph.

En tapant cette commande :

git log --graph --all --oneline

Vous aurez par exemple :

Cela vous donne en fait les relations entre les branches, le début des hash des commits, la position de HEAD et des branches et le début des messages de validation.
L'extension Git Graph

L'extension Git Graph est excellente pour la visualisation des branches. Vous pouvez le voir comme un git log --graph amélioré.

