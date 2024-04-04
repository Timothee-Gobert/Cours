# Récuperer des données

## Syntaxe SQL de base

Dans cette leçon nous allons voir la syntaxe *SQL* de base, nous verrons ensuite tous les mots clés.

### Utilisation des majuscules

En *SQL*, l'utilisation des majuscules n'est pas requise par le langage lui-même, mais elle suit une convention largement adoptée pour améliorer la lisibilité des requêtes.

Les mots-clés *SQL* tels que `SELECT`, `FROM`, `WHERE`, etc., sont habituellement écrits en majuscules, tandis que les noms d'identifiants (noms de tables, de colonnes) sont écrits en minuscules ou suivent la casse définie lors de leur création.

Par exemple :

```sql
SELECT name, age FROM users WHERE age >= 18;
```
 
### Points-virgules

Le point-virgule est le terminateur standard d'une instruction *SQL*.

Il indique au système de gestion de base de données (*SGBD*) que votre instruction est complète et prête à être exécutée.

Dans les scripts contenant plusieurs instructions *SQL*, le point-virgule est essentiel pour séparer chaque instruction :

```sql
SELECT name FROM users;
UPDATE users SET age = age + 1 WHERE name = 'Alice';
```
 
### Commentaires

Les commentaires sont des textes ignorés par le *SGBD*, servant à expliquer des parties du code *SQL* ou à désactiver temporairement certaines parties d'une requête.

*SQL* supporte deux types de commentaires :

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

Bien que les espaces blancs supplémentaires (espaces, tabulations, nouvelles lignes) ne soient pas significatifs en *SQL* et n'affectent pas l'exécution de vos requêtes, un bon usage de l'indentation et des espaces peut rendre votre code beaucoup plus lisible.

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

