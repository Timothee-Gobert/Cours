## Répertoire distant et Gitlab

Introduction aux répertoires distants et à Gitlab
Les répertoires distants

Un répertoire ou dépôt distant Git permet de rendre accessible en permanence, pour toute votre équipe, l'historique d'un projet.

Ils permettent ce qu'on appelle le développement distribué.

Vous pouvez créer votre propre serveur Git, mais nous ne le verrons pas dans ce cours pour trois raisons :

Premièrement, c'est très long et complexe, et personne ne le fait. C'est d'ailleurs pour cette raison que Github, Gitlab et Bitbucket valent si cher. Créer et maintenir un serveur distant offrant les fonctionnalités nécessaires à un projet moderne est un métier en soi.

Deuxièmement, Gitlab offre un nombre hallucinant de fonctionnalités gratuitement et en open source.

Troisièmement, si vous tenez absolument à héberger vos source que sur vos serveurs, vous pouvez installer Gitlab Community Edition.
Le choix de Gitlab

Nous choisissons de vous montrer Gitlab plutôt que Github pour deux raisons.

Premièrement, Github a été racheté par Microsoft en 2018.

Deuxièmement, Gitlab a plus de fonctionnalités offertes gratuitement, alors qu'il faut rapidement s'abonner avec Github pour avoir accès aux fonctionnalités intéressantes.

Gitlab ne risque pas de disparaître, l'entreprise a déjà levé plus de 440 millions de dollars et est en forte croissance. Elle est utilisée par énorméments d'entreprises et d'institutions, comme par exemple la NASA ou le CERN.

Commencer par créer un compte dès maintenant sur Gitlab. 

Cloner et gérer un répertoire distant
Cloner un répertoire distant

Pour créer une copie d'un répertoire distant sur votre ordinateur, il faut utiliser la commande git clone.

Vous tester localement en copiant le dépôt disant du projet Linux.

Placez vous dans un dossier pour vos projets et faites :

git clone https://github.com/torvalds/linux.git

Vous allez vous retrouvez avec une copie du noyau Linux sur votre ordinateur !
Créer un projet sur Gitlab

Une fois votre compte créé et votre email validé, allez tout d'abord sur l'icône de profil tout en haut à droite.

Cliquez ensuite sur Settings.

Dans la colonne de gauche, vers le bas, cliquez sur Preferences.

Dans Localization sélectionnez language Français et First day of the week Monday puis sauvegardez.

En haut à gauche cliquez sur Projets > Vos projets.

Ensuite cliquez sur Create Project.

Donnez n'importe quel nom au projet, laissez la visibilité par défaut à privé et cliquez sur create project.

Vous arriverez alors sur une nouvelle interface Project overview > Détails.

Gitlab vous indiquera que Le dépôt de ce projet est vide. Ce qui est tout à fait normal !
Cloner votre projet vide

Sur votre ordinateur, ouvrez un projet vide et suivez les instructions données par Gitlab dans Create a new repository.

Par exemple :

git clone https://gitlab.com/erwan.vallee/test.git
cd test
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

Lorsque vous exécuterez la première instruction, Gitlab vous demandera votre mot de passe. Ce qui est tout à fait normal car nous avons créé le projet en visibilité privée : seuls les membres autorisés pourront y accéder.

La deuxième commande vous permet de vous placer dans le dossier que vous venez de cloner, qui est vide en apparence mais contient le dossier caché .git.

La troisième commande crée un fichier vide au format markdown.

Ensuite nous indexons le fichier et nous créons un premier commit.

Enfin, nous utilisons une commande spéciale que nous étudierons en détails, git push -u qui va envoyer notre commit sur le dépôt distant. Il faudra bien sûr entrer de nouveau notre mot de passe.

Nous allons afficher le fichier de configuration de Git.

Vous pouvez remarquer que Git a enregistré l'URL du dépôt distant pour que vous n'avez pas à la taper à chaque fois :

cat .git/config

Donne en effet :

[core]
  repositoryformatversion = 0
  filemode = true
  bare = false
  logallrefupdates = true
[remote "origin"]
  url = https://gitlab.com/erwan.vallee/test.git
  fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
  remote = origin
  merge = refs/heads/master

Remarquez dans [remote "origin"] la propriété url.
Les références distantes

Les références distantes sont tous les pointeurs vers les objets Git de votre dépôt distant : les branches, les tags (que nous verrons plus tard) etc.

Vous pouvez lister l'ensemble des pointeurs distants avec :

git ls-remote

Dans notre exemple, avec notre répertoire qui a un seul commit et une seule branche cela donne :

From https://gitlab.com/erwan.vallee/test.git
0ffc52c48d6c5b21afa549829475a11508249793	HEAD
0ffc52c48d6c5b21afa549829475a11508249793	refs/heads/master

Nous pouvons retrouver ces références dans .git/refs/remotes/ :

cat .git/refs/remotes/origin/master

Donne :

0ffc52c48d6c5b21afa549829475a11508249793

Ces références distantes ne sont pas modifiables localement, elles sont gérées automatiquement par le serveur Git.

Les branches distantes ont la forme suivante : (nom-dépôt-distant)/(nom-branche).

Par défaut le nom donné localement au dépôt distant est origin et nous vous déconseillons de le modifier, même si c'est très facile comme nous le verrons.

La branche principale pour le dépôt distant est donc par défaut : origin/master.

Après un git clone et en ayant fait un seul commit que nous avons envoyé, nous sommes donc dans la situation suivante :
La commande git remote

La commande git remote vous sera peu utile car lorsque vous clonerez un dépôt distant, la configuration se fera automatiquement.

Nous allons quand même voir les commandes principales.

git remote add origin URL|SSH permet d'ajouter un répertoire distant en lui donnant le nom par défaut origin.

git remote -v permet de lister les répertoires distants.

git remote rm nom permet de supprimer le répertoire distant donné.

git remote rename ancienNom nom permet de remplacer le nom donné localement au répertoire distant.
Ajouter des membres à votre projet

Pour ajouter d'autres membres de votre équipe sur un projet vous devez d'abord sélectionner le projet dans Projets > Vos projets en haut à gauche.

Ensuite, dans la colonne de gauche tout en bas allez dans Paramètres.

Cliquez sur Membres.

Vous pouvez choisir qui inviter en utilisant soit leur identifiant Gitlab, soit leur adresse email.

Cliquez sur en savoir plus si vous souhaitez le détail des permissions.

En général vous donnerez le statut de developer aux membres de votre équipe et le statut de maintainer (qui peuvent quasiment tout faire) aux lead devs. 

Mise à jour des pointeurs distants avec git fetch
La divergence des branches

Lorsque vous travaillez en équipe, les branches locales et distantes vont constamment diverger ! Et c'est tout à fait normal, Git est justement prévu pour cela, c'est un gestionnaire de version décentralisé.

Vous aurez donc par exemple :

Dans cette situation vous avez un commit A localement sur lequel pointe votre branche locale master.

Un ou plusieurs autres membres de votre équipe ont continué à travaillé et ont publié de nouveaux changements sur la branche distante origin/master.

Vous avez donc deux commits en retard !

Par défaut vous ne verrez pas cette divergence, localement vous aurez toujours :

En effet, Git a besoin de faire une requête sur le réseau pour connaitre l'état du dépôt distant !

C'est justement le rôle de la commande git fetch.
La commande git fetch

git fetch met à jour localement les branches distantes.

En détails, git fetch va télécharger les pointeurs des branches et des tags distants ainsi que tous les objets (commits, blobs et arbres) nécessaires.

Par défaut, si aucun dépôt distant n'est précisé, git fetch utiliera origin.

git fetch est seulement une mise à jour des branches locales, aucun changement ne sera appliqué ni à votre répertoire de travail, ni à votre index, ni à votre historique local.

Il s'agit donc d'une commande de mise à jour locale sans aucun danger.

Après cette commande vous pouvez inspecter les branches distantes :

git branch -r

Qui donnera dans notre exemple :

origin/master

Vous pouvez également voir l'ensemble des références distantes pour origin avec :

ls .git/refs/remotes/origin/

Dans notre exemple nous avons seulement une référence distante :

master

Visualiser l'état d'une branche distante

Vous pouvez tout à fait mettre votre répertoire de travail à la version d'une branche distante. Dans ce cas, vous êtes en mode HEAD détaché et vous ne devez pas faire de modifications.

Comme nous l'avons vu les branches distantes ne peuvent en effet pas être modifiée localement.

Dans notre exemple nous pouvons faire :

git checkout origin/master

Ce qui donne :

Note : extraction de 'origin/master'.

Vous êtes dans l'état « HEAD détachée ». Vous pouvez visiter, faire des modifications
expérimentales et les valider. Il vous suffit de faire une autre extraction pour
abandonner les commits que vous faites dans cet état sans impacter les autres branches

Si vous voulez créer une nouvelle branche pour conserver les commits que vous créez,
il vous suffit d'utiliser « checkout -b » (maintenant ou plus tard) comme ceci :

  git checkout -b <nom-de-la-nouvelle-branche>

HEAD est maintenant sur 0ffc52c add README

Git vous prévient que vous êtes dans l'état HEAD détaché et que si vous faites des modifications elles seront perdues sauf si vous créez une branche.

Nous sommes en fait dans cet état :
Mettre à jour qu'une branche

Pour mettre à jour seulement une branche distante vous pouvez faire :

git fetch origin branche

Seuls les objets de cette branche distante seront alors téléchargés.

Cela peut être utile si vous êtes dans une équipe importante sur un grand projet (plusieurs dizaines de développeurs) et que vous ne travaillez que sur quelques branches.

En effet, si le projet est important et qu'il y a des dizaines de branches, vous ne voudrez pas forcément charger tous les objets de toutes ces branches à chaque mise à jour locale.

Mais la plupart du temps, lorsque vous n'êtes que moins d'une dizaine dans votre équipe, avec quelques branches il suffit de faire git fetch.
Exemple standard d'intégration des branches distantes

Prenons un exemple fréquent, vous êtes dans cette situation :

Très classiquement, vous avez une branche de production master, une branche de développement develop et une branche pour chaque fonctionnalité créée depuis develop.

Après un git fetch, vous aurez quelque chose comme :

remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Dépaquetage des objets: 100% (6/6), fait.
Depuis https://gitlab.com/erwan.vallee/test
  * [nouvelle branche] auth       -> origin/auth
  * [nouvelle branche] develop    -> origin/develop

Git détaille ce qu'il a téléchargé : le nombre d'objets nécessaires et les nouvelles références.

Ici il nous dit que nous avons deux branches distantes nouvelles.

Nous pouvons maintenant lister les branches distantes :

git branch -r

Ce qui donnera :

origin/auth
origin/develop
origin/master

Créer une branche locale au même point qu'une branche distante

Maintenant nous voulons travailler sur la fonctionnalité d'authentification. Comment devons nous procéder ?

git checkout develop

Oui, mais nous n'avons pas de branche locale develop ! Nous n'avons qu'une branche distante origin/develop !

C'est vrai, mais Git est suffisamment intelligent pour savoir que si vous faites un checkout sur une branche ayant le même nom qu'une branche distante vous voulez en fait créer une branche locale du même nom qui suivra la branche distante. C'est d'ailleurs ce qu'il vous indique :

La branche 'develop' est paramétrée pour suivre la branche distante 'develop' depuis 'origin'.
Basculement sur la nouvelle branche 'develop'

Si vous faites :

git log --oneline

Vous aurez l'indication que HEAD pointe bien sur la branche develop qui pointe bien sur le même commit que la branche distante origin/develop :

23fef78 (HEAD -> develop, origin/develop) Update README.md

Mettre à jour une branche locale

Maintenant, toujours sur le même exemple, nous souhaitons mettre à jour notre branche master au même point que origin/master.

Rien de plus simple, nous nous plaçons sur la branche locale master, si ce n'est pas déjà fait :

git checkout master

Ensuite nous vérifions les changements apportés par la branche distante origin/master :

git log origin/master

Dans notre exemple nous aurions par exemple :

commit 51f9e69592ead2f104c56d670d2237f202cadb1a (origin/master)
Author: Jean Paul <jean@hotmail.com>
Date:   Fri Mar 20 14:47:33 2020 +0000

    Update README.md

commit 68ed863c50c66c3f25757b7a126d7262b0efa6d2
Author: Jean Paul <jean@hotmail.com>
Date:   Fri Mar 20 14:46:04 2020 +0000

    Update README.md

commit 0ffc52c48d6c5b21afa549829475a11508249793 (master)
Author: Erwan Vallée <contact@dyma.fr>
Date:   Thu Mar 19 18:27:48 2020 +0100

    add README

Nous avons bien deux commits postérieurs effectués par Jean Paul.

Nous pouvons voir les changements pour chaque commit en allant dans VS Code sur GitGraph ou en faisant :

git diff HEAD origin/master

Cela vous donnera la différence entre la dernière version sur origin/master et votre version sur votre branche master locale sur laquelle pointe HEAD.

Vous pouvez ensuite intégrer les changements en rebasant ou en fusionnant comme nous l'avons appris.

Dans la plupart des cas, si vous nous trouvons sur la même branche locale et distante nous privilégierons un rebasage, dans le cas contraire une fusion. Cela permet de garder un historique propre, nous détaillerons tout cela dans le chapitre suivant.

Dans notre cas nous faisons donc :

git rebase origin/master

Ce qui aboutit à :

git log

Donnant :

commit 51f9e69592ead2f104c56d670d2237f202cadb1a (HEAD -> master, origin/master)
Author: Erwan Vallée <erwan1309@hotmail.com>
Date:   Fri Mar 20 14:47:33 2020 +0000

    Update README.md

commit 68ed863c50c66c3f25757b7a126d7262b0efa6d2
Author: Erwan Vallée <erwan1309@hotmail.com>
Date:   Fri Mar 20 14:46:04 2020 +0000

    Update README.md

commit 0ffc52c48d6c5b21afa549829475a11508249793
Author: Erwan Vallée <contact@dyma.fr>
Date:   Thu Mar 19 18:27:48 2020 +0100

    add README

Nous avons donc bien mis à jour notre branche locale :
Détails sur fetch

Dans le fichier .git/config nous avons la ligne suivante :

[remote "origin"]
  fetch = +refs/heads/*:refs/remotes/origin/*

Vous n'aurez jamais à la modifier, mais nous allons simplement l'expliquer.

Cela signifie que pour le répertoire distant origin, nous allons appliquer les règles suivantes pour fetch :

Récupère toutes les branches distantes (car +refs/heads/* signifie toutes les références dans heads, c'est-à-dire toutes les branches).

Met à jours toutes les branches de suivi correspondantes localement (c'est-à-dire toutes les branches dans refs/remotes/origin/).

Autrement dit, cette configuration permet de mettre à jour toutes les branches distantes localement lorsque vous ne précisez rien en faisant git fetch. 

Le raccourci git pull
La commande git pull

La commande git pull est le raccourci de deux commandes git fetch puis git merge.

Lorsque vous faites la commande, l'éditeur s'ouvre pour avoir l'opportunité de modifier le message du commit de fusion.

Par exemple, si nous sommes dans cette situation :

Et que nous faisons :

git pull

Nous aurons :

Le commit G étant un commit de fusion.

Si il y a des conflits, vous devrez bien sûr les résoudre.

Si vous ne souhaitez pas résoudre les conflits de fusion immédiatement vous pouvez annuler la fusion :

git reset --merge

L'option --rebase

Vous pouvez également utiliser l'option rebase pour effectuer un rebasage plutôt qu'une fusion après le fetch :

git pull --rebase

Vous avez également l'option dans la partie Source Control de VS Code, en cliquant sur les trois points pour plus d'options, elle est intitulée Pull (Rebase).

Nous reprenons la situation initiale, mais cette fois-ci nous utilisons le rebasage.

Notez bien l'ordre des commits !

Nous nous plaçons sur origin/master puis nous ajoutons les commits sur master après le commit ancêtre commun, donc les commits divergents.

Ici nous n'avons que le commit D.

Notez que si vous avez des conflits vous aurez le message suivant :

Username for 'https://gitlab.com': erwan.vallee
Password for 'https://erwan.vallee@gitlab.com':
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 0
Dépaquetage des objets: 100% (9/9), fait.
Depuis https://gitlab.com/erwan.vallee/gittest
    8df0cb2..46f33f6  master     -> origin/master
Rembobinage préalable de head pour pouvoir rejouer votre travail par-dessus...
Application de  master D
Utilisation de l'information de l'index pour reconstruire un arbre de base...
M	README.md
Retour à un patch de la base et fusion à 3 points...
Fusion automatique de README.md
CONFLIT (contenu) : Conflit de fusion dans README.md
error: Échec d'intégration des modifications.
le patch a échoué à 0001 master D
Utilisez 'git am --show-current-patch' pour visualiser le patch en échec

Résolvez tous les conflits manuellement, marquez-les comme résolus avec
"git add/rm <fichier en conflit>", puis lancez "git rebase --continue".
Si vous préférez sauter ce commit, lancez "git rebase --skip". Pour arrêter
et revenir à l'état antérieur à la commande, lancez "git rebase --abort".

Git vous explique bien ce qu'il se passe. Le commit sur la branche locale master placé après la branche distante origin/master modifie le même fichier, ici REAME.md.

Il faut ouvrir VS Code, aller dans l'onglet Source Control puis résoudre le conflit en choisissant une version des changements.

Ensuite, il faut valider les changements avec git add ou + sur le fichier qui contenait des conflits.

Enfin, il faut faire git rebase --continue. Et vous aurez alors :

Application de  master D

Confirmant la fin du rebasage.

Pour l'instant notre historique n'a pas été envoyé sur le dépôt distant.

Un git log ou une visite sur GitGraph confirmera la situation finale :

commit 62c7cecae29938c52f8b0b69fbffb5c62be3c49d (HEAD -> master)
Author: Erwan Vallée <jean@gmail.com>
Date:   Sat Mar 21 13:42:55 2020 +0100

    master D

commit 46f33f60b503fd59b590649decc4b3e48704f187 (origin/master)
Author: Erwan Vallée <paul@hotmail.com>
Date:   Sat Mar 21 12:43:41 2020 +0000

    F origin

commit 1d14920507b9b8df80d3e50e3233f886aeb70d8c
Author: Erwan Vallée <paul@hotmail.com>
Date:   Sat Mar 21 12:43:30 2020 +0000

    E origin

commit 6d2a1bdad936302db9557a58d1cc2aae340a99cc
Author: Erwan Vallée <paul@hotmail.com>
Date:   Sat Mar 21 12:43:16 2020 +0000

    C origin

commit 8df0cb2d537a0f6d71bb4aedc4b217c53395f6d9
Author: Erwan Vallée <jean@gmail.com>
Date:   Sat Mar 21 13:41:55 2020 +0100

    B master

commit 8df0cb2d537a0f6d71bb4aedc4b217c53395f6d9
Author: Erwan Vallée <jean@gmail.com>
Date:   Sat Mar 21 13:41:51 2020 +0100

    A master

Mettre à jour le dépôt distant avec git push
La commande git push

La commande git push permet de mettre à jour les références sur le dépôt distant.

Autrement dit, elle permet d'envoyer vos changements pour mettre à jour le dépôt distant.

Ce que vous devez bien comprendre, c'est qu vos branches locales ne sont pas automatiquement synchronisées avec votre dépôt distant. Il faut utiliser git push.

Comme pour les autres commandes, si aucun dépôt distant n'est précisé, origin sera utilisé.

Pour ce qui est de la branche, par défaut, c'est la branche sur laquelle est positionnée HEAD qui est push vers la branche distante correspondante.

Donc par défaut, si vous êtes sur master :

git push

Équivaut à :

git push origin master

Mise à jour en avance rapide

Pour ne pas perdre aucune étape de l'historique des modifications et permettre à tout le monde de travailler avec le même dépôt distant, il existe une règle : seuls les commits résultants en une fusion en avance rapide peuvent être push.

Prenons un exemple :

Ici aucun problème ! Si nous faisons git push, comme il n'y a pas de commit divergeant sur origin/master, alors les commits C et D peuvent être appliqués avec une fusion en avance rapide, et aucun risque de perdre le moindre changement.

Après un git push nous aurons simplement :
Impossibilité de mise à jour

En revanche, si vous êtes dans cette situation :

Vous aurez cette erreur si vous faites git push :

Username for 'https://gitlab.com': erwan.vallee
Password for 'https://erwan.vallee@gitlab.com':
To https://gitlab.com/erwan.vallee/gittest.git
  ! [rejected]        master -> master (non-fast-forward)
error: impossible de pousser des références vers 'https://gitlab.com/erwan.vallee/gittest.git'
astuce: Les mises à jour ont été rejetées car la pointe de la branche courante est derrière
astuce: son homologue distant. Intégrez les changements distants (par exemple 'git pull ...')
astuce: avant de pousser à nouveau.
astuce: Voir la 'Note à propos des avances rapides' dans 'git push --help' pour plus d'information.

Vous devez avant de git push récupérer les changements avec git fetch ou git pull et rebaser ou fusionner. Une fois que c'est fait vous pourrez git push.

Il y a un moyen de passer outre cette limitation :

git push  --force

Cela force la mise à jour du répertoire disant. Par défaut les branches master et develop sur Gitlab sont protégées et cette commande est interdite (vous pouvez modifier cette interdiction mais nous le déconseillons).

Et c'est très bien ainsi, forcer le push signifie détruire des changements et modifier irrémédiablement l'historique distant, donc également pour votre équipe.

Si vous avez push de mauvais commits utilisez revert, mais n'utilisez jamais force, c'est une extrêmement mauvaise pratique !
Supprimer une branche distante

Vous pouvez supprimer une branche distante en faisant :

git push origin :branche

Ou :

git push --delete branche

Nous ne nous attardons pas car vous pouvez supprimer une branche depuis Gitlab et c'est comme cela que vous ferez dans la majorité des cas.

Vous supprimerez des branches distantes dans deux cas : le travail est abandonné et la branche devient inutile, ou la branche a été fusionnée / rebasée et donc il ne sert à rien de maintenir la référence.

C'est les mêmes cas que pour la suppression d'une branche locale. Simplement quand vous supprimez une branche localement, la branche distante correspondante origin/branche n'est pas supprimée. 

Liens entre branches locales et branches distantes
Pousser une nouvelle branche

Si vous créez une branche locale, par exemple auth :

git checkout -b auth

Et que vous faites un ou plusieurs commits puis que vous essayez de faire git push vous aurez :

git push
fatal: La branche courante auth n'a pas de branche amont.
Pour pousser la branche courante et définir la distante comme amont, utilisez

    git push --set-upstream origin auth

Cela signifie que Git ne sait pas vers quelle branche distante pousser ces commits. Par défaut, il ne va pas présupposer que vous voulez créer une branche distante origin/auth et la synchroniser.

Vous pouvez le déclarer en faisant :

git push origin auth

Qui est un raccourci pour :

git push origin auth:auth

Cela signifie prend ma branche locale auth et envoie la sur origin pour créer la branche distante auth.

Vous pourriez ainsi modifier le nom de la branche sur le dépôt distant, par exemple pour l'appeler authentification :

git push origin auth:authentification

Mais c'est assez déconseillé car vous pourriez vous y perdre.

Et cela fonctionnera, car vous précisez où Git doit push :

Username for 'https://gitlab.com': erwan.vallee
Password for 'https://erwan.vallee@gitlab.com':
Décompte des objets: 8, fait.
Delta compression using up to 12 threads.
Compression des objets: 100% (4/4), fait.
Écriture des objets: 100% (8/8), 723 bytes | 361.00 KiB/s, fait.
Total 8 (delta 0), reused 0 (delta 0)
remote:
remote: To create a merge request for auth, visit:
remote:   https://gitlab.com/erwan.vallee/gittest/-/merge_requests/new?merge_request%5Bsource_branch%5D=auth
remote:
To https://gitlab.com/erwan.vallee/gittest.git
  * [new branch]      auth -> auth

Cependant vous devrez le faire à chaque fois ! Ce qui n'est pas pratique !

Il vaut mieux paramétrer votre branche locale pour dire à Git que la branche locale auth doit suivre la branche distante origin/auth.

Pour ce faire vous pouvez faire :

git push --set-upstream origin auth

Ou l'option -u qui est un alias (raccourci) :

git push -u origin auth

Si vous voulez faire l'association branche locale / branche distante mais pas push toute suite, vous pouvez faire, si HEAD pointe sur la branche locale :

git branch --set-upstream origin/auth

Ou également :

git branch -u origin/auth

Vous aurez ensuite le message de confirmation suivant :

La branche 'auth' est paramétrée pour suivre la branche distante 'auth' depuis 'origin'.
Everything up-to-date

Vous n'aurez alors plus besoin de préciser où push ou pull lorsque vous vous trouvez sur la branche.

Vous pourrez donc faire git push ou git pull sans rien préciser.
Suivre une branche distante

Pour continuer nous allons devoir distinguer trois types de branche :

La branche locale est la branche qui est dans .git/refs/heads/, par exemple .git/refs/heads/auth. Elle stocke la référence du commit de la branche sur votre machine.

La branche amont (upstream branch) est la branche sur le dépôt distant dans .git/refs/heads/. C'est une branche auquelle vous n'avez jamais accès (sauf si vous êtes administrateur du dépôt distant). Elle est gérée par Git automatiquement.

La branche de suivi est la branche dans .git/refs/remotes/origin/. Il s'agit d'une branche sur votre machine qui se met à jour en utilisant la branche amont lorsque vous faites git fetch ou git pull. Elle est appelée branche de suivi car elle suit la branche amont.
Le suivi par défaut des branches lors d'un clonage

Par défaut lorsque vous clonez un dépôt distant, la branche master est configurée pour suivre la branche origin/master, et c'est le cas pour toutes les branches existantes. Par exemple, si une branche amont auth existe sur le dépôt distant et que vous clonez le répertoire, si vous allez sur la branche en faisant :

git checkout auth

Vous aurez le message :

La branche 'auth' est paramétrée pour suivre la branche distante 'auth' depuis 'origin'.
Basculement sur la nouvelle branche 'auth'

Autrement dit, la branche locale auth va automatiquement suivre origin/auth.
Suivre une branche distante

Pour suivre une branche distante, vous pouvez soit faire git fetch puis git checkout branche, où branche est le nom de la branche de suivi, soit utiliser la commande :

git checkout --track origin/branche

Dans les deux cas vous aller checkout sur la branche de suivi et pouvoir continuer le travail dessus.
Suivre une branche distante lorsqu'on a déjà des changements

Vous avez des changements dans votre répertoire de travail et vous voudriez les appliquer sur une branche distante que vous n'avez pas encore récupérée.

Aucun problème, vous pouvez faire :

git checkout -b branche origin/branche

Lister les branches de suivi

Pour voir les associations branches locales / branches de suivi, vous pouvez faire :

git branch -vv

L'option -vv signifie verbose de niveau 2. -v étant verbose de niveau 1. C'est l'option qui donne le plus d'informations sur les branches.

Cela donnera par exemple :

* auth   02d9540 [origin/auth] auth1
  master 46f33f6 [origin/master] F origin

Où auth est la branche locale et origin/auth la branche de suivi associée. auth1 étant le début du message de validation du commit pointé par la branche auth.

Git vous signalera également si une branche est en retard ou en avance par rapport à sa branche de suivi. Par exemple, si nous créons un commit sur auth, nous aurons :

* auth   3e0ce37 [origin/auth: en avance de 1] auth2
  master 46f33f6 [origin/master] F origin

Nous avons bien l'indication en avance de 1 pour en avance de un commit par rapport à la branche de suivi. 

Répertoire distant et configurations
Mettre en cache ses identifiants

Cela peut devenir pénible de taper tout le temps ses identifiants.

Nous verrons bientôt comment utiliser le protocole ssh et ainsi utiliser une paire de clés cryptographiques.

Mais en attendant, où si vous voulez rester en HTTPS, vous pouvez faire :

git config --global credential.helper cache

Vos identifiants seront mis en cache et vous ne seront demandé qu'au maximum une fois toutes les 15 minutes.

Vous pouvez modifier la durée de cache par défaut en faisant :

git config --global credential.helper cache --timeout nombreDeSecondes

Dans le cas où votre ordinateur est partagé ou que vous voulez changer d'utilisateur et / ou de mot de passe (par exemple après une mise à jour du mot de passe sur Gitlab) vous pouvez faire :

git credential-cache exit

Rebaser par défaut lors des git pull

Pour mettre en configuration globale le fait de toujours rebaser lors d'un pull sans avoir à faire git pull --rebase, vous pouvez faire :

git config --global pull.rebase true

Et comme nous l'avons dit, VS Code offre aussi dans Source Control puis … la possibilité de faire rapidement un git pull --rebase avec Pull (Rebase). 

Utiliser le protocole SSH avec Git et Gitlab
Le protocole SSH

SSH est un protocole de connexion sécurisé qui permet d'authentifier et de chiffrer les segments TCP (les paquets transportés sur le réseau).

SSH peut utiliser l'authentification par mot de passe ou en utilisant la cryptographie asymétrique. C'est cette dernière qui est utilisée avec Git.

La cryptographie asymétrique utilise une clé publique et une clé privée. La clé publique est ajoutée sur les serveurs où l'on souhaite se connecter par un administrateur, la clé privé reste uniquement sur l'ordinateur.

Ces deux clés permettent dans cette configuration deux choses : l'authentification et le chiffrement des communications.

Voici le déroulement :

1 - La clé publique ajoutée au serveur Git ou sur Gitlab ou Github comme une clé d'authentification valide pour ce compte.

2 - Lorsque vous essayez de faire une action sur l'hôte distant, Git va utiliser SSH si vous avez cloné le répertoire en SSH, sinon nous verrons comment le configurer. Il va envoyer une demande de connexion authentifiée par une signature utilisant la clé privée.

3 - L'hôte distant utilise la clé publique pour s'assurer de l'authenticité du message : en effet, seul le détenteur de la clé privée peut effectuer une signature correspondant à la clé publique.

4 - Les communications sont ensuite chiffrées dans les deux sens.
Générer une paire de clés

Vous pouvez commencer par vérifier si une paire de clé existe sur votre machine :

cd ~/.ssh
ls

Si vous avez un fichier id_rsa et un fichier id_rsa.pub, vous avez une paire de clé publique / privée.

Nous allons utiliser la librairie ssh-keygen qui est disponible sur tous les environnements (Windows, Linux et MacOS) et installée par défaut..

Il suffit de faire :

ssh-keygen

Cela générera par défaut une paire de clés de longueur de 2048 bits en utilisant l'algorithme RSA.

Tapez entrée à toutes les questions, sauf si vous utilisez un ordinateur partagé dans ce cas entrez un mot de passe lorsque cela vous sera demandé. Ce mot de passe sera demandé à chaque fois que la paire de clé sera utilisée.

Par défaut les clés seront enregistrées dans /home/utilisateur/.ssh/.

Où utilisateur est le nom de votre utilisateur courant.

Les options courantes sont :

-t permettant de spécifier le type d'algorithme utilisé pour la génération de clés. Laissez le par défaut, RSA est le standard.

-C pour comment : cela permet d'insérer un commentaire dans la clé. Le plus souvent on précise un email ou un identifiant pour permettre plus facilement de connaitre le propriétaire de la clé (attention ces informations sont incluses dans la clé publique et sont donc accessibles).

-b permet de préciser la longueur de la clé en bits. Par défaut, suivant votre environnement ce sera 2048 bits et parfois 3072 bits. Vous pouvez préciser -b 4096 pour allonger la clé et la rendre plus sécurisée, mais cela ne sert à rien en l'état actuelle des puissances de calcul disponible.

Pour retrouver vos clés rendez vous dans :

cd /home/utilisateur/.ssh

Ou, le raccourci :

cd ~/.ssh

Vous devrez copier la clé publique pour la mettre sur Gitlab.

Affichez là en faisant :

cat ~/.ssh/id_rsa.pub

Elle doit ressembler à :

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC+OAb59N+WS68D
  /xiYdwPPqAOqbrhFAney4jJVMiKMVrt0rYC9V+80mjKB9WA1jpf4e+3AtrXMDqiq2rbd
  MuQbkjzbjSCMzl57vJlXkixNpAgzdF3HU13RhMvi+PVMrbKwPIMMQPgdJIemRm85HI4G
  Pz0WaTDezBwLOvTbmiPzJl/LbfmcCGD8ViciPOhlLFcj3nPLzmU3PtE8UC57SoiYaXYl
  Guc+p0tQN7TjvITQHNvSvukzOQTZjC/w+52v6tmjULej6nLXRN4wTEpB21lnYw+UYRY2
  egd385yGVx13kIs/4DR9apD/rW4BqwOHuVDHwur5f41tlys3Rg0hAgysSWjhQaT+qt7s
  y3PgkM7p0IAXeGuOst7IKXt+vTTgUrhBMVWiOb1dHvCcdjaCASdQ/3U1JE5Anqe+eoim
  zSr3vJQfpZfZwS34lQqqRnQgIjwiHXPPJxpFxd7mctt7AnVCLMpOnKZLpaRRVei3FjQ6
  mwlqOXpanVyivvIVx60MHItKbw1ZV4/NOHri5B+3qC4OQKIVB0iUmmjxEqxXXw84EV5v
  AmasWrztShBID+WCezvYIG3Bw3NHiVxv+cnBmrRSLLToZGkAFbjoDLE7mckLVXtUmA+/
  wWTuznI7O//7jxgm7eowbyM9JmLIrtx1hvuIBg9KsTkAYhj59s5knd37eQ==
  erwan@erwan-XPS-8930

Vous pouvez ensuite la copier et vous rendre sur Gitlab.

Allez sur Profil (en haut à droite), puis Paramètres.

Ensuite dans la colonne de gauche allez dans Clefs SSH.

Copier la clé dans l'espace et cliquez sur Ajouter une clef.

Vous pouvez maintenant vous connecter en utilisant SSH !
Configuration Git pour utiliser SSH

Lorsque vous clonez un projet, vous utilisez soit le lien SSH soit le lien HTTPS.

Jusqu'à maintenant nous avions utilisé HTTPS car nous n'avions pas configuré de clés.

Une fois vos clés configurées, utilisez tout le temps SSH lors du clonage, c'est bien plus pratique de ne jamais avoir à taper de mot de passe !

Si vous avez cloné un projet en utilisant HTTP et que vous voulez utiliser SSH, il va simplement falloir modifier la configuration.

Pour ce faire, commencez par vérifier que vous utilisez HTTPS :

git remote -v

Si vous avez par exemple :

origin	https://gitlab.com/erwan.vallee/gittest.git (fetch)
origin	https://gitlab.com/erwan.vallee/gittest.git (push)

Les liens commencent bien par https ce qui signifie que vous utilisez bien le protocole HTTPS.

Il suffit de le modifier en faisant :

git remote set-url origin git@gitlab.com:erwan.vallee/gittest.git

Le lien se trouve sur la page d'accueil de votre projet sur Gitlab lorsque vous cliquez sur Clone (en bleu).

Le tour est joué ! Vous pouvez ensuite utiliser SSH sur ce projet pour vos connexions vers le dépôt distant sur Gitlab. 

Optimisation et nettoyage
Nettoyer les objets orphelins

La commande git prune permet de supprimer les objets orphelins Git.

Comme nous l'avons vu, les objets orphelins sont des objets de la base de données Git qui ne sont plus référencés par aucun autre objet : ils sont inatteignables et donc inutiles.

Vous pouvez voir les objets orphelins avec la commande que nous avons déjà vue dans la leçon sur le reflog :

git fsck --full

Ou vous pouvez également utiliser git prune :

git prune --dry-run

L'option --dry-run ou -n permet d'afficher ce qui serait supprimé avec git prune sans le faire immédiatement.

Ce qui vous donne par exemple :

6e0b5d695be895f493329fec0f70ede2bab4021e blob
f1c829d7a0e20bbd61986ce39bbbcf030240bc10 blob

Ici, cela signifie que nous avons deux fichiers qui ne sont référencés par aucun arbre.

Pour les supprimer vous pouvez faire :

git prune --verbose

Cela va supprimer les objets orphelins listés par --dry-run et les afficher grâce à l'option --verbose.

Vous remarquerez que des commits ou d'autres objets orphelins ne seront pas listés.

La raison est qu'ils ne sont pas encore expirés : comme nous l'avons appris, les objets sont conservés dans le reflog pendant 90 jours par défaut pour des raisons de sécurité.

A noter que les objets orphelins sont conservés 30 jours par défaut dans le reflog.

La commande git prune permet donc de supprimé les objets orphelins expirés.

Nous pouvons forcer l'expiration des objets orphelins avec la commande suivante :

git reflog expire --expire-unreachable=all --all

Dans notre exemple, si nous refaisons :

git prune --dry-run

Nous aurons cette fois ci :

1384c27644939f51759e4559014a800e1821a261 commit
62c7cecae29938c52f8b0b69fbffb5c62be3c49d commit
86c40dfab371d55e699e08101f3adec2c481dc27 commit
8a9de7c64a5c3478413c2b6a40d4389b9916515e commit

Ce sont les commits orphelins dont nous avons forcé l'expiration.
Supprimer les branches de suivi inutiles

Rappelez vous que nous avons vu la distinction entre les branches amonts, situées sur le dépôt distant, et les branches de suivis qui sont dans .git/refs/remotes/origin/ et qui suivent les branches amonts.

Lorsque les branches amonts sont supprimées du dépôt distant, les branches de suivi ne sont pas supprimées automatiquement, même après avoir git fetch.

Pour les supprimer il faut faire :

git fetch --prune

L'option --prune va supprimer les branches de suivi dont les branches amonts associées n'existent plus sur le dépôt distant.

Si vous voulez que les branches de suivi soient automatiquement supprimées lorsque les branches amonts n'existent plus et que vous faîtes un git fetch ou un git prune, vous pouvez faire :

git config --global fetch.prune true

Optimiser sa base de données locale Git

La commande git gc (pour garbage collection) permet de procéder au nettoyage et à l'optimisation de sa base de données locale git.

C'est une commande utilitaire très puissante :

Premièrement, elle va supprimer les objets orphelins expirés (comme git prune).

Deuxièmement, elle va compresser les objets Git en utilisant un algorithme de compression différentielle. Il va identifier les objets similaires, les grouper en packs et ne sauvegarder que les différences entre ces objets.

Nous avons vu en effet que par défaut, les objets sont simplement compressés avec zlib mais ne sont pas compressés différentiellement.

Ces objets compressés, sont placés dans le dossier .git/objects/pack.

Cette commande est utilisée par Git régulièrement lors des commandes git pull, git merge, git rebase et git commit.

De cette manière, le dépôt local est optimisé régulièrement. 