# Les variables et les procédures et les fonctions stockées

Les variables utilisateurs
Les variables

Dans MySQL, les variables sont des espaces de stockage nommés qui peuvent contenir des données. Il y a plusieurs types de variables, chacune avec son propre usage et portée :

    Variables système : ce sont des variables préconfigurées qui contrôlent le comportement de MySQL. Elles peuvent être globales, affectant le serveur entier, ou de session, affectant la connexion courante. Exemple : max_connections, wait_timeout.

    Variables utilisateur : ce sont des variables définies par l'utilisateur, spécifiques à chaque session, et qui ne nécessitent pas de déclaration préalable. Elles sont préfixées par @ et leur valeur est maintenue jusqu'à la fin de la session. Exemple : SET @mon_compteur = 1;.

    Variables locales : utilisées au sein des blocs de code comme les procédures stockées et les fonctions, elles doivent être déclarées avec DECLARE et leur portée est limitée au bloc dans lequel elles sont définies.

Dans cette leçon nous allons commencer par voir les variables utilisateur.

Nous verrons qu'avec les procédures stockées nous utiliserons des variables utilisateur mais aussi des variables locales.

 
Les variables utilisateur

Les variables utilisateur offrent un moyen pratique de stocker et de manipuler des données temporairement durant l'exécution de vos requêtes SQL.

Elles sont spécifiques à chaque session de connexion à la base de données et se distinguent par leur préfixe @.
Déclaration et initialisation

Les variables utilisateur n'ont pas besoin d'être déclarées avant d'être utilisées.

Vous pouvez les initialiser et leur attribuer une valeur en utilisant l'instruction SET ou l'opérateur d'assignation := ou l'instruction INTO.

SET @ma_variable = 100;
-- Ou
SELECT @ma_variable := 100;

INTO est généralement utilisé pour stocker le résultat d'une requête SELECT directement dans une variable.

Il peut aussi être utilisé pour stocker les résultats dans une liste de variables si la requête renvoie plusieurs colonnes.

SELECT colonne1, colonne2 INTO @var1, @var2 FROM table;

Utilisation

Une fois initialisée, la variable utilisateur peut être utilisée dans des requêtes SQL au lieu de valeurs constantes.

SELECT @ma_variable;
UPDATE ma_table SET colonne = @ma_variable WHERE id = 1;

 
Exemples
Stockage d'un résultat simple

Supposons que vous vouliez stocker le nombre total de films dans la base de données pour le réutiliser dans d'autres requêtes plus tard (mais durant la même session) :

SELECT COUNT(*) INTO @nombre_total_films FROM movies;

Vous pourrez ensuite récupérer la valeur :

SELECT @nombre_total_films;

Utilisation dans une condition

Vous pouvez utiliser une variable utilisateur pour stocker une valeur à utiliser dans des conditions de requête.

SET @annee_recherche = 2000;
SELECT * FROM movie WHERE YEAR(release_date) = @annee_recherche;

Stockage de valeurs agrégées

Stockez le résultat d'une fonction d'agrégation pour l'utiliser dans des calculs ultérieurs.

SELECT MAX(budget) INTO @budget_max FROM movie;
SELECT title FROM movie WHERE budget = @budget_max;

Passage de valeurs entre requêtes

Les variables utilisateur peuvent servir à transmettre des valeurs entre les requêtes.

SET @id_film = (SELECT movie_id FROM movie WHERE title = 'Inception');
UPDATE movie SET revenue = revenue * 1.1 WHERE movie_id = @id_film;

Introduction aux procédures stockées
Qu'est-ce qu'une procédure stockée ?

Une procédure stockée est un ensemble d'instructions SQL précompilées que vous pouvez sauvegarder et exécuter à plusieurs reprises.

Elles sont particulièrement utiles pour :

    encapsuler des logiques complexes
    réduire la duplication du code
    améliorer la performance grâce à la minimisation du trafic réseau entre les clients et le serveur

 
Créer une procédure stockée

Pour créer une procédure stockée dans MySQL, utilisez la syntaxe suivante :

DELIMITER //

CREATE PROCEDURE nom_procedure()
BEGIN
  -- Votre code SQL ici
END //

DELIMITER ;

DELIMITER change le délimiteur par défaut de MySQL pour permettre l'utilisation du ; dans le corps de la procédure.

BEGIN et END entourent le corps de la procédure.

 
Utiliser une procédure stockée

Pour appeler une procédure stockée, utilisez CALL suivi du nom de la procédure :

CALL NomProcedure();

 
Exemple
Liste des films et de leur statut de production

Cette procédure récupère tous les films avec leur titre et leur statut de production (par exemple, 'Released', 'Post-Production').

DELIMITER //

CREATE PROCEDURE ListerStatutFilms()
BEGIN
  SELECT title, movie_status FROM movie;
END //

DELIMITER ;

Pour appeler cette procédure : 

CALL ListerStatutFilms();

Liste des personnes et leur rôle le plus récent dans un film

Cette procédure va lister chaque personne et le nom du dernier film dans lequel elle a joué, en se basant sur l'ordre des entrées dans la table de distribution des films.

DELIMITER //

CREATE PROCEDURE DernierRoleParPersonne()
BEGIN
  SELECT
    person_name,
    title AS dernier_film,
    dernier_film_id
	FROM
		(SELECT
			movie_cast.person_id, MAX(movie_cast.movie_id) AS dernier_film_id
		FROM
			movie_cast
		GROUP BY movie_cast.person_id) AS d
	JOIN
		person USING(person_id)
	JOIN
		movie AS m ON m.movie_id = d.dernier_film_id;
END //

DELIMITER ;

Voici une explication détaillée de la procédure stockée :

    DELIMITER // : cette commande change le délimiteur par défaut de MySQL (qui est ;) en //. Cela est nécessaire car le corps de la procédure stockée contient des ;, qui autrement seraient interprétés comme la fin de la commande CREATE PROCEDURE, ce qui causerait une erreur de syntaxe.

    CREATE PROCEDURE DernierRoleParPersonne() : c'est la commande qui commence la définition de la procédure stockée appelée DernierRoleParPersonne. Les parenthèses après le nom indiquent que cette procédure ne prend aucun paramètre.

    BEGIN ... END : ces mots-clés délimitent le corps de la procédure stockée. Tout ce qui est écrit entre BEGIN et END fait partie de la procédure.

    La requête SQL à l'intérieur du corps de la procédure fait plusieurs choses :
        La sous-requête (SELECT movie_cast.person_id, MAX(movie_cast.movie_id) AS dernier_film_id FROM movie_cast GROUP BY movie_cast.person_id) AS d sélectionne l'ID le plus élevé de movie_id pour chaque person_id dans la table movie_cast, en supposant que c'est le dernier film dans lequel la personne a joué. Elle utilise GROUP BY pour regrouper toutes les entrées de movie_cast par person_id, et MAX(movie_cast.movie_id) pour trouver l'ID le plus élevé (dernier) pour chaque groupe.
        Ensuite, la sous-requête est jointe à la table person avec JOIN person USING(person_id) pour récupérer le nom de la personne correspondant à chaque person_id.
        Enfin, elle est jointe à la table movie avec JOIN movie AS m ON m.movie_id = d.dernier_film_id pour obtenir le titre du film correspondant à chaque movie_id trouvé.

    DELIMITER ; : Cette commande remet le délimiteur par défaut à ; après la fin de la définition de la procédure.

Pour l'appeler :

CALL DernierRoleParPersonne();

Procédures stockées et paramètres
Les paramètres des procédures stockées

Les procédures stockées peuvent avoir trois types de paramètres : IN, OUT, et INOUT.

    IN : les paramètres IN permettent de passer des valeurs à la procédure stockée. Ils sont utilisés comme des constantes à l'intérieur de la procédure. Tout changement apporté à ces paramètres à l'intérieur de la procédure n'affecte pas la valeur du paramètre qui a été passé.

    OUT : les paramètres OUT sont utilisés pour retourner des valeurs de la procédure au programme appelant. La procédure peut modifier la valeur du paramètre OUT et cette nouvelle valeur est disponible après l'exécution de la procédure.

    INOUT : les paramètres INOUT sont une combinaison des types IN et OUT. Ils permettent à la fois de passer des valeurs à la procédure et de retourner des résultats via le même paramètre.

Note : les arguments sont les valeurs passées.

La syntaxe est :

TYPE_DE_PARAMETRE nomParamètre TYPE_DONNEE [DEFAULT valeur]

 
Valeurs par défaut

Il est souvent utile de définir des valeurs par défaut pour les paramètres d'une procédure stockée.

Si un paramètre NULL est fourni lors de l'appel de la procédure, la valeur par défaut est utilisée à la place.

Par exemple :

DELIMITER //

CREATE PROCEDURE AddGenre(
    IN genreName VARCHAR(255),
    IN description TEXT
)
BEGIN
    IF description IS NULL THEN
      SET description = 'Valeur par défaut';
    END IF;
    INSERT INTO genre (genre_name, description) VALUES (genreName, description);
END //

DELIMITER ;

 
Exemples
Procédure stockée avec un paramètre IN

Supposons que vous vouliez obtenir tous les films d'un genre spécifique.

DELIMITER //

CREATE PROCEDURE GetMoviesByGenre(IN genreName VARCHAR(255))
BEGIN
    SELECT title, release_date
    FROM movie
    JOIN movie_genres USING (movie_id)
    JOIN genre USING (genre_id)
    WHERE genre_name = genreName;
END //

DELIMITER ;

Pour appeler cette procédure en passant en argument le genre :

CALL GetMoviesByGenre('Action');

Procédure stockée avec un paramètre OUT

Supposons que vous vouliez compter le nombre de films d'une certaine catégorie.

DELIMITER //

CREATE PROCEDURE CountMoviesByGenre(IN genreName VARCHAR(255), OUT count INT)
BEGIN
    SELECT COUNT(*)
    INTO count
    FROM movie
    JOIN movie_genres USING (movie_id)
    JOIN genre USING (genre_id)
    WHERE genre_name = genreName;
END //

DELIMITER ;

Pour appeler cette procédure et récupérer le résultat :

CALL CountMoviesByGenre('Comedy', @movieCount);
SELECT @movieCount;

Procédure stockée avec un paramètre INOUT

Imaginons que vous voulez mettre à jour le statut d'un film et vérifier le statut précédent.

DELIMITER //

CREATE PROCEDURE UpdateMovieStatus(INOUT movieId INT, INOUT status VARCHAR(255))
BEGIN
    -- Sauvegarde le statut actuel dans le paramètre 'status'
    SELECT movie_status INTO status FROM movie WHERE movie_id = movieId;
    -- Met à jour le statut
    UPDATE movie SET movie_status = status WHERE movie_id = movieId;
END //

DELIMITER ;

Pour appeler cette procédure :

SET @movieId = 1;
SET @newStatus = 'Released';
CALL UpdateMovieStatus(@movieId, @newStatus);
SELECT @newStatus;  -- Renvoie le statut précédent

Les variables locales dans les procédures stockées
Les variables Locales

Les variables locales sont des identificateurs qui stockent des valeurs temporaires.

Elles sont utilisées au sein des blocs de code, comme les procédures stockées, les fonctions ou les triggers.

Elles ont une portée limitée au bloc de code (entre BEGIN ... END) dans lequel elles sont déclarées et ne sont pas accessibles en dehors de celui-ci.
Déclaration des variables locales

Pour déclarer une variable locale, on utilise la commande DECLARE au début d'un bloc de code, suivi du nom de la variable et de son type de données.

DECLARE ma_variable INT;

Lors de la déclaration, il est possible d'initialiser la variable avec une valeur.

DECLARE ma_variable INT DEFAULT 10;

 
Utilité

L'utilisation de variables locales dans une procédure stockée, peut être avantageuse pour plusieurs raisons, même si dans certains cas, elles pourraient être omises :

    Clarté et maintenance : les variables locales nomment explicitement les valeurs importantes dans le cadre de la procédure, ce qui rend le code plus lisible et plus facile à maintenir. Elles agissent comme des étiquettes descriptives pour les données manipulées.

    Performance : dans des cas plus complexes, l'utilisation de variables locales peut minimiser le nombre de requêtes à la base de données. Par exemple, stocker un résultat intermédiaire dans une variable locale évite de faire plusieurs requêtes pour obtenir la même information.

    Réutilisation : une fois qu'une valeur est récupérée dans une variable locale, elle peut être réutilisée plusieurs fois dans la procédure sans nécessiter de nouvelles requêtes SQL.

    Modularité : les variables locales permettent de décomposer des opérations complexes en étapes plus petites et gérables, ce qui simplifie le débogage et le test de parties individuelles de la procédure.

    Sécurité des types : en utilisant des variables locales, on s'assure que les données sont du type attendu. Cela peut prévenir des erreurs subtiles qui pourraient survenir si les types de données n'étaient pas correctement gérés.

    Gestion des erreurs : les variables locales peuvent être utilisées pour gérer les erreurs de manière plus granulaire, en permettant des vérifications et des contrôles conditionnels basés sur les états intermédiaires de la procédure.

 
Exemple

Prenons un exemple complet :

DELIMITER //

CREATE PROCEDURE FindLatestMovieByDirector(IN directorName VARCHAR(255))
BEGIN
  -- Déclaration de variables locales pour stocker l'ID et le titre du dernier film
  DECLARE latestMovieID INT;
  DECLARE latestMovieTitle VARCHAR(255);
  DECLARE latestMovieReleaseDate DATE;

  -- Recherche de l'ID du film le plus récent du réalisateur donné
  SELECT movie_id, title, MAX(release_date) INTO latestMovieID, latestMovieTitle, latestMovieReleaseDate
  FROM movie
  JOIN movie_crew USING(movie_id)
  JOIN person USING(person_id)
  WHERE person_name = directorName AND job = 'Director'
  GROUP BY movie_id
  ORDER BY release_date DESC
  LIMIT 1;

  -- Affichage des détails du dernier film
  SELECT latestMovieID AS 'ID', latestMovieTitle AS 'Title', latestMovieReleaseDate AS 'Release Date';
END //

DELIMITER ;

Nous pouvons ensuite utiliser cette procédure :

CALL FindLatestMovieByDirector('Christopher Nolan');

Validation des procédures stockées
Validation

La validation consiste à vérifier que les données entrées dans une procédure stockée sont correctes et respectent certaines règles ou formats attendus.

Cela peut inclure la vérification du type de données, des contraintes de clé étrangère (nous verrons les contraintes plus loin dans la formation), etc.

 
L'instruction SIGNAL

SIGNAL est une instruction qui permet à une procédure stockée, une fonction ou un déclencheur de retourner une erreur.

L'instruction SIGNAL est utilisée pour générer une condition d'erreur, ce qui provoque normalement l'interruption de l'exécution du bloc de code courant.

SQLSTATE est une chaîne de cinq caractères qui signale une exception. Les états SQL sont définis par la norme SQL et 45000 est une valeur générique qui représente une "exception non gérée par l'utilisateur". C'est souvent utilisé pour les erreurs définies par l'utilisateur.

Voici quelques valeurs SQLSTATE couramment utilisées avec SIGNAL :

    '01000' : avertissement général.

    '02000' : non trouvé. Cela pourrait indiquer qu'une recherche ou une requête n'a pas retourné de résultats, par exemple dans le cas d'une instruction SELECT qui ne trouve aucune ligne correspondante.

    '23000' : violation de contrainte d'intégrité, par exemple une violation de clé étrangère ou de clé primaire.

    '45000' : exception définie par l'utilisateur. C'est un code d'état générique que vous pouvez utiliser pour signaler des erreurs d'application personnalisées.

    'HY000' : erreur générale SQLSTATE. Si aucun autre code SQLSTATE n'est approprié, HY000 est utilisé.

Si vous voulez la liste de tous les codes standards, c'est par ici.

MESSAGE_TEXT est une variable de l'instruction SIGNAL qui permet de définir le message d'erreur retourné par l'exception.

 
Exemples
Exemple simple

DELIMITER //

CREATE PROCEDURE CheckAge(age INT)
BEGIN
  IF age < 18 THEN
    SIGNAL SQLSTATE '45000'
      SET MESSAGE_TEXT = 'Vous devez être âgé d’au moins 18 ans.';
  END IF;
END //

DELIMITER ;

Dans cet exemple, si la procédure stockée CheckAge est appelée avec une valeur age inférieure à 18, l'instruction SIGNAL déclenchera une exception avec le message "Vous devez être âgé d’au moins 18 ans."

Cela peut être utile pour valider les entrées dans les procédures stockées et assurer que les règles métier sont respectées.
Exemple

Prenons un exemple :

DELIMITER //

CREATE PROCEDURE AddMovie(
    IN titleInput VARCHAR(255),
    IN releaseDateInput DATE
)
BEGIN
    -- Validation de l'entrée pour le titre
    IF titleInput IS NULL OR titleInput = '' THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Le titre ne peut pas être vide.';
    END IF;

    -- Validation de la date de sortie
    IF releaseDateInput > CURRENT_DATE() THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'La date de sortie ne peut pas être dans le futur.';
    END IF;

    -- Insertion après validation
    INSERT INTO movie (title, release_date) VALUES (titleInput, releaseDateInput);
END //

DELIMITER ;

Dans cet exemple, la procédure AddMovie valide que le titre du film n'est pas vide et que la date de sortie n'est pas dans le futur avant d'insérer les données dans la base de données.

Essayez ensuite :

CALL AddMovie('Bob leponge', '2050-2-2');

Ou :

CALL AddMovie('', '2030-2-2');

Les fonctions stockées
Définition

Les fonctions stockées sont des routines qui vous permettent de regrouper des instructions SQL pour créer des opérations réutilisables à la demande, semblables aux fonctions dans les langages de programmation.

Contrairement aux procédures stockées, les fonctions stockées retournent toujours une valeur.

 
Création d'une fonction stockée

Pour créer une fonction stockée, on utilise la syntaxe CREATE FUNCTION.

Par exemple :

DELIMITER //

CREATE FUNCTION GetGreeting(name VARCHAR(100))
RETURNS VARCHAR(150)
DETERMINISTIC
BEGIN
    RETURN CONCAT('Bonjour ', name, ' !');
END //

DELIMITER ;

Cette fonction prend un nom en paramètre et retourne une salutation personnalisée.

    RETURNS: spécifie le type de la valeur de retour de la fonction.
    DETERMINISTIC: indique si la fonction retourne le même résultat pour les mêmes entrées à chaque appel.

Paramètres des fonctions stockées

Les fonctions stockées peuvent avoir des paramètres qui passent des données à la fonction.

Contrairement aux procédures stockées, les fonctions ne peuvent avoir que des paramètres IN.
Variables locales

Dans le corps d'une fonction, vous pouvez déclarer des variables locales qui ne sont visibles que dans la fonction.
Retourner une valeur

Utilisez l'instruction RETURN pour retourner une valeur :

RETURN expression;

 
Utilisation des fonctions stockées

Pour appeler une fonction stockée, vous l'utilisez comme n'importe quelle autre fonction dans une requête SQL :

SELECT NomFonction(arg1, arg2...);
-- Ou par exemple : --
SELECT col1, NomFonction(id) FROM table;

 
Exemples
Trouver l'âge d'un film

DELIMITER //

CREATE FUNCTION GetMovieAge(titre VARCHAR(1000))
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE release_year INT;
    SELECT YEAR(release_date) INTO release_year FROM movie WHERE title = titre;
    RETURN YEAR(CURRENT_DATE) - release_year;
END //

DELIMITER ;

Puis :

SELECT title, GetMovieAge(title) FROM movie;

 
Différences clés procédures / fonctions stockées

Retour de valeur : les fonctions retournent toujours une valeur. Les procédures ne retournent pas de valeur, mais peuvent retourner des paramètres de sortie.

Utilisation dans une requête SQL : les fonctions peuvent être utilisées dans des requêtes SQL, tandis que ce n'est pas possible avec des procédures.

Invocations : les procédures sont appelées avec la commande CALL, tandis que les fonctions sont invoquées comme n'importe quelle autre fonction SQL.

Complexité des opérations : les procédures sont plus adaptées aux opérations complexes et peuvent inclure des instructions telles que CREATE TABLE, ce qui n'est pas possible avec les fonctions.

Utilisez donc une procédure stockée pour encapsuler une logique complexe, pour améliorer les performances des tâches répétitives, et pour manipuler ou modifier des données.

Et utilisez une fonction stockée plutôt lorsque vous avez besoin de calculs ou de transformations qui peuvent être encapsulés dans une fonction et réutilisés dans plusieurs requêtes SQL.

