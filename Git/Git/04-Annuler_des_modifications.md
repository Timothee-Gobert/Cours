## Annuller des modifications

Branche master et HEAD
Rappels sur les zones de Git

Nous avons vu que la version des fichiers tels qu'ils apparaissent dans l'éditeur de code est appelée repertoire de travail (ou working directory).

Nous avons vu que la version des fichiers qui a été ajoutée à l'index avec git add sont dits indexés ou en zone de transit (staging area).

Nous avons enfin vu que la version des fichiers qui était sauvegardée avec git commit depuis la zone de transit est ajoutée au répertoire Git (repository), appelé également dépôt.

Dans ce chapitre nous allons nous intéresser en détails à comment défaire des changements qui sont soit dans le répertoire de travail, soit indexés.

Nous allons apprendre comment faire pour annuler des modifications et revenir à l'état souhaité sans perdre ou en perdant ces modifications. C'est donc un chapitre important !
Qu'est-ce que la branche master ?

Nous allons voir les branches dans un chapitre dédié de manière approfondi, mais nous allons voir déjà une première approche de ce qu'est une branche.

Nous avons appris qu'un commit contient notamment une référence à un arbre ou tree.

Cet arbre contient des références vers des blobs (des fichiers en binaire) ou d'autres arbres (s'il y a des sous dossiers) qui contiennent eux-mêmes des références à des arbres ou des blobs et ainsi de suite. L'ensemble constitue un instantané du projet (appelé également version ou sauvegarde). Il permet d'avoir la structure du projet (comment sont organisés les fichiers et dossiers entre eux) et tous les contenus en binaire.

Tous les commits (sauf le tout premier commit d'un répertoire appelé commit racine) ont également la référence à leur commit parent (comme nous le verrons un commit peut avoir plusieurs parents en cas de fusion).

Les commits forment donc une chaîne de commits grâce à ces références.

Une branche est simplement un pointeur, également appelé référence, vers un commit.

master est la branche principale créée par défaut par Git lors du git init.

Au fur et à mesure que vous créez des commits sur une branche, la référence de la branche se déplace automatiquement au dernier commit.

Prenons un exemple :

Au début du projet nous avons que le commit A, la branche master pointe sur celui-ci.

Ensuite, nous faisons un second commit B qui a pour parent le commit A, la branche master pointe alors automatiquement vers le commit B.

Ainsi de suite jusqu'au commit D qui est dans l'exemple le dernier commit réalisé.

Nous pouvons vérifier qu'une branche est simplement une référence en faisant :

cat .git/refs/heads/master

cat permet d'afficher le contenu d'un fichier.

Cela vous donnera par exemple : d23b1bff8c89251f8335f8869dd18e9c7d16e5eb, soit tout simplement le hash du dernier commit.

C'est tout ce que nous avons besoin de savoir pour le moment.
Qu'est-ce que HEAD ?

Nous verrons plus tard que nous pouvons avoir plusieurs branches, naviguer entre les commits et plein d'autres choses !

Mais alors comment Git sait où vous-vous trouvez actuellement ? Comment vous met t-il la bonne version du projet dans votre répertoire de travail ? C'est le rôle de HEAD !

HEAD est une référence, ou pointeur, vers le commit, la branche, ou le tag sur laquelle vous vous trouvez actuellement.

Autrement dit elle point toujours directement, ou indirectement, vers un commit.

Par défaut, HEAD pointe vers la branche master qui pointe vers le dernier commit.

C'est un peu difficile à appréhender au début, c'est pour cela que nous vous avons préparé un schéma avec les différents liens, chaque lien est une référence.

Nous pouvons vérifier que HEAD pointe bien vers master :

cat .git/HEAD

Cela vous donnera cela :

ref: refs/heads/master

Il s'agit d'un chemin relatif vers la branche master, qui pour rappel contient le hash du dernier commit. 

git checkout
La commande git checkout

La commande git checkout est l'une des plus utilisées dans Git.

Elle permet de se déplacer vers une autre branche ou vers un tag (comme nous le verrons), vers un autre commit, ou de restaurer la version indexée.

Il faut vraiment voir la commande checkout comme une commande de navigation de HEAD. Autrement dit elle permet de déplacer HEAD où vous souhaitez.

Nous verrons dans le chapitre suivant comment fonctionne le checkout sur les branches.

Dans cette leçon nous allons voir comment naviguer entre les commits.
Naviguer entre les commits

La commande git checkout hashCommit permet de déplacer HEAD vers le commit spécifié et de mettre à jour le répertoire de travail et l'index à la version correspondant à ce commit.

Prenons la configuration suivante :

Par défaut, comme nous l'avons vu, HEAD pointe sur une branche, en l'occurrence la branche principale master créée par Git par défaut.

Mais si nous faisons :

git checkout hashCommitB

Que va t'il se passer ? Nous serons dans la situation suivante :

Nous avons déplacé HEAD vers le commit B.

Si nous faisons :

cat .git/HEAD

Nous avons maintenant directement le hash du commit B, par exemple :

9e633d56381d9f0335320f22ccf3f1562d1f80d8

Si nous faisons :

git status

Nous aurons en rouge :

HEAD detached at 9e633d5
nothing to commit, working tree clean

Nous allons voir ensemble ce que cela signifie.
L'état DETACHED HEAD

Attention ! lorsque vous naviguez sur un commit en utilisant git checkout commit vous êtes dans un état très particulier appelé DETACHED HEAD.

L'état détaché signifie que HEAD ne fait référence à aucune branche et donc si vous effectuez des modifications vous ne pourrez pas les indexer et les sauvegarder.

Plus exactement vous pourrez le faire mais Git nettoie automatiquement les commits qui ne sont plus accessible par une branche.

Par exemple, si vous effectuez des modifications en étant sur le commit B puis que vous les indexez et les sauvegardez vous aurez l'état suivant :

Le problème est que le commit E n'est pas référencé par une branche.

Si vous vous replacez sur master en faisant git checkout master, plus aucune référence ne sera conservée pour le commit E.

Il sera très difficile d'y accéder : il faudra retrouver le hash du commit E et faire git checkout hash.

Git va détruire le commit E automatiquement si aucune branche n'y fait référence après une période de 90 jours.

Le cas d'utilisation de git checkout commit est donc simplement de lancer votre projet à cette version donnée pour regarder comment il était.

Il ne faut surtout pas faire de modifications, ou alors il faut créer une branche comme nous le verrons.
Annuler définitivement des modifications

Avec la commande git checkout -- fichier vous pouvez rétablir la version du fichier pointée par HEAD.

Notez que git checkout fichier et git checkout -- fichier font exactement la même chose. La différence est que si une branche a le même nom qu'un fichier (c'est improbable mais quand même), ce sera la branche qui sera checkout. Il vaut donc mieux dans un gros projet utiliser -- fichier.

Autrement dit, vous avez effectué des modifications sur un fichier dans votre répertoire de travail et vous ne voulez pas les conserver, vous pouvez toutes les supprimer définitivement et rétablir la version indexée.

Si vous n'avez pas indexé de modifications alors l'index correspondra à la dernière version sauvegardée, donc au dernier commit.

Attention avec cette commande ! C'est l'une des rares commandes qui supprime définitivement des données en Git.

Réfléchissez avant de git checkout -- fichier.
Restaurer une version spécifique pour un fichier particulier

Vous pouvez également, avec git checkout commit -- fichier restaurer le fichier donné à une version spécifique, à savoir celle correspondant au commit.

Dans ce cas HEAD ne sera pas déplacé, la version du fichier indexée et de travail seront remplacées par la version du fichier spécifié.

Par exemple :

git checkout c5f567 -- fichier1.txt

Attention ! Si le fichier1.txt contenait des modifications non sauvegardées (commit) elles seront définitivement perdues, et ce même si elles étaient indexées.

Comme vous pouvez le voir la commande git checkout n'est pas à prendre à la légère ! Vous pouvez perdre des modifications qui n'ont pas été sauvegardées avec git commit. 

git clean
La commande git clean

La commande git clean permet de supprimer tous les fichiers non suivis du répertoire de travail, sauf ceux contenus dans .gitignore.

Attention ! Les fichiers sont définitivement perdus après cette commande.

C'est une commande peu utilisée, que nous allons montrer brièvement. Pas besoin de passer beaucoup de temps dessus !
Options

L'option -n, pour dry-run, permet de savoir quels fichiers seront supprimés avant de procéder à leur suppression.

git clean -n

L'option -f, pour force, permet d'enclencher la suppression des fichiers. C'est une mesure de protection car les fichiers sont définitivement supprimés :

git clean -f

L'option -i, pour interactive permet de lancer le mode interactif de Git comme montré dans la vidéo :

git clean -i

git revert
Annuler les changements d'un commit

La commande git revert permet de créer un nouveau commit qui va annuler tous les changements effectués par le commit spécifié.

Cette commande est très utile pour annuler un commit contenant des erreurs en conservant l'historique de toutes les modifications.

git revert commit

Où commit est le hash du commit dont vous voulez annuler les modifications.

Par défaut, l'éditeur par défaut sera ouvert pour spécifier le message de validation qui sera pré-rempli par Revert + le message de validation du commit que vous voulez annuler.

Si vous ne voulez pas avoir à modifier le message et que celui par défaut vous convient, vous pouvez faire :

git revert commit --no-edit

Dans ce cas le commit se fera immédiatement avec le message prérempli.
Annuler les changements d'un commit sans créer de nouveau commit

Parfois vous voudrez annuler les changements d'un commit et faire quelques modifications supplémentaires avant d'effectuer un nouveau commit.

Dans ce cas il faut utiliser git revert --no-commit :

git revert --no-commit commit

Où commit est le hash du commit dont vous voulez annuler les modifications.

Cette commande va annuler tous les changements effectués par le commit cible, mais ne va pas créer de nouveau commit.

Elle va annuler les changements dans le répertoire de travail et dans l'index sans créer de commit. 

git reset

La commande git reset est l'une des plus importantes de Git. Il est très important de bien la comprendre et bien la maîtriser.

Elle permet de faire beaucoup de choses, et peut être dangereuse dans certains cas car elle peut provoquer des pertes de données irréversibles.
Désindexer des fichiers ou des dossiers indexés

La commande git reset fichier est l'inverse de git add fichier : elle permet de désindexer un fichier en conservant les changements dans le répertoire de travail.

En fait, elle prend la version du fichier du dernier commit et la met dans l'index. Cela revient effectivement à désindexer un fichier.

Si vous spécifiez un dossier, tous les fichiers contenus dans ce dossier, et le dossier lui-même seront désindexés.

Sur VS Code cela revient à appuyer sur le bouton - dans l'onglet Source Control sur un fichier dans le groupe STAGED CHANGES.

Désindexer un fichier est utile si vous voulez diviser vos modifications en plusieurs commits.

Par exemple, mettons que vous avez travaillez sur une fonctionnalité et une correction de bug, et que par inadvertance vous avez tout indexé avec git add ..

Pourtant vous souhaitez faire un commit pour la correction de bug et un autre pour la nouvelle fonctionnalité.

Vous allez donc pouvoir désindexer les fichiers modifiés pour la correction de bug avec git reset, commit les modifications restantes pour la fonctionnalité, puis réindexer les fichiers pour la résolution de bug et faire un second commit.
Déplacer la branche sur laquelle pointe HEAD

La commande git reset commit permet de déplacer la branche sur laquelle pointe HEAD sur le commit sélectionné.

Nous avons vu que HEAD contient la référence vers la branche sélectionnée, par défaut master.

Par défaut git reset commit va donc déplacer la branche master sur le commit spécifié.

Prenons par exemple la situation suivante :

Que se passe t-il si nous faisons :

git reset B

Où B est le hash du commit B.

Voici le résultat :

HEAD pointe toujours sur master :

cat .git/HEAD

Donne toujours :

ref: refs/heads/master

Si vous faites git status, vous obtiendrez quelque chose comme cela :

On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   fichier1.txt
  modified:   fichier2.txt

no changes added to commit (use "git add" and/or "git commit -a")

En fait, par défaut, la dernière version (celle du commit D dans notre exemple) est conservée dans le répertoire de travail.

Par contre, les versions correspondants aux commits C et D de notre exemples sont retirés du dépôt, elles ne sont plus sauvegardées.
Les trois options

Il existe trois options avec git reset qu'il est très important de connaitre.
git reset --soft

La première option est --soft, elle permet de déplacer la branche sur laquelle pointe HEAD sur le commit spécifié sans rien faire d'autre.

Autrement dit voici la version des fichiers dans les trois zones Git :

Dans le répertoire de travail la version est celle correspondant au dernier commit : rien ne change par rapport à avant de lancer la commande.

Dans l'index la version est celle correspondant au dernier commit : rien ne change non plus par rapport à avant de lancer la commande.

Dans le répertoire, HEAD pointe sur master et master pointe sur le commit spécifié. Ce qui signifie qu'il n'y a plus aucune référence vers les commits suivants (C et D dans notre exemple). Ces versions ne sont donc plus sauvegardées dans le dépôt !
git reset --mixed

La seconde option --mixed, est celle par défaut, c'est-à-dire celle utilisée en faisant git reset commit sans passer d'option.

La différence avec --soft se situe au niveau de l'index : dans l'index, la version des fichiers correspond au commit spécifié.

Autrement dit, les modifications des commits C et D n'existent plus dans l'index. Elles sont uniquement présentes dans le répertoire de travail.
git reset --hard

La dernière option --hard est dangereuse : les modifications sont définitivement perdues.

Autrement dit elles ne sont plus dans le répertoire de travail, plus dans l'index et plus dans le dépôt !
Supprimer un commit

Attention ! Cet exemple n'est valide que si vous n'avez pas push (mis à jour le dépôt distant avec vos versions, comme nous le verrons plus tard).

Si vous avez push il ne faut pas suivre ce qui suit mais utiliser revert.

Prenons la situation suivante :

Imaginons que le commit C soit la moitié d'une fonctionnalité et qu'il contient comme message chantier en cours. Vous n'avez pas push ni le commit C ni le D.

Vous aimeriez supprimer le commit C car la version D contient la fonctionnalité terminée avec un message de validation correct.

Commençons par faire :

git reset --soft B

Ou :

git reset --soft HEAD~2

HEAD~2 ou HEAD^^ signifient remonte de deux commits à partir du commit pointé par HEAD. Comme HEAD pointe sur master qui pointe sur D, cela revient à pointer sur B :

Comme nous avons fait soft, l'index et le répertoire de travail contiennent la version des fichiers correspondant au commit D.

Nous n'avons plus qu'à git commit !

Cela créera un nouveau commit avec la version des fichiers correspondant au commit D qui n'existe plus !

En résumé, nous avons écarté complètement le commit C et créé un nouveau commit qui a la même version des fichiers que celle du commit D.

Nous obtenons :

Le commit E et l'arbre E ont la même version des fichiers que le commit D et l'arbre D mais sont des objets différents. 