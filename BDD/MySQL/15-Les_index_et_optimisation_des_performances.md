# Les index et optimisation des performances

Introduction aux index avec MySQL
Qu'est-ce qu'un index ?

Un index est une structure de données associée à une table de base de données qui permet d'accélérer les opérations de recherche et de tri sur cette table.

Les index sont utilisés pour trouver rapidement les lignes d'une table qui correspondent à une condition de recherche spécifiée.

Sans index, MySQL doit lire chaque ligne de la table pour trouver les lignes correspondantes à une requête. Avec un index, MySQL peut rapidement localiser les données sans avoir à parcourir toute la table, ce qui réduit considérablement le temps d'accès.

Voyons deux analogies courantes pour comprendre l'intérêt des index.
La bibliothèque

Imaginez que vous êtes dans une grande bibliothèque remplie de milliers de livres. Votre tâche est de trouver un livre spécifique. Sans un système d'organisation, vous devriez parcourir chaque étagère et lire chaque titre jusqu'à trouver le livre désiré, ce qui pourrait prendre énormément de temps.

Ici, la bibliothèque représente votre base de données, et les livres représentent les lignes de données dans une table. Parcourir chaque étagère est similaire à parcourir chaque ligne de la table pour trouver une donnée spécifique, une opération qui peut être très lente pour de grandes quantités de données.

Un index, dans ce contexte, est comparable au catalogue de la bibliothèque, qui vous permet de savoir exactement où se trouve le livre que vous cherchez. Au lieu de parcourir chaque étagère, vous consultez le catalogue, trouvez la localisation du livre, et vous dirigez directement vers lui. De la même manière, un index dans une base de données permet à MySQL de trouver rapidement les données recherchées sans avoir à lire chaque ligne de la table.
L'annuaire

Pensez à comment vous utilisez un annuaire téléphonique pour trouver le numéro de quelqu'un. L'annuaire est organisé alphabétiquement, ce qui vous permet de sauter directement vers la section appropriée (disons la lettre "M" pour "Martin") sans avoir à lire chaque nom depuis le début. Une fois que vous êtes dans la bonne section, il ne vous reste plus qu'à parcourir une petite portion de la page pour trouver le nom spécifique et obtenir le numéro de téléphone associé.

Dans cette analogie, l'annuaire téléphonique représente votre table de base de données, et les noms classés alphabétiquement sont les données. L'organisation alphabétique de l'annuaire fonctionne comme un index : elle vous permet de trouver rapidement l'information recherchée (le numéro de téléphone) en réduisant considérablement le nombre de pages (ou lignes) à examiner.

Ces deux analogies illustrent comment les index rendent la recherche de données spécifiques rapide et efficace, en évitant de devoir examiner chaque enregistrement un par un. En pratique, les index sont essentiels pour optimiser les performances des requêtes dans les bases de données, surtout lorsque les volumes de données sont importants.

 
Avantages et inconvénients des index

Les index ont plusieurs avantages :

    Ils améliorent la vitesse des requêtes de base de données.
    Ils permettent de faire respecter certaines contraintes de base de données, comme les contraintes d'unicité et de clé étrangère.

Cependant, ils ont aussi quelques inconvénients :

    Ils prennent de l'espace disque pour stocker les structures d'index.
    Ils peuvent ralentir les opérations d'écriture (INSERT, UPDATE, DELETE) car les index doivent être mis à jour à chaque modification de données.

 
Utilisation des index par MySQL

MySQL utilise les index pour les opérations suivantes :

    Recherche avec WHERE : les index permettent de trouver rapidement les lignes correspondant à une clause WHERE.

    Élimination des lignes : si plusieurs index sont disponibles, MySQL choisit celui qui réduit le plus le nombre de lignes à examiner.

    Index sur plusieurs colonnes : un index sur plusieurs colonnes permet des recherches sur n'importe quel préfixe de gauche de l'index. Par exemple, un index sur (col1, col2, col3) supporte les recherches sur (col1), (col1, col2), et (col1, col2, col3).

    Joins : les index améliorent l'efficacité des opérations de jointure, surtout lorsque les colonnes impliquées sont de types et de tailles identiques.

    Comparaisons de chaînes : pour que les index soient utilisés lors de comparaisons entre colonnes de chaînes non binaires, celles-ci doivent utiliser le même jeu de caractères.

    MIN() et MAX() : MySQL peut optimiser la recherche des valeurs MIN() ou MAX() pour une colonne indexée spécifique, ce qui peut accélérer les requêtes.

    Tri et regroupement : les index sont utilisés pour trier ou regrouper des tables, en particulier si le tri ou le regroupement s'effectue sur un préfixe de gauche d'un index utilisable.

    Index couvrant : si une requête n'utilise que des colonnes qui sont toutes incluses dans un index, MySQL peut récupérer les données directement de l'arbre d'index sans avoir à lire les lignes de la table, ce qui accélère considérablement la requête.

Nous verrons cela dans ce chapitre mais c'est pour avoir un premier aperçu du champ d'application des index.

 
Visualiser les index d'une table

Pour visualiser les index d'une table dans MySQL, vous pouvez utiliser la commande SHOW INDEX FROM suivie du nom de la table. Voici la syntaxe :

SHOW INDEX FROM nom_de_la_table;

Nous allons voir maintenant que toutes les clés primaires et les clés étrangères ont un index créés par MySQL.

Remplacez nom_de_la_table par le nom de la table pour laquelle vous souhaitez afficher les index. Cette commande renverra un tableau avec les colonnes suivantes :

    Table : le nom de la table.
    Non_unique : indique si l'index est unique (0) ou non unique (1).
    Key_name : le nom de l'index.
    Seq_in_index : la position de la colonne dans l'index.
    Column_name : le nom de la colonne indexée.
    Collation : la collation utilisée pour l'index.
    Cardinality : le nombre approximatif de valeurs uniques dans l'index.
    Sub_part : la longueur de la partie de la colonne indexée, si l'index est un index partiel.
    Packed : indique si l'index est compressé (NULL signifie non).
    Null : indique si la colonne peut contenir des valeurs NULL.
    Index_type : le type d'index (BTREE, HASH, etc.).
    Comment : des commentaires sur l'index.

Essayons sur notre base de données movies :

SHOW INDEXES FROM movie;

Remarquons qu'il existe un index par défaut pour la PRIMARY KEY de la table :

Voici la signification de chaque colonne :

    Table : le nom de la table, dans ce cas "movie".
    Non_unique : indique si l'index est unique (0) ou non unique (1). Dans ce cas, la valeur est 0, ce qui signifie que l'index est unique.
    Key_name : le nom de l'index, dans ce cas "PRIMARY", ce qui signifie que c'est l'index primaire de la table.
    Seq_in_index : la position de la colonne dans l'index. Dans ce cas, la valeur est 1, ce qui signifie que c'est la première colonne de l'index.
    Column_name : le nom de la colonne indexée, dans ce cas "movie_id".
    Collation : la collation utilisée pour l'index. Dans ce cas, la valeur est "A", ce qui signifie que la collation est ascendante.
    Cardinality : le nombre approximatif de valeurs uniques dans l'index. Dans ce cas, la valeur est 4688, ce qui signifie que l'index contient environ 4688 valeurs uniques.
    Sub_part : la longueur de la partie de la colonne indexée, si l'index est un index partiel. Dans ce cas, la colonne est vide, ce qui signifie que l'index utilise la longueur complète de la colonne.
    Packed : indique si l'index est compressé. Dans ce cas, la colonne est vide, ce qui signifie que l'index n'est pas compressé.
    Null : indique si la colonne peut contenir des valeurs NULL. Dans ce cas, la colonne est vide, ce qui signifie que la colonne ne peut pas contenir de valeurs NULL.
    Index_type : le type d'index. Dans ce cas, la valeur est "BTREE", ce qui signifie que l'index est un index B-tree.
    Comment : des commentaires sur l'index. Dans ce cas, la colonne est vide, ce qui signifie qu'il n'y a pas de commentaire sur l'index.
    Visible : des commentaires supplémentaires sur l'index. Dans ce cas, la valeur est "YES", ce qui signifie que l'index est visible.

Essayons maintenant sur la table movie_cast :

SHOW indexes FROM movie_cast;

Vous pourrez voir trois index correspondants aux trois clés étrangères dans la table.

 
Créer un index

L'instruction CREATE INDEX permet de créer un index sur une ou plusieurs colonnes d'une table.

CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name
    ON tbl_name (key_part,...)
    [index_option]
    [algorithm_option | lock_option] ...

    UNIQUE, FULLTEXT, SPATIAL : ces options spécifient le type d'index. UNIQUE garantit que chaque valeur dans l'index est unique. FULLTEXT est utilisé pour les recherches textuelles intégrales, principalement avec les colonnes CHAR, VARCHAR, ou TEXT. SPATIAL est utilisé pour les données spatiales et nécessite que les colonnes soient de type spatial comme GEOMETRY.

    index_name : nom de l'index à créer.

    tbl_name : nom de la table sur laquelle l'index est créé.

    key_part : spécifie les colonnes à inclure dans l'index et peut inclure une longueur de préfixe pour les types de colonnes de chaîne et peut être suivi de ASC ou DESC pour définir l'ordre de tri de l'index (bien que ASC et DESC ne soient pas supportés pour les index HASH et les index spatiaux).

 
Exemple

Pour créer un index sur la colonne release_date de la table movie, voici comment faire :

CREATE INDEX idx_release_date ON movie(release_date);

Notez bien que par convention le nom de l'index commence par idx_ puis la ou les colonnes concernées par l'index.

Vérifiez que l'index est bien créé :

SHOW INDEXES FROM movie;

L'instruction EXPLAIN
Qu'est-ce que la commande EXPLAIN ?

La commande EXPLAIN est un outil puissant pour les développeurs et les administrateurs de bases de données cherchant à optimiser leurs requêtes.

Elle fournit des informations détaillées sur la manière dont MySQL exécute une instruction donnée, offrant ainsi un aperçu précieux de l'efficacité de l'exécution de la requête et des opportunités d'optimisation.

EXPLAIN fonctionne avec les instructions SELECT, DELETE, INSERT, REPLACE, et UPDATE.

Lorsqu'utilisé avec une instruction explicable, EXPLAIN affiche des informations provenant de l'optimiseur sur le plan d'exécution de l'instruction. Cela inclut la manière dont les tables sont jointes et l'ordre de ces jointures.

EXPLAIN permet principalement deux choses :

    Optimisation des index : EXPLAIN aide à identifier où ajouter des index sur les tables pour que l'instruction s'exécute plus rapidement, en utilisant des index pour trouver les lignes.

    Ordre de jointure : permet de vérifier si l'optimiseur joint les tables dans un ordre optimal.

 
Format de sortie

Lorsque vous utilisez EXPLAIN, MySQL retourne une série de colonnes fournissant des informations sur chaque table impliquée dans la requête. Voici quelques-unes des colonnes clés et ce qu'elles signifient :

    id : identifiant de la requête SELECT. Utile pour les requêtes impliquant des sous-requêtes ou des unions.
    select_type : type de SELECT, indiquant si la requête est simple, fait partie d'une union, est une sous-requête, etc.
    table : nom de la table concernée par la ligne de sortie.
    type : type de jointure, allant de "system" (le meilleur) à "ALL" (le moins bon, indiquant un scan complet de la table).
    possible_keys : indique les index que MySQL peut choisir d'utiliser pour la requête.
    key : l'index que MySQL a effectivement décidé d'utiliser.
    key_len : longueur de la clé utilisée, utile pour comprendre la partie de l'index utilisée.
    ref : colonnes ou constantes comparées à l'index.
    rows : estimation du nombre de lignes que MySQL doit examiner pour exécuter la requête.
    Extra : informations supplémentaires sur le plan d'exécution, comme "Using index" ou "Using temporary".

 
Interprétation des résultats de EXPLAIN

    Optimisation des index : si possible_keys contient des index mais que key est NULL ou que l'index utilisé n'est pas optimal, vous pourriez avoir besoin d'ajuster vos index.
    Types de jointure : un type "ALL" indique un scan complet de la table, souvent un signe que la requête peut être optimisée, par exemple, en ajoutant des index.
    Utilisation d'un index couvrant : si la colonne Extra indique "Using index", cela signifie que la requête peut être satisfaite entièrement à partir de l'index, ce qui est généralement très performant.
    Scans de table complets : les valeurs "Using filesort" ou "Using temporary" dans la colonne Extra peuvent indiquer des opérations coûteuses que vous pourriez vouloir éviter.

 
Exemples
Premier exemple

EXPLAIN SELECT * FROM movie WHERE release_date = '2015-12-25';

Nous aurons :

    id : identifiant de la ligne de sortie. Dans ce cas, la valeur est 1, ce qui signifie que c'est la première ligne de sortie.
    select_type : type de la requête SELECT. Dans ce cas, la valeur est SIMPLE, ce qui signifie que la requête ne contient pas de sous-requêtes ou d'unions.
    table : nom de la table à laquelle la ligne de sortie fait référence. Dans ce cas, la valeur est "movie", ce qui signifie que la ligne de sortie fait référence à la table "movie".
    partitions : nom des partitions auxquelles la ligne de sortie fait référence. Dans ce cas, la valeur est NULL, ce qui signifie que la table n'est pas partitionnée ou que la requête ne fait référence à aucune partition spécifique.
    type : type de jointure utilisé pour accéder à la table. Dans ce cas, la valeur est "ref", ce qui signifie que MySQL utilise un index pour rechercher les lignes correspondantes.
    possible_keys : liste des index pouvant être utilisés pour optimiser la requête. Dans ce cas, la valeur est "idx_release_date", ce qui signifie que l'index "idx_release_date" peut être utilisé pour optimiser la requête.
    key : index utilisé pour optimiser la requête. Dans ce cas, la valeur est également "idx_release_date", ce qui signifie que MySQL utilise cet index pour optimiser la requête.
    key_len : longueur de la clé utilisée pour la recherche dans l'index. Dans ce cas, la valeur est 4, ce qui signifie que la longueur de la clé est de 4 octets.
    ref : valeur utilisée pour la comparaison constante dans la recherche d'index. Dans ce cas, la valeur est "const", ce qui signifie que MySQL utilise une valeur constante pour la recherche.
    rows : nombre estimé de lignes à examiner pour trouver les lignes correspondantes. Dans ce cas, la valeur est 3, ce qui signifie que MySQL estime que 3 lignes doivent être examinées pour trouver les lignes correspondantes.
    filtered : pourcentage estimé de lignes qui seront filtrées en utilisant la clause WHERE. Dans ce cas, la valeur est 100,00, ce qui signifie que toutes les lignes correspondent à la clause WHERE.
    Extra : informations supplémentaires sur la manière dont MySQL exécute la requête. Dans ce cas, la valeur est NULL, ce qui signifie qu'il n'y a pas d'informations supplémentaires à afficher.

Deuxième exemple

Comparons à une requête qui n'utilise pas d'index :

EXPLAIN SELECT * FROM movie WHERE title = 'Inception';

Nous aurons alors :

    id : identifiant de la ligne de sortie. Dans ce cas, la valeur est 1, ce qui signifie que c'est la première ligne de sortie.
    select_type : type de la requête SELECT. Dans ce cas, la valeur est SIMPLE, ce qui signifie que la requête ne contient pas de sous-requêtes ou d'unions.
    table : nom de la table à laquelle la ligne de sortie fait référence. Dans ce cas, la valeur est "movie", ce qui signifie que la ligne de sortie fait référence à la table "movie".
    partitions : nom des partitions auxquelles la ligne de sortie fait référence. Dans ce cas, la valeur est NULL, ce qui signifie que la table n'est pas partitionnée ou que la requête ne fait référence à aucune partition spécifique.
    type : type de jointure utilisé pour accéder à la table. Dans ce cas, la valeur est "ALL", ce qui signifie que MySQL effectue une lecture complète de la table pour trouver les lignes correspondantes.
    possible_keys : liste des index pouvant être utilisés pour optimiser la requête. Dans ce cas, la valeur est NULL, ce qui signifie qu'aucun index ne peut être utilisé pour optimiser la requête.
    key : index utilisé pour optimiser la requête. Dans ce cas, la valeur est également NULL, ce qui signifie que MySQL ne peut pas utiliser d'index pour optimiser la requête.
    key_len : longueur de la clé utilisée pour la recherche dans l'index. Dans ce cas, la valeur est NULL, ce qui signifie que MySQL ne peut pas utiliser d'index pour optimiser la requête.
    ref : valeur utilisée pour la comparaison constante dans la recherche d'index. Dans ce cas, la valeur est également NULL, ce qui signifie que MySQL ne peut pas utiliser d'index pour optimiser la requête.
    rows : nombre estimé de lignes à examiner pour trouver les lignes correspondantes. Dans ce cas, la valeur est 4688, ce qui signifie que MySQL doit examiner toutes les lignes de la table pour trouver les lignes correspondantes.
    filtered : pourcentage estimé de lignes qui seront filtrées en utilisant la clause WHERE. Dans ce cas, la valeur est 10,00, ce qui signifie que seulement 10 % des lignes correspondent à la clause WHERE.

Autres exemples

Toutes les requêtes suivantes bénéficieront de l'index, que ce soit pour compter des lignes, trier, rechercher etc.

Liste des 10 derniers films sortis :

EXPLAIN SELECT * FROM movie ORDER BY release_date DESC LIMIT 10;

Compte des films sortis avant une certaine date :

EXPLAIN SELECT COUNT(*) FROM movie WHERE release_date < '2010-01-01';

Recherche de films sortis entre deux dates :

EXPLAIN SELECT * FROM movie WHERE release_date BETWEEN '2014-01-01' AND '2014-12-31';

Moyenne des revenus des films pour une année donnée :

EXPLAIN SELECT AVG(revenue) FROM movie WHERE YEAR(release_date) = 2023;

Les index composites
Qu'est-ce qu'un index composite ?

Les index à colonnes multiples, également connus sous le nom d'index composites, permettent à MySQL d'effectuer des recherches efficaces sur plusieurs colonnes d'une table.

Imaginez que vous avez un carnet d'adresses. Vous pourriez vouloir chercher une personne par son nom de famille, puis par son prénom.

Un index à colonnes multiples dans une base de données fonctionne de la même manière : il combine plusieurs colonnes dans un seul index, ce qui vous permet de rechercher des données en utilisant une combinaison de ces colonnes.

 
Comment fonctionne-t-il ?

Prenons l'exemple d'un index sur les colonnes last_name et first_name.

Si vous créez un index composite sur ces deux colonnes dans cet ordre, MySQL peut l'utiliser pour accélérer les requêtes qui recherchent à la fois sur last_name et first_name, mais aussi pour celles qui ne recherchent que sur last_name.

C'est parce que last_name est le premier élément de l'index, et dans un index composite, les données sont triées d'abord par le premier élément.
Utilisation de l'index à colonnes multiples

L'index composite peut être utilisé pour des requêtes qui :

Testent toutes les colonnes de l'index :

SELECT * FROM test WHERE last_name='Dupont' AND first_name='Jean';

Testent seulement la première colonne :

SELECT * FROM test WHERE last_name='Dupont';

Limitations de l'index à colonnes multiples

MySQL ne peut pas utiliser un index composite si votre requête ne commence pas par la première colonne de l'index ou si elle utilise OR.

Par exemple, si votre index est sur (last_name, first_name), la requête suivante ne peut pas utiliser l'index :

SELECT * FROM test WHERE first_name='Jean';

Celle-ci non plus :

SELECT * FROM test WHERE last_name='Dupont' OR first_name='Jean';

MySQL doit considérer deux ensembles de données distincts : un ensemble où last_name est 'Dupont', et un autre ensemble où first_name est 'Jean'. Ces deux ensembles peuvent se trouver à des emplacements très différents dans l'index, car l'index est d'abord trié par last_name, puis par first_name au sein de chaque entrée last_name.

 
Exemple

Supprimons l'index créé précédemment :

DROP INDEX idx_release_date ON movie;

Créons un index composite :

CREATE INDEX idx_movie_popularity_release ON movie(popularity, release_date);

Cet index permettra d'accélérer les requêtes qui filtrent ou trient les résultats sur popularity et release_date.
Requêtes utilisant l'index

Sélectionner les films par ordre de popularité et de date de sortie la plus récente :

EXPLAIN SELECT * FROM movie ORDER BY popularity DESC, release_date DESC;

Trouver les films avec une popularité spécifique dans une plage de dates :

EXPLAIN SELECT * FROM movie
WHERE popularity > 500
  AND release_date BETWEEN '2010-01-01' AND '2016-01-01'
ORDER BY release_date DESC;

Sélectionner les films les plus récents avec une popularité supérieure à un seuil :

EXPLAIN SELECT * FROM movie
WHERE popularity >= 200
ORDER BY release_date DESC, popularity DESC
LIMIT 10;

Requêtes n'utilisant pas l'index

Requête qui filtre uniquement sur la seconde colonne de l'index (popularity) sans mentionner la première (release_date):

EXPLAIN SELECT * FROM movie WHERE popularity > 100;

Requête qui utilise OR entre les colonnes de l'index:

EXPLAIN SELECT * FROM movie WHERE release_date = '2015-01-01' OR popularity > 100;

Ici, même si les deux colonnes font partie de l'index, l'utilisation de OR signifie que la requête cherche des enregistrements qui correspondent soit à la première condition, soit à la seconde, ce qui ne correspond pas à une séquence continue dans l'index composite et, par conséquent, ne peut être efficacement optimisé par l'index.

Les index prefix
Qu'est-ce qu'un index prefix ?

Les index prefix avec MySQL sont un concept permettant d'optimiser les performances des requêtes sur des tables contenant des colonnes de type chaîne de caractères (comme CHAR, VARCHAR, TEXT) ou des données binaires (BLOB).

Un index prefix utilise seulement les premiers N premiers caractères d'une colonne pour l'indexation.

Cette approche est particulièrement utile pour les colonnes contenant des textes longs ou des données binaires, où indexer la totalité de chaque entrée serait inefficace en termes d'espace disque et de performance.

Ce type d'index n'est pas fait pour rechercher dans un champ texte, nous verrons qu'il existe un autre type d'index. Il n'index que les premiers caractères et peut augmenter les performances pour les très larges tables où l'on a besoin d'effectuer des recherches du type "qui commence par...".

 
Fonctionnement d'un index prefix

Le fonctionnement exact est le suivant : si un terme de recherche dépasse la longueur du prefix indexé, l'index est utilisé pour exclure les lignes non correspondantes.

Les lignes restantes sont ensuite examinées une par une pour identifier les correspondances potentielles.

Cela signifie que, même si un index prefix ne garantit pas une correspondance directe avec le terme de recherche complet, il aide à réduire significativement le nombre de lignes à examiner.

 
Pourquoi utiliser un index prefix ?

    Optimisation de l'espace : utiliser un index prefix peut considérablement réduire la taille de l'index, surtout pour les colonnes contenant de grands volumes de texte.
    Amélioration des performances : en réduisant la taille de l'index, les opérations d'indexation et de recherche peuvent être accélérées, car il y a moins de données à parcourir.

 
Syntaxe

La syntaxe est la suivante :

CREATE INDEX nom_index ON table (nom_colonne(n));

Où n est le nombre de caractères à indexer.

 
Exemple

Supposons que nous avons un champ de recherche dans une application et que nous ayons des millions d'entrées dans notre table movie.

Nous pourrions accélérer les recherches avec un index prefix sur la colonne title.

CREATE INDEX idx_title_prefix ON movie (title(10));

Requêtes bénéficiant de l'index

Recherche de films par un titre débutant par "Incep" :

EXPLAIN SELECT * FROM movie WHERE title LIKE 'Incep%';

Même avec cette requête :

EXPLAIN SELECT * FROM movie WHERE title LIKE 'In%';

MySQL a à regarder 61 lignes au lieu de plus de 4000.
Requêtes ne pouvant pas utiliser du tout l'index

Recherche de films dont le titre contient "love" n'importe où :

EXPLAIN SELECT * FROM movie WHERE title LIKE '%love%';

Les index textuels
Qu'est-ce qu'un index FULLTEXT ?

Les index FULLTEXT sont conçus pour permettre des recherches de texte intégral dans les bases de données MySQL.

Contrairement aux index classiques ou aux index prefix qui fonctionnent sur des préfixes de colonne, les index FULLTEXT prennent en compte l'intégralité du contenu des colonnes de type CHAR, VARCHAR, ou TEXT.

Les index FULLTEXT sont différents des index traditionnels tels que les index B-Tree ou les index hash, car ils utilisent une technique de recherche en texte intégral qui prend en compte la langue naturelle, les synonymes, les pluriels et autres variantes linguistiques. Les recherches en texte intégral peuvent être beaucoup plus lentes que les recherches utilisant des index traditionnels, mais les index FULLTEXT peuvent améliorer considérablement les performances des recherches en texte intégral.

 
Syntaxe et utilisation

La création d'un index FULLTEXT dans MySQL se fait à l'aide de la commande CREATE FULLTEXT INDEX, qui a la syntaxe suivante :

CREATE FULLTEXT INDEX nom_de_lindex ON nom_de_la_table (colonne1, colonne2, ...);

Où nom_de_lindex est le nom que vous donnez à l'index, nom_de_la_table est le nom de la table sur laquelle l'index est créé, et colonne1, colonne2, ..., sont les colonnes de la table sur lesquelles l'index est appliqué.

Pour utiliser un index FULLTEXT pour rechercher dans une table, il faut utiliser la fonction MATCH() en combinaison avec AGAINST().
MATCH()

    MATCH() prend en argument les colonnes sur lesquelles la recherche de texte intégral est effectuée. Ces colonnes doivent être celles qui ont été indexées avec l'index FULLTEXT.
    La syntaxe est la suivante : MATCH(colonne1, colonne2, ...)

AGAINST()

    AGAINST() prend en argument une chaîne de caractères qui est le texte à rechercher dans les colonnes spécifiées par MATCH().
    Vous pouvez spécifier un mode de recherche dans AGAINST(). Les deux modes les plus courants sont le mode naturel (par défaut) et le mode booléen.
    La syntaxe est la suivante : AGAINST('texte à rechercher' [IN BOOLEAN MODE]).

Mode de recherche

MySQL propose deux modes de recherche FULLTEXT

    Mode naturel (Natural Language Mode) :
        C'est le mode par défaut pour FULLTEXT recherches.
        Il est conçu pour traiter la requête de recherche comme une phrase en langage naturel.
        Il retourne les résultats avec un score de pertinence calculé en fonction de la fréquence des mots, de leur présence dans les documents et de leur absence dans l'ensemble de la base de données.

MATCH (colonne) AGAINST ('texte de recherche')

    Mode booléen (Boolean Mode)
        Permet une recherche plus fine avec l'utilisation d'opérateurs booléens comme + (doit inclure), - (ne doit pas inclure), * (wildcard), entre autres.
        Il n'utilise pas de score de pertinence calculé et retourne les résultats seulement en fonction de la correspondance avec les critères booléens.

MATCH (colonne) AGAINST ('+texte -exclure' IN BOOLEAN MODE)

La syntaxe pour rechercher est donc :

SELECT titre, MATCH(colonne_texte) AGAINST('recherche' IN BOOLEAN MODE) AS score
FROM table
WHERE MATCH(colonne_texte) AGAINST('recherche' IN BOOLEAN MODE);

 
Recommandations sur les performances
MySQL propose quelques pistes pour améliorer les performances de ses requêtes sur des index FULLTEXT :

    MySQL applique des optimisations pour les requêtes FULLTEXT qui se limitent à retourner l'identifiant du document (clé primaire) et éventuellement le score de recherche.
    Les requêtes FULLTEXT sont plus efficaces lorsqu'elles sont combinées avec un tri par score descendant et une clause LIMIT pour récupérer les N premières correspondances. Ces requêtes ne doivent pas contenir de clauses WHERE et doivent avoir une unique clause ORDER BY par ordre descendant.
    Les requêtes qui récupèrent seulement le compte (COUNT(*)) des lignes correspondant à un terme de recherche sont optimisées si elles n'incluent pas de clauses WHERE supplémentaires.

 
Exemples

Créons un index sur overview :

CREATE FULLTEXT INDEX idx_fulltext_on_overview ON movie(overview);

Une fois l'index créé, vous pouvez effectuer des recherches de texte intégral à l'aide de la fonction MATCH() ... AGAINST() :

SELECT movie_id, title, overview, MATCH(overview) AGAINST('space adventure') AS score
FROM movie
WHERE MATCH(overview) AGAINST('space adventure');

Remarquez bien la colonne score de recherche (score). Les résultats sont triés du plus pertinent au moins pertinent.
Mode booléen

Recherche de films contenant obligatoirement le mot "love" mais pas le mot "war" dans leur résumé :

SELECT movie_id, title, overview
FROM movie
WHERE MATCH(overview) AGAINST('+love -war' IN BOOLEAN MODE);

    Le signe + devant "love" indique que le mot "love" doit être présent dans le overview pour qu'une ligne soit retournée.
    Le signe - devant "war" exclut les lignes dont le overview contient le mot "war".

Recherche de films contenant soit le mot "space" soit le mot "adventure", avec une priorité plus élevée pour ceux qui contiennent "space" :

SELECT movie_id, title, overview
FROM movie
WHERE MATCH(overview) AGAINST('+space adventure' IN BOOLEAN MODE);

    Le signe + indique que le mot "space" est obligatoire.
    Le mot "adventure" sans signe est optionnel, mais si présent, il augmente la pertinence de la ligne dans les résultats de recherche.

Les index descendants
Qu'est-ce qu'un index descendant ?

Les index décroissants (DESC) permettent de stocker les valeurs de clé dans un ordre décroissant dans l'index.

Avant que cette fonctionnalité ne soit implémentée, bien que les index pouvaient être parcourus dans l'ordre inverse, cela entraînait une pénalité de performance. Avec les index décroissants, un parcours d'index dans l'ordre direct est plus efficace.

Quand vous définissez un index comme décroissant, MySQL organise les données de l'index de telle sorte que les valeurs les plus élevées soient en premier.

Cela peut améliorer les performances des requêtes qui trient les données dans un ordre décroissant, car l'index peut être utilisé directement sans nécessiter un tri supplémentaire.

C'est un sujet assez avancé donc nous ne détaillerons pas beaucoup.

 
Exemple

Supposons que nous créions cette table et ces 4 index :

CREATE TABLE t (
  c1 INT, c2 INT,
  INDEX idx1 (c1 ASC, c2 ASC),
  INDEX idx2 (c1 ASC, c2 DESC),
  INDEX idx3 (c1 DESC, c2 ASC),
  INDEX idx4 (c1 DESC, c2 DESC)
);

    idx1 sera utilisé pour ORDER BY c1 ASC, c2 ASC.
    idx2 sera utilisé pour ORDER BY c1 ASC, c2 DESC.
    idx3 sera utilisé pour ORDER BY c1 DESC, c2 ASC.
    idx4 sera utilisé pour ORDER BY c1 DESC, c2 DESC.

Les index couvrants
Qu'est-ce qu'un index couvrant ?

Les index couvrants, ou covering indexes, sont des index qui contiennent toutes les colonnes nécessaires pour répondre à une requête.

En d'autres termes, si une requête peut être entièrement satisfaite en utilisant uniquement les informations contenues dans l'index, sans avoir à consulter la table de données sous-jacente, alors cet index est dit couvrant pour cette requête.

Les avantages sont les suivants :

    Vitesse d'accès accrue : les données nécessaires sont toutes dans l'index, évitant ainsi des lectures supplémentaires dans la table de données.
    Réduction de la charge I/O : moins de lectures de disque sont nécessaires puisque les données sont lues directement depuis l'index qui est généralement plus petit que la table de données.
    Meilleure utilisation du cache : les index sont souvent mieux représentés dans le cache de la base de données, permettant ainsi des accès plus rapides.

 
Exemple

Créons un index composite :

CREATE INDEX idx_movie_release_revenue ON movie(release_date, revenue);

Voici des exemples de requêtes qui seront couvertes par l'index :

EXPLAIN SELECT release_date, revenue FROM movie WHERE release_date BETWEEN '2016-01-01' AND '2016-12-31';

EXPLAIN SELECT SUM(revenue) FROM movie WHERE release_date >= '2012-05-01' AND release_date <= '2015-05-31';

EXPLAIN SELECT COUNT(*), AVG(revenue) FROM movie WHERE release_date >= '2010-01-01' AND release_date <= '2010-06-30';

Recommandations générales sur la performance

Voici les recommandations les plus communes pour optimiser les performances de MySQL.

 
Optimisation de la structure de la table

    Tables plus petites sont plus performantes : évitez de stocker des données inutiles. Concentrez-vous sur les problèmes actuels sans anticiper sur des problèmes futurs hypothétiques.

    Utilisation de types de données adaptés : privilégiez les types de données les plus petits possibles, comme TINYINT pour l'âge plutôt qu'un INT.

    Clés primaires : assurez-vous que chaque table ait une clé primaire pour une identification unique et rapide des enregistrements.

    Clés primaires courtes et numériques : les clés primaires devraient être courtes et de type numérique pour une recherche plus rapide.

    Éviter les BLOBs : les objets de grande taille (BLOBs) augmentent la taille de la base et impactent négativement les performances. Stockez les fichiers sur disque si possible.

    Partitionnement vertical : si une table contient de nombreuses colonnes peu fréquemment lues, envisagez de la scinder en plusieurs tables liées par une relation un-à-un.
    Tables de résumé/cache : pour les requêtes coûteuses, créez des tables de résumé ou cache. Utilisez des événements ou des déclencheurs pour les mettre à jour.

 
Optimisations des requêtes

    Sélection des colonnes spécifiques : évitez SELECT * et spécifiez uniquement les colonnes nécessaires.

    Limitation des résultats : utilisez LIMIT pour réduire le nombre de lignes retournées par les requêtes.

    Optimisation de l'utilisation de LIKE : évitez les expressions LIKE avec un joker initial (%) qui empêchent l'utilisation d'index.
    Transactions judicieuses : Regroupez les modifications en transactions pour réduire le nombre d'écritures sur disque.

 
Optimisations des index

    Éviter les scans de table complets : utilisez EXPLAIN pour identifier et optimiser les requêtes effectuant des scans complets.

    Conception d'index : indexez les colonnes utilisées dans les clauses WHERE et ORDER BY. Envisagez des index composites et assurez-vous que l'ordre des colonnes dans les index est optimisé par rapport à l'utilisation dans les requêtes.

    Élimination des index inutiles : supprimez les index dupliqués, redondants, ou inutilisés.

 
Autres Recommandations

    Utilisation de la compression : Pour les grandes tables, la compression peut réduire l'espace disque utilisé et améliorer les performances de lecture.

    Maintenance régulière : Utilisez des commandes comme OPTIMIZE TABLE pour maintenir l'efficacité de votre base de données.

