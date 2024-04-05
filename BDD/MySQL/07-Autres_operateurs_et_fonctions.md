# Autres opérateurs et fonctions

Les opérateurs spécifiques aux sous-requêtes
Les opérateurs spécifiques aux sous-requêtes

Les opérateurs ANY, IN, SOME, ALL, EXISTS, et NOT EXISTS permettent d'effectuer des comparaisons complexes entre des ensembles de données.

Ces opérateurs sont utilisés avec des sous-requêtes pour filtrer les résultats selon des critères spécifiques. Seul IN peut être utilisé sans sous-requête, par exemple :

SELECT * FROM movie WHERE genre_id IN (1, 2, 3);

 
Opérateurs ANY et SOME

Les opérateurs ANY et SOME sont synonymes et permettent de vérifier si une condition est vraie pour n'importe quelle valeur d'un sous-ensemble.

 
Opérateur ALL

L'opérateur ALL vérifie si une condition est vraie pour toutes les valeurs d'un sous-ensemble.

SELECT * FROM movie
WHERE budget > ALL (SELECT budget FROM movie WHERE YEAR(release_date) = 2016);

 
Opérateurs EXISTS / NOT EXISTS

L'opérateur EXISTS teste l'existence d'enregistrements retournés par une sous-requête. Si la sous-requête retourne au moins un enregistrement, la condition est vraie.

Par exemple pour avoir tous les films avec au moins une entrée dans movie_cast :

SELECT m.title
FROM movie m
WHERE EXISTS (
    SELECT m.movie_id
    FROM movie_cast mc
    WHERE m.movie_id = mc.movie_id
);

 
Exemples pratiques
Sélectionner des films ayant un budget supérieur à celui de n'importe quel film réalisé par 'Christopher Nolan'

SELECT title, budget
FROM movie
WHERE budget > ANY (
    SELECT movie.budget
    FROM movie
    JOIN movie_crew ON movie.movie_id = movie_crew.movie_id
    JOIN person ON movie_crew.person_id = person.person_id
    WHERE person_name = 'Christopher Nolan' AND job = 'Director'
);

Trouver les films qui n'ont pas de casting dans la base de données

SELECT title
FROM movie m
WHERE NOT EXISTS (
    SELECT *
    FROM movie_cast mc
    WHERE m.movie_id = mc.movie_id
);

Les fonctions mathématiques
Les fonctions disponibles

MySQL propose un grand nombre de fonctions mathématiques :
Nom 	Description
ABS() 	Retourne la valeur absolue
ACOS() 	Retourne l'arc cosinus
ASIN() 	Retourne l'arc sinus
ATAN() 	Retourne l'arc tangente
ATAN2(), ATAN() 	Retourne l'arc tangente des deux arguments
CEIL() 	Retourne la plus petite valeur entière supérieure ou égale à l'argument
CEILING() 	Retourne la plus petite valeur entière supérieure ou égale à l'argument
CONV() 	Convertit les nombres entre différentes bases numériques
COS() 	Retourne le cosinus
COT() 	Retourne la cotangente
CRC32() 	Calcule une valeur de contrôle de redondance cyclique
DEGREES() 	Convertit les radians en degrés
EXP() 	Élève à la puissance de
FLOOR() 	Retourne la plus grande valeur entière inférieure ou égale à l'argument
LN() 	Retourne le logarithme naturel de l'argument
LOG() 	Retourne le logarithme naturel du premier argument
LOG10() 	Retourne le logarithme en base 10 de l'argument
LOG2() 	Retourne le logarithme en base 2 de l'argument
MOD() 	Retourne le reste
PI() 	Retourne la valeur de pi
POW() 	Retourne l'argument élevé à la puissance spécifiée
POWER() 	Retourne l'argument élevé à la puissance spécifiée
RADIANS() 	Retourne l'argument converti en radians
RAND() 	Retourne une valeur aléatoire à virgule flottante
ROUND() 	Arrondit l'argument
SIGN() 	Retourne le signe de l'argument
SIN() 	Retourne le sinus de l'argument
SQRT() 	Retourne la racine carrée de l'argument
TAN() 	Retourne la tangente de l'argument
TRUNCATE() 	Tronque à un nombre spécifié de décimales

 
Présentation des fonctions mathématiques les plus utilisées
ABS() : valeur absolue

Retourne la valeur absolue d'un nombre, c'est-à-dire le nombre sans son signe.

ABS(-5) retourne 5. Utilisé pour obtenir la magnitude d'un nombre indépendamment de sa direction (positif/négatif).
ROUND(X, D) : arrondissement

Arrondit un nombre à un nombre spécifié de décimales.

ROUND(3.14159, 2) retourne 3.14. Essentiel pour formater les nombres flottants lors de la présentation des résultats.
CEILING() : arrondi à l'entier supérieur

Retourne le plus petit entier supérieur ou égal au nombre donné.

CEILING(2.1) retourne 3. Utile pour des calculs de quantités ou d'allocations où les fractions ne sont pas pratiques.
FLOOR() : arrondi à l'entier inférieur

Retourne le plus grand entier inférieur ou égal au nombre donné.

FLOOR(2.9) retourne 2. Utilisé dans des scénarios où il faut arrondir vers le bas à l'entier le plus proche.
RAND() : nombre aléatoire

Génère un nombre aléatoire flottant entre 0 et 1.

RAND() pourrait retourner 0.345678. Pratique pour générer des données d'échantillonnage ou pour des opérations nécessitant un élément aléatoire.

 
Exemples

SELECT title, ROUND(vote_average, 1) AS rounded_vote_average FROM movie;

Pour sélectionner un film aléatoirement :

SELECT title FROM movie ORDER BY RAND() LIMIT 1;

Voici comment cela fonctionne :

    RAND() : cette fonction génère un nombre flottant aléatoire pour chaque ligne de la table movie. Le nombre est compris entre 0 (inclus) et 1 (exclus).

    ORDER BY RAND() : cela ordonne les lignes de la table movie de manière aléatoire en utilisant les valeurs générées par la fonction RAND(). Étant donné que RAND() génère un nouveau nombre aléatoire pour chaque ligne, l'ordre des lignes après cette opération est totalement aléatoire.

    LIMIT 1 : cette partie de la requête limite le résultat à une seule ligne. Après que les lignes ont été ordonnées de manière aléatoire, LIMIT 1 sélectionne le premier enregistrement de cet ensemble ordonné aléatoirement.

Les fonctions pour les dates et les temps
Présentation des fonctions

MySQL propose un grand nombre de fonctions pour les dates et les temps :
Nom 	Description
ADDDATE() 	Ajoute des intervalles de temps à une valeur de date
ADDTIME() 	Ajoute du temps
CONVERT_TZ() 	Convertit d'un fuseau horaire à un autre
CURDATE() 	Retourne la date actuelle
CURRENT_DATE(), CURRENT_DATE 	Synonymes pour CURDATE()
CURRENT_TIME(), CURRENT_TIME 	Synonymes pour CURTIME()
CURRENT_TIMESTAMP(), CURRENT_TIMESTAMP 	Synonymes pour NOW()
CURTIME() 	Retourne l'heure actuelle
DATE() 	Extrait la partie date d'une expression date ou datetime
DATE_ADD() 	Ajoute des intervalles de temps à une valeur de date
DATE_FORMAT() 	Formate la date comme spécifié
DATE_SUB() 	Soustrait un intervalle de temps d'une date
DATEDIFF() 	Soustrait deux dates
DAY() 	Synonyme pour DAYOFMONTH()
DAYNAME() 	Retourne le nom du jour de la semaine
DAYOFMONTH() 	Retourne le jour du mois (0-31)
DAYOFWEEK() 	Retourne l'indice du jour de la semaine
DAYOFYEAR() 	Retourne le jour de l'année (1-366)
EXTRACT() 	Extrait une partie d'une date
FROM_DAYS() 	Convertit un numéro de jour en date
FROM_UNIXTIME() 	Formate un timestamp Unix en date
GET_FORMAT() 	Retourne une chaîne de format de date
HOUR() 	Extrait l'heure
LAST_DAY 	Retourne le dernier jour du mois pour l'argument
LOCALTIME(), LOCALTIME 	Synonyme pour NOW()
LOCALTIMESTAMP, LOCALTIMESTAMP() 	Synonyme pour NOW()
MAKEDATE() 	Crée une date à partir de l'année et du jour de l'année
MAKETIME() 	Crée un temps à partir de l'heure, minute, seconde
MICROSECOND() 	Retourne les microsecondes de l'argument
MINUTE() 	Retourne la minute de l'argument
MONTH() 	Retourne le mois de la date passée
MONTHNAME() 	Retourne le nom du mois
NOW() 	Retourne la date et l'heure actuelles
PERIOD_ADD() 	Ajoute une période à une année-mois
PERIOD_DIFF() 	Retourne le nombre de mois entre deux périodes
QUARTER() 	Retourne le trimestre d'une date
SEC_TO_TIME() 	Convertit des secondes en format 'hh:mm:ss'
SECOND() 	Retourne la seconde (0-59)
STR_TO_DATE() 	Convertit une chaîne en date
SUBDATE() 	Synonyme pour DATE_SUB() lorsqu'invoqué avec trois arguments
SUBTIME() 	Soustrait des temps
SYSDATE() 	Retourne le moment auquel la fonction s'exécute
TIME() 	Extrait la portion de temps de l'expression passée
TIME_FORMAT() 	Formate comme temps
TIME_TO_SEC() 	Retourne l'argument converti en secondes
TIMEDIFF() 	Soustrait du temps
TIMESTAMP() 	Avec un seul argument, cette fonction retourne l'expression date ou datetime; avec deux arguments, la somme des arguments
TIMESTAMPADD() 	Ajoute un intervalle à une expression datetime
TIMESTAMPDIFF() 	Retourne la différence de deux expressions datetime, en utilisant les unités spécifiées
TO_DAYS() 	Retourne l'argument date converti en jours
TO_SECONDS() 	Retourne l'argument date ou datetime converti en secondes depuis l'An 0
UNIX_TIMESTAMP() 	Retourne un timestamp Unix
UTC_DATE() 	Retourne la date UTC actuelle
UTC_TIME() 	Retourne l'heure UTC actuelle
UTC_TIMESTAMP() 	Retourne la date et l'heure UTC actuelles
WEEK() 	Retourne le numéro de la semaine
WEEKDAY() 	Retourne l'indice du jour de la semaine
WEEKOFYEAR() 	Retourne la semaine calendaire de la date (1-53)
YEAR() 	Retourne l'année
YEARWEEK() 	Retourne l'année et la semaine

 
Fonctions les plus utilisées
CURDATE() - Retourne la date actuelle.

Par exemple : SELECT CURDATE(); pourrait retourner '2024-03-15', la date d'aujourd'hui.
CURTIME() - Retourne l'heure actuelle.

Exemple : SELECT CURTIME(); pourrait donner '13:45:07', représentant l'heure actuelle.
NOW() - Retourne la date et l'heure actuelles.

Exemple : SELECT NOW(); peut retourner '2024-02-15 13:45:07', indiquant à la fois la date et l'heure actuelles.
DATE() - Extrait la partie date d'une expression date ou datetime.

Exemple : SELECT DATE('2024-02-15 13:45:07'); retourne '2024-02-15', isolant la partie date de l'expression datetime.
DATE_FORMAT(date, format) - Formate la date comme spécifié.

La chaîne de format peut inclure une variété de spécificateurs qui indiquent quelle partie de la valeur de date doit être affichée et comment. Voici quelques exemples de spécificateurs de format :

    %Y : Année sur quatre chiffres.
    %m : Mois sous forme de deux chiffres.
    %d : Jour du mois sous forme de deux chiffres.
    %H : Heure (00 à 23).
    %i : Minutes (00 à 59).
    %s : Secondes (00 à 59).
    %a : Nom abrégé du jour de la semaine.
    %b : Nom abrégé du mois.
    %W : Nom complet du jour de la semaine.
    %M : Nom complet du mois.

Exemple : SELECT DATE_FORMAT(NOW(), '%W, %M %d, %Y'); pourrait donner 'Vendredi, Février 15, 2024', formatant la date actuelle dans un format lisible.
DATEDIFF() - Soustrait deux dates et retourne la différence en jours.

Exemple : SELECT DATEDIFF('2024-02-15', '2024-01-01'); retourne 45, calculant le nombre de jours entre deux dates.
TIMESTAMPDIFF() - Retourne la différence entre deux expressions datetime, en utilisant les unités spécifiées.

Exemple : SELECT TIMESTAMPDIFF(MONTH, '2024-01-01', '2024-02-15'); donne 1, montrant le nombre de mois complets entre deux dates.
ADDDATE() et DATE_ADD() - Ajoutent des intervalles de temps à une valeur de date.

Exemple : SELECT ADDDATE('2024-02-15', INTERVAL 10 DAY); retourne '2024-02-25', ajoutant 10 jours à la date spécifiée.
SUBDATE() et DATE_SUB() - Soustraient un intervalle de temps d'une date.

Exemple : SELECT SUBDATE('2024-02-15', INTERVAL 1 MONTH); donne '2024-01-15', soustrayant un mois de la date spécifiée.
UNIX_TIMESTAMP() - Convertit une date en timestamp Unix.

Exemple : SELECT UNIX_TIMESTAMP('2024-02-15 13:45:07'); pourrait retourner 1708160707, le nombre de secondes depuis le '1970-01-01 00:00:00' UTC jusqu'à la date et l'heure spécifiées.

 
Exemples
Trouver les films sortis cette année

Cette requête utilise CURDATE() pour obtenir la date actuelle et YEAR() pour extraire l'année actuelle. Elle récupère les titres des films (title) et leurs dates de sortie (release_date) de la table movie pour les films sortis cette année. 

Il n'y aura ici pas de résultat car la table n'a que des films jusqu'en 2016 comme vu précédemment.

SELECT title, release_date
FROM movie
WHERE YEAR(release_date) = YEAR(CURDATE());

Calculer l'âge moyen des films par genre

Cette requête combine YEAR(), CURDATE(), et AVG() pour calculer l'âge moyen des films dans chaque genre. Elle joint les tables movie et genre via une table de liaison pour grouper les résultats par genre.

SELECT
    genre_name,
    AVG(YEAR(CURDATE()) - YEAR(release_date)) AS average_age
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
GROUP BY genre_name;

Les fonctions pour les chaînes de caractères
Présentation des fonctions

MySQL propose un grand nombre de fonctions pour les chaînes de caractères :
Nom 	Description
ASCII() 	Retourne la valeur numérique du caractère le plus à gauche
BIN() 	Retourne une chaîne contenant la représentation binaire d'un nombre
BIT_LENGTH() 	Retourne la longueur de l'argument en bits
CHAR() 	Retourne le caractère pour chaque entier passé
CHAR_LENGTH() 	Retourne le nombre de caractères dans l'argument
CHARACTER_LENGTH() 	Synonyme de CHAR_LENGTH()
CONCAT() 	Retourne une chaîne concaténée
CONCAT_WS() 	Retourne concaténer avec séparateur
ELT() 	Retourne la chaîne à l'indice numéro
EXPORT_SET() 	Retourne une chaîne telle que pour chaque bit défini dans les bits de valeur, vous obtenez une chaîne on et pour chaque bit non défini, vous obtenez une chaîne off
FIELD() 	Index (position) du premier argument dans les arguments suivants
FIND_IN_SET() 	Index (position) du premier argument dans le second argument
FORMAT() 	Retourne un nombre formaté au nombre spécifié de décimales
HEX() 	Représentation hexadécimale d'une valeur décimale ou chaîne
INSERT() 	Insère une sous-chaîne à la position spécifiée jusqu'au nombre spécifié de caractères
INSTR() 	Retourne l'indice de la première occurrence de la sous-chaîne
LCASE() 	Synonyme de LOWER()
LEFT() 	Retourne le nombre spécifié de caractères les plus à gauche
LENGTH() 	Retourne la longueur d'une chaîne en octets
LIKE 	Correspondance de motif simple
LOAD_FILE() 	Charge le fichier nommé
LOCATE() 	Retourne la position de la première occurrence de la sous-chaîne
LOWER() 	Retourne l'argument en minuscules
LPAD() 	Retourne l'argument chaîne, complété à gauche avec la chaîne spécifiée
LTRIM() 	Supprime les espaces de début
MAKE_SET() 	Retourne un ensemble de chaînes séparées par des virgules qui ont le bit correspondant dans les bits défini
MATCH() 	Effectue une recherche en texte intégral
MID() 	Retourne une sous-chaîne à partir de la position spécifiée
NOT LIKE 	Négation de la correspondance de motif simple
NOT REGEXP 	Négation de REGEXP
OCT() 	Retourne une chaîne contenant la représentation octale d'un nombre
OCTET_LENGTH() 	Synonyme de LENGTH()
ORD() 	Retourne le code caractère pour le caractère le plus à gauche de l'argument
POSITION() 	Synonyme de LOCATE()
QUOTE() 	Échappe l'argument pour utilisation dans une déclaration SQL
REGEXP 	Si la chaîne correspond à l'expression régulière
REGEXP_INSTR() 	Indice de début de la sous-chaîne correspondant à l'expression régulière
REGEXP_LIKE() 	Si la chaîne correspond à l'expression régulière
REGEXP_REPLACE() 	Remplace les sous-chaînes correspondant à l'expression régulière
REGEXP_SUBSTR() 	Retourne la sous-chaîne correspondant à l'expression régulière
REPEAT() 	Répète une chaîne le nombre de fois spécifié
REPLACE() 	Remplace les occurrences d'une chaîne spécifiée
REVERSE() 	Inverse les caractères dans une chaîne
RIGHT() 	Retourne le nombre spécifié de caractères les plus à droite
RLIKE 	Si la chaîne correspond à l'expression régulière
RPAD() 	Ajoute la chaîne le nombre de fois spécifié
RTRIM() 	Supprime les espaces de fin
SOUNDEX() 	Retourne une chaîne soundex
SOUNDS LIKE 	Compare les sons
SPACE() 	Retourne une chaîne du nombre d'espaces spécifié
STRCMP() 	Compare deux chaînes
SUBSTR() 	Retourne la sous-chaîne spécifiée
SUBSTRING() 	Retourne la sous-chaîne spécifiée
SUBSTRING_INDEX() 	Retourne une sous-chaîne d'une chaîne avant le nombre spécifié d'occurrences du délimiteur
TRIM() 	Supprime les espaces de début et de fin
UCASE() 	Synonyme de UPPER()
UNHEX() 	Retourne une chaîne contenant la représentation hexadécimale d'un nombre
UPPER() 	Convertit en majuscules
WEIGHT_STRING() 	Retourne la chaîne de poids pour une chaîne

 
Exemples pratiques
Trouver le nombre de caractères dans les titres des films :

SELECT title, CHAR_LENGTH(title) AS title_length FROM movie;

Convertir tous les noms de genres en majuscules pour uniformiser l'affichage :

SELECT UPPER(name) AS genre_uppercase FROM genre;

Trouver des films dont le titre contient un mot spécifique, en ignorant la casse :

SELECT title FROM movie WHERE LOWER(title) LIKE '%war%';

Les fonctions et opérateurs de contrôle du flux
Définition

Les opérateurs de contrôle de flux permettent de manipuler les résultats des requêtes en fonction de certaines conditions.

Ces opérateurs et fonctions incluent CASE, IF(), IFNULL(), et NULLIF().

 
Opérateur CASE

L'opérateur CASE évalue une liste de conditions et retourne une valeur lorsque la première condition est remplie.

Si aucune condition n'est remplie, il retourne la valeur spécifiée dans la clause ELSE, ou NULL si cette clause n'est pas présente.

Il existe deux syntaxes pour l'opérateur CASE :

CASE valeur
    WHEN valeur_comparée THEN résultat
    [WHEN valeur_comparée THEN résultat ...]
    [ELSE résultat]
END

On peut aussi utiliser cette syntaxe :

CASE
    WHEN condition THEN résultat
    [WHEN condition THEN résultat ...]
    [ELSE résultat]
END

 
Fonction IF()

La fonction IF() permet d'exécuter une condition.

Si cette condition est vraie, la fonction retourne une valeur ; sinon, elle en retourne une autre.

IF(condition, valeur_si_vrai, valeur_si_faux)

 
Fonction IFNULL()

La fonction IFNULL() retourne la première expression si elle n'est pas NULL. Sinon, elle retourne la seconde expression.

IFNULL(expression1, expression2)

 
Fonction NULLIF()

La fonction NULLIF() compare deux expressions. Si elles sont égales, la fonction retourne NULL. Sinon, elle retourne la première expression.

NULLIF(expression1, expression2)

 
Exemples pratiques
Utilisation de CASE pour catégoriser les films par année de sortie

SELECT
    title,
    release_date,
    CASE
        WHEN YEAR(release_date) <= 2000 THEN 'Classique'
        WHEN YEAR(release_date) BETWEEN 2001 AND 2010 THEN 'Moderne'
        ELSE 'Récent'
    END AS categorie
FROM
    movie;

Utilisation de CASE pour classifier les films par popularité

SELECT
    title,
    popularity,
    CASE
        WHEN popularity < 5 THEN 'Peu populaire'
        WHEN popularity BETWEEN 5 AND 15 THEN 'Moyennement populaire'
        ELSE 'Très populaire'
    END AS niveau_popularite
FROM
    movie;

Utilisation de IF() pour différencier les films selon qu'ils ont généré des bénéfices ou non, en prenant en compte le budget et les revenus

SELECT
    title,
    budget,
    revenue,
    IF(revenue > budget,
        'Bénéficiaire',
        'Non bénéficiaire') AS statut_financier
FROM
    movie;

