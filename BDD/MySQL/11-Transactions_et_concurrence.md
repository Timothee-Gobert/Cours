# Transactions et concurrence

Introduction aux principes ACID et aux transactions
Que sont les transactions ?

Les transactions sont des mécanismes essentiels dans les systèmes de gestion de base de données (SGBD) tels que MySQL, qui permettent de maintenir l'intégrité des données en traitant plusieurs opérations comme une seule unité logique.

Les propriétés ACID des transactions garantissent que la base de données reste dans un état cohérent et fiable, même en cas d'erreurs ou de problèmes inattendus.

 
Les principes ACID

ACID est un acronyme qui se réfère aux quatre propriétés clés des transactions dans les bases de données :

    Atomicité (Atomicity)
    Cohérence (Consistency)
    Isolation (Isolation)
    Durabilité (Durability)

Chacune de ces propriétés joue un rôle essentiel dans l'assurance que les transactions sont exécutées de manière fiable.

Note : nous verrons la syntaxe en détail dans la prochaine leçon, les exemples sont là pour comprendre les principes, ils ne sont pas fonctionnels.
Atomicité

L'atomicité assure que toutes les opérations au sein d'une transaction sont exécutées comme une seule unité indivisible.

Autrement dit, soit elles sont toutes réussies ou, en cas d'échec, elles sont toutes annulées.

Avec MySQL, cela signifie que si une partie de la transaction échoue, l'ensemble de la transaction sera annulé (un "rollback" sera effectué), et la base de données sera retournée à son état avant le début de la transaction.

Pour illustrer cela avec un exemple simple dans MySQL, considérons un scénario où vous gérez une base de données pour une librairie en ligne. Vous voulez mettre à jour le stock d'un livre après une vente et également augmenter le nombre de ventes pour ce livre. Les deux opérations doivent réussir pour que la transaction soit complète :

START TRANSACTION;

-- Décrémenter le stock pour un livre après une vente
UPDATE livres SET stock = stock - 1 WHERE id_livre = 1;

-- Si l'opération précédente réussit, incrémenter le compteur de ventes
UPDATE statistiques_ventes SET nombre_ventes = nombre_ventes + 1 WHERE id_livre = 1;

-- Simulons une vérification : le stock ne doit pas être négatif
DECLARE @stock INT;
SELECT @stock = stock FROM livres WHERE id_livre = 1;
IF @stock < 0 THEN
  -- Si le stock est négatif, annuler la transaction
  ROLLBACK;
ELSE
  -- Sinon, valider toutes les opérations
  COMMIT;
END IF;

Dans cet exemple, si la mise à jour du stock ou des statistiques échoue, ou si le stock devient négatif, l'instruction ROLLBACK est exécutée. Cela annule toutes les opérations effectuées dans cette transaction, préservant ainsi l'intégrité des données de la base de données. Si tout se passe bien, la transaction est validée avec COMMIT, et toutes les modifications sont définitivement appliquées à la base de données.
Cohérence

La cohérence garantit que chaque transaction va conduire la base de données d'un état valide à un autre état valide, en préservant toutes les règles et contraintes définies dans la base de données.

Les transactions ne compromettent pas l'intégrité de la base de données ; si une transaction ne peut pas maintenir la cohérence de la base de données, elle ne sera pas effectuée.

Par exemple :

-- Supposons que la table `comptes` ait une contrainte de solde non négatif
START TRANSACTION;
UPDATE comptes SET solde = solde - 100 WHERE client_id = 1;
-- Si le solde devient négatif, la contrainte sera violée et la transaction sera annulée
-- ROLLBACK; --
COMMIT;

Isolation

L'isolation détermine comment les modifications apportées par une transaction sont visibles par d'autres transactions.

MySQL offre plusieurs niveaux d'isolation qui peuvent être configurés, du moins isolé (READ UNCOMMITTED) au plus isolé (SERIALIZABLE). Le niveau d'isolation choisi peut affecter la performance et la concurrence de la base de données.

Exemple :

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT * FROM comptes WHERE client_id = 1;
-- D'autres transactions ne pourront pas modifier ces lignes jusqu'à ce que la transaction soit terminée
COMMIT;

Nous verrons l'isolation en détail dans une leçon dédiée.
Durabilité

La durabilité assure que, une fois qu'une transaction a été validée (commit), ses résultats sont permanents et persistants, même en cas de panne système.

Cela signifie que les modifications apportées par la transaction ne seront pas perdues en cas de défaillance du système et seront récupérables après un redémarrage de la base de données.

MySQL utilise des mécanismes tels que les journaux de transactions (transaction logs) pour garantir que même en cas de panne, les transactions validées ne seront pas perdues.

Exemple :

START TRANSACTION;
INSERT INTO comptes (client_id, solde) VALUES (2, 200);
COMMIT;
-- Après le COMMIT, même en cas de crash, les données insérées seront persistées

Syntaxe des transactions
Le mode autocommit

MySQL fonctionne par défaut avec le mode autocommit activé, ce qui signifie que chaque instruction est exécutée et validée individuellement comme sa propre transaction.

Pour contrôler ce comportement, utilisez la commande SET :

SET autocommit = 0; -- pour désactiver l'autocommit
SET autocommit = 1; -- pour activer l'autocommit

 
Syntaxe des transactions
Démarrage

Pour démarrer une transaction dans MySQL, vous pouvez utiliser la commande START TRANSACTION. 

START TRANSACTION;

Ou alors l'alias BEGIN :

BEGIN;

Une fois qu'une transaction est commencée avec START TRANSACTION, le mode autocommit est désactivé pour cette session jusqu'à ce que la transaction soit terminée, ce qui signifie que les modifications apportées à la base de données ne sont pas encore permanentes.

Après cette commande, vous pouvez exécuter une série d'opérations SQL qui forment la transaction. 
Validation

La commande COMMIT valide la transaction en cours, rendant toutes les modifications permanentes. Voici comment vous pourriez valider une transaction :

COMMIT;

Lorsque vous exécutez COMMIT, toutes les modifications de données effectuées dans la transaction sont écrites de manière durable sur le disque. Après un COMMIT, le mode autocommit est réactivé si aucune autre transaction n'est démarrée.
Annulation

La commande ROLLBACK annule la transaction en cours, annulant toutes les modifications qui ont été faites. Voici comment vous pourriez annuler une transaction :

ROLLBACK;

Un ROLLBACK peut être nécessaire si vous rencontrez une erreur pendant la transaction ou si vous souhaitez délibérément annuler les modifications pour une autre raison.

Après un ROLLBACK, le mode autocommit est réactivé si aucune autre transaction n'est démarrée.

 
Exemples

Imaginons que vous vouliez effectuer une série d'opérations qui transfèrent de l'argent d'un compte à un autre tout en enregistrant cette transaction dans un journal.

START TRANSACTION;
UPDATE comptes SET solde = solde - 100 WHERE compte_id = 1;
UPDATE comptes SET solde = solde + 100 WHERE compte_id = 2;
INSERT INTO journal_transactions (type, montant) VALUES ('transfert', 100);
COMMIT;

Autre exemple de transaction utile, la gestion de stocks :

START TRANSACTION;

-- Ajouter une nouvelle commande
INSERT INTO commandes (client_id, date_commande) VALUES (1234, NOW());

-- Mise à jour du stock des articles commandés
UPDATE articles SET quantite_en_stock = quantite_en_stock - 2 WHERE article_id = 5678;

-- Vérification que le stock ne devient pas négatif
SELECT quantite_en_stock INTO @quantite FROM articles WHERE article_id = 5678;
IF @quantite < 0 THEN
  ROLLBACK;
ELSE
  COMMIT;
END IF;

 
Bonnes pratiques pour les transactions

L'utilisation de transactions peut avoir des implications sur les performances, notamment en raison des verrous nécessaires pour maintenir l'isolation des transactions.

De plus, une transaction non validée qui reste ouverte trop longtemps peut entraîner l'accumulation de verrous, affectant ainsi la concurrence et les performances de la base de données. 

Il faut donc généralement respecter les bonnes pratiques suivantes :

    Utilisez des transactions pour des opérations qui nécessitent de modifier plusieurs tables ou enregistrements de manière atomique.
    N'utilisez pas de transactions pour des opérations simples ou pour des lectures, à moins que vous n'ayez besoin de cohérence dans les lectures.
    Veillez à ne pas laisser des transactions ouvertes, car cela pourrait bloquer d'autres opérations et affecter les performances.

Introduction à la concurrence
Introduction à la concurrence

Dans un système de gestion de base de données, les problèmes de concurrence surviennent lorsque plusieurs transactions s'exécutent en parallèle et interagissent de manière à compromettre l'intégrité des données. 

Les problèmes de concurrence courants incluent les mises à jour perdues, les lectures sales, les lectures non répétables et les lectures fantômes.

 
Problèmes de concurrences potentiels
Mises à jour perdues (Lost Updates)

Les mises à jour perdues se produisent lorsque deux transactions ou plus lisent le même enregistrement et le modifient ensuite sur la base de la valeur lue, mais l'une des modifications écrase les autres sans prendre en compte les changements intermédiaires.

Exemple :

    La Transaction A lit une valeur X.
    La Transaction B lit la valeur X.
    La Transaction A modifie X en X' et valide ses modifications.
    La Transaction B, sans tenir compte des modifications de A, modifie X en X'' et valide ses modifications. Les modifications de A sont perdues.

Lectures sales (Dirty Reads)

Une lecture sale se produit lorsqu'une transaction lit les modifications apportées par une autre transaction qui n'a pas encore été validée (commit).

Si la transaction qui a apporté les modifications est annulée (rollback), la première transaction aura lu des données qui n'ont jamais été validées et qui sont donc considérées comme "sales".

Exemple :

    La Transaction A modifie une valeur X en X' mais ne valide pas encore ses modifications.
    La Transaction B lit la valeur X' modifiée par A.
    Si A est annulée, B a lu une valeur qui n'existe plus dans la base de données.

Lectures non répétables (Non-repeatable Reads)

Une lecture non répétable se produit lorsqu'une transaction lit le même enregistrement à plusieurs reprises et constate que les données ont changé entre les lectures, généralement en raison de modifications validées par une autre transaction entretemps.

Exemple :

    La Transaction A lit une valeur X.
    La Transaction B modifie la valeur X en X' et valide ses modifications.
    La Transaction A relit X et obtient la nouvelle valeur X', constatant ainsi une modification des données entre les deux lectures.

Lectures fantômes (Phantom Reads)

Les lectures fantômes se produisent dans les situations où une transaction répète une requête retournant un ensemble de résultats et découvre que le résultat a changé à cause de l'ajout ou de la suppression de lignes (par une autre transaction) qui satisfont aux critères de la requête.

Exemple :

    La Transaction A exécute une requête qui retourne un ensemble de lignes.
    La Transaction B ajoute ou supprime des lignes affectant le même ensemble de résultats.
    La Transaction A réexécute la même requête et obtient un ensemble de résultats différent, incluant ou excluant les nouvelles lignes ajoutées ou supprimées par B.

L'isolation en détails
L'isolation des transactions

Les niveaux d'isolation des transactions déterminent dans quelle mesure une transaction doit être isolée des modifications apportées par d'autres transactions simultanées.

Chaque niveau offre un équilibre différent entre la performance et la précision des données.

 
Gestion des problèmes de concurrence

MySQL offre différents niveaux d'isolation des transactions pour gérer ces problèmes :

    Read Uncommitted : permet les lectures sales, ne protège pas contre les lectures non répétables ni les lectures fantômes.
    Read Committed : empêche les lectures sales, mais permet les lectures non répétables.
    Repeatable Read (niveau par défaut dans InnoDB) : empêche les lectures sales et les lectures non répétables, mais peut permettre certaines formes de lectures fantômes.
    Serializable : le niveau d'isolation le plus élevé, empêche les lectures sales, les lectures non répétables et les lectures fantômes, en verrouillant les enregistrements impliqués dans les critères de sélection d'une transaction.

 
Exemples sur Workbench

Chaque scénario illustrera un type de problème de concurrence.

Notez que pour exécuter ces exemples, vous devrez utiliser deux connexions différentes dans la même instance de Workbench pour simuler les transactions concurrentes.
Mises à jour perdues

Ce scénario est fictif car ce n'est pas possible avec MySQL et InnoDB.

Supposons que deux requêtes essaient de mettre à jour le budget d'un film en même temps.

Connexion 1 :

-- Cette session démarre une transaction pour mettre à jour le budget d'un film
START TRANSACTION;
SELECT budget FROM movie WHERE movie_id = 1;
-- Imaginez que le budget actuel soit de 1 000 000 et que cette session veuille ajouter 500 000

Connexion 2 :

-- Cette session essaie également de mettre à jour le budget en même temps
START TRANSACTION;
UPDATE movie SET budget = budget + 250000 WHERE movie_id = 1;
COMMIT;

Connexion 1 continue :

-- La session 1 n'est pas au courant de la mise à jour qui vient de se produire dans la session 2
UPDATE movie SET budget = budget + 500000 WHERE movie_id = 1;
COMMIT;

Lectures sales

Pour ce scénario, vous devez avoir un niveau d'isolation qui permet les lectures sales (READ UNCOMMITTED).

Il faut exécuter ligne par ligne pour que cela fonctionne sur Workbench.

Connexion 1 :

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
-- Cette session fait une modification sur le budget d'un film sans valider la transaction
UPDATE movie SET budget = budget + 100000 WHERE movie_id = 1;
-- Ne validez pas encore la transaction

Connexion 2 :

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
-- Cette session lit les modifications non validées
SELECT budget FROM movie WHERE movie_id = 1;
COMMIT;

Connexion 1 continue :

-- Session 1 décide d'annuler la mise à jour
ROLLBACK;

Résultat : la connexion 2 a lu une valeur de budget qui a été modifiée mais jamais validée (lecture sale) et qui est par la suite annulée par la connexion 1.
Lectures non répétables

Ce scénario nécessite un niveau d'isolation READ COMMITTED.

Connexion 1 :

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
-- Cette session lit le budget d'un film
SELECT budget FROM movie WHERE movie_id = 1;
-- Imaginez une pause ici

Connexion 2 :

-- Cette session met à jour le budget et valide la transaction
START TRANSACTION;
UPDATE movie SET budget = budget + 300000 WHERE movie_id = 1;
COMMIT;

Connexion 1 continue :

-- La session 1 lit à nouveau le budget et remarque un changement
SELECT budget FROM movie WHERE movie_id = 1;
COMMIT;

Résultat : la connexion 1 voit deux valeurs différentes pour le même budget dans la même transaction, ce qui constitue une lecture non répétable.
Lectures fantômes

Ce scénario n'est pas testable avec InnoDB qui ne permet pas les lectures fantômes.

Connexion 1 :

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
-- Cette session fait une sélection basée sur une condition
SELECT * FROM movie WHERE budget > 1000000;
-- Imaginez une pause ici

Connexion 2 :

-- Cette session ajoute un nouveau film qui correspond à la condition de la session 1
START TRANSACTION;
INSERT INTO movie (movie_id, title, budget) VALUES (5, 'New Blockbuster', 2000000);
COMMIT;

Connexion 1 continue :

-- La session 1 refait la même sélection et voit maintenant un nouveau film
SELECT * FROM movie WHERE budget > 1000000

 
Cas d'utilisation des niveaux d'isolation

Dans la plupart des cas vous n'aurez pas besoin de gérer les transactions et les niveaux d'isolation.

Mais si vous avez besoin de transactions, voici dans quels cas utiliser chaque niveau d'isolation.
READ UNCOMMITTED

Ce niveau est utile lorsque les exigences de cohérence des données sont minimales et que les performances sont plus critiques. Il est rarement utilisé dans la pratique en raison des problèmes de concurrence comme les lectures sales qu'il autorise.

Exemple : dans un système de logging où il est acceptable de lire des données non validées (comme des logs en cours d'écriture), READ UNCOMMITTED peut être utilisé pour améliorer les performances de lecture.
READ COMMITTED

Ce niveau est souvent utilisé dans les applications où la cohérence des données est importante mais où les lectures non répétables sont acceptables. Il offre de meilleures performances que des niveaux d'isolation plus élevés.

Exemple : un système de réservation en ligne où les utilisateurs peuvent voir les dernières places disponibles qui ont été validées. Même si les données peuvent changer rapidement, il est essentiel que les utilisateurs ne voient pas de données non validées.
REPEATABLE READ

Ce niveau est le niveau par défaut pour InnoDB dans MySQL. Il est recommandé pour la plupart des applications nécessitant un bon équilibre entre intégrité des données et performance. Il prévient les lectures non répétables mais peut permettre certaines lectures fantômes.

Exemple : un système de gestion des stocks où il est important qu'une transaction qui lit plusieurs fois le stock d'un article pendant son exécution voie toujours la même quantité, même si d'autres transactions sont en cours.
SERIALIZABLE

C'est le niveau le plus élevé d'isolation qui garantit la sérialisation complète des transactions. Il est utilisé lorsque l'intégrité des données est critique et que les transactions doivent être traitées comme si elles étaient exécutées de manière séquentielle.

Exemple : dans un système bancaire, lorsqu'une transaction implique le transfert d'argent entre comptes, le niveau SERIALIZABLE peut être utilisé pour assurer qu'aucune autre transaction ne puisse interférer, évitant ainsi les problèmes de concurrence comme les lectures fantômes.

Les deadlocks
Qu'est-ce qu'un deadlock ?

Les deadlocks sont des situations dans un système de gestion de base de données où deux ou plusieurs transactions attendent indéfiniment les unes sur les autres pour libérer des verrous sur des ressources. 

Un deadlock se produit généralement lorsqu'il y a une concurrence pour des ressources partagées, comme des lignes dans une table, et que chaque transaction détient un verrou tout en essayant d'acquérir un verrou détenu par l'autre transaction.

Prenons un exemple pour mieux comprendre.

Considérez le scénario suivant avec deux transactions et deux ressources (par exemple, deux lignes dans une table):

    La Transaction A détient un verrou sur la Ressource 1 et demande un verrou sur la Ressource 2.
    La Transaction B détient un verrou sur la Ressource 2 et demande un verrou sur la Ressource 1.
    Ni la Transaction A ni la Transaction B ne peut progresser, car chacune attend que l'autre libère le verrou qu'elle détient.

 
Que se passe-t-il lors d'un deadlock ?

MySQL détecte automatiquement les deadlocks et choisit une transaction victime pour résoudre le problème.

La transaction victime est interrompue et reçoit un message d'erreur indiquant qu'un deadlock a été détecté et qu'elle a été choisie pour le rollback.

Les verrous détenus par cette transaction sont libérés, ce qui permet aux autres transactions en attente de continuer.

 
Comment gérer les deadlocks ?

Les applications doivent être conçues pour gérer les erreurs de transaction, y compris les deadlocks.

Cela signifie généralement capturer l'erreur de deadlock et réessayer la transaction.

Réduire la taille et la durée des transactions peut aider à minimiser les deadlocks. Si possible, essayez de verrouiller les ressources dans le même ordre dans toutes les transactions.

 
Exemple de deadlock

Voici un exemple simple illustrant un deadlock :

Transaction A :

START TRANSACTION;
SELECT * FROM comptes WHERE compte_id = 1 FOR UPDATE;
-- Une pause est introduite ici, attendant une action de la part de Transaction B

Transaction B :

START TRANSACTION;
SELECT * FROM comptes WHERE compte_id = 2 FOR UPDATE;
-- Transaction B tente maintenant d'obtenir un verrou sur le compte_id = 1
SELECT * FROM comptes WHERE compte_id = 1 FOR UPDATE;

Transaction A continue :

-- Transaction A tente d'obtenir un verrou sur le compte_id = 2, ce qui crée un deadlock
SELECT * FROM comptes WHERE compte_id = 2 FOR UPDATE;

