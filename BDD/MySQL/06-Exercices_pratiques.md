# Exercices pratiques

## Exercices : les sous-requêtes

Dans ce chapitre nous allons pratiquer tout ce que nous avons vu jusqu'à maintenant :

- sous-requêtes
- jointures
- agrégations

### Enoncés

_Dans ces premiers exercices, il peut y avoir l'utilisation de fonctions d'agrégation mais ni de jointures ni de `GROUP BY`._

- Retrouvez tous les films dont le budget est supérieur au film Inception et triez les résultats par budget décroissant.
- Retrouvez tous les films dont le revenu est supérieur au revenu moyens des films sortis après le 1er janvier 2000 et triez les résultats par budget décroissant.
- Retrouvez tous les films dont les USA ne font pas parti des pays de production du film.
- Obtenez le titre de chaque film accompagné du nombre total de films sortis la même année.

## Solutions : les sous-requêtes

### Retrouvez tous les films dont le budget est supérieur au film Inception et triez les résultats par budget décroissant.

```sql
SELECT
    *
FROM
    movie
WHERE
    budget > (SELECT
            budget
        FROM
            movie
        WHERE
            title = 'Inception')
ORDER BY budget DESC;
```

### Retrouvez tous les films dont le revenu est supérieur au revenu moyens des films sortis après le 1er janvier 2000 et triez les résultats par budget décroissant.

```sql
SELECT
    *
FROM
    movie
WHERE
    budget > (SELECT
            AVG(revenue)
        FROM
            movie
        WHERE
            release_date > '2000-01-01')
ORDER BY budget DESC;
```

### Retrouvez tous les films dont les USA ne font pas parti des pays de production du film

```sql
SELECT
    title, movie_id
FROM
    movie
WHERE
    movie_id NOT IN (SELECT
            movie_id
        FROM
            production_country
        WHERE
            country_id = (SELECT
                    country_id
                FROM
                    country
                WHERE
                    country_iso_code = 'US'));
```

### Obtenez le titre de chaque film accompagné du nombre total de films sortis la même année

```sql
SELECT
    title,
    release_date,
    (SELECT
            COUNT(*)
        FROM
            movie
        WHERE
            YEAR(release_date) = YEAR(m.release_date)) AS total_films_same_year
FROM
    movie m;
```

**Ici, la sous-requête est corrélée parce qu'elle dépend de la requête principale.** Elle utilise une valeur (_m.release_date_) qui vient de la ligne courante de la requête extérieure. Pour chaque film examiné dans la requête principale, la sous-requête doit être réévaluée, car elle dépend de la date de sortie de ce film spécifique.

La sous-requête doit être exécutée pour chaque ligne de la table movie, ce qui signifie qu'elle effectue un calcul complet pour chaque film. Si la table movie contient de nombreuses lignes, cela peut entraîner beaucoup d'exécutions de la sous-requête, ce qui est coûteux en termes de performances.

## Exercices : les jointures

_Ces exercices peuvent comporter des sous-requêtes, des agrégations et toujours des jointures. Ils ne comportent pas de `GROUP BY`._

### Enoncés

- Trouvez les acteurs qui n'ont jamais joué dans un film d'action ni dans une comédie, ni dans un drame, ni dans un film d'aventure.
- Créez une requête pour identifier toutes les paires d'acteurs qui ont joué ensemble dans au moins un film. Affichez les noms des acteurs dans chaque paire et le titre du film dans lequel ils ont joué ensemble. Triez par le nom du premier acteur, puis pas le nom du second acteur, puis par titre du film.

## Solutions : les jointures

### Trouvez les acteurs qui n'ont jamais joué dans un film d'action ni dans une comédie, ni dans un drame, ni dans un film d'aventure

```sql
SELECT
    person_name, person_id
FROM
    person
WHERE
    person_id NOT IN (SELECT DISTINCT
            person_id
        FROM
            movie_cast
                JOIN
            movie_genres USING (movie_id)
                JOIN
            genre USING (genre_id)
        WHERE
            genre_name IN ('Comedy' , 'Drama', 'Adventure', 'Action'));
```

### Créez une requête pour identifier toutes les paires d'acteurs qui ont joué ensemble dans au moins un film. Affichez les noms des acteurs dans chaque paire et le titre du film dans lequel ils ont joué ensemble. Triez par le nom du premier acteur, puis pas le nom du second acteur, puis par titre du film

```sql
SELECT
    p1.person_name AS Actor_1,
    p2.person_name AS Actor_2,
    m.title AS Movie_Title
FROM
    movie_cast AS mc1
JOIN
    movie_cast AS mc2 ON mc1.movie_id = mc2.movie_id AND mc1.person_id <> mc2.person_id
JOIN
    person AS p1 ON mc1.person_id = p1.person_id
JOIN
    person AS p2 ON mc2.person_id = p2.person_id
JOIN
    movie AS m ON mc1.movie_id = m.movie_id
ORDER BY
    Actor_1, Actor_2, Movie_Title;
```

## Exercices : les agrégations

### Enoncés

- Trouvez les acteurs ayant joué dans plus de 30 films et triez les par nombre de films.
- Écrivez une requête pour trouver le revenu total généré par les films pour chaque pays de production et triez par revenu total décroissant.
- Trouver le film le plus populaire par genre et triez alphabétiquement par genre.
- Trouvez les 15 acteurs ayant joué dans le plus grand nombre de genres différents.

## Solutions : les agrégations

### Trouvez les acteurs ayant joué dans plus de 30 films et triez les par nombre de films

```sql
SELECT
    person_name, COUNT(*) AS number_of_movies
FROM
    person
        JOIN
    movie_cast USING (person_id)
GROUP BY person_name
HAVING COUNT(*) > 30
ORDER BY number_of_movies DESC;
```

### Écrivez une requête pour trouver le revenu total généré par les films pour chaque pays de production et triez par revenu total décroissant

```sql
SELECT
    country_name,
    SUM(revenue) AS total_revenue
FROM
    country
JOIN
    production_country USING(country_id)
JOIN
    movie USING(movie_id)
GROUP BY
    country_name
ORDER BY total_revenue DESC;
```

### Trouver le film le plus populaire par genre et triez alphabétiquement par genre

```sql
SELECT
    genre_name,
    (SELECT
         title
     FROM
         movie AS m
     JOIN
         movie_genres AS mg USING(movie_id)
     WHERE
         mg.genre_id = g.genre_id
     ORDER BY
         m.popularity DESC
     LIMIT 1) AS most_popular_movie
FROM
    genre AS g
ORDER BY genre_name;
```

### Trouvez les 15 acteurs ayant joué dans le plus grand nombre de genres différents

```sql
SELECT
    person_name AS Acteur,
    COUNT(DISTINCT genre_id) AS Genres_Uniques
FROM
    person
        JOIN
    movie_cast USING (person_id)
        JOIN
    movie_genres USING (movie_id)
GROUP BY person_id
ORDER BY Genres_Uniques DESC
LIMIT 15;
```
