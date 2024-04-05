# Gestion des utilisateurs et sécurisation

Les comptes utilisateurs MySQL
Les comptes utilisateurs MySQL

MySQL gère les comptes d'utilisateur dans la table user de la base de données système mysql.

Un compte est défini par un nom d'utilisateur et l'hôte ou les hôtes clients à partir desquels l'utilisateur peut se connecter au serveur.

 
Syntaxe des noms de compte

La syntaxe standard est 'user_name'@'host_name'.

Le nom d'hôte (host_name) est facultatif. Si seul le user_name est spécifié, il est interprété comme 'user_name'@'%', ce qui signifie que l'utilisateur peut se connecter de n'importe quel hôte.

Nom d'utilisateur : une valeur vide (la chaîne vide) correspond à tout nom d'utilisateur.

Nom d'hôte : peut prendre plusieurs formes et autoriser l'utilisation de caractères génériques :

    Une valeur d'hôte peut être un nom d'hôte ou une adresse IP (IPv4 ou IPv6).
    'localhost' indique l'hôte local, '127.0.0.1' indique l'interface de bouclage IPv4 et '::1' l'interface de bouclage IPv6.

Les noms d'utilisateur et d'hôte n'ont pas besoin d'être entre guillemets s'ils sont valides sans guillemets. Des guillemets sont nécessaires si le user_name contient des caractères spéciaux ou si le host_name contient des caractères spéciaux ou des caractères génériques comme . ou %.

Les guillemets peuvent être des guillemets inversés (`), des guillemets simples ('), ou des guillemets doubles ("). 

Note : les parties user_name et host_name, si elles sont entre guillemets, doivent l'être séparément. Donc, on écrira 'user'@'localhost' et non 'user@localhost'.

 
Les comptes réservés

Lors de l'installation de MySQL, les utilisateurs suivants sont créés :

    'root'@'localhost' : utilisé à des fins administratives, ce compte possède tous les privilèges, est un compte système et peut effectuer n'importe quelle opération. Bien que le nom de ce compte ne soit pas strictement "réservé" au sens où certaines installations peuvent renommer le compte root pour dissimuler un compte hautement privilégié avec un nom bien connu, il est fortement déconseillé de modifier ce compte en raison de son rôle central dans la gestion du système de base de données.

    'mysql.sys'@'localhost' : utilisé comme DEFINER pour les objets du schéma sys. L'utilisation du compte mysql.sys évite les problèmes qui pourraient survenir si un administrateur de base de données (DBA) renomme ou supprime le compte root. Ce compte est verrouillé, ce qui signifie qu'il ne peut pas être utilisé pour des connexions clients.

    'mysql.session'@'localhost' : utilisé en interne par les plugins pour accéder au serveur. Ce compte est également verrouillé pour empêcher son utilisation pour des connexions clients. Il est désigné comme un compte système.

    'mysql.infoschema'@'localhost' : utilisé comme DEFINER pour les vues de INFORMATION_SCHEMA. Comme pour mysql.sys, l'utilisation de mysql.infoschema est conçue pour éviter les problèmes en cas de renommage ou de suppression du compte root par un DBA. Ce compte est verrouillé et ne peut donc pas être utilisé pour des connexions clients.

 
Création d'un utilisateur

La commande CREATE USER est utilisée pour créer de nouveaux comptes d'utilisateurs.

Elle permet de configurer plusieurs aspects d'un compte, notamment l'authentification, les rôles, les options TLS, les limites de ressources, la gestion des mots de passe, ainsi que d'autres propriétés telles que des commentaires ou des attributs.

La commande contrôle également si les comptes sont initialement verrouillés ou déverrouillés.

Il y a énormément d'options pour la création des utilisateurs, voici un résumé pour savoir ce qui est disponible.
Options d'authentification (auth_option)

IDENTIFIED BY 'mot de passe' : définit un mot de passe pour le compte utilisateur.

IDENTIFIED BY RANDOM PASSWORD : demande à MySQL de générer un mot de passe aléatoire pour l'utilisateur.

IDENTIFIED WITH auth_plugin : spécifie le plugin d'authentification à utiliser pour le compte.

IDENTIFIED WITH auth_plugin BY 'mot de passe' : utilise un plugin d'authentification spécifié avec un mot de passe fourni.

IDENTIFIED WITH auth_plugin AS 'mot de passe' : utilise un plugin d'authentification spécifié avec une chaîne d'authentification, souvent un hachage de mot de passe.

Les plugins d'authentification disponibles sont :

    mysql_native_password : le plugin d'authentification traditionnel qui utilise un hachage de mot de passe SHA1. C'est le plugin par défaut pour MySQL 5.7 et les versions antérieures.
    caching_sha2_password : introduit à partir de MySQL 8.0, il utilise un hachage SHA-256 et offre de meilleures performances et une meilleure sécurité. C'est le plugin d'authentification par défaut pour MySQL 8.0 et les versions ultérieures.
    sha256_password : semblable à caching_sha2_password, mais sans la mise en cache des hachages de mot de passe, ce qui peut entraîner une légère baisse des performances.
    auth_socket : utilisé principalement sur les systèmes Unix et Linux, permet une authentification basée sur les sockets, où les utilisateurs sont authentifiés par le système d'exploitation. Cela signifie que si un utilisateur se connecte depuis le même système où le serveur MySQL est exécuté, il est authentifié automatiquement s'il se connecte en utilisant le socket Unix de MySQL.
    mysql_old_password : un ancien plugin d'authentification utilisé dans les versions de MySQL antérieures à 5.7. Il utilise un hachage de mot de passe moins sécurisé et est obsolète dans les versions plus récentes.
    pam : le plugin PAM (Pluggable Authentication Modules) permet à MySQL de déléguer l'authentification à PAM, ce qui ouvre la possibilité d'intégrer MySQL à des systèmes d'authentification centralisés comme LDAP, Kerberos ou d'autres systèmes compatibles PAM.
    windows : ce plugin permet l'authentification des utilisateurs MySQL via le système d'authentification Windows, en utilisant les mécanismes d'authentification intégrés de Windows.
    dialog : ce plugin est souvent utilisé avec d'autres systèmes d'authentification externes. Il affiche une invite de dialogue au client qui peut être utilisée pour collecter des informations d'authentification supplémentaires.
    authentication_ldap_sasl : plugin d'authentification permettant l'intégration avec un annuaire LDAP en utilisant SASL (Simple Authentication and Security Layer).
    authentication_ldap_simple : permet l'authentification contre un serveur LDAP en utilisant une liaison simple, sans utiliser SASL.

Options TLS (tls_option)

REQUIRE SSL : exige une connexion sécurisée.

REQUIRE X509 : exige un certificat X509 valide pour la connexion.

REQUIRE ISSUER 'issuer' : spécifie l'émetteur du certificat.

REQUIRE SUBJECT 'subject' : spécifie le sujet du certificat.

REQUIRE CIPHER 'cipher' : spécifie le chiffrement à utiliser.
Limites de ressources (resource_option)

MAX_QUERIES_PER_HOUR nombre: limite le nombre de requêtes qu'un utilisateur peut exécuter par heure.

MAX_UPDATES_PER_HOUR nombre: limite le nombre de mises à jour par heure.

MAX_CONNECTIONS_PER_HOUR nombre: limite le nombre de connexions par heure.

MAX_USER_CONNECTIONS nombre: limite le nombre total de connexions simultanées.
Options de gestion de mot de passe (password_option)

PASSWORD EXPIRE : force l'expiration du mot de passe.

PASSWORD HISTORY N : empêche la réutilisation des N derniers mots de passe.

PASSWORD REUSE INTERVAL N DAY : définit un intervalle avant qu'un mot de passe ne puisse être réutilisé.

FAILED_LOGIN_ATTEMPTS N et PASSWORD_LOCK_TIME N : configurent le suivi des échecs de connexion et le verrouillage temporaire du compte après N tentatives infructueuses.
Commentaires et attributs (COMMENT et ATTRIBUTE)

COMMENT 'commententaire' : ajoute un commentaire descriptif au compte utilisateur.

ATTRIBUTE 'json_object' : ajoute des attributs au format JSON au compte utilisateur.
Options de verrouillage de compte (lock_option)

ACCOUNT LOCK : verrouille le compte pour empêcher toute connexion.

ACCOUNT UNLOCK : déverrouille le compte pour permettre les connexion

 
Exemples de création d'utilisateurs

Création d'un utilisateur administrateur avec un mot de passe spécifié :

CREATE USER 'admin'@'localhost'
IDENTIFIED BY 'MotDePasseSécurisé!';

Création d'un utilisateur qui ne peut se connecter que via SSL :

CREATE USER 'secure_user'@'%'
IDENTIFIED BY 'EncoreUnMotDePasseSécurisé!'
REQUIRE SSL;

Création d'un utilisateur avec des limites de ressources définies :

CREATE USER 'limited_user'@'localhost'
IDENTIFIED BY 'MotDePasseSécurisé!'
WITH MAX_QUERIES_PER_HOUR 100 MAX_CONNECTIONS_PER_HOUR 10;

Création d'un utilisateur avec un mot de passe qui expire après 90 jours :

CREATE USER 'temp_user'@'localhost'
IDENTIFIED BY 'MotDePasseSécurisé!'
PASSWORD EXPIRE INTERVAL 90 DAY;

Création d'un utilisateur avec des tentatives de connexion limitées :

CREATE USER 'try_user'@'localhost'
IDENTIFIED BY 'MotDePasseSécurisé!'
FAILED_LOGIN_ATTEMPTS 5 PASSWORD_LOCK_TIME 2;

Voir et modifier les utilisateurs
Afficher les utilisateurs MySQL
Avec une requête SQL

Vous pouvez utiliser cette commande pour lister tous les utilisateurs et leurs privilèges (nous verrons les privilèges dans la prochaine leçon) :

SELECT * FROM mysql.user;

Dans l'interface Workbench

Pour afficher les utilisateurs de MySQL à l'aide de MySQL Workbench, suivez les étapes suivantes :

    Ouvrez MySQL Workbench et connectez-vous à votre serveur MySQL.
    Dans le volet de navigation de gauche, cliquez sur l'onglet Administration puis sur "Users and Privileges".
    Vous verrez une liste de tous les utilisateurs et vous pourrez tout gérer : les mots de passe, la méthode d'authentification, les hôtes, les noms d'utilisateurs, les limites, les rôles et les privilèges.

 
Modifier les utilisateurs

Vous pouvez modifier les utilisateurs depuis l'interface Workbench vue précédemment, mais vous pouvez aussi le faire avec des requêtes SQL.

Il faut dans ce cas utiliser ALTER USER et la syntaxe est la même que pour la création :

ALTER USER 'utilisateur'@'hôte' IDENTIFIED BY 'nouveau mot de passe';

 
Supprimer des utilisateurs

Vous pouvez supprimer un utilisateur avec la commande DROP USER :

DROP USER 'utilisateur'@'hôte';

Les privilèges
Introduction aux privilèges et aux tables de privilèges
Les privilèges

Les privilèges sont des droits d'accès qui permettent aux utilisateurs de réaliser certaines opérations dans MySQL.

Il existe différents types de privilèges qui correspondent à des actions spécifiques, comme le droit de lire des données (SELECT), d'écrire des données (INSERT, UPDATE, DELETE), de créer des objets dans la base de données (CREATE), d'administrer des utilisateurs (CREATE USER), etc.

Les privilèges peuvent être accordés ou révoqués par les administrateurs de la base de données à des utilisateurs ou à des rôles spécifiques. Nous verrons les rôles dans la prochaine leçon.

Ils peuvent être appliqués à différents niveaux, y compris le serveur entier, une base de données spécifique, une table spécifique, ou même une colonne spécifique dans une table.
Les tables de privilèges (Grant Tables)

Les grant tables, ou tables de privilèges, sont des tables système internes utilisées par MySQL pour stocker des informations sur les privilèges accordés.

MySQL utilise ces tables pour déterminer si un utilisateur a le droit d'exécuter une action spécifique.

Les tables de privilèges principales incluent user, db, tables_priv, columns_priv, et procs_priv, chacune stockant des informations relatives à différents types de privilèges (globaux, par base de données, par table, par colonne, et par routine, respectivement).

Lorsqu'un utilisateur tente d'exécuter une action, MySQL consulte les grant tables pour vérifier si l'utilisateur dispose des privilèges nécessaires.

Les modifications apportées aux privilèges via les commandes GRANT et REVOKE sont reflétées dans ces tables.

 
Les types de privilèges

Il existe trois types de privilèges :

    Privilèges administratifs : ces privilèges permettent de gérer le fonctionnement du serveur MySQL. Ils sont considérés comme globaux car ils ne sont pas spécifiques à une base de données en particulier.

    Privilèges de base de données : ces privilèges s'appliquent à une base de données spécifique et à tous les objets qu'elle contient. Ils peuvent être accordés pour des bases de données spécifiques ou de manière globale.

    Privilèges sur les objets de base de données : ces privilèges peuvent être accordés pour des objets spécifiques dans une base de données, comme les tables, vues, et procédures stockées.

 
Attribuer des privilèges

Pour attribuer des privilèges à un utilisateur on utilise la commande GRANT. 

La syntaxe est :

GRANT privilège1, privilège2 ON cible TO 'utilisateur'@'hôte' [WITH GRANT OPTION];

    privilège1, privilège2, etc. représentent les privilèges que vous souhaitez attribuer.
    cible spécifie sur quoi les privilèges sont accordés, comme une base de données (database.*), une table spécifique (database.table), ou toutes les bases de données (*.*).
    'utilisateur'@'hôte' identifie le compte utilisateur. 'hôte' peut être remplacé par '%' pour indiquer tout hôte.
    WITH GRANT OPTION permet à l'utilisateur de transmettre les privilèges à d'autres utilisateurs.

 
Révoquer des privilèges

Pour retirer des privilèges, on utilise la commande REVOKE. La syntaxe générale est la suivante :

REVOKE privilège1, privilège2 ON cible FROM 'utilisateur'@'hôte';

    privilège1, privilège2, etc. représentent les privilèges que vous souhaitez retirer.
    cible et 'utilisateur'@'hôte' ont la même signification que dans la commande GRANT.

 
Exemples

Attribuer tous les privilèges sur une base de données spécifique à un utilisateur :

GRANT ALL PRIVILEGES ON ma_base_de_donnees.* TO 'mon_utilisateur'@'localhost';

Révoquer tous les privilèges d'un utilisateur sur une base de données spécifique :

REVOKE ALL PRIVILEGES ON ma_base_de_donnees.* FROM 'mon_utilisateur'@'localhost';

Accorder le privilège de création d'utilisateurs et de gestion des rôles :

GRANT CREATE USER, ROLE_ADMIN ON *.* TO 'admin_user'@'localhost';

Permettre à un utilisateur d'exécuter une procédure stockée :

GRANT EXECUTE ON PROCEDURE ma_base_de_donnees.ma_procedure TO 'executeur_procedure'@'localhost';

Permettre à un utilisateur de gérer les événements pour une base de données :

GRANT EVENT ON ma_base_de_donnees.* TO 'gestionnaire_evenements'@'%';

 
Exemple WITH GRANT OPTION

Imaginons que vous souhaitiez permettre à un utilisateur, nommé admin_bdd, de non seulement accéder à toutes les tables de la base de données exemple_db mais aussi de pouvoir accorder ces mêmes privilèges à d'autres utilisateurs. Voici comment vous pourriez procéder :

GRANT ALL PRIVILEGES ON exemple_db.* TO 'admin_bdd'@'localhost' WITH GRANT OPTION;

    ALL PRIVILEGES accorde tous les privilèges sur toutes les tables de la base de données exemple_db à l'utilisateur admin_bdd.
    WITH GRANT OPTION permet à admin_bdd de donner à d'autres utilisateurs les mêmes privilèges qu'il possède sur exemple_db.

Après l'exécution de cette commande, admin_bdd peut exécuter des commandes GRANT pour accorder à d'autres utilisateurs des privilèges sur la base de données exemple_db. Par exemple, admin_bdd pourrait exécuter la commande suivante pour accorder des privilèges de sélection à un autre utilisateur :

GRANT SELECT ON exemple_db.* TO 'utilisateur_lecture'@'localhost';

Si à un moment donné vous décidez que admin_bdd ne devrait plus avoir la capacité d'accorder des privilèges à d'autres, vous pouvez révoquer cette option spécifique sans retirer les autres privilèges :

REVOKE GRANT OPTION FOR ALL PRIVILEGES ON exemple_db.* FROM 'admin_bdd'@'localhost';

 
Bonnes pratiques

Supposons que nous avons une application Web (par exemple Node.js ou Spring Boot) et que nous souhaitions créer un utilisateur pour cette application.

Pour une application Web, nous aurons généralement besoin des privilèges SELECT, INSERT, UPDATE, et DELETE. Supposons que notre base de données utilisée par cette application s'appelle maBaseDeDonnees.

Nous attriburerions les privilèges suivants :

GRANT SELECT, INSERT, UPDATE, DELETE ON maBaseDeDonnees.* TO 'monUtilisateurApp'@'localhost';

Nous devons en effet respecter les bonnes pratiques suivantes :

    Principe de moindre privilège : accordez uniquement les privilèges strictement nécessaires à votre application pour fonctionner. Évitez d'utiliser GRANT ALL PRIVILEGES, sauf si c'est absolument nécessaire.
    Séparation des privilèges : si votre application comporte plusieurs composants ou services qui interagissent avec la base de données de manière différente, envisagez de créer des utilisateurs distincts avec des ensembles de privilèges minimisés pour chaque composant.

 
Tableau des privilèges disponibles

Il y a de nombreux privilèges disponibles pour contrôler très finement l'accès des utilisateurs :
Privilège 	Colonne de la table de privilèges 	Contexte
ALL [PRIVILEGES] 	Synonyme pour « tous les privilèges » 	Administration du serveur
ALTER 	Alter_priv 	Tables
ALTER ROUTINE 	Alter_routine_priv 	Routines stockées
CREATE 	Create_priv 	Bases de données, tables ou index
CREATE ROLE 	Create_role_priv 	Administration du serveur
CREATE ROUTINE 	Create_routine_priv 	Routines stockées
CREATE TABLESPACE 	Create_tablespace_priv 	Administration du serveur
CREATE TEMPORARY TABLES 	Create_tmp_table_priv 	Tables
CREATE USER 	Create_user_priv 	Administration du serveur
CREATE VIEW 	Create_view_priv 	Vues
DELETE 	Delete_priv 	Tables
DROP 	Drop_priv 	Bases de données, tables ou vues
DROP ROLE 	Drop_role_priv 	Administration du serveur
EVENT 	Event_priv 	Bases de données
EXECUTE 	Execute_priv 	Routines stockées
FILE 	File_priv 	Accès aux fichiers sur l'hôte du serveur
GRANT OPTION 	Grant_priv 	Bases de données, tables ou routines stockées
INDEX 	Index_priv 	Tables
INSERT 	Insert_priv 	Tables ou colonnes
LOCK TABLES 	Lock_tables_priv 	Bases de données
PROCESS 	Process_priv 	Administration du serveur
PROXY 	Voir la table proxies_priv 	Administration du serveur
REFERENCES 	References_priv 	Bases de données ou tables
RELOAD 	Reload_priv 	Administration du serveur
REPLICATION CLIENT 	Repl_client_priv 	Administration du serveur
REPLICATION SLAVE 	Repl_slave_priv 	Administration du serveur
SELECT 	Select_priv 	Tables ou colonnes
SHOW DATABASES 	Show_db_priv 	Administration du serveur
SHOW VIEW 	Show_view_priv 	Vues
SHUTDOWN 	Shutdown_priv 	Administration du serveur
SUPER 	Super_priv 	Administration du serveur
TRIGGER 	Trigger_priv 	Tables
UPDATE 	Update_priv 	Tables ou colonnes
USAGE 	Synonyme pour « aucun privilège » 	Administration du serveur

Les rôles
Introduction aux rôles

Les rôles sont des ensembles nommés de privilèges qui peuvent être accordés à des utilisateurs pour simplifier la gestion des autorisations.

Les rôles peuvent être créés, modifiés et supprimés à l'aide des instructions CREATE ROLE, ALTER ROLE et DROP ROLE.

Les privilèges peuvent être accordés ou révoqués à un rôle à l'aide des instructions GRANT et REVOKE, de la même manière que pour les utilisateurs.

Les rôles peuvent être associés à des utilisateurs à l'aide de l'instruction GRANT.

Lorsqu'un utilisateur se connecte à MySQL, il hérite des privilèges qui lui sont accordés directement, ainsi que des privilèges accordés à tous les rôles qui lui sont associés.

 
Créer un rôle

Choisissez des noms de rôles qui décrivent clairement leur but ou les privilèges qu'ils accordent.

Pour créer un rôle, utilisez la commande CREATE ROLE. Par exemple, pour créer un rôle appelé gestionnaire_db :

CREATE ROLE gestionnaire_db;

Une fois le rôle créé, vous pouvez lui attribuer des privilèges. Par exemple, pour permettre au rôle gestionnaire_db de créer des bases de données et de gérer les utilisateurs :

GRANT CREATE ON *.* TO gestionnaire_db;
GRANT CREATE USER ON *.* TO gestionnaire_db;

 
Attribution de rôles aux utilisateurs

Pour attribuer le rôle gestionnaire_db à un utilisateur nommé alice :

GRANT gestionnaire_db TO alice;

Pour que les privilèges associés à un rôle soient effectifs pour un utilisateur, l'utilisateur doit activer le rôle. Par exemple, alice peut activer son rôle gestionnaire_db en utilisant :

SET ROLE gestionnaire_db;

 
SET DEFAULT ROLE

SET DEFAULT ROLE définit quels rôles deviennent automatiquement actifs lorsqu'un utilisateur se connecte au serveur. Cela affecte les sessions futures et non la session courante.

Par exemple :

SET DEFAULT ROLE ALL TO 'app_user'@'localhost';

Définit tous les rôles accordés à app_user@localhost comme étant actifs par défaut lors de la connexion.

SET DEFAULT ROLE 'app_read', 'app_write' TO 'app_user'@'localhost';

Définit spécifiquement app_read et app_write comme étant les rôles actifs par défaut pour app_user@localhost.

 
Gestion des rôles

Pour voir les rôles existants et leurs privilèges, utilisez SHOW GRANTS FOR en spécifiant le rôle :

SHOW GRANTS FOR 'gestionnaire_db';

Pour révoquer un privilège d'un rôle :

REVOKE CREATE ON *.* FROM gestionnaire_db;

Pour supprimer un rôle qui n'est plus nécessaire :

DROP ROLE gestionnaire_db;