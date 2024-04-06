# Les vues

## Introduction aux vues

### Qu'est-ce qu'une vue ?

**Les vues en _SQL_ sont des tables virtuelles basées sur le résultat d'une requête _SQL_.**

Elles ressemblent à des tables ordinaires dans la mesure où elles contiennent des lignes et des colonnes de données, et vous pouvez interroger une vue comme vous le feriez avec une table réelle.

**Une vue ne contient pas de données elle-même,** mais affiche les données stockées dans d'autres tables. Vous pouvez penser à une vue comme un raccourci ou un filtre appliqué à une ou plusieurs tables.

### Pourquoi utiliser des vues ?

- **Simplification des requêtes :** les vues peuvent rendre les requêtes complexes plus accessibles en les enregistrant pour une utilisation répétée.
- **Sécurité :** elles peuvent limiter l'accès aux données, affichant seulement certaines colonnes ou lignes aux utilisateurs.
- **Abstraction :** elles permettent de changer les données sous-jacentes sans affecter les utilisateurs qui accèdent à ces données à travers la vue.

### Création d'une vue

La syntaxe de base pour créer une vue est :

```sql
CREATE VIEW nom_de_la_vue AS
SELECT colonnes
FROM table
WHERE conditions;
```

### Exemple d'une vue utile

Créons une première vue :

```sql
CREATE VIEW cast_details AS
    SELECT
        title, character_name, person_name
    FROM
        movie_cast
            JOIN
        movie USING (movie_id)
            JOIN
        person USING (person_id);
```

Cette vue nous permet de simplifier de nombreuses requêtes, nous allons en voir quelques-unes.

#### Trouver tous les rôles joués par un acteur spécifique

Sans la vue, vous devriez effectuer une jointure entre les tables **movie_cast**, **movie**, et **person**. Avec la vue, c'est plus simple :

```sql
SELECT
    title, character_name
FROM
    cast_details
WHERE
    person_name = 'Johnny Depp';
```

#### Lister tous les acteurs d'un film spécifique

Une autre requête qui serait simplifiée par cette vue :

```sql
SELECT
    person_name, character_name
FROM
    cast_details
WHERE
    title = 'Inception';
```

#### Trouver tous les films dans lesquels un personnage spécifique apparaît

Si vous recherchez tous les films avec un personnage particulier, comme "James Bond", la vue rend la requête très directe :

```sql
SELECT
    title
FROM
    cast_details
WHERE
    character_name = 'James Bond';
```

#### Compter le nombre d'apparitions d'un acteur

Pour connaître le nombre de films dans lesquels un acteur a joué, utilisez la vue pour une requête agrégée :

```sql
SELECT
    COUNT(*) AS number_of_movies
FROM
    cast_details
WHERE
    person_name = 'Robert de Niro';
```

#### Trouver des doublons dans le casting

Si vous voulez trouver des erreurs ou des doublons dans votre base de données où un acteur est listé plusieurs fois pour le même rôle dans un film, la vue peut aider :

```sql
SELECT
    title, character_name, person_name, COUNT(*)
FROM
    cast_details
GROUP BY title , character_name , person_name
HAVING COUNT(*) > 1;
```

### Pour aller plus loin : les algorithmes des vues

Dans _MySQL_, lors de la création ou de la modification d'une vue, vous pouvez spécifier un algorithme à utiliser pour traiter la vue.

L'algorithme détermine comment _MySQL_ exécute la vue lorsqu'elle est utilisée dans une requête. Il y a trois options possibles pour l'algorithme.

#### _UNDEFINED_

L'algorithme _UNDEFINED_ permet au système de choisir l'algorithme à utiliser lors de l'exécution de la vue. _MySQL_ décide automatiquement d'utiliser l'algorithme _MERGE_ ou _TEMPTABLE_ en fonction de la requête et de la façon dont la vue est utilisée.

C'est l'option par défaut si aucun algorithme n'est explicitement spécifié lors de la création de la vue.

#### _MERGE_

L'algorithme _MERGE_ intègre la définition de la vue dans la requête principale. En d'autres termes, lorsqu'une vue est référencée dans une requête, la requête de la vue est fusionnée avec la requête principale, et le serveur _MySQL_ exécute le tout comme une seule requête combinée.

Cet algorithme permet d'utiliser les index des tables sous-jacentes, ce qui peut rendre l'exécution plus efficace.

Cependant, toutes les vues ne sont pas éligibles à être traitées avec l'algorithme _MERGE_. Par exemple, les vues qui contiennent certaines clauses comme `DISTINCT`, `GROUP BY`, des fonctions d'agrégation ou des sous-requêtes ne peuvent pas être fusionnées.

#### _TEMPTABLE_

L'algorithme _TEMPTABLE_ crée une table temporaire pour stocker le résultat de la vue. La requête principale exécute ensuite ses opérations sur cette table temporaire.

Cela signifie que la vue est matérialisée dans une table temporaire, ce qui peut avoir un impact sur les performances car cela implique une étape supplémentaire pour générer la table temporaire avant que la requête principale ne soit exécutée.

Les vues traitées avec _TEMPTABLE_ peuvent inclure des éléments qui ne sont pas autorisés avec _MERGE_, comme les fonctions d'agrégation, `DISTINCT`, et d'autres. Cependant, puisque la vue est matérialisée, les index sur les tables sous-jacentes ne sont pas utilisés, ce qui peut être moins performant.

Retenez en résumé que _MERGE_ est plus performant mais qu'il n'est pas possible de l'utiliser avec toutes les requêtes. Partout où c'est possible, ce sera _MERGE_ qui sera sélectionné par _MySQL_.

## Modifier ou supprimer une vue

### Modification d'une Vue

Pour modifier une vue existante, vous avez deux options :

1. Utiliser `CREATE OR REPLACE VIEW`
2. Utiliser `ALTER VIEW`

Il est recommandé d'utiliser `ALTER VIEW` si vous voulez conserver les privilèges et faire des modifications spécifiques à la définition sans affecter d'autres attributs de la vue.

Optez pour `CREATE OR REPLACE VIEW` si vous voulez une manière rapide de remplacer une vue sans se soucier de la suppression de la vue existante ou si la vue a besoin d'une redéfinition complète.

#### `CREATE OR REPLACE VIEW`

La commande `CREATE OR REPLACE VIEW` est utilisée pour créer une nouvelle vue ou remplacer une vue existante. Voici ses caractéristiques :

- **Facilité d'utilisation :** si la vue existe déjà, elle sera remplacée sans avoir besoin de la supprimer explicitement.
- **Nouvelle définition :** vous pouvez redéfinir complètement la vue, y compris la requête `SELECT`, les noms de colonnes, etc.
- **Réinitialisation des privilèges :** lorsque vous remplacez une vue, les privilèges attribués spécifiquement à la vue précédente ne sont pas conservés et doivent être réattribués.

#### `ALTER VIEW`

La commande `ALTER VIEW` permet de modifier la définition d'une vue existante sans la supprimer et la recréer. Voici quelques points clés concernant `ALTER VIEW` :

- **Modification de la structure :** vous pouvez changer la structure de la vue en spécifiant une nouvelle requête `SELECT` qui définit la vue.
- **Changement de l'algorithme :** vous pouvez changer l'algorithme (_UNDEFINED_, _MERGE_ ou _TEMPTABLE_) utilisé pour la vue.
- **Sécurité _SQL_ :** `ALTER VIEW` permet de modifier le contexte de sécurité (_DEFINER_ ou _INVOKER_) de la vue.

Cependant, `ALTER VIEW` possède des limitations :

- **Nom de la vue :** vous ne pouvez pas changer le nom de la vue avec `ALTER VIEW`. Vous devez supprimer et recréer la vue pour cela.
- **Colonnes :** si vous voulez changer le nom des colonnes de la vue, vous devez spécifier toute la liste des colonnes de la vue dans votre nouvelle définition.
- **Conservation des privilèges :** lorsque vous utilisez `ALTER VIEW`, tous les privilèges attribués à la vue restent intacts après la modification.

### Exemples

#### Utiliser `ALTER VIEW`

Supposons que vous souhaitiez ajouter une colonne _cast_order_ à la vue **cast_details**. Vous pourriez le faire comme ceci :

```sql
ALTER VIEW cast_details AS
    SELECT
        title,
        character_name,
        person_name,
        cast_order
    FROM
        movie_cast
            JOIN
        movie USING (movie_id)
            JOIN
        person USING (person_id);
```

Cette commande `ALTER VIEW` modifie la définition de la vue **cast_details** pour inclure la colonne _cast_order_.

#### Utiliser `CREATE OR REPLACE VIEW`

La commande `CREATE OR REPLACE VIEW` vous permet de remplacer une vue existante par une nouvelle définition sans avoir à la supprimer d'abord.

Voici comment vous pouvez utiliser cette commande pour apporter la même modification que celle décrite ci-dessus :

```sql
CREATE OR REPLACE VIEW cast_details AS
    SELECT
        title, character_name, person_name, cast_order
    FROM
        movie_cast
            JOIN
        movie USING (movie_id)
            JOIN
        person USING (person_id);
```

### Suppression d'une Vue

Si vous avez besoin de supprimer une vue, utilisez la commande `DROP VIEW` :

```sql
DROP VIEW nom_de_la_vue;
```

## Vues modifiables et insérables

### Définition

**Les vues insérables et modifiables sont des vues sur lesquelles vous pouvez exécuter des instructions `INSERT`, `UPDATE`, ou `DELETE` pour modifier les données des tables sous-jacentes.**

Cela permet de simplifier des opérations complexes sur les données en cachant certaines complexités dans la définition de la vue.

### Limitations des vues insérables et modifiables :

Pour qu'une vue soit considérée comme insérable ou modifiable, elle doit respecter certaines règles :

1. **Aucune fonction d'agrégation :** la vue ne doit pas contenir de fonctions d'agrégation (`SUM()`, `COUNT()`, etc.).
2. **Pas de `DISTINCT` :** le mot-clé `DISTINCT` ne doit pas être utilisé dans la définition de la vue.
3. **Pas de `GROUP BY` ou `HAVING` :** ces clauses ne doivent pas être présentes.
4. **Pas d'`UNION` :** la vue ne doit pas être définie avec une union.
5. **Pas de sous-requêtes dans `SELECT` :** aucune colonne de la vue ne doit être définie par une sous-requête (sauf pour `INSERT`).
6. **Pas de sous-requêtes corrélées**
7. **Jointures :** certaines jointures complexes rendent la vue non modifiable.
8. **Restrictions sur les colonnes :** la vue doit inclure toutes les colonnes non NULL sans valeur par défaut de la table de base pour être insérable.
9. **Colonnes simples :** toutes les colonnes de la vue doivent être des références directes aux colonnes de la table, sans expressions ou calculs.

Ces limitations sont dues au fait que _MySQL_ doit être capable de traduire de manière univoque les modifications de la vue en modifications des tables sous-jacentes.

Si une vue contient des éléments qui ne sont pas directement liés à des colonnes spécifiques des tables sous-jacentes, _MySQL_ ne peut pas déterminer comment appliquer les modifications.

Si une vue ne respecte pas ces règles, vous ne pourrez pas insérer, mettre à jour, ou supprimer des enregistrements via la vue. Vous obtiendrez une erreur si vous tentez de le faire.

### Exemple simple

Supposons que nous ayons une table avec cette structure :

```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(100),
    salary DECIMAL(10, 2)
);
```

Nous pouvons créer une vue qui sélectionne uniquement les employés d'un certain département, disons _Development_ :

```sql
CREATE VIEW dev_employees AS
SELECT id, name, salary
FROM employees
WHERE department = 'Development';
```

Dans ce cas, la vue **dev_employees** est insérable et modifiable car elle ne contient pas de fonctions d'agrégation, de mots-clés `GROUP BY` / `DISTINCT`, de sous-requêtes ou d'autres éléments qui empêcheraient les mises à jour. Elle est basée sur une simple condition `WHERE` qui ne modifie pas les données mais les filtre simplement.

Nous pouvons par exemple modifier un employé en utilisant la vue :

```sql
UPDATE dev_employees
SET salary = 75000
WHERE name = 'John Doe';
```

### Exemple avec la vue que nous avions créée

#### Mise à jour

```sql
UPDATE cast_details
SET
    character_name = 'Bob l\'éponge'
WHERE
    person_name = 'Leonardo DiCaprio'
        AND title = 'Inception';
```

Vérifiez que la table sous-jacente **movie_cast** a été modifiée :

```sql
SELECT * FROM movies.movie_cast WHERE character_name = 'Bob l\'éponge';
```

#### Insertion

Par contre :

```sql
INSERT INTO cast_details (title, character_name, person_name, cast_order)
VALUES ('Nouveau Film', 'Nouveau Personnage', 'Nouvel Acteur', 1);
```

Donnera l'erreur :

> _Error Code: 1393. Can not modify more than one base table through a join view 'movies.cast_details'._

L'erreur _MySQL_ _Error Code: 1393_ indique que vous tentez de modifier plusieurs tables de base en même temps par le biais d'une vue qui utilise des jointures (`JOIN`).

Si une vue est basée sur une jointure de plusieurs tables, comme c'est le cas avec **cast_details**, vous ne pouvez généralement modifier qu'une seule des tables sous-jacentes.

Pour éviter cette erreur, il faut s'assurer que sa requête ne concerne que des colonnes provenant d'une seule table de base référencée par la vue. Si vous avez besoin de modifier des colonnes de plusieurs tables, vous devez effectuer des commandes séparées pour chaque table.

Par exemple, pour une vue qui joint les tables **movie_cast**, **movie** et **person**, vous ne pouvez pas exécuter une commande `UPDATE` qui tente de mettre à jour des colonnes dans **movie_cast** et **movie** en même temps. Vous devrez exécuter deux commandes `UPDATE` distinctes, une pour chaque table.

#### Suppression

Même chose par exemple ici :

```sql
DELETE FROM cast_details
WHERE character_name = 'Inconnu';
```

L'erreur _Error Code: 1395_ signifie que vous tentez d'effectuer une suppression sur une vue (**cast_details**) qui est basée sur une jointure de plusieurs tables.

## La clause WITH CHECK OPTION

La clause `WITH CHECK OPTION` est utilisée lors de la création ou de la modification de vues modifiables (c'est-à-dire des vues dans lesquelles des opérations d'insertion ou de mise à jour sont permises).

Cette clause est importante pour maintenir l'intégrité des données dans une vue en s'assurant que toutes les modifications respectent les conditions définies dans la clause `WHERE` de la vue.

Quand vous utilisez `WITH CHECK OPTION`, _MySQL_ s'assure que toutes les insertions ou mises à jour dans la vue ne rendront pas une ligne de la vue invisible.

Une ligne est considérée comme invisible si elle ne respecte pas la condition de la clause `WHERE` de la définition de la vue. En d'autres termes, `WITH CHECK OPTION` garantit que les modifications ne peuvent pas créer des données qui ne seraient pas sélectionnées par la requête `SELECT` de la vue.

### Exemples

Supposons que nous avons une table **employees** avec une colonne _age_ :

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT
);
```

Maintenant, nous créons une vue **young_employees** qui ne sélectionne que les employés de moins de 30 ans :

```sql
CREATE VIEW young_employees AS
SELECT id, name, age
FROM employees
WHERE age < 30
WITH CHECK OPTION;
```

L'option `WITH CHECK OPTION` signifie que toutes les insertions ou mises à jour à travers la vue **young_employees** doivent satisfaire la condition `age < 30`.

#### Exemple d'insertion valide

```sql
INSERT INTO young_employees (id, name, age) VALUES (1, 'John Doe', 25);
```

#### Exemple d'insertion non valide

```sql
INSERT INTO young_employees (id, name, age) VALUES (2, 'Jane Smith', 35);
```
