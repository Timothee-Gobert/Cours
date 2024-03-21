## Inspecter un repertoire et des fichiers

Visualiser l'historique d'un projet Git avec git log
Visualiser l'historique avec git log

Lorsque vous aurez plein de commits, ou lorsque vous arrivez sur un projet, il peut être utile de visualiser rapidement l'historique des commits.

Pour ce faire il suffit d'utiliser la commande :

git log

Voici un exemple de ce qu'elle produit :

commit d23b1bff8c89251f8335f8869dd18e9c7d16e5eb (HEAD -> master)
Author: dymafr <jean@gmail.com>
Date:   Tue Mar 10 14:16:48 2020 +0100

    deuxième commit

commit 9e633d56381d9f0335320f22ccf3f1562d1f80d8
Author: dymafr <jean@gmail.com>
Date:   Mon Mar 9 12:50:43 2020 +0100

    Premier commit

La commande git log affiche les commits du plus récent (donc du dernier) vers le plus anciens.

Pour chaque commit elle donne le hash entier du commit, l'auteur, la date et le message de validation.

Si il y a trop de commits à afficher dans le terminal, Git passe en mode pagination, vous pouvez remonter le fil et utiliser q pour quitter la visualisation.
L'option p pour patch text

Cette option permet d'ajouter en plus un résultat similaire à git diff.

Elle permet d'afficher en fait la différence avec le commit précédent, prenons un exemple :

git log -p

Donnera :

commit d23b1bff8c89251f8335f8869dd18e9c7d16e5eb (HEAD -> master)
Author: dymafr <jean@gmail.com>
Date:   Tue Mar 10 14:16:48 2020 +0100

    deuxième commit

diff --git a/fichier1.txt b/fichier1.txt
index d00491f..b4de394 100644
--- a/fichier1.txt
+++ b/fichier1.txt
@@ -1 +1 @@
-1
+11

puis d’autres commits plus anciens à la suite...

Vous voyez que vous avez les mêmes informations qu'avec un git diff.

En fait, cela revient à faire git diff hash (où hash est à chaque fois le hash du commit précédent).

Vous pouvez ainsi visualiser rapidement les modifications effectuées depuis le commit précédent.
Limiter le nombre de commits affichés

Vous pouvez passer en option un nombre de commits à afficher avec -2 par exemple.

Vous pouvez par exemple faire :

git log -p -2

Cela affichera les deux derniers commits avec les modifications par rapport au commit précédent pour chacun des deux commits.
Afficher un résumé des modifications pour chaque commit

Une option intéressante est --stat qui permet d'afficher pour chaque commit seulement un résumé des modifications effectuées par rapport au commit précédent :

git log --stat

Elle donnera par exemple :

commit d23b1bff8c89251f8335f8869dd18e9c7d16e5eb (HEAD -> master)
Author: dymafr <jean@gmail.com>
Date:   Tue Mar 10 14:16:48 2020 +0100

    deuxième commit

  fichier1.txt | 2 +-
  1 file changed, 1 insertion(+), 1 deletion(-)

commit 9e633d56381d9f0335320f22ccf3f1562d1f80d8
Author: dymafr <jean@gmail.com>
Date:   Mon Mar 9 12:50:43 2020 +0100

    Premier commit

  dossier/fichier3.txt | 1 +
  fichier1.txt         | 1 +
  fichier2.txt         | 1 +
  3 files changed, 3 insertions(+)

Vous pouvez ainsi voir en un clin d'oeil les fichiers modifiés par un commit.

Elle permet de voir : la liste des fichiers modifiés par le commit, combien de fichier ont été modifiés en tout et combien de lignes ont été ajoutées et supprimées.
Obtenir uniquement le hash et le message de validation

Une option permet de n'afficher que le hash et le message de validation pour chaque commit.

C'est utile pour accéder rapidement à un hash précédent pour pouvoir par exemple, comme nous le verrons, naviguer entre plusieurs commit.

git log --pretty=oneline

Cela vous donnera par exemple :

d23b1bff8c89251f8335f8869dd18e9c7d16e5eb (HEAD -> master) deuxième commit
9e633d56381d9f0335320f22ccf3f1562d1f80d8 Premier commit

Vous pouvez aussi faire :

git log --oneline

Qui ne donnera que le début du hash de chaque commit (suffisamment pour l'identifier) :

d23b1bf (HEAD -> master) deuxième commit
9e633d5 Premier commit

Filtrer les commits

Nous avons vu que nous pouvions passer un nombre pour filtrer les commits en n'affichant que les n derniers commits.

Il existe de très nombreuses options permettant de filtrer les commits, nous allons montrer quelques exemples pour voir ce qu'il est possible de faire.
Filtrer suivant l'auteur du commit

Pour ne voir que les commits effectués par un auteur en particulier vous vouvez utiliser l'option --author :

git log --author="dymafr"

Filtrer suivant des périodes

Vous pouvez n'afficher que les commit effectués dans les 7 derniers jours :

git log --since=1.weeks

Filtrer suivant des mots clés contenus dans les messages de validation

Vous pouvez également filtrer les commits dont le message de validation contient les mots clés précisés en utilisant l'option --grep :

git log --grep Premier

Cela filtrera les commits qui ont Premier dans leur message de validation.
Filtrer suivant des mots clés contenus dans les modifications

Une option très puissante est de chercher dans les modifications des commits et de filtrer ceux qui contiennent le mot clé :

git log -S methodeModifiee

Vous aurez tous les commits qui ont touchés au nom de cette méthode !

Il faut utiliser des guillemets si vous voulez utiliser plusieurs mots clés :

git log -S 'Linus Torvald'

Visualiser l'historique d'un fichier avec git blame
Visualiser qui a modifié chaque ligne

Git possède une commande qui permet de vous indiquer pour chaque ligne d'un fichier qui l'a modifié et quand : git blame.

Prenons un fichier d'un projet important, par exemple la célèbre librairie moment permettant de manipuler facilement des dates en JavaScript.

Faisons git blame sur un des fichiers :

git blame fichier

Nous utilisons l'option -L pour spécifier un intervalle de lignes par exemple de 1 à 4 :

git blame -L 1,4 fichier

Nous avons donc par exemple, pour les 4 premières lignes :

6928b84f9 (Iskren Chernev       2015-03-22 15:00:07 -0700 1) import extend from '../utils/extend';
6928b84f9 (Iskren Chernev       2015-03-22 15:00:07 -0700 2) import { createUTC } from './utc';
06b987a21 (Brian Wyant          2015-04-22 10:28:24 -0400 3) import getParsingFlags from '../create/parsing-flags';
421b78421 (maggie@tempworks.com 2016-03-15 23:32:32 -0500 4) import some from '../utils/some';

Pour chaque ligne nous avons les informations suivantes :

Premièrement, nous avons le début du hash du dernier commit qui a modifié la ligne.

Deuxièmement, nous avons l'identité de celui qui a modifié la ligne pour la dernière fois.

Troisièmement, nous avons la date de la dernière modification de la ligne.

Ensuite, nous avons le numéro de la ligne par exemple 2).

Enfin, nous avons le contenu actuel de la ligne.
Visualiser les mouvements de code

Git est tellement puissant qu'il peut repérer si une ligne de code a été déplacé depuis un autre fichier.

Mettons que vous refactorez votre code et que vous déplaciez trois lignes d'un fichier vers un autre fichier, grâce à git blame vous allez pouvoir le voir !

L'option -C permet de détecter les lignes déplacées ou copiées depuis d'autres fichiers dans le commit où elles ont été ajoutées au fichier.

Par défaut l'option va détecter les suites de 40 caractères qui ont été déplacés / copiés.

git blame -C -L 3,6 fichier

Donne par exemple :

06b987a21 src/lib/create/valid.js (Brian Wyant          2015-04-22 10:28:24 -0400 3) import getParsingFlags from '../create/parsing-flags';
421b78421 src/lib/create/valid.js (maggie@tempworks.com 2016-03-15 23:32:32 -0500 4) import some from '../utils/some';
d5262a01b lib/create/valid.js     (Tim Wood             2015-01-11 11:15:03 -0800 5)
d5262a01b lib/create/valid.js     (Tim Wood             2015-01-11 11:15:03 -0800 6) export function isValid(m) {

Nous pouvons voir qu'il y a eu un déplacement des lignes 5 et 6 qui étaient dans lib/create/valid.js et maintenant sont dans src/lib/create/valid.js (qui est l'état actuel du fichier).

Nous verrons dans les leçons suivantes que nous utilisons des extensions de VS Code plutôt que directement git blame, mais elles se basent sur cette commande. 

L'onglet Source Control de VS Code
Visual Studio Code supporte Git par défaut

Par défaut, Visual Studio Code (VS Code) permet d'utiliser très facilement Git grâce à l'onglet Source Control :

Il s'agit du troisième onglet à gauche en partant du haut, son symbole est celui de commits symbolisés par des ronds reliés entre eux et d'une branche qui part vers la droite (nous verrons très en détails les branches dans un chapitre dédié).

L'icône Source Control a une petite bulle bleu indiquant le nombre de fichiers modifiés, dès lors que vous avez au moins un fichier modifiés.
Les trois états des fichiers dans VS Code

VS Code sépare les fichiers modifiés en trois groupes :

Premièrement, les fichiers sous CHANGES : à savoir les fichiers qui ont au moins une modification et qui ne sont pas indexés. Autrement dit il s'agit des fichiers modifiés sur le répertoire de travail et qui ne sont pas ajoutés en zone de transit.

Deuxièmement, les fichiers sous STAGED CHANGES : ce sont les fichiers qui sont indexés dans la zone de transit.

Troisièmement, les fichiers sous MERGE CHANGES : nous verrons plus tard cette catégorie qui apparaît lors des fusions.

Voici un exemple avec des fichiers modifiés indexés et non indexés :
Indexer un fichier avec VS Code

Lorsque vous avez des fichiers modifiés et sauvegardés, vous pouvez les indexer avec git add, comme nous l'avons vu jusqu'à maintenant.

Il est également possible d'utiliser le bouton +, intitulé Stage changes lorsque vous passez la souris (comme sur l'image précédente).

Il a exactement le même effet que de faire git add fichier.
Désindexer un fichier avec VS Code

Pour les fichiers indexés, qui se trouvent dans le groupe STAGED CHANGES, vous verrez que vous avez un bouton -. Il permet d'annuler l'indexation.

Comme nous le verrons cela revient en fait à utiliser git reset fichier.
Effectuer un commit

Lorsque vous avez des changements indexés et que vous voulez les sauvegarder définitivement dans une version, nous avons vu que vous pouviez faire :

git commit -m "Message de validation"

Sur VS Code, vous pouvez également remplir la case Message avec le message de validation puis cliquer sur la flèche de validation.

Attention ! Si vous n'avez aucun fichier dans STAGED CHANGES, c'est-à-dire que vous n'avez que des fichiers modifiés mais non indexés et que vous utilisez la case Message pour créer un commit, VS Code va considérer que vous voulez faire :

git commit -am "Message de validation"

A savoir que tous vos fichiers sous CHANGES seront indexés et commit en une fois.

C'est très agréable quand vous maîtrisez Git car vous allez plus vite, mais au début nous vous le déconseillons car vous allez faire des erreurs dans ce que vous allez sauvegarder.
Visualisation des changements dans un fichier

Dans un fichier, VS Code a un code couleur qui va vous indiquer les lignes ajoutées, modifiées et supprimées :

Le rouge correspond à la suppression d'une ligne. Dans l'exemple cela signifie que la ligne 2 a été supprimé (et donc l'ancienne ligne 3 est devenue la ligne 2).

Le vert correspond à l'ajout d'une ligne. Dans l'exemple cela signifie que la ligne 3 a été ajoutée.

Le bleu correspond à la modification d'une ligne. Dans l'exemple cela signifie que les lignes 9, 10 et 11 ont été modifiées.

Vous pouvez ainsi facilement visualiser les changements.

Lorsque vous indexé un fichier contenant des changements, ou que vous effectuez un commit avec git -am, les changements ne sont plus affichés dans VS Code.

C'est logique : les changements sont identifiés par rapport à l'index, ou zone de transit. Lorsque vous indexez les changements, il n'y a plus de différences entre l'index et le répertoire de travail et VS Code ne les affiche donc plus !

Cela revient en fait à faire git diff, seulement c'est beaucoup plus visuel grâce à VS Code !
Visualisation des changements d'un fichier avec la version précédente

Lorsque vous cliquez sur le nom d'un fichier dans l'onglet Source Control, l'outil de comparaison vous permet de visualiser les changements exacts.

Par exemple si vous cliquez sur un fichier modifié mais non indexé (dans CHANGES), vous verrez la version du fichier du dernier commit à gauche et la version du répertoire de travail à droite.

Si vous cliquez sur un fichier modifié et indexé (dans STAGED CHANGES), vous verrez la version du répertoire de travail à gauche et celle indexée à droite.

A chaque fois le rouge dénote la version précédente et le vert la nouvelle version.
Utiliser la barre d'accès rapide

Pour accéder à la barre d'accès rapide appelez Command Palette sur VS Code faites :

Sur Windows et Linux : Ctrl + Maj + P.

Sur MacOS : Pomme + Maj + P.

Vous pouvez ensuite taper des commandes Git et VS Code auto-complétera au mieux.

Par exemple, comme nous le verrons plus tard lorsque nous aurons vu les branches et les tags, si vous faites git checkout et cliquez sur Git: Checkout to, vous pourrez naviguer sur les tags et les branches. 

L'extension Git lens
L'extension GitLens

L'extension VS Code GitLens permet principalement de visualiser directement dans l'éditeur les informations fournies par git blame.

Mais elle a également de nombreuses fonctionnalités dont nous allons voir les principales.
Vérifier qui a modifié une ligne

La fonctionnalité principale est de pouvoir voir quelles lignes ont été modifiées quand et par qui directement dans VS Code.

Ainsi lorsque vous ouvrirez un fichier vous aurez :

Dans l'exemple Eric Amodio (c'est le créateur de l'extension) a modifié la ligne il y a 8 mois et le début du message de validation est Supercharged.

Si vous passez la souris sur le texte en gris vous aurez une popup vous donnant plus d'informations et vous permettant d'effectuer des actions :

Vous pouvez cliquer sur Changes pour accéder aux modifications apportées par le dernier fichier par le dernier commit. (Equivalent d'un git diff entre les deux derniers commits).
Vérifier qui a modifié un fichier

En haut de tous les fichiers vous aurez maintenant également une nouvelle ligne :

Cette ligne vous indique :

Premièrement, qui a modifié le fichier en dernier et quand.

Deuxièmement, combien de personnes ont touché au fichier (x authors) et l'auteur principal (celui qui a le plus écrit de lignes).

Si vous cliquez vous aurez un espace qui vous mettra pour chaque ligne le message du dernier commit l'ayant modifiée et sa date. Vous pourrez passer la souris pour avoir plus d'informations (qui a modifié, début du hash du commit etc) :

Si vous cliquez sur un commit dans la colonne de gauche (par exemple celui qui est actuellement sélectionné dans l'exemple avec le message Supercharged), toutes les lignes qui ont été modifiés par ce commit seront surlignés sur le fichier (à droite).

Vous pouvez donc voir très facilement quelles lignes ont été modifiées par tel ou tel commit.

Il existe de nombreuses autres fonctionnalités plus avancées mais elles nécessitent de voir plus de notions Git.

Vous avez déjà un premier aperçu de cet outil très puissant qui vous permettra de savoir rapidement qui a fait quoi ! 