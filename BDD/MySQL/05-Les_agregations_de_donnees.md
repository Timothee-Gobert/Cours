# Les agrégations de données

## Introduction aux fonctions d'agrégation

### Restaurer la base de données

Cliquez du droit sur la base de données **movies** dans _Workbench_ et cliquez sur _Drop Schema_ et validez pour supprimer la base de données.

Rouvrez le dump et réexécutez le script ou [téléchargez ce dossier](https://dyma-courses-assets.s3.fr-par.scw.cloud/MySQL/movies_db.zip) et décompressez le si vous ne l'avez plus.

### Que sont les fonctions d'agrégation ?

**Les fonctions d'agrégation sont utilisées pour calculer une seule valeur de résultat à partir d'un ensemble de valeurs d'entrée.**

Les fonctions d'agrégation sont souvent utilisées dans les rapports et les analyses de données, où les résumés de données sont essentiels.

Par exemple, elles permettent de répondre à des questions comme "Quel est le produit le plus vendu ?" ou "Quel département a la plus haute satisfaction client ?".

Elles sont donc particulièrement utiles pour résumer des données, telles que le calcul de la moyenne des salaires dans une entreprise, ou la détermination du chiffre d'affaires total sur une période donnée.

### Fonctions d'agrégation de base

Les fonctions d'agrégation sont essentielles en _SQL_ pour résumer les données, permettant d'obtenir des informations telles que les moyennes, les totaux et les comptages à partir de grandes tables.

Voici la liste complète :

| Nom              | Description                                                   |
| :--------------- | :------------------------------------------------------------ |
| AVG()            | Renvoie la valeur moyenne de l'argument                       |
| BIT_AND()        | Renvoie le ET logique sur les bits                            |
| BIT_OR()         | Renvoie le OU logique sur les bits                            |
| BIT_XOR()        | Renvoie le OU exclusif logique sur les bits                   |
| COUNT()          | Renvoie le nombre de lignes retournées                        |
| COUNT(DISTINCT)  | Renvoie le nombre de différentes valeurs                      |
| GROUP_CONCAT()   | Renvoie une chaîne de caractères concaténée                   |
| JSON_ARRAYAGG()  | Renvoie le jeu de résultats sous forme d'un seul tableau JSON |
| JSON_OBJECTAGG() | Renvoie le jeu de résultats sous forme d'un seul objet JSON   |
| MAX()            | Renvoie la valeur maximale                                    |
| MIN()            | Renvoie la valeur minimale                                    |
| STD()            | Renvoie l'écart-type de la population                         |
| STDDEV()         | Renvoie l'écart-type de la population                         |
| STDDEV_POP()     | Renvoie l'écart-type de la population                         |
| STDDEV_SAMP()    | Renvoie l'écart-type de l'échantillon                         |
| SUM()            | Renvoie la somme                                              |
| VAR_POP()        | Renvoie la variance standard de la population                 |
| VAR_SAMP()       | Renvoie la variance de l'échantillon                          |
| VARIANCE()       | Renvoie la variance standard de la population                 |

#### `AVG()` - calcul de la moyenne

La fonction `AVG()` est utilisée pour calculer la valeur moyenne d'un ensemble de valeurs numériques, excluant les valeurs _NULL_.

Exemple - pour calculer la note moyenne de tous les films :

```sql
SELECT AVG(vote_average) AS note_moyenne FROM movie;
```

**Il est courant d'utiliser une fonction d'agrégation avec une ou plusieurs conditions,** afin d'agréger uniquement certaines données.

Par exemple, pour obtenir la moyenne des notes des films sortis après une certaine année :

```sql
SELECT
    AVG(vote_average) AS note_moyenne_recent
FROM
    movie
WHERE
    release_date >= '2010-01-01'
```

Il est aussi fréquent d'utiliser des `UNION` pour comparer des ensembles de données :

```sql
SELECT
    'A partir de 2000' AS période,
    AVG(vote_average) AS note_moyenne_recent
FROM
    movie
WHERE
    release_date >= '2000-01-01'
UNION SELECT
    'Avant 2000' AS période,
    AVG(vote_average) AS note_moyenne_anciens
FROM
    movie
WHERE
    release_date < '2000-01-01';
```

#### `COUNT()` - comptage des lignes

**`COUNT()` compte le nombre d'entrées dans une table.**

**`COUNT(*)` compte toutes les lignes, tandis que `COUNT(colonne)` compte les lignes où la colonne spécifiée n'est pas _NULL_.**

Exemple - pour compter le nombre total de films :

```sql
SELECT COUNT(*) AS total_movies FROM movie;
```

Pour compter le nombre de films ayant un budget défini :

```sql
SELECT
    COUNT(budget) AS movies_with_budget
FROM
    movie
WHERE
    budget <> 0;
```

#### `COUNT(DISTINCT)` - compter les valeurs distinctes

**La fonction `COUNT(DISTINCT)` est utilisée pour obtenir le nombre de valeurs uniques dans une colonne, excluant les valeurs _NULL_.**

Cela est particulièrement utile pour analyser la diversité des données ou pour compter le nombre d'éléments uniques, tels que le nombre de clients différents ayant passé une commande.

Par exemple, supposons que nous ayons une table **user_logs** avec une colonne _user_id_. Pour compter le nombre d'utilisateurs uniques ayant accédé à une application sur une période donnée nous pouvons faire :

```sql
SELECT
    COUNT(DISTINCT user_id) AS utilisateurs_uniques
FROM
    user_logs
WHERE
    access_date BETWEEN '2023-01-01' AND '2023-01-31';
```

#### `MAX()` et `MIN()` - valeurs maximales et minimales

**`MAX()` et `MIN()` sont utilisées pour trouver respectivement la valeur la plus élevée et la plus basse d'une colonne. Cela peut être un nombre, une date ou une chaîne de caractères.**

Exemple - trouver le film le plus récent et le plus ancien :

```sql
SELECT
    MAX(release_date) AS newest_movie,
    MIN(release_date) AS oldest_movie
FROM
    movie;
```

Pour trouver le film avec la plus longue et la plus courte durée :

```sql
SELECT
    MAX(runtime) AS longest_movie,
    MIN(runtime) AS shortest_movie
FROM
    movie;
```

#### `SUM()` - totalisation des valeurs

`SUM()` retourne la somme totale des valeurs numériques d'une colonne.

Exemple - pour obtenir la somme totale des revenus de tous les films :

```sql
SELECT SUM(revenue) AS total_revenue FROM movie;
```

Prenons un exemple plus complexe, nous voulons savoir si les films du genre Action ont plus ou moins rapporté que les films du genre Comedy :

```sql
SELECT
    'Action' AS genre,
    SUM(revenue) AS total_revenue
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
WHERE
    genre_name = 'Action'

UNION

SELECT
    'Comedy' AS genre,
    SUM(revenue) AS total_revenue
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
WHERE
    genre_name = 'Comedy';
```

## Fonctions d'agrégation plus avancées

Nous allons voir rapidement ces fonctions car elles sont moins utilisées, mais c'est bien de savoir qu'elles existent.

### `STD()`, `STDDEV()`, `STDDEV_POP()`, `STDDEV_SAMP()`

Ces fonctions (qui sont des alias) calculent l'[écart type](https://fr.wikipedia.org/wiki/%C3%89cart_type) pour un ensemble de valeurs, ce qui est utile pour analyser la dispersion des données autour de la moyenne.

Par exemple, pour calculer l'écart type des revenus des films :

```sql
SELECT STDDEV(revenue) FROM movie;
```

### `VAR_POP()`, `VAR_SAMP()`, `VARIANCE()`

Ces fonctions (qui sont des alias) calculent la [variance](<https://fr.wikipedia.org/wiki/Variance_(math%C3%A9matiques)>) des valeurs, une mesure de la dispersion similaire à l'écart type mais en utilisant le carré des écarts par rapport à la moyenne.

Par exemple, pour calculer la variance des votes des films :

```sql
SELECT VAR_POP(vote_count) FROM movie;
```

### `JSON_ARRAYAGG()`, `JSON_OBJECTAGG()`

Ces fonctions agrègent les résultats d'une requête en un tableau _JSON_ ou en un objet _JSON_, respectivement.

Par exemple, pour obtenir un tableau _JSON_ des titres de films et un objet _JSON_ des films avec leur revenu :

```sql
SELECT JSON_ARRAYAGG(title) AS movies_json FROM movie;
SELECT JSON_OBJECTAGG(title, revenue) AS movie_revenues FROM movie;
```

### `GROUP_CONCAT()` - concaténation des valeurs de groupe

`GROUP_CONCAT()` combine les valeurs de plusieurs lignes en une seule chaîne de caractères.

Exemple - pour créer une liste de tous les titres de films séparés par des virgules:

```sql
SELECT GROUP_CONCAT(title ORDER BY title SEPARATOR ', ') AS all_movies_titles FROM movie;
```

`BIT_AND()`, `BIT_OR()`, `BIT_XOR()`

Ces fonctions effectuent des opérations bit à bit sur un ensemble de valeurs pour une colonne donnée. Elles sont utiles pour des calculs spécifiques qui nécessitent une manipulation au niveau du bit.

## Regroupements complexes avec la clause GROUP BY

### Définition

**La clause `GROUP BY` est un outil puissant pour regrouper les lignes qui ont les mêmes valeurs dans des colonnes spécifiées.**

Elle est souvent utilisée en combinaison avec des fonctions d'agrégation (comme `SUM()`, `AVG()`, `COUNT()`, etc.) pour effectuer des calculs sur chaque groupe de lignes.

La syntaxe de base de `GROUP BY` est la suivante :

```sql
SELECT colonne1, fonction_agrégation(colonne2)
FROM table
WHERE condition
GROUP BY colonne1;
```

_colonne1_ dans le `GROUP BY` est la colonne selon laquelle les données seront regroupées, et _fonction_agrégation(colonne2)_ applique une fonction d'agrégation sur les données regroupées.

Il est également possible de grouper par plusieurs colonnes.

### Exemples

Rien de mieux que de bien pratiquer pour bien comprendre. C'est une clause très importante et très utilisée.

#### Calculer le nombre de films par genre et triez par nombre de films

```sql
SELECT
    genre_name, COUNT(*) AS nombre_de_films
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
GROUP BY genre_name
ORDER BY nombre_de_films DESC;
```

#### Calculer le revenu des films par genre et triez par revenu

```sql
SELECT
    genre_name, SUM(revenue) AS revenue_total
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
GROUP BY genre_name
ORDER BY revenue_total DESC;
```

#### Calculer la durée moyenne des films par genre et triez par durée

```sql
SELECT
    genre_name, AVG(runtime) AS durée_moyenne
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
GROUP BY genre_name
ORDER BY durée_moyenne DESC;
```

#### Obtenez le nombre de films par année et par genre et triez par nombre de films

_Utilisez la fonction `YEAR()` pour obtenir l'année à partir d'une date._

```sql
SELECT
    YEAR(release_date) AS Année,
    genre_name AS Genre,
    COUNT(*) AS Nombre_de_Films
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
GROUP BY YEAR(release_date) , genre_name
ORDER BY Nombre_de_Films DESC
```

#### Extraire les informations de genre d'un film et les concaténer en une seule chaîne de caractères pour chaque film

```sql
SELECT
    title AS Titre,
    GROUP_CONCAT(genre_name
        SEPARATOR ', ') AS Genres
FROM
    movie_genres
        JOIN
    genre USING (genre_id)
        JOIN
    movie USING (movie_id)
GROUP BY movie_id
ORDER BY title;
```

#### Affichez tous les films et le nombre d'acteurs pour chaque film, y compris les films sans acteurs et triez par le nombre d'acteurs

```sql
SELECT title, COUNT(DISTINCT person_id) AS num_actors
FROM movie
INNER JOIN movie_cast USING(movie_id)
GROUP BY title
ORDER BY num_actors DESC;
```

## Filtrer des données agrégées avec la clause HAVING

### Définition

**La clause `HAVING` est un outil puissant en _SQL_ qui permet de filtrer les résultats d'une requête `SELECT` qui utilise la clause `GROUP BY`.**

Alors que la clause `WHERE` **filtre les lignes avant l'agrégation**, `HAVING` **filtre les groupes après l'agrégation.**

C'est particulièrement utile lorsque vous souhaitez appliquer des conditions sur des valeurs agrégées comme la somme, la moyenne, le compte, etc.

La syntaxe est :

```sql
SELECT colonnes
FROM table
GROUP BY colonnes
HAVING condition;
```

**Retenez bien que :**

- Contrairement à `WHERE`, `HAVING` peut utiliser des fonctions d'agrégation.
- `HAVING` s'utilise souvent en combinaison avec `GROUP BY`, mais peut être utilisée sans si la requête inclut des fonctions d'agrégation.

### Exemples

#### Trouver les genres avec moins de 200 films et triez par nombre de films

```sql
SELECT
    genre_name, COUNT(*) AS Nombre_de_Films
FROM
    genre
        JOIN
    movie_genres USING (genre_id)
GROUP BY genre_name
HAVING Nombre_de_Films < 200
ORDER BY Nombre_de_Films DESC;
```

#### Trouver les genres dont le revenu moyen dépasse 100 millions et triez par revenu

```sql
SELECT
    genre_name, AVG(revenue) AS Revenu_Moyen
FROM
    movie
        JOIN
    movie_genres USING (movie_id)
        JOIN
    genre USING (genre_id)
GROUP BY genre_name
HAVING Revenu_Moyen > 100000000
ORDER BY Revenu_Moyen DESC;
```

#### Trouver les années où la note moyenne des films est supérieure à 7.5 et triez par note

```sql
SELECT
    YEAR(release_date) AS Année,
    AVG(vote_average) AS Note_Moyenne
FROM
    movie
GROUP BY YEAR(release_date)
HAVING AVG(vote_average) > 7.5
ORDER BY Note_Moyenne DESC;
```

## Résultats récapitulatifs avec GROUP BY WITH ROLLUP

### Le modificateur `WITH ROLLUP`

La clause `GROUP BY` accompagnée du modificateur `WITH ROLLUP`, offre une puissance analytique significative en permettant de produire des résultats récapitulatifs à plusieurs niveaux d'agrégation en une seule requête.

La syntaxe est la suivante :

```sql
SELECT colonnes, fonction_agrégat(colonne)
FROM table
GROUP BY colonnes WITH ROLLUP;
```

### Exemples

#### Obtenez le revenu par année en filtrant pour les années sans données sur le revenu et en triant par revenu. Obtenez également le total global avec `ROLLUP`

```sql
SELECT YEAR(release_date) AS Année, SUM(revenue) AS Revenu_total
FROM movie
GROUP BY Année WITH ROLLUP
HAVING Revenu_total > 0
ORDER BY Revenu_total DESC;
```

#### Obtenez la durée moyenne des films par année de sortie et par genre, avec des totaux pour chaque année et un total global

```sql
SELECT
    YEAR(release_date) AS release_year,
    genre_name,
    AVG(runtime) AS average_runtime
FROM
    movie
        JOIN
    movie_genres USING(movie_id)
        JOIN
    genre USING(genre_id)
GROUP BY release_year , genre_name WITH ROLLUP
ORDER BY release_year;
```

_Le total pour chaque année aura un genre à **NULL** et le total global aura **NULL** dans les deux premières colonnes. Nous trions par années `ASC` pour le montrer._
