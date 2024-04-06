# Les déclencheurs et les événements

## Introduction aux déclencheurs

### Qu'est-ce qu'un déclencheur ?

Les déclencheurs (_triggers_) dans _MySQL_ offrent un moyen puissant de réagir automatiquement à des événements spécifiques dans la base de données.

#### Syntaxe

**Déclaration de création :** on commence par la commande C`REATE TRIGGER` suivie du nom que l'on souhaite donner au déclencheur.

**Moment d'activation :** on spécifie ensuite quand le déclencheur doit être activé en utilisant `BEFORE` ou `AFTER`, ce qui indique si l'action doit se produire avant ou après l'événement de la table (par exemple, `BEFORE INSERT`).

**Événement déclencheur :** on indique l'événement qui va activer le déclencheur. Cela peut être `INSERT`, `UPDATE` ou `DELETE`.

**Table cible :** on précise sur quelle table le déclencheur doit agir avec `ON nom_de_table`.

**Portée de l'action :** on utilise `FOR EACH ROW` pour indiquer que le déclencheur doit s'exécuter pour chaque ligne affectée par l'événement.

**Corps du déclencheur :** enfin, on définit l'action que le déclencheur doit exécuter. Ce bloc peut contenir une simple instruction ou plusieurs instructions encapsulées dans un bloc `BEGIN` ... `END`.

```sql
DELIMITER //

CREATE TRIGGER nom_du_déclencheur
BEFORE|AFTER événement ON nom_de_table
FOR EACH ROW
BEGIN
    -- Instructions SQL à exécuter par le déclencheur
END;

DELIMITER ;
```

**Il est recommandé de préfixer le nom du déclencheur par l'événement : _insert_nom_, _update_nom_ ou _delete_nom_ pour s'y retrouver plus facilement.**

#### Cas d'utilisation

Voici quelques-uns des cas d'utilisation les plus fréquents des déclencheurs :

- **Audit et logging :** enregistrer des changements dans les tables de logs pour suivre qui a modifié quoi et quand. Cela inclut la sauvegarde des anciennes valeurs avant qu'elles ne soient mises à jour ou supprimées.

- **Validation de données :** vérifier les données pour s'assurer qu'elles respectent certaines règles métiers avant de les insérer ou de les mettre à jour. Par exemple, s'assurer qu'une adresse email est dans un format valide ou que le solde d'un compte n'est jamais négatif.

- **Mise à jour automatique des données :** mettre à jour automatiquement certaines colonnes lors de modifications de données.

- **Synchronisation de données :** maintenir la synchronisation entre des tables ou des bases de données distinctes lorsque les données sont insérées, mises à jour ou supprimées.

- **Enrichissement de données :** ajouter ou transformer des données lorsqu'elles sont insérées. Par exemple, calculer un champ comme une TVA ou un total lors de l'insertion d'une commande.

- **Déclenchement de processus externes :** appeler des procédures stockées ou des fonctions qui exécutent des opérations qui vont au-delà des simples modifications de la base de données, comme envoyer des notifications ou des messages à des systèmes externes.

- **Archivage de données :** archiver automatiquement les enregistrements qui sont supprimés ou remplacés, par exemple, en les déplaçant vers une table d'archive.

### Exemples

#### Déclencheur pour le suivi des modifications de popularité d'un film

Chaque fois que la popularité d'un film est mise à jour, nous pourrions vouloir enregistrer l'ancienne valeur quelque part pour suivre comment la popularité d'un film évolue.

```sql
CREATE TRIGGER before_movie_popularity_update
BEFORE UPDATE ON movie
FOR EACH ROW
BEGIN
IF OLD.popularity <> NEW.popularity THEN
INSERT INTO movie_popularity_audit(movie_id, old_popularity, new_popularity, change_date)
VALUES (OLD.movie_id, OLD.popularity, NEW.popularity, NOW());
END IF;
END;
```

#### Déclencheur pour valider le budget avant insertion d'un film

On pourrait vouloir s'assurer que le budget inséré pour un film n'est pas négatif.

```sql
CREATE TRIGGER validate_movie_budget
BEFORE INSERT ON movie
FOR EACH ROW
BEGIN
IF NEW.budget < 0 THEN
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Le budget d\'un film ne peut pas être négatif.';
END IF;
END;
```

#### Déclencheur pour la mise à jour automatique du nombre de films d'une personne

Lorsqu'un membre de l'équipe d'un film (_cast_ ou _crew_) est ajouté, nous pourrions vouloir mettre à jour automatiquement un compteur de films pour cette personne.

```sql
CREATE TRIGGER update_movie_count_after_insert_cast
AFTER INSERT ON movie_cast
FOR EACH ROW
BEGIN
UPDATE person
SET movie_count = movie_count + 1
WHERE person_id = NEW.person_id;
END;
```

### Mise en pratique

Notre objectif est d'ajouter une colonne _last_modified_ à la table **movie** et créer un déclencheur qui met à jour ce champ à chaque fois qu'un film est mis à jour.

#### Étape 1 : ajouter la colonne _last_modified_

Vous commencerez par ajouter une nouvelle colonne à votre table _movie_. Voici la requête SQL pour ajouter cette colonne :

```sql
ALTER TABLE movie ADD COLUMN last_modified TIMESTAMP;
```

Cette requête ajoute une colonne de type `TIMESTAMP`.

#### Étape 2 : créer le déclencheur

Une fois la colonne ajoutée, vous pouvez créer le déclencheur :

```sql
DELIMITER //

CREATE TRIGGER update_last_modified
BEFORE UPDATE ON movie
FOR EACH ROW
BEGIN
SET NEW.last_modified = NOW();
END //

DELIMITER ;
```

_NEW.last_modified_ fait référence à la nouvelle valeur de la colonne _last_modified_ pour la ligne qui est en train d'être mise à jour.

Essayez une mise à jour :

```sql
UPDATE `movies`.`movie`
SET
vote_average = 2
WHERE
`movie_id` = 5;
```

## Lister, modifier ou supprimer un déclencheur

### Lister les déclencheurs

Pour **voir les déclencheurs** définis dans une base de données _MySQL_, vous pouvez utiliser la commande `SHOW TRIGGERS`.

Cette commande affiche une liste des déclencheurs avec des informations détaillées telles que le nom du déclencheur, le type d'événement (`INSERT`, `UPDATE`, `DELETE`), le timing (`BEFORE`, `AFTER`), la table associée, et la définition du déclencheur (la commande _SQL_ qui est exécutée).

```sql
SHOW TRIGGERS;
```

Vous pouvez également filtrer les déclencheurs pour une table spécifique ou pour ceux qui correspondent à certains critères en utilisant la clause `LIKE` ou `WHERE` :

```sql
SHOW TRIGGERS LIKE 'nom*de_table';
SHOW TRIGGERS WHERE `Table` = 'nom_de_table';
SHOW TRIGGERS LIKE 'update*%';
```

### Modifier les déclencheurs

**Vous ne pouvez pas modifier directement un déclencheur existant.**

Si vous devez changer la définition d'un déclencheur, vous devez le supprimer puis le recréer avec la nouvelle définition.

Cependant, avant de le supprimer, vous voudrez peut-être voir la définition actuelle du déclencheur.

Vous pouvez obtenir la définition d'un déclencheur avec la commande `SHOW CREATE TRIGGER` :

```sql
SHOW CREATE TRIGGER nom_du_déclencheur;
```

Cela vous donnera la commande exacte utilisée pour créer le déclencheur, que vous pouvez copier et modifier selon vos besoins.

### Supprimer les déclencheurs

Pour supprimer un déclencheur, vous utilisez la commande `DROP TRIGGER`.

Vous devez spécifier le nom du déclencheur et, si vous n'utilisez pas la base de données par défaut, vous devez également spécifier le nom de la base de données.

```sql
DROP TRIGGER [IF EXISTS] [nom_de_base_de_données.]nom_du_déclencheur;
```

L'utilisation de `IF EXISTS` est optionnelle mais recommandée, car cela évite une erreur si le déclencheur n'existe pas.

### Bonnes pratiques

Toujours sauvegarder la définition du déclencheur avant de le supprimer (par exemple dans un fichier qui est mis sur un répertoire Git)

Tester les changements de déclencheur dans un environnement de développement avant de les appliquer en production.

Utiliser `IF EXISTS` pour éviter les erreurs lorsque vous supprimez des déclencheurs qui peuvent ne pas exister.

## Introduction aux événements

### Définition

Les événements sont des tâches automatisées qui sont exécutées à des intervalles prévus ou à des moments spécifiques, similaires aux tâches _cron_ dans les systèmes d'exploitation _Unix_.

Ils sont très utiles pour effectuer des tâches de maintenance ou des opérations récurrentes, telles que des nettoyages de table réguliers ou des rapports périodiques.

### Activation du planificateur d'événements

Avant de créer des événements, assurez-vous que le planificateur d'événements de _MySQL_ est activé (il l'est par défaut normalement). Vous pouvez vérifier cela avec la commande suivante :

```sql
SHOW VARIABLES LIKE 'event_scheduler';
```

Si le planificateur n'est pas activé, vous pouvez l'activer en exécutant :

```sql
SET GLOBAL event_scheduler = ON;
```

### Création d'un événement

Pour créer un événement, vous utilisez la commande `CREATE EVENT`.

```sql
CREATE EVENT [IF NOT EXISTS] nom_de_l_evenement
ON SCHEDULE schedule
[ON COMPLETION [NOT] PRESERVE]
DO
corps_de_l_evenement;
```

- `CREATE EVENT` : démarre la déclaration pour créer un événement.
- `IF NOT EXISTS` : Optionnel. Empêche une erreur si un événement du même nom existe déjà.
- `nom_de_l_evenement` : le nom unique pour identifier l'événement.
- `ON SCHEDULE` : définit le calendrier de l'événement. Ce peut être un intervalle récurrent ou un moment spécifique pour l'exécution unique.
- `ON COMPLETION NOT PRESERVE` : Optionnel. Par défaut, l'événement est supprimé après son exécution si cette option est spécifiée. Sinon, il est préservé.
- `DO` : spécifie la ou les instructions SQL à exécuter lorsque l'événement est déclenché.

#### Les options de planification (`ON SCHEDULE`)

- `AT timestamp [+ INTERVAL interval]` : exécute l'événement une fois à un moment spécifique. Vous pouvez optionnellement ajouter un intervalle pour retarder l'exécution.
- `EVERY interval [STARTS timestamp] [ENDS timestamp]` : exécute l'événement à des intervalles réguliers. Vous pouvez spécifier un début et/ou une fin pour ces intervalles.

L'intervalle spécifie la fréquence à laquelle l'événement doit s'exécuter. Il peut être exprimé en secondes, minutes, heures, jours, semaines, mois, ou années. Par exemple, `EVERY 1 DAY`, `EVERY 2 WEEKS`, `EVERY 1 MONTH`.

Voici les intervalles disponibles :

- `YEAR` : `EVERY 2 YEAR` - Exécute l'événement tous les 2 ans.
- `QUARTER` : `EVERY 1 QUARTER` - Exécute l'événement tous les trimestres.
- `MONTH` : `EVERY 3 MONTH` - Exécute l'événement tous les 3 mois.
- `DAY` : `EVERY 10 DAY` - Exécute l'événement tous les 10 jours.
- `HOUR` : `EVERY 4 HOUR` - Exécute l'événement toutes les 4 heures.
- `MINUTE` : `EVERY 30 MINUTE` - Exécute l'événement toutes les 30 minutes.
- `WEEK` : `EVERY 1 WEEK` - Exécute l'événement chaque semaine.
- `SECOND` : `EVERY 45 SECOND` - Exécute l'événement toutes les 45 secondes.
- `YEAR_MONTH` : `EVERY '1-2' YEAR_MONTH` - Exécute l'événement tous les 1 an et 2 mois.
- `DAY_HOUR` : `EVERY '2 5' DAY_HOUR` - Exécute l'événement tous les 2 jours et 5 heures.
- `DAY_MINUTE` : `EVERY '1 15' DAY_MINUTE` - Exécute l'événement tous les 1 jour et 15 minutes.
- `DAY_SECOND` : `EVERY '1 10' DAY_SECOND` - Exécute l'événement tous les 1 jour et 10 secondes.
- `HOUR_MINUTE` : `EVERY '3:30' HOUR_MINUTE` - Exécute l'événement toutes les 3 heures et 30 minutes.
- `HOUR_SECOND` : `EVERY '2:15' HOUR_SECOND` - Exécute l'événement toutes les 2 heures et 15 secondes.
- `MINUTE_SECOND` : `EVERY '5:10' MINUTE_SECOND` - Exécute l'événement toutes les 5 minutes et 10 secondes.

### Exemples

#### Evénement de nettoyage

```sql
CREATE EVENT if not exists nettoyage_hebdomadaire
ON SCHEDULE EVERY 1 WEEK STARTS '2025-01-01 00:00:00'
DO
DELETE FROM table_temporaire WHERE date < NOW() - INTERVAL 1 MONTH;
```

Cet exemple crée un événement nommé _nettoyage_hebdomadaire_ qui démarre le 1er janvier 2025 à minuit et s'exécute toutes les semaines. À chaque exécution, il supprime les enregistrements de plus d'un mois de la table _table_temporaire_.

#### Nettoyage quotidien

```sql
CREATE EVENT if not exists nettoyage_journalier
ON SCHEDULE EVERY 1 DAY
STARTS (TIMESTAMP(CURRENT_DATE) + INTERVAL 1 DAY) + INTERVAL 0 HOUR
DO
DELETE FROM table_temporaire WHERE date_creation < CURRENT_DATE;
```

Dans cet exemple, _nettoyage_journalier_ est le nom de l'événement, et il supprime les lignes de la table _table_temporaire_ qui ont une _date_creation_ avant le jour courant.

Pour le premier événement :

- `STARTS` : indique le moment précis où l'événement commencera à s'exécuter selon la planification définie.
- `(TIMESTAMP(CURRENT_DATE) + INTERVAL 1 DAY)` : calcule le timestamp qui représente minuit (le début) du jour suivant. CURRENT_DATE renvoie la date actuelle sans information de temps, et + INTERVAL 1 DAY ajoute un jour à cette date, programmant ainsi l'événement pour commencer à minuit le lendemain.
- `+ INTERVAL 0 HOUR` : cette partie est techniquement redondante dans cet exemple, car elle ajoute zéro heure au moment de début déjà déterminé. Elle illustre comment on pourrait décaler l'heure de début si nécessaire.

#### Exemple pratique

```sql
CREATE EVENT if not exists cleanup_old_cast_entries
ON SCHEDULE EVERY 1 WEEK
DO
DELETE FROM movie_cast
WHERE movie_id IN (SELECT movie_id FROM movie WHERE movie_status = 'Cancelled');
```

## Gestion des événements

### Voir les événements

Pour lister les événements dans la base de données courante, utilisez la commande `SHOW EVENTS` :

```sql
SHOW EVENTS;
```

Pour obtenir des informations sur un événement spécifique, utilisez :

```sql
SHOW EVENTS LIKE 'nom_de_l_evenement';
```

### Modifier un événement

Pour modifier un événement existant, vous devez utiliser la commande `ALTER EVENT`.

Voici un exemple de modification de l'événement précédent pour qu'il s'exécute toutes les heures :

```sql
ALTER EVENT nettoyage_journalier ON SCHEDULE EVERY 1 HOUR STARTS CURRENT_TIMESTAMP;
```

### Désactiver et activer un événement

Si vous souhaitez arrêter temporairement un événement sans le supprimer, vous pouvez le désactiver :

```sql
ALTER EVENT nom_de_l_evenement DISABLE;
```

Pour le réactiver, utilisez :

```sql
ALTER EVENT nom_de_l_evenement ENABLE;
```

### Supprimer un événement

Pour supprimer un événement lorsque vous n'en avez plus besoin, utilisez la commande `DROP EVENT` :

```sql
DROP EVENT if exists nom_de_l_evenement;
```
