# Création de table

Nous avons vu comment créer des tables avec Workbench en utilisant l'outil de Forward Engineering et comment les modifier avec l'outil de synchronisation, mais il faut comprendre les syntaxes et savoir comment le faire sans Workbench également.
Syntaxe de création d'une table

La syntaxe générale pour créer une table dans MySQL est :

CREATE TABLE nom_de_la_table (
    nom_colonne1 type_de_donnée1 contraintes1,
    nom_colonne2 type_de_donnée2 contraintes2,
    ...
    nom_colonneN type_de_donnéeN contraintesN,
    contrainte_table1,
    contrainte_table2,
    ...
);

CREATE TABLE : c'est la commande utilisée pour créer une nouvelle table dans la base de données.

nom_de_la_table : le nom que vous souhaitez attribuer à votre table. Ce nom doit être unique au sein de la base de données.

Pour chaque colonne de la table, vous spécifiez :

    nom_colonne : le nom de la colonne dans la table. Chaque nom de colonne doit être unique au sein de sa table.
    type_de_donnée : le type de données que cette colonne va contenir (par exemple, INT pour un entier, VARCHAR pour une chaîne de caractères variable, DATE pour une date, etc.).
    contraintes : les règles appliquées aux valeurs de la colonne (par exemple, NOT NULL pour indiquer que la colonne ne peut pas avoir de valeur NULL, AUTO_INCREMENT pour une valeur qui s'incrémente automatiquement, PRIMARY KEY, FOREIGN KEY, etc.).

 
Exemples de création

DROP DATABASE test;
CREATE DATABASE IF NOT EXISTS test;
CREATE TABLE test.Employes (
    employe_id INT PRIMARY KEY AUTO_INCREMENT,
    prenom VARCHAR(50) NOT NULL,
    nom VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    date_naissance DATE,
    salaire DECIMAL(10, 2)
);

Rappels :

    employe_id INT PRIMARY KEY, : définit une colonne employe_id comme clé primaire de la table. Les clés primaires identifient de manière unique chaque enregistrement dans la table. Le type INT signifie que la colonne stockera des entiers. En tant que clé primaire, chaque valeur dans cette colonne doit être unique.

    prenom VARCHAR(50) NOT NULL, : crée une colonne prenom pouvant stocker des chaînes de caractères (texte) jusqu'à 50 caractères. NOT NULL signifie que chaque employé doit avoir un prénom, et cette colonne ne peut pas être laissée vide.

    nom VARCHAR(50) NOT NULL, : similaire à prenom, cette colonne stocke le nom de famille de l'employé et ne peut pas être vide.

    email VARCHAR(100) UNIQUE NOT NULL, : crée une colonne email pour stocker l'adresse email de l'employé. UNIQUE garantit que chaque email est unique dans la table, empêchant ainsi les doublons. NOT NULL indique que chaque enregistrement doit avoir une adresse email.

    date_naissance DATE, : définit une colonne date_naissance pour stocker la date de naissance de l'employé. Le type DATE est utilisé pour stocker des dates (année, mois, jour).

    salaire DECIMAL(10, 2) : crée une colonne salaire qui peut stocker des nombres décimaux avec jusqu'à 10 chiffres au total, dont 2 après la virgule, ce qui est utile pour représenter des montants d'argent. 

CREATE TABLE commandes (
    commande_id INT PRIMARY KEY AUTO_INCREMENT,
    date_commande DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    statut VARCHAR(20) DEFAULT 'En attente',
    montant_total DECIMAL(10, 2)
);

Rappels :

    DEFAULT CURRENT_TIMESTAMP définit la valeur par défaut de cette colonne à la date et l'heure actuelles au moment de l'insertion de l'enregistrement. Si aucune valeur n'est spécifiée lors de l'insertion, MySQL utilise la date et l'heure actuelles.
    DEFAULT 'En attente' spécifie que si aucune valeur n'est donnée pour cette colonne lors de l'insertion d'un enregistrement, la valeur par défaut 'En attente' sera utilisée.

CREATE TABLE utilisateurs (
    utilisateur_id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    est_actif BOOL NOT NULL DEFAULT 1,
    accepte_newsletter BOOL NOT NULL DEFAULT 0,
    est_verifie BOOL NOT NULL DEFAULT 0
);

Gestion des relations ( i changed the order )
Créer une table avec une clé étrangère

Lors de la création d'une table, vous pouvez définir une clé étrangère en utilisant la syntaxe suivante :

CREATE TABLE nom_table_enfant (
    colonne1 type_donnée,
    colonne2 type_donnée,
    ...
    CONSTRAINT nom_contrainte_fk FOREIGN KEY (colonne_fk) REFERENCES nom_table_parent(colonne_cible) [options_de_contrainte]
);

    nom_table_enfant : le nom de la table où la clé étrangère est définie.
    colonne_fk : la colonne dans la table enfant qui sera la clé étrangère.
    nom_contrainte_fk : un nom optionnel pour la contrainte de clé étrangère.
    nom_table_parent : la table contenant la clé primaire ou unique à laquelle la clé étrangère fait référence.
    colonne_cible : la colonne dans la table parent que la clé étrangère référence.
    options_de_contrainte : des instructions supplémentaires sur le comportement de la clé étrangère, telles que ON DELETE CASCADE ou ON UPDATE SET NULL.
        ON DELETE CASCADE : si un enregistrement de la table parent est supprimé, alors les enregistrements correspondants dans la table enfant seront automatiquement supprimés.
        ON DELETE SET NULL : si un enregistrement de la table parent est supprimé, les valeurs dans la colonne de clé étrangère dans les enregistrements correspondants de la table enfant seront mises à NULL.
        ON UPDATE CASCADE : si la valeur de la clé primaire ou unique dans la table parent est modifiée, alors la valeur correspondante dans la table enfant sera automatiquement mise à jour.
        ON UPDATE SET NULL : similaire à ON DELETE SET NULL, mais appliqué lors de la mise à jour de l'enregistrement parent.

 
Exemples de créations de tables avec clés étrangères

CREATE TABLE departements (
    departement_id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(255) NOT NULL
);

CREATE TABLE employes (
    employe_id INT PRIMARY KEY AUTO_INCREMENT,
    prenom VARCHAR(255) NOT NULL,
    nom VARCHAR(255) NOT NULL,
    departement_id INT,
    FOREIGN KEY (departement_id) REFERENCES departements(departement_id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

Cette structure permet de lier chaque employé à un département. Si un département est supprimé, la référence dans les employés sera mise à NULL pour éviter la rupture de lien, et si un département est mis à jour, la référence sera automatiquement mise à jour (CASCADE).

CREATE TABLE projets (
    projet_id INT AUTO_INCREMENT PRIMARY KEY,
    titre VARCHAR(255) NOT NULL,
    description TEXT,
    date_debut DATE,
    date_fin DATE
);

CREATE TABLE taches (
    tache_id INT AUTO_INCREMENT PRIMARY KEY,
    titre VARCHAR(255) NOT NULL,
    description TEXT,
    projet_id INT,
    est_complete BOOLEAN NOT NULL DEFAULT FALSE,
    FOREIGN KEY (projet_id) REFERENCES projets(projet_id)
        ON DELETE CASCADE
);

Dans cet exemple, chaque tâche est liée à un projet. La suppression d'un projet entraînera la suppression de toutes ses tâches associées (CASCADE).

 
Ajouter une contrainte de clé étrangère à une table existante

Pour ajouter une contrainte de clé étrangère :

ALTER TABLE nom_table ADD CONSTRAINT nom_contrainte FOREIGN KEY (nom_colonne) REFERENCES autre_table(colonne_dans_autre_table);

    nom_contrainte : un nom unique pour la contrainte.
    nom_colonne : la colonne dans la table actuelle qui sera la clé étrangère.
    autre_table : la table contenant la clé primaire à laquelle la clé étrangère fait référence.
    colonne_dans_autre_table : la colonne dans autre_table à laquelle nom_colonne fait référence.

 
Supprimer une Contrainte de Clé Étrangère

Pour supprimer une contrainte de clé étrangère :

ALTER TABLE nom_table DROP FOREIGN KEY nom_contrainte;

    nom_contrainte : Le nom de la contrainte de clé étrangère à supprimer.

Ces opérations sur la structure des tables doivent être utilisées avec prudence, surtout sur des tables contenant des données importantes. Il est recommandé de toujours effectuer une sauvegarde de la base de données avant d'appliquer des modifications structurelles.

Modification et suppression des tables ( i changed the order)
Introduction à la commande ALTER TABLE

La commande ALTER TABLE est utilisée pour modifier la structure d'une table existante dans une base de données.

Elle offre une grande flexibilité et permet d'effectuer de nombreuses opérations sans avoir à recréer la table.

 
Ajouter une colonne

Pour ajouter une nouvelle colonne à une table existante :

ALTER TABLE nom_table ADD nom_colonne type_de_donnée [contrainte];

    nom_table : Le nom de la table à modifier.
    nom_colonne : Le nom de la nouvelle colonne à ajouter.
    type_de_donnée : Le type de données pour la nouvelle colonne (par exemple, VARCHAR(255), INT, etc.).
    contrainte : Facultatif. Vous pouvez spécifier des contraintes pour la colonne (par exemple, NOT NULL, UNIQUE, DEFAULT 'valeur', etc.).

 
Supprimer une colonne

Pour supprimer une colonne existante :

ALTER TABLE nom_table DROP COLUMN nom_colonne;

 
Modifier le type d'une colonne

Pour changer le type de données d'une colonne existante :

ALTER TABLE nom_table MODIFY COLUMN nom_colonne nouveau_type_de_donnée;

    nom_colonne : le nom de la colonne dont le type doit être modifié.
    nouveau_type_de_donnée : le nouveau type de données pour la colonne.

 
Renommer une colonne

Pour renommer une colonne :

ALTER TABLE nom_table RENAME COLUMN nom_colonne_actuel TO nouveau_nom_colonne;

 
Changer la position d'une colonne

MySQL permet également de modifier la position d'une colonne dans la table :

ALTER TABLE nom_table MODIFY COLUMN nom_colonne type_de_donnée AFTER autre_colonne;

    nom_colonne : le nom de la colonne à déplacer.
    type_de_donnée : le type de données de la colonne (doit être répété même si non modifié).
    autre_colonne : la colonne après laquelle nom_colonne sera placée. Utilisez FIRST pour la placer en première position.

 
Suppression de tables avec DROP TABLE

La commande DROP TABLE supprime une table et toutes les données qu'elle contient. C'est une opération irréversible.

DROP TABLE nom_table;

Gestion des contraintes check
Rappels sur la contrainte CHECK

Comme nous l'avons déjà vu, la contrainte CHECK est utilisée pour limiter les valeurs qui peuvent être placées dans une colonne.

Depuis MySQL 8.0.16, cette contrainte est pleinement prise en charge, ce qui signifie que vous pouvez désormais définir des conditions qui doivent être remplies pour que les données soient insérées ou mises à jour dans une table. 

 
Syntaxe de base

La syntaxe générale pour ajouter une contrainte CHECK lors de la création d'une table est la suivante :

CREATE TABLE nom_table (
    colonne1 type_données contrainte,
    colonne2 type_données contrainte,
    ...
    CONSTRAINT nom_contrainte_check CHECK (condition)
);

Vous pouvez également ajouter une contrainte CHECK à une colonne existante en utilisant la syntaxe ALTER TABLE :

ALTER TABLE nom_table
ADD CONSTRAINT nom_contrainte_check CHECK (condition);

 
Exemples

Supposons que vous vouliez vous assurer que l'âge d'un employé est entre 18 et 65 ans. Vous pouvez le faire comme ceci :

CREATE TABLE Employes (
    employe_id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(100) NOT NULL,
    age INT,
    CONSTRAINT age_valide CHECK (age >= 18 AND age <= 65)
);

Pour vérifier que les adresses e-mail suivent un format spécifique (simple exemple avec un caractère '@'), vous pourriez utiliser :

CREATE TABLE Utilisateurs (
    utilisateur_id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255),
    CONSTRAINT email_format CHECK (email LIKE '%@%')
);

Si vous avez une table de produits et que vous voulez vous assurer que les produits appartiennent uniquement à certaines catégories :

CREATE TABLE Produits (
    produit_id INT PRIMARY KEY AUTO_INCREMENT,
    categorie VARCHAR(50),
    CONSTRAINT categorie_check CHECK (categorie IN ('Électronique', 'Livres', 'Vêtements'))
);

Pour s'assurer que les dates de début sont avant les dates de fin dans une table d'événements :

CREATE TABLE Evenements (
    evenement_id INT PRIMARY KEY AUTO_INCREMENT,
    date_debut DATE,
    date_fin DATE,
    CONSTRAINT dates_valides CHECK (date_debut < date_fin)
);

 
Bonnes pratiques

Nommez vos contraintes : donner un nom explicite à vos contraintes CHECK facilite l'identification et la gestion des contraintes dans de grandes bases de données.

Utilisez des validations précises : soyez aussi précis que possible avec vos conditions pour éviter les données inattendues.

Testez vos contraintes : avant de déployer des modifications en production, assurez vous de tester vos contraintes pour éviter des erreurs ou des comportements inattendus.

Jeux de caractères et collations
Jeux de caractères et collations
Jeux de caractères (Character Sets)

Un jeu de caractères définit l'ensemble des symboles et des encodages (c'est-à-dire, la manière dont les caractères sont stockés en mémoire ou sur disque). Par exemple, utf8mb4 est un jeu de caractères qui peut stocker n'importe quel caractère universel.

Si nous imaginons un jeu de caractères qui comprend quatre lettres (A, B, a, b) avec des codages numériques spécifiques (A = 0, B = 1, a = 2, b = 3). Chaque lettre est un symbole, et le numéro associé est son codage. Le jeu de caractères est donc la collection de ces associations symbole / codage.
Collations

Une collation est un ensemble de règles pour comparer des caractères dans un jeu de caractères particulier. Cela inclut des règles de comparaison de chaînes de caractères, comme la sensibilité à la casse et l'ordre alphabétique. Par exemple, utf8mb4_general_ci est une collation pour utf8mb4 qui compare les caractères sans tenir compte de la casse (ci pour case-insensitive).
Valeurs par défaut

Le jeu de caractères et la collation par défaut du serveur MySQL sont utf8mb4 et utf8mb4_0900_ai_ci, respectivement.

utf8mb4 est un jeu de caractères Unicode qui supporte pleinement le standard Unicode. Il est capable de stocker n'importe quel symbole Unicode, y compris les émoticônes (emoji) et les symboles spéciaux qui requièrent jusqu'à 4 octets par caractère. C'est une extension du jeu de caractères utf8 de MySQL, qui ne supporte que les caractères Unicode nécessitant jusqu'à 3 octets par caractère.

utf8mb4_0900_ai_ci est une collation pour le jeu de caractères utf8mb4. Le suffixe ai signifie "accent insensitive" (insensible aux accents), et ci signifie "case insensitive" (insensible à la casse). Cette collation est utilisée pour comparer des chaînes de caractères d'une manière qui ignore les différences de casse et d'accents, ce qui est utile pour de nombreuses applications internationales où ces différences ne doivent pas affecter les résultats de comparaison.

Cependant, MySQL offre la flexibilité de spécifier des jeux de caractères à différents niveaux : serveur, base de données, table, colonne et littéral de chaîne.

 
Système de défaut à plusieurs niveaux

MySQL utilise un système de défaut à plusieurs niveaux pour l'assignation des jeux de caractères, permettant une granularité fine dans la gestion des données textuelles :

    Serveur : le jeu de caractères et la collation par défaut pour le serveur peuvent être définis dans le fichier de configuration de MySQL (my.cnf ou my.ini).
    Base de données : chaque base de données peut avoir son propre jeu de caractères et sa collation par défaut, héritant du serveur si non spécifié.
    Table : les tables peuvent définir leur propre jeu de caractères et collation, héritant de la base de données si non spécifié.
    Colonne : les colonnes peuvent spécifier leur propre jeu de caractères et collation, héritant de la table si non spécifié.
    Littéral de chaîne : les littéraux de chaîne peuvent spécifier un jeu de caractères pour des comparaisons ou des opérations spécifiques.

 
Exemples de modification

Il est très facile de modifier la collation ou le jeu de caractères d'une table ou d'une colonne. Nous ne rentrerons pas dans les détails car c'est assez rare. Il faut avoir des besoins de performance ou des spécifications particuliers.
Modifier le jeu de caractères et la collation d'une table

ALTER TABLE ma_table CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

Définir le jeu de caractères et la collation lors de la création d'une colonne

CREATE TABLE ma_table ( ma_colonne VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL );

