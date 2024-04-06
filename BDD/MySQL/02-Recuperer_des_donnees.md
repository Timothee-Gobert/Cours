# Récuperer des données

## Syntaxe SQL de base

Dans cette leçon nous allons voir la syntaxe _SQL_ de base, nous verrons ensuite tous les mots clés.

### Utilisation des majuscules

En _SQL_, l'utilisation des majuscules n'est pas requise par le langage lui-même, mais elle suit une convention largement adoptée pour améliorer la lisibilité des requêtes.

Les mots-clés _SQL_ tels que `SELECT`, `FROM`, `WHERE`, etc., sont habituellement écrits en majuscules, tandis que les noms d'identifiants (noms de tables, de colonnes) sont écrits en minuscules ou suivent la casse définie lors de leur création.

Par exemple :

```sql
SELECT name, age FROM users WHERE age >= 18;
```

### Points-virgules

Le point-virgule est le terminateur standard d'une instruction _SQL_.

Il indique au système de gestion de base de données (_SGBD_) que votre instruction est complète et prête à être exécutée.

Dans les scripts contenant plusieurs instructions _SQL_, le point-virgule est essentiel pour séparer chaque instruction :

```sql
SELECT name FROM users;
UPDATE users SET age = age + 1 WHERE name = 'Alice';
```

### Commentaires

Les commentaires sont des textes ignorés par le _SGBD_, servant à expliquer des parties du code _SQL_ ou à désactiver temporairement certaines parties d'une requête.

_SQL_ supporte deux types de commentaires :

- **Commentaires sur une ligne :** commencent par `--` et s'étendent jusqu'à la fin de la ligne.

```sql
-- Ceci est un commentaire sur une ligne
SELECT * FROM users; -- Ceci est un autre commentaire
```

- **Commentaires multilignes :** encadrés par `/*` au début et `*/` à la fin, ils peuvent s'étendre sur plusieurs lignes.

```sql
/*
Ceci est un commentaire
qui couvre plusieurs lignes
*/
SELECT * FROM users;
```

### Espaces et indentation

Bien que les espaces blancs supplémentaires (espaces, tabulations, nouvelles lignes) ne soient pas significatifs en _SQL_ et n'affectent pas l'exécution de vos requêtes, un bon usage de l'indentation et des espaces peut rendre votre code beaucoup plus lisible.

Par exemple :

```sql
SELECT
    user_name,
    user_age
FROM
    user_accounts
WHERE
    user_age >= 18;
```

## Première requête SQL

### Utiliser une base de données

Avant d'effectuer une requête il faut sélectionner la base de données (ou schéma) contre laquelle la requête va être effectuée.

Autrement dit, sélectionner une base de données détermine le contexte dans lequel les requêtes _SQL_ sont exécutées. Sans sélectionner une base de données, vous devrez qualifier chaque table avec le nom de sa base de données, ce qui peut rendre les requêtes plus longues et moins lisibles.

Sans :

```sql
SELECT * FROM movies.movie
```

Avec :

```sql
SELECT * FROM movie
```

#### Utiliser une base de données par défaut

Faites un clic droit sur le nom du schéma et choisissez "_Set as Default Schema_".

Cela rendra ce schéma le contexte par défaut pour toutes les opérations et requêtes _SQL_ dans cette session.

#### Utilisation de l'éditeur _SQL_

**Ouverture d'un nouvel onglet _SQL_ :** dans l'onglet de connexion active, cliquez sur l'icône "_New Query Tab_" dans la barre d'outils pour ouvrir un nouvel onglet d'éditeur _SQL_.

**Commande `USE` :** dans le nouvel onglet _SQL_, tapez la commande `USE nom_de_la_base_de_données`; en remplaçant _nom_de_la_base_de_données_ par le nom de la base de données que vous souhaitez sélectionner.

Exécutez cette commande en cliquant sur l'éclair.

```sql
USE movies;
```

Après l'exécution, toutes les requêtes SQL tapées dans cet onglet seront exécutées dans le contexte de la base de données sélectionnée.

### Première requête

Exécutez cette requête avec l'éclair :

```sql
USE movies;
SELECT * from movie;
```

Cette requête _SQL_ donnée comporte deux instructions distinctes. Il faut donc obligatoirement mettre un premier point virgule après la première instruction (essayez sans).

Le point virgule de la dernière instruction est optionnel mais nous recommandons de toujours le mettre pour débuter.

```sql
SELECT * FROM movie;
```

Cette instruction est une requête _SQL_ qui demande la récupération de toutes les données de la table _movie_.

C'est une des requêtes les plus basiques et les plus utilisées en SQL, permettant de consulter le contenu d'une table.

- `SELECT` est le mot-clé utilisé pour spécifier les données à récupérer. L'utilisation de `*` indique que toutes les colonnes de la table doivent être retournées.
- `FROM` est le mot-clé qui spécifie la table à partir de laquelle récupérer les données.
- `movie` est le nom de la table dont les données sont demandées.

**Vous pouvez d'ailleurs exécuter cette requête de base en cliquant du droit dans Workbench sur une Table et en sélectionnant _Select Rows - Limit 1000_.**

Remarquez que la requête est alors :

```sql
SELECT * FROM movies.movie;
```

## Les clauses SELECT et LIMIT

### La clause `SELECT`

La clause `SELECT` est l'une des instructions _SQL_ les plus fondamentales et les plus utilisées.

Elle permet d'extraire des données spécifiques à partir de bases de données relationnelles.

La forme la plus simple de la clause `SELECT` est :

```sql
SELECT colonne1, colonne2 FROM nom_de_la_table;
```

Ici, _colonne1_ et _colonne2_ sont les noms des colonnes que vous souhaitez récupérer, et _nom_de_la_table_ est la table depuis laquelle vous voulez extraire les données.

**L'ordre des colonnes importe !** Si vous inversez colonne1 et colonne2, les colonnes du tableau de sortie seront également inversées.

### Sélection de toutes les colonnes

Pour sélectionner toutes les colonnes d'une table, utilisez le caractère `*` :

```sql
SELECT * FROM nom_de_la_table;
```

_Ceci est particulièrement utile pour l'exploration rapide des données, mais nous verrons qu'il est recommandé de spécifier les colonnes pour améliorer la performance des requêtes sur de grandes bases de données._

### La clause `LIMIT`

La clause `LIMIT` est une fonctionnalité SQL permettant de limiter le nombre de lignes renvoyées par une instruction `SELECT`.

Elle est particulièrement utile pour paginer les résultats d'une requête ou pour récupérer un sous-ensemble spécifique de données.

Lorsque `LIMIT` est utilisé avec deux arguments, le premier argument spécifie le décalage de la première ligne à renvoyer, et le second spécifie le nombre maximum de lignes à renvoyer

```sql
SELECT * FROM tbl LIMIT 5, 10;  -- Renvoie les lignes 6 à 15
```

Si `LIMIT` est utilisé avec un seul argument, cet argument spécifie le nombre de lignes à retourner depuis le début du jeu de résultats.

```sql
SELECT * FROM tbl LIMIT 5;  # Renvoie les 5 premières lignes
```

### Mise en pratique

Récupérez les colonnes _title_ et _vote_average_ de la table _movie_ dans la base de données _movies_ et limitez à 10 résultats.

## La clause WHERE et les opérateurs de comparaison communs

### La clause `WHERE`

La clause `WHERE` est un outil puissant en _SQL_, utilisée pour filtrer les enregistrements qui répondent à une condition spécifique.

Elle permet d'extraire, de modifier ou de supprimer des données précises d'une base de données relationnelle.

La syntaxe générale pour utiliser la clause `WHERE` dans une requête _SQL_ est la suivante :

```sql
SELECT colonnes FROM table WHERE condition;
```

### Opérateurs de comparaison les plus communs

Nous ne verrons pas tous les opérateurs dès cette leçon mais seulement les plus utilisé.

#### Opérateurs de comparaison

La clause `WHERE` peut être utilisée avec des opérateurs de comparaison tels que `=`, `<>` (qui est la même chose que `!=`), `>`, `<`, `>=`, `<=` pour filtrer les données en fonction de critères simples :

```sql
SELECT nom, age FROM utilisateurs WHERE age >= 18;
```

Autre exemple pour une date :

```sql
SELECT * FROM table WHERE date >= '2022-01-01';
```

Sélectionne les enregistrements à partir du 1er janvier 2022, y compris cette date.

#### Conditions composées - `AND` `OR` `NOT`

Pour des filtrages plus complexes, vous pouvez combiner plusieurs conditions en utilisant les opérateurs logiques `AND` (ou `&&`), `OR` (ou `||`), et `NOT` (ou `!`) :

```sql
SELECT nom, ville FROM utilisateurs WHERE age >= 18 AND ville = 'Paris';
```

#### Utilisation d'intervalles - `BETWEEN` `min` `AND` `max`

L'opérateur `BETWEEN` permet de sélectionner les valeurs dans une plage donnée :

```sql
SELECT * FROM produits WHERE prix BETWEEN 10 AND 20;
```

**Cela inclut les valeurs limites.**

Lorsque vous utilisez `BETWEEN` `min` `AND` `max`, _SQL_ vérifie si l'expression est à la fois supérieur ou égal à `min` et inférieur ou égal à `max`.

```sql
SELECT 2 BETWEEN 1 AND 3; -- Renvoie 1 (vrai), car 2 est entre 1 et 3.
SELECT 2 BETWEEN 3 AND 1; -- Renvoie 0 (faux), car 2 n'est pas entre 3 et 1.
SELECT 1 BETWEEN 2 AND 3; -- Renvoie 0 (faux), car 1 est inférieur à 2.
SELECT 'b' BETWEEN 'a' AND 'c'; -- Renvoie 1 (vrai), car 'b' est alphabétiquement entre 'a' et 'c'.
SELECT 2 BETWEEN 2 AND '3'; -- Renvoie 1 (vrai), la chaîne '3' est convertie en nombre pour la comparaison.
SELECT 2 BETWEEN 2 AND 'x-3'; -- Renvoie 0 (faux), car 'x-3' ne peut pas être converti de manière significative en un nombre.
```

#### Filtrage par listes - `IN (valeur,...)`

L'opérateur `IN` permet de spécifier plusieurs valeurs dans une clause `WHERE` :

```sql
SELECT * FROM films WHERE genre IN ('comédie', 'horreur');
```

#### Nullité des données - `NULL`

Pour vérifier si une colonne contient `NULL`, utilisez `IS NULL` ou `IS NOT NULL` :

```sql
SELECT * FROM utilisateurs WHERE adresse IS NULL;
```

#### Recherche de motifs - `LIKE`

`LIKE` en _SQL_ est utilisé pour effectuer des recherches de motifs dans les données textuelles, offrant une grande souplesse dans les conditions de filtrage

`LIKE` retourne 1 (vrai) si le motif spécifié correspond à l'expression donnée, et 0 (faux) sinon. Si l'expression ou le motif est `NULL`, le résultat est également `NULL`.

`LIKE` supporte deux caractères jokers principaux :

- `%` : représente n'importe quelle séquence de caractères, y compris aucune.
- `_` : représente exactement un caractère.

Par exemple :

```sql
SELECT 'David!' LIKE 'David_'; -- Renvoie 1, car '_' correspond à un caractère.
SELECT 'David!' LIKE '%D%v%'; -- Renvoie 1, '%' peut correspondre à n'importe quelle séquence de caractères.
```

Les comparaisons effectuées avec `LIKE` ne sont pas sensibles à la casse.

### Mise en pratique

- Sélectionnez tous les films ayant un _vote_average_ supérieur à 8.
- Trouvez les films avec une _popularity_ supérieure à _140_ et un _vote_count_ supérieur à _3000_.
- Sélectionnez les films dont le temps de lecture est compris entre 90 et 120 minutes.
- Affichez les films dont le _movie_status_ est soit '_Rumored_' soit '_Post Production_'.
- Trouvez les films dont _overview_ contient 'god'.
- Sélectionnez les films qui n'ont pas de _homepage_.
- Affichez les films qui ne sont pas en statut '_Released_'.
- Trouvez les films qui sont sortis après le 15 mars 2016.
- Trouvez les films qui ont soit un _budget_ supérieur à 100 millions, soit un _revenue_ supérieur à 300 millions.

## Conditions composées

### Ordre d'évaluation des conditions

**_MySQL_ évalue les conditions en suivant la priorité des opérateurs : `NOT` est évalué en premier, suivi de `AND`, puis `OR`.**

Cependant, cette hiérarchie peut conduire à des résultats inattendus si on ne structure pas explicitement l'ordre d'évaluation.

Nous allons voir plusieurs exemples d'ordre d'évaluation puis voir comment y remédier facilement avec l'utilisation de parenthèses.

#### Exemple 1 : priorité de `AND` sur `OR`

```sql
SELECT * FROM employes WHERE nom = 'Alice' OR nom = 'Bob' AND departement = 'IT';
```

**Intention :** vous pourriez penser que cette requête sélectionne tous les employés nommés Alice ou tous les employés nommés Bob qui travaillent dans le département IT.

**Réalité :** en raison de la priorité plus élevée de AND sur OR, la condition est interprétée comme `(nom = 'Alice') OR (nom = 'Bob' AND departement = 'IT')`. Cela signifie que tous les employés nommés Alice seront sélectionnés, quelle que soit leur département, plus les employés nommés Bob uniquement s'ils travaillent dans le département IT.

#### Exemple 2 : combinaison de `NOT` avec `AND` et `OR`

```sql
SELECT * FROM produits WHERE NOT categorie = 'Électronique' AND prix < 100 OR couleur = 'Rouge';
```

**Intention :** vous pourriez vouloir exclure tous les produits électroniques coûtant moins de 100 ou sélectionner tous les produits rouges.

**Réalité :** `AND` a une priorité plus élevée que OR, donc la requête est interprétée comme `(NOT categorie = 'Électronique' AND prix < 100) OR couleur = 'Rouge'`. Cela signifie que la requête sélectionne tous les produits non électroniques de moins de 100, plus tous les produits rouges, indépendamment de leur catégorie ou prix.

#### Exemple 3 : utilisation de `NOT` avec `OR`

```sql
SELECT * FROM commandes WHERE NOT etat = 'Livré' OR etat = 'En préparation';
```

**Intention :** l'intention pourrait être d'exclure les commandes qui ont été livrées ou de sélectionner celles en préparation.

**Réalité :** la requête est interprétée comme `NOT (etat = 'Livré') OR etat = 'En préparation'`, sélectionnant ainsi toutes les commandes qui ne sont pas livrées, y compris celles en préparation, mais aussi celles dans tous les autres états, ce qui n'était probablement pas l'intention.

#### Exemple 4 : mélange de `NOT`, `AND` et `OR`

```sql
SELECT * FROM bibliothèque WHERE NOT auteur = 'Alice' AND genre = 'Science-fiction' OR titre LIKE '%futur%';
```

**Intention :** l'idée pourrait être d'exclure les livres d'Alice sur la science-fiction ou de sélectionner les livres dont le titre contient "futur".

**Réalité :** en raison de la priorité des opérateurs, la condition est interprétée comme `(NOT auteur = 'Alice' AND genre = 'Science-fiction') OR titre LIKE '%futur%'`, sélectionnant tous les livres qui ne sont pas d'Alice et qui sont de science-fiction, plus tous les livres dont le titre contient "futur", indépendamment de l'auteur ou du genre.

### Utilisation des parenthèses

Les parenthèses sont cruciales pour définir l'ordre dans lequel les conditions composées sont évaluées, permettant de grouper les conditions logiquement et d'assurer que l'opération souhaitée est effectuée en premier.

Nous vous recommandons d'utiliser des parenthèses dès lors que vous utilisez plus de deux conditions.

```sql
SELECT * FROM employes WHERE (departement = 'Ventes' OR departement = 'IT') AND salaire > 3000;
```

Et avec _NOT_ :

```sql
SELECT * FROM commandes WHERE NOT (etat = 'Annulée' OR etat = 'En attente');
```

### Mise en pratique

Entrainez vous :

1. Sélectionnez les titres et la note moyenne des films dont le budget est supérieur à 50 millions et dont la note moyenne est supérieure à 7.
2. Trouvez les films dont le budget est de plus de 100 millions et sortis après 2010 ou ceux ayant un _popularity_ supérieur à 100.
3. Sélectionnez les films qui ne sont pas en post-production et qui ont un budget de plus de 250 millions.
4. Trouvez les films dont le titre commence par 'A' et qui ont soit un _vote_count_ supérieur à 500 soit un _runtime_ inférieur à 120 minutes.
5. Sélectionnez les films qui ne sont ni 'Cancelled' ni 'Post Production' et dont la note moyenne est soit inférieure à 5, soit supérieure à 8.

## Les opérateurs arithmétiques et les alias

### Les opérateurs arithmétiques

Dans le langage _SQL_, les opérateurs arithmétiques permettent de réaliser des calculs sur les données numériques

Les opérateurs arithmétiques en _SQL_ incluent `+` (addition), - (soustraction), `*` (multiplication), `/` (division), et `%` (modulo).

Ils peuvent être utilisés dans les requêtes `SELECT` pour effectuer des calculs sur les colonnes de la table.

#### Exemples d'Utilisation

**Addition :** calculer le coût total en ajoutant une taxe à un prix.

```sql
SELECT prix + taxe AS prix_total FROM produits;
```

`Multiplication :` calculer le revenu total en multipliant le prix par la quantité vendue.

```sql
SELECT prix * quantite AS revenu_total FROM ventes;
```

**Modulo :** Trouver les lignes avec un identifiant pair.

```sql
SELECT * FROM produits WHERE id % 2 = 0;
```

### Les alias

**Les alias en _SQL_ sont utilisés pour renommer temporairement une colonne ou une table dans les résultats d'une requête avec le mot clé `AS`.**

Ils rendent les résultats plus compréhensibles et facilitent la référence à des colonnes calculées ou à des tables jointes.

**Division :** trouver le prix moyen par article dans une commande puis renommer la colonne de sortie en _prix_moyen_ :

```sql
SELECT montant_total / nombre_articles AS prix_moyen FROM commandes;
```

### Mise en pratique guidée

Ajoutez une colonne profit que nous calculons pour chaque film :

```sql
SELECT *, (revenue - budget) AS profit FROM movie;
```

Grâce à cette colonne récupérez les films ayant fait un profit de plus de 1 milliard de dollars.

_En **SQL**, vous ne pouvez pas directement assigner un alias à une expression dans la clause `WHERE`. L'alias doit être défini dans la liste de sélection (`SELECT`) et ne peut être utilisé dans une clause `WHERE` directement. Nous devons donc faire :_

```sql
SELECT *, (revenue - budget) AS profit FROM movie WHERE (revenue - budget) > 1000000000;
```

Trouvez les films dont le revenu est supérieur à deux fois le budget, et qui ont un budget de plus de 50 millions, et ajoutez une colonne ROI pour avoir le _return on Investment_ de chaque film :

```sql
SELECT *, (revenue / budget) AS ROI FROM movie WHERE (revenue > 2 * budget) AND budget > 50000000;
```

## Expressions régulières

### L'opérateur _REGEXP_

_REGEXP_ dans _MySQL_ est un puissant outil de recherche de motifs qui permet de réaliser des recherches complexes basées sur des expressions régulières.

Contrairement à l'opérateur `LIKE`, qui ne permet que des recherches simples avec les jokers `%` et `_`, _REGEXP_ offre une flexibilité bien plus grande pour la correspondance de motifs dans les chaînes de caractères.

Une expression régulière (ou _regex_) est une séquence de caractères qui définit un motif de recherche.

Avec _REGEXP_, vous pouvez filtrer les enregistrements dans une table _MySQL_ en fonction de motifs complexes.

Syntaxe de base :

```sql
SELECT colonne FROM table WHERE colonne REGEXP 'motif';
```

### Caractéristiques des expressions régulières

#### Syntaxe

Les expressions régulières utilisent des métacaractères ayant des significations spéciales :

- `.` : correspond à n'importe quel caractère unique.
- `^` : indique le début d'une chaîne.
- `$` : indique la fin d'une chaîne.
- `*` : zéro ou plusieurs occurrences du caractère précédent.
- `+` : une ou plusieurs occurrences du caractère précédent.
- `?` : zéro ou une occurrence du caractère précédent.
- `[abc]` : correspond à n'importe quel caractère unique dans l'ensemble spécifié (ici a, b, ou c).
  - `[a-dX]` correspond à un caractère parmi "a", "b", "c", "d", ou "X";
  - `[^a-dX]` exclut ces caractères.
- `(abc)` : correspond à l'expression exacte entre parenthèses.
- `{n}` correspond exactement à n occurrences de l'élément précédent; `{m,n}` à entre _m_ et _n_ occurrences.
  - `a*` équivaut à `a{0,}`, correspondant à zéro ou plus occurrences de "a".
  - `a+` est identique à `a{1,}`, signifiant une ou plus occurrences de "a".
  - `a?` se traduit par `a{0,1}`, pour zéro ou une occurrence de "a".

#### Exemples d'utilisation

Sélectionne les noms d'employés commençant par 'A' :

```sql
SELECT nom FROM employes WHERE nom REGEXP '^A';
```

Sélectionne les noms se terminant par une lettre majuscule :

```sql
SELECT nom FROM employes WHERE nom REGEXP '[A-Z]$';
```

Sélectionne les adresses commençant par un numéro suivi de 'Rue' :

```sql
SELECT adresse FROM clients WHERE adresse REGEXP '^[0-9]+ Rue';
```

Sélectionner des noms commençant par 'Al' ou 'Be' :

```sql
SELECT nom FROM employes WHERE nom REGEXP '^(Al|Be)';
```

### Mise en pratique

1. Sélectionner les films dont le titre commence par "The".
2. Trouver les films dont le titre se termine par "man".
3. Sélectionner les films dont le titre contient "a" suivi de n'importe quel caractère, puis "e".
4. Trouver les films dont le titre contient une lettre entre "a" et "e" comme premier caractère.
5. Sélectionner les films dont le titre commence par "Coll" ou "Call".
6. Trouver les films dont le titre contient "oo" deux fois.
7. Sélectionner les films dont le titre ne commence pas par une voyelle.
8. Trouver les films dont le titre commence par "The" ou "A".

### Expressions régulières plus avancées

#### Utilisation de classes de caractères

Les classes de caractères _POSIX_ étendent les capacités de correspondance en permettant de cibler des types spécifiques de caractères.

Par exemple, `[[:digit:]]` correspond à tout chiffre numérique, tandis que `[[:alpha:]]` cible n'importe quelle lettre alphabétique.

```sql
SELECT * FROM movie WHERE title REGEXP '[[:digit:]]{4}';
```

Cette requête sélectionne les films dont le titre contient une séquence de quatre chiffres, utile pour trouver des titres avec des années.

Voici la liste des classes utilisables :

| Nom de la classe | Signification                                        |
| :--------------: | :--------------------------------------------------- |
|      alnum       | Caractères alphanumériques                           |
|      alpha       | Caractères alphabétiques                             |
|      blank       | Caractères d'espacement                              |
|      cntrl       | Caractères de contrôle                               |
|      digit       | Caractères numériques                                |
|      graph       | Caractères graphiques                                |
|      lower       | Caractères alphabétiques en minuscules               |
|      print       | Caractères graphiques ou espace                      |
|      punct       | Caractères de ponctuation                            |
|      space       | Espace, tabulation, nouvelle ligne et retour chariot |
|      upper       | Caractères alphabétiques en majuscules               |
|      xdigit      | Caractères numériques hexadécimaux                   |

#### Les bornes de mot

Une borne de mot `\\b` permet de rechercher des mots complets, en assurant que la correspondance se fait aux limites des mots, et non au milieu des mots.

```sql
SELECT * FROM movie WHERE title REGEXP '\\blove\\b';
```

Cette requête retourne les films dont le titre contient le mot "love" comme un mot entier, et non pas les titres dont des mots contiennent le motif _love_.

#### Notes sur la sensibilité à la casse - [avancé]

**Par défaut, les expressions régulières sont sensibles à la casse.**

Mais la sensibilité à la casse, dépend également de la collation de la colonne interrogée.

**Une collation détermine comment les données de chaîne sont comparées et triées dans la base de données.**

Par exemple, la collation de la colonne _title_ de la table _movie_ est définie pour être insensible à la casse (par exemple, _utf8mb4_0900_ai_ci_ où _ci_ signifie _case-insensitive_), et la requête :

```sql
SELECT title FROM movie WHERE title REGEXP 'love';
```

ne sera pas sensible à la casse. Cela signifie que la requête correspondra à "love", "Love", "LOVE", etc., indépendamment de la casse des lettres dans le mot "love".

En revanche, si la collation avait été sensible à la casse (par exemple, _utf8_general_cs_ où _cs_ signifie _case-sensitive_), la même requête n'aurait retourné que les titres correspondant exactement à la casse spécifiée dans l'expression régulière.

## Trier les résultats

### Trier les résultats avec ORDER BY

**La clause `ORDER BY` dans _MySQL_ est un outil puissant pour trier les résultats d'une requête selon un ou plusieurs critères.**

Cette fonctionnalité est essentielle pour organiser les données de manière significative, faciliter l'analyse ou améliorer la présentation des résultats. Voici une exploration détaillée de son utilisation.

`ORDER BY` permet de trier les résultats d'une requête `SELECT` en ordre ascendant (`ASC`) ou descendant (`DESC`) selon les valeurs d'une ou plusieurs colonnes.

**Syntaxe :**

```sql
SELECT colonne1, colonne2 FROM table ORDER BY colonne1 ASC, colonne2 DESC;
```

Cette requête trie d'abord les résultats selon _colonne1_ en ordre ascendant, puis trie les résultats avec des valeurs identiques de _colonne1_ selon _colonne2_ en ordre descendant.

### Fonctionnement

#### Tri ascendant et descendant

- **Ascendant (ASC) :** c'est l'ordre par défaut si aucun mot-clé n'est spécifié. Il organise les données du plus petit au plus grand, ou alphabétiquement de A à Z.
- **Descendant (DESC) :** trie les données du plus grand au plus petit, ou alphabétiquement de Z à A.

#### Tri selon plusieurs colonnes

Il est possible de trier selon plusieurs colonnes en les listant après `ORDER BY`, séparées par des virgules. MySQL effectue le tri selon l'ordre des colonnes spécifié.

#### Tri des valeurs nulles

Par défaut, _MySQL_ place les valeurs _NULL_ en premier avec `ORDER BY ... ASC` et en dernier avec `ORDER BY ... DESC`.

_Nous verrons que nous pouvons utiliser des fonctions comme `COALESCE` pour gérer différemment les valeurs **NULL** si nécessaire._

#### Exemples Pratiques

Trier des utilisateurs d'abord par nom de famille, puis par prénom, tous en ordre alphabétique :

```sql
SELECT nom, prenom FROM utilisateurs ORDER BY nom ASC, prenom ASC;
```

Affiche des utilisateurs du plus âgé au plus jeune :

```sql
SELECT nom, age FROM utilisateurs ORDER BY age DESC;
```

Trier des employés selon leur salaire annuel estimé, du plus élevé au plus bas.

```sql
SELECT nom, salaire FROM employes ORDER BY (salaire * 12) DESC;
```

#### Mise en pratique

Trier les films par titre dans l'ordre alphabétique :

```sql
SELECT title FROM movie ORDER BY title ASC;
```

Trier les films par popularité décroissante. En cas de popularité égale, trier par titre de manière alphabétique :

```sql
SELECT title, popularity FROM movie ORDER BY popularity DESC, title ASC;
```

Essayez par vous même :

- Trier les films par date de sortie la plus récente. Pour les films sortis le même jour, les trier par note moyenne décroissante.
- Calculer le revenu net (revenue - budget) pour chaque film et trier les résultats par revenu net décroissant. Inclure uniquement les films avec un revenu net positif.

## Introduction aux sous-requêtes

### Définition des sous-requêtes

**Une sous-requête, aussi appelée requête imbriquée, est une requête _SQL_ qui est insérée à l'intérieur d'une autre requête _SQL_.**

Les sous-requêtes peuvent être utilisées dans diverses parties d'une requête principale, y compris dans les clauses `SELECT`, `FROM` et `WHERE`.

Les sous-requêtes sont exécutées en premier, et leur résultat est utilisé par la requête principale.

Nous allons voir deux exemples rapides et les pratiquerons largement dans les chapitres suivants avec des requêtes sur nos bases de données d'exemple.

### Exemple dans la clause `WHERE`

Les sous-requêtes utilisées dans la clause `WHERE` permettent de filtrer les résultats de la requête principale en se basant sur les résultats de la sous-requête.

Imaginez une base de données d'une librairie avec deux tables : _livres_ et _auteurs_. La table _livres_ contient des colonnes _titre_, _annee_publication_, et _auteur_id_. La table auteurs contient _auteur_id_, _nom_auteur_.

**Objectif :** trouver tous les livres publiés par "Victor Hugo".

```sql
SELECT titre FROM livres
WHERE auteur_id = (SELECT auteur_id FROM auteurs WHERE nom_auteur = 'Victor Hugo');
```

Le fonctionnement est le suivant :

1. La sous-requête `(SELECT auteur_id FROM auteurs WHERE nom_auteur = 'Victor Hugo')` est exécutée en premier. Elle cherche l'_auteur_id_ correspondant à "Victor Hugo" dans la table _auteurs_.
2. La sous-requête retourne l'_auteur_id_ de Victor Hugo.
3. La requête principale utilise cet _auteur_id_ pour filtrer tous les livres dans la table livres qui correspondent à cet auteur.

### Exemple plus avancé

**Objectif :** Calculer pour chaque livre le nombre total de livres publiés par le même auteur.

```sql
SELECT
  titre,
  (SELECT COUNT(*) FROM livres AS l2 WHERE l2.auteur_id = l1.auteur_id) AS nombre_livres_auteur
FROM livres AS l1;
```

1. Pour chaque ligne (livre) dans la table _livres_ (alias l1), la sous-requête est exécutée.
2. La sous-requête `(SELECT COUNT(*) FROM livres AS l2 WHERE l2.auteur_id = l1.auteur_id)` compte le nombre total de livres (`COUNT(*)`) dans la table _livres_ (alias _l2_) qui ont le même _auteur_id_ que le livre courant de la requête principale (l1.auteur_id).
3. Le résultat de la sous-requête, qui est le nombre de livres par auteur, est retourné comme une colonne supplémentaire _nombre_livres_auteur_ pour chaque ligne de la requête principale.
