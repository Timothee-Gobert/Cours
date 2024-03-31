# Création et architecture d'une application Next.js

Installation avec create-next-app

Après avoir installé Node.js et son utilitaire npm, nous allons pouvoir commencer à installer notre première application Next.js.
Installation de l'application Next.js

Il est possible d'installer une application Next.js à partir d'un projet React.js existant. Ici, nous procéderons à l'installation grâce à cette ligne de commande :

npx create-next-app@latest

npx, à la différence de npm, nous permet d'exécuter des paquets et non pas de les enregistrer, les installer.

Le tag latest nous permettra de profiter de la dernière version de disponible de Next.

L'option --experimental-app n'est plus à ajouter, elle permettait de définir que l'on souhait utiliser la version app router de Next quand elle n'était pas encore stable. Depuis juin 2023, cette version est stable.
Les paramètres de l'exécution

Une fois ce paquet exécuté, vous pourrez définir un nom de projet qui vous sera demandé. Ce nom sera celui du dossier créé ainsi que le nom de votre projet dans votre fichier package.json.

Egalement, vous aurez la possibilité de choisir si l'installation de votre application vient également avec Typescript. Langage développé par Microsoft, c'est un surensemble de Javascript qui vous permet, entre autres, de typer les variables et retours de votre code.

Nous avions installé lors des précédentes sessions l'extension ESLINT pour notre IDE, ici nous préciserons que nous souhaitons l'inclure aux dépendances de notre projet. Celui-ci vient avec une configuration pré-établie pour Next.js.

Vous pouvez choisir d'inclure un dossier supplémentaire src dans votre application pour bien compartimenter votre code. 

Enfin, vous devriez choisir d'utiliser la version du routage sous format de dossier App. Celle-ci est la toute dernière version de routage de Next. C'est celle-là que nous allons utiliser dans nos exemples.

Un alias d'import par défaut vous sera proposé, qui vous permettra de remplacer une partie de vos chemins vers le dossier src par un format raccourci (@/ par défaut).
Découverte du projet installé

Comme pour tout projet Node, vous trouverez un répertoire node_modules qui contient la totalité des dépendances requises pour le fonctionnement de votre projet.

Le fichier package.json, lui, vous donnera des informations sur les paquets installés, notamment React et Next. Aussi, vous y retrouverez plusieurs scripts :

    dev : permet le lancement d'un serveur de développement, avec notamment le Fast Refresh qui vous permettra de profiter d'un rechargement de votre serveur lors de chaque édition de code
    build : permet de préparer votre application à la production.
    start : permet le lancement d'un serveur de production, il s'appuiera sur le dossier .next créé par l'exécution de votre script de build précédent.
    lint : exécute le linter ESLint sur l'ensemble de votre code pour trouver des possibles problèmes. Ce script ne nous intéressera pas forcément étant donné que l'extension installée sur notre IDE remplit déjà ce rôle.

Parmi ces fichiers installés, le package-lock.json sera généré pour vous et vous listera l'ensemble des paquets installés avec leur version exacte.