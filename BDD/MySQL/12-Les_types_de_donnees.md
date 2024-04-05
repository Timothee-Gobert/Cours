# Les types de données

Types numériques : les entiers
Les types entiers

Les types entiers sont utilisés pour stocker des données numériques qui ne contiennent pas de fractions (pas de nombre après la virgule).

Ils sont couramment utilisés pour stocker des données comme les identifiants, les quantités, les codes, etc. MySQL offre une variété de types entiers pour s'adapter à différentes tailles et besoins de stockage.
TINYINT

    Plage de valeurs : -128 à 127 pour la version signée, et 0 à 255 pour la version non signée.
    Utilisation courante : pour de très petits entiers ou pour économiser de l'espace disque quand les valeurs ne sont pas grandes. Souvent utilisé pour stocker des valeurs booléennes (où 0 est FALSE et 1 est TRUE).

SMALLINT

    Plage de valeurs : -32768 à 32767 pour la version signée, et 0 à 65535 pour la version non signée.
    Utilisation courante : pour des entiers légèrement plus grands, comme des scores, des petits compteurs, ou des identifiants.

MEDIUMINT

    Plage de valeurs : -8388608 à 8388607 pour la version signée, et 0 à 16777215 pour la version non signée.
    Utilisation courante : pour des entiers de taille moyenne qui dépassent les limites de SMALLINT, comme des identifiants d'utilisateurs dans des systèmes avec plus d'utilisateurs.

INT ou INTEGER

    Plage de valeurs : -2147483648 à 2147483647 pour la version signée, et 0 à 4294967295 pour la version non signée.
    Utilisation courante : c'est le type entier le plus couramment utilisé dans MySQL. Il est utilisé pour une grande variété de besoins, tels que les identifiants d'articles, les compteurs de visiteurs, etc.

BIGINT

    Plage de valeurs : -9223372036854775808 à 9223372036854775807 pour la version signée, et 0 à 18446744073709551615 pour la version non signée.
    Utilisation courante : pour des entiers très grands, comme des identifiants uniques à l'échelle mondiale, des compteurs extrêmement élevés, ou lorsqu'il est nécessaire de stocker des valeurs au-delà de la capacité d'un INT.

Tableau récapitulatif
Type 	Stockage (octets) 	Valeur Min Signée 	Valeur Min Non Signée 	Valeur Max Signée 	Valeur Max Non Signée
TINYINT 	1 	-128 	0 	127 	255
SMALLINT 	2 	-32768 	0 	32767 	65535
MEDIUMINT 	3 	-8388608 	0 	8388607 	16777215
INT 	4 	-2147483648 	0 	2147483647 	4294967295
BIGINT 	8 	-2^63 	0 	2^63 - 1 	2^64 - 1

Notez que les valeurs pour BIGINT sont exprimées en puissances de 2, car elles dépassent les capacités d'affichage.
SERIAL

C'est un type spécial qui est un alias pour BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE, souvent utilisé pour créer des clés primaires simples.

Nous en reparlerons lorsque nous verrons les clés primaires en détail.

 
Attributs des types entiers

    UNSIGNED : utilisé pour indiquer qu'un type entier ne contiendra que des valeurs positives ou nulles.
    AUTO_INCREMENT : utilisé pour créer un identifiant unique pour de nouvelles lignes. Souvent utilisé avec PRIMARY KEY. Déclenche la génération de la prochaine valeur de séquence, qui est généralement la valeur la plus élevée actuellement dans la table plus un.

 
Qu'est-ce que l'overflow et comment le gérer ?

L'overflow, ou dépassement de capacité, est un problème qui survient lorsqu'une opération arithmétique sur des nombres entiers atteint une valeur qui dépasse la plage de valeurs maximale que le type de données peut stocker.

Les types de données entiers dans les bases de données ont une plage de valeurs limitée déterminée par le nombre d'octets alloués pour le stockage du type de données.

Si vous essayez d'insérer une valeur en overflow/underflow vous aurez : Error Code: 1264. Out of range value for column 'colonne' at row n.

Voici les problèmes potentiels :

Dépassement de capacité positif (Overflow) : Se produit lorsque vous essayez d'ajouter deux grands nombres positifs ou d'augmenter la valeur d'un nombre entier au-delà de sa valeur maximale autorisée.

Exemple : pour TINYINT, la valeur maximale est 127. Si vous essayez de stocker 128, il y aura un overflow.

Dépassement de capacité négatif (Underflow) : se produit lorsque vous essayez de soustraire un grand nombre d'un nombre plus petit, menant la valeur résultante en dessous de la valeur minimale autorisée.

Exemple : pour TINYINT, la valeur minimale est -128. Si vous essayez de stocker -129, il y aura un underflow.

Multiplication et division : l'overflow peut également se produire avec la multiplication de deux entiers qui donne un résultat au-delà de la plage autorisée, ou lors de la division par zéro ou un nombre proche de zéro, ce qui peut produire un très grand nombre.

 
Bonnes pratiques et note

Lorsque vous utilisez des types entiers, il est important de choisir un type qui peut gérer toutes les valeurs possibles que vous attendez sans gaspiller d'espace disque. Cela augmentera les performances des requêtes et cela optimisera l'utilisation de l'espace disque de votre base de données.

Note : depuis MySQL 8.0.17, l'attribut de largeur d'affichage (M) pour les types entiers est obsolète. De même, l'attribut ZEROFILL est obsolète. Nous ne les verrons donc pas dans la formation.

Types numériques : DECIMAL, NUMERIC, FLOAT et DOUBLE
Les types numériques non entiers

Les types numériques non entiers comprennent les types à virgule flottante et à point fixe. 

Ils sont utilisés pour stocker des nombres qui peuvent avoir des fractions, ce qui les rend essentiels pour représenter des valeurs telles que les mesures, les devises, et d'autres quantités qui nécessitent une précision au-delà des nombres entiers.

 
Les types à point fixe

Les types à point fixe (Fixed-Point Types), DECIMAL et NUMERIC, sont conçus pour stocker des valeurs numériques exactes.

Ils sont particulièrement utiles lorsque la précision exacte est cruciale, comme c'est souvent le cas pour les données monétaires.

Dans MySQL, le type NUMERIC est traité comme un alias de DECIMAL.

MySQL stocke les valeurs DECIMAL dans un format binaire optimisé pour la précision mathématique. Cela diffère du stockage des types à virgule flottante (FLOAT et DOUBLE), qui peuvent présenter des erreurs d'arrondi en raison de la représentation approximative des nombres à virgule flottante.

Les valeurs sont stockées en tant que chaînes, ce qui permet de préserver la précision exacte.
Précision et échelle (precision et scale)

Lors de la déclaration d'une colonne DECIMAL, vous pouvez (et devez généralement) spécifier la précision et l'échelle.

La précision représente le nombre total de chiffres significatifs que la colonne peut stocker, tandis que l'échelle indique le nombre de ces chiffres qui peuvent apparaître après la virgule décimale.

Par exemple :

salary DECIMAL(5,2)

Dans cet exemple, la précision est de 5 et l'échelle est de 2. Cela signifie que la colonne salary peut stocker des valeurs allant de -999.99 à 999.99, inclusivement.
Limites et conversion

Le nombre maximal de chiffres pour DECIMAL est de 65, avec une échelle (D) pouvant aller jusqu'à 30.

Si une valeur assignée à une colonne DECIMAL a plus de chiffres après la virgule décimale que ce que l'échelle spécifiée permet, cette valeur est convertie en respectant l'échelle définie.

Cette conversion peut entraîner une troncature de la valeur, comportement qui peut varier légèrement selon le système d'exploitation, mais qui aboutit généralement à la suppression des chiffres excédentaires.
Utilisation de DECIMAL

Les colonnes DECIMAL sont idéales pour stocker des valeurs nécessitant une précision absolue, telles que les montants financiers.

Par exemple, lorsqu'il s'agit de stocker des salaires, des prix, ou des coûts, où arrondir au chiffre le plus proche pourrait entraîner des erreurs significatives en cumulé.

 
Les types à virgule flottante

Les types à virgule flottante (Floating-Point Types), FLOAT et DOUBLE sont utilisés pour représenter des valeurs numériques approximatives.

Ces types ne stockent pas les valeurs numériques de manière exacte, ce qui les rend appropriés pour les cas où une grande plage de valeurs est plus importante que la précision absolue de chaque valeur individuelle.

Étant donné que les valeurs à virgule flottante sont approximatives et non stockées comme des valeurs exactes, les traiter comme exactes dans les comparaisons peut conduire à des problèmes. Elles sont également sujettes à des dépendances liées à la plate-forme ou à l'implémentation.
Stockage et précision

FLOAT : utilise 4 octets pour les valeurs à simple précision. Il peut stocker des valeurs approximatives avec une précision d'environ 7 chiffres décimaux.

DOUBLE : utilise 8 octets pour les valeurs à double précision. Il peut stocker des valeurs approximatives avec une précision d'environ 15 chiffres décimaux.
Spécification de la précision

La norme SQL permet de spécifier facultativement la précision pour FLOAT en bits (par exemple, FLOAT(p)), mais MySQL utilise cette précision uniquement pour déterminer la taille de stockage.

Une précision de 0 à 23 résulte en une colonne FLOAT à simple précision de 4 octets.

Une précision de 24 à 53 entraîne une colonne DOUBLE à double précision de 8 octets.
Bonnes pratiques et notes

FLOAT et DOUBLE sont mieux utilisés pour les données scientifiques ou les cas où les calculs impliquent des nombres qui peuvent dépasser la précision d'un DECIMAL, ou quand la performance de calcul est plus critique que la précision exacte des nombres.

Lors de la comparaison de valeurs à virgule flottante, il est préférable d'utiliser une marge d'erreur ou "epsilon" pour tenir compte de l'imprécision. Par exemple, au lieu de vérifier l'égalité directe, vérifiez si la différence entre les deux nombres est inférieure à un petit seuil.

Soyez conscient de l'arrondissement lors de l'insertion ou de la mise à jour des valeurs dans les colonnes FLOAT ou DOUBLE. 

Pour les données nécessitant une précision exacte, comme l'argent, utilisez plutôt DECIMAL. Réservez l'utilisation de FLOAT et DOUBLE pour les cas où la représentation approximative est acceptable et souhaitable.

Note : à partir de MySQL 8.0.17, la syntaxe non standard FLOAT(M,D) et DOUBLE(M,D) est obsolète, et il est attendu que le support pour celle-ci soit retiré dans une future version de MySQL. Nous ne verrons donc pas cette syntaxe dans la formation.

Chaînes : CHAR, VARCHAR, BINARY, VARBINARY
Les types de données chaîne

Les types de données chaîne (String Data Types) sont utilisés pour stocker des textes de différentes longueurs et formats. Ils incluent CHAR, VARCHAR, BINARY, VARBINARY, BLOB, TEXT, ENUM, et SET. 

Nous allons présenter brièvement chaque type puis entrer plus en détail.
CHAR et VARCHAR

CHAR(M) : chaîne de caractères de longueur fixe. M spécifie la longueur de la chaîne en caractères, avec un maximum de 255. CHAR est toujours complété par des espaces à droite jusqu'à la longueur spécifiée lorsqu'il est stocké. Les espaces de fin sont supprimés lors de la récupération des valeurs CHAR à moins que le mode SQL PAD_CHAR_TO_FULL_LENGTH ne soit activé.

VARCHAR(M) : chaîne de caractères de longueur variable. M représente la longueur maximale de la chaîne en caractères, avec un maximum de 65 535. La longueur effective maximale de VARCHAR dépend de la taille maximale de la ligne et du jeu de caractères utilisé.
BINARY et VARBINARY

BINARY(M) : chaîne d'octets de longueur fixe. M spécifie la longueur de la chaîne en octets (bytes), avec un maximum de 255.

VARBINARY(M) : chaîne d'octets de longueur variable. M représente la longueur maximale de la chaîne en octets.
BLOB et TEXT

BLOB et TEXT : utilisés pour stocker de grandes quantités de données binaires (BLOB) ou textuelles (TEXT).

Il existe quatre tailles : TINY, MEDIUM, LONG, et la taille par défaut (sans préfixe), qui déterminent la taille maximale de stockage.
ENUM et SET

ENUM('value1', 'value2', ...) : permet de stocker une valeur choisie dans une liste spécifiée de valeurs possibles. ENUM est représenté en interne comme un entier.

SET('value1', 'value2', ...) : permet de stocker zéro ou plusieurs valeurs choisies dans une liste spécifiée. SET est également représenté en interne comme un entier.

 
CHAR vs VARCHAR

CHAR est un type de longueur fixe qui alloue un espace de stockage constant pour chaque entrée, quelle que soit la longueur réelle de la chaîne stockée. Si la chaîne stockée est plus courte que la longueur définie pour la colonne CHAR, MySQL complète la chaîne avec des espaces à droite jusqu'à atteindre cette longueur fixe.

VARCHAR est un type de longueur variable qui stocke les chaînes de caractères de manière plus efficace en n'allouant que l'espace nécessaire pour chaque entrée plus un ou deux octets supplémentaires pour enregistrer la longueur de la chaîne. Si la chaîne est plus courte que la longueur maximale définie, seul l'espace nécessaire est utilisé.

CHAR est idéal pour stocker des données de longueur relativement constante, comme les codes postaux, les codes de pays, et autres identifiants de taille fixe.

Dans tous les autres cas on utilisera VARCHAR.
Exemple d'utilisation du stockage
Valeur 	CHAR(4) Stockage 	VARCHAR(4) Stockage
'' (vide) 	' ' (4 octets) 	'' (1 octet)
'ab' 	'ab ' (4 octets) 	'ab' (3 octets)
'abcd' 	'abcd' (4 octets) 	'abcd' (5 octets)
'abcdefgh' 	'abcd' (4 octets) 	'abcd' (5 octets)

'' (vide) : Pour CHAR(4), MySQL stocke la valeur vide comme quatre espaces, utilisant 4 octets. Pour VARCHAR(4), seulement un octet est utilisé pour indiquer la longueur de la chaîne, qui est 0 dans ce cas.

'ab' : Avec CHAR(4), les valeurs sont complétées avec des espaces à droite pour atteindre la longueur fixe de 4, d'où l'utilisation de 4 octets. VARCHAR(4) stocke la valeur 'ab' plus un octet de préfixe de longueur, pour un total de 3 octets.

'abcd' : Comme 'abcd' remplit exactement la longueur spécifiée pour CHAR(4) et VARCHAR(4), les deux utilisent la longueur spécifiée pour le stockage. Cependant, VARCHAR(4) ajoute un octet supplémentaire pour le préfixe de longueur, d'où les 5 octets de stockage.

'abcdefgh' : Lorsque la valeur dépasse la longueur maximale de la colonne, CHAR(4) tronque la valeur à 'abcd', utilisant toujours 4 octets. VARCHAR(4) fait de même mais inclut le préfixe de longueur, donc utilise 5 octets. Il est important de noter que ce comportement de troncature pour VARCHAR(4) s'applique uniquement en mode non strict SQL ; en mode strict, une erreur serait générée plutôt que de permettre la troncature. Le mode strict est activé par défaut.
Quelle taille utiliser pour VARCHAR ?

Si les données que vous prévoyez de stocker varient considérablement en longueur, VARCHAR est un bon choix car il utilise uniquement l'espace nécessaire pour stocker les données plus un ou deux octets supplémentaires pour la longueur de la chaîne.

Déterminez la taille maximale des données que vous prévoyez de stocker dans la colonne.

Par exemple, si vous stockez des adresses email, une valeur comme VARCHAR(255) est courante car elle est suffisamment longue pour la plupart des adresses email tout en restant sous la limite de 256 caractères souvent utilisée dans les spécifications et les normes.

 
BINARY vs VARBINARY
BINARY

BINARY est similaire au type CHAR mais pour les données binaires. Il stocke des chaînes d'octets de longueur fixe.

Lorsque des valeurs BINARY sont stockées, elles sont complétées à droite avec des zéros (0x00) jusqu'à atteindre la longueur spécifiée. Ce remplissage assure que toutes les valeurs BINARY ont la même longueur fixe dans la base de données.
VARBINARY

VARBINARY est similaire au type VARCHAR mais pour les données binaires. Il stocke des chaînes d'octets de longueur variable.

La longueur maximale autorisée est la même pour BINARY et VARBINARY que pour CHAR et VARCHAR, mais elle est mesurée en octets plutôt qu'en caractères.
Exemples d'utilisation

BINARY est souvent utilisé pour stocker des hashs cryptographiques, tels que ceux générés par SHA-256 ou MD5. Ces hashs ont une longueur fixe et représentent une empreinte digitale des données, souvent utilisée pour vérifier l'intégrité des données ou stocker des mots de passe de manière sécurisée.

Par exemple : pour stocker un hash SHA-256, qui a une longueur de 32 bytes, vous pouvez utiliser une colonne BINARY(32). Cela garantit que le hash est stocké et récupéré byte par byte sans modification.

BINARY peut également être utilisé pour stocker des identifiants uniques générés par des systèmes ou des applications, comme les UUID (Universally Unique Identifier). L'utilisation de BINARY(16) pour les UUID est plus efficace en termes d'espace que de les stocker sous forme de chaînes de caractères.

En résumé, BINARY est idéal pour stocker des clés de cryptographie ou d'autres données sensibles qui doivent être préservées dans un format exact, car il garantit que les données sont stockées et récupérées sans modification.

VARBINARY est adapté pour stocker des données chiffrées, dont la longueur peut varier en fonction du mode de chiffrement et des données d'entrée. En utilisant VARBINARY, vous pouvez stocker des données chiffrées de manière sécurisée tout en optimisant l'espace de stockage.

Exemple : pour stocker des données utilisateur chiffrées où la taille des données chiffrées peut varier.

Chaînes : TEXT et BLOB
TEXT et BLOB
Types BLOB

Les types BLOB sont utilisés pour stocker des données binaires, comme des images, des fichiers audio, ou des documents binaires. Ils sont traités comme des chaînes d'octets plutôt que des chaînes de caractères.

Il existe quatre types de BLOB qui varient en fonction de la taille maximale des données qu'ils peuvent contenir : TINYBLOB, BLOB, MEDIUMBLOB, et LONGBLOB. La principale différence entre ces types est leur capacité de stockage maximale.

Les valeurs BLOB sont comparées et triées en fonction des valeurs numériques des octets. Ils utilisent l'ensemble de caractères et la collation binaires.

Les colonnes BLOB ne peuvent pas avoir de valeurs par défaut et, pour indexer une colonne BLOB, vous devez spécifier une longueur de préfixe d'index.
Types TEXT

Les types TEXT sont utilisés pour stocker des chaînes de caractères de grande taille, comme des articles de blog, des commentaires ou d'autres formes de texte long. Ils sont traités comme des chaînes de caractères non binaires.

Comme pour les BLOB, il existe quatre types de TEXT : TINYTEXT, TEXT, MEDIUMTEXT, et LONGTEXT, qui correspondent aux quatre types BLOB en termes de capacité de stockage.

    TINYTEXT : Peut stocker jusqu'à 255 caractères.
    TEXT : Peut stocker jusqu'à 65 535 caractères (environ 64KB).
    MEDIUMTEXT : Peut stocker jusqu'à 16 777 215 caractères (environ 16MB).
    LONGTEXT : Peut stocker jusqu'à 4 294 967 295 caractères (environ 4GB).

Il est important de noter que ces limites sont basées sur le nombre de caractères. La taille réelle en octets peut varier, surtout si vous utilisez un jeu de caractères multibyte comme utf8mb4, où chaque caractère peut occuper jusqu'à 4 octets. La longueur effective des données que vous pouvez stocker dépendra donc également du jeu de caractères utilisé pour la colonne.

Les colonnes TEXT ne peuvent pas avoir de valeurs par défaut et, pour indexer une colonne TEXT, une longueur de préfixe d'index doit être spécifiée.

 
Exemples d'utilisation de TEXT

    Articles de blog ou nouvelles : un champ TEXT ou MEDIUMTEXT est idéal pour stocker le corps d'articles de blog, de nouvelles ou de publications sur des forums, où le contenu peut être relativement long et varier en taille.

    Commentaires utilisateurs : pour les applications qui incluent des fonctionnalités de commentaires, un champ TEXT peut être utilisé pour stocker les commentaires des utilisateurs sur des articles, des produits, ou des services.

    Scripts ou code source : les développeurs peuvent utiliser des champs TEXT pour stocker des scripts, des configurations, ou du code source dans des applications de gestion de configuration ou de déploiement automatisé.

 
Exemples d'Utilisation de BLOB

    Images de profil utilisateur : bien que stocker des images directement dans une base de données ne soit pas toujours recommandé, un champ BLOB peut être utilisé pour cela, en particulier si les images sont de petite taille et que la facilité d'accès est une priorité.

    Documents PDF ou Word : pour une application qui nécessite l'accès rapide à des documents utilisateurs (comme des CVs ou des rapports), les stocker dans un champ BLOB peut être pratique pour une récupération facile directement depuis la base de données.

 
Bonnes pratiques

Utilisez TEXT et BLOB avec modération : privilégiez l'utilisation de ces types pour des données qui doivent être étroitement couplées avec d'autres données dans la base de données et qui ne sont pas excessivement volumineuses.

Optez pour des solutions de stockage externes : pour les fichiers médias, les documents volumineux, ou les données qui doivent être facilement accessibles depuis différentes applications, utilisez des solutions spécialisées comme Amazon S3, Google Cloud Storage, ou des systèmes de block storage. Stockez uniquement des références ou des URL de ces fichiers dans votre base de données MySQL.

Évaluez les besoins de performance et d'accessibilité : si l'accès rapide aux données depuis la base de données est une priorité et que la taille des données est gérable, TEXT et BLOB peuvent être appropriés. Pour des accès moins fréquents ou pour des données volumineuses, les solutions de stockage externe sont préférables.

Chaînes : ENUM et SET
ENUM

Le type ENUM est un objet chaîne qui permet de choisir une valeur parmi une liste de valeurs permises explicitement énumérées dans la spécification de la colonne lors de la création de la table.

ENUM est particulièrement efficace pour les colonnes ayant un ensemble limité de valeurs possibles. Les chaînes spécifiées comme valeurs d'entrée sont automatiquement encodées sous forme de nombres, ce qui rend le stockage de données ENUM plus compact que les chaînes de caractères équivalentes stockées dans des colonnes VARCHAR ou CHAR.

Les requêtes et les résultats sont plus lisibles car MySQL convertit automatiquement les valeurs numériques en chaînes correspondantes lors des requêtes, rendant les données plus faciles à comprendre sans nécessiter de jointure supplémentaire ou de logique d'application pour décoder les valeurs.

Exemple :

size ENUM('x-small', 'small', 'medium', 'large', 'x-large')

Chaque valeur d'énumération a un index, débutant par 1 pour la première valeur énumérée. La valeur d'erreur de chaîne vide a un index de 0.

Il est déconseillé d'utiliser des nombres comme valeurs ENUM car cela peut entraîner une confusion entre les index numériques internes et les valeurs littérales chaînes.
Exemples d'utilisation

Statut de commande : pour une application e-commerce, où chaque commande peut avoir des statuts tels que 'nouveau', 'en traitement', 'expédié', 'livré', 'retourné'.

statut ENUM('nouveau', 'en traitement', 'expédié', 'livré', 'retourné')

Niveaux d'accès utilisateur : dans un système de gestion des utilisateurs, pour définir différents niveaux d'accès comme 'admin', 'éditeur', 'utilisateur'.

niveau_acces ENUM('admin', 'éditeur', 'utilisateur')

Bonnes pratiques

Bien que ENUM ait ses avantages, il existe des situations où d'autres types de données sont préférables :

    Grande liste de valeurs qui peut évoluer : si l'ensemble de valeurs peut grandir ou changer fréquemment, utiliser ENUM peut devenir gênant car chaque modification nécessite une commande ALTER TABLE, ce qui peut être coûteux en termes de performance. Dans ces cas, utiliser une table séparée et une clé étrangère peut être plus flexible.

    Internationalisation : les valeurs ENUM sont stockées telle quelle dans la base de données, ce qui peut poser problème si vous devez supporter plusieurs langues. Une table de référence avec des identifiants permet une meilleure internationalisation.

    Analyse des données : pour l'analyse des données, avoir des valeurs ENUM dans des tables séparées avec des clés étrangères peut simplifier les requêtes d'agrégation et de jointure, en plus de rendre les données plus normalisées.

    Recherche de performance : l'utilisation d'indices sur des colonnes avec des types numériques (INT, TINYINT, etc.) peut être plus performante que l'utilisation d'ENUM pour des grandes bases de données avec des requêtes complexes.

 
SET

Le type SET est un objet de chaîne qui peut contenir zéro ou plusieurs valeurs, chacune devant être choisie parmi une liste de valeurs permises spécifiées lors de la création de la table.

Une colonne de type SET peut stocker une combinaison de zéro ou plus de valeurs spécifiées lors de la définition de la table. Les valeurs multiples sont séparées par des virgules dans une seule entrée de colonne.

Les valeurs sont également stockées comme des entiers, mais chaque bit de l'entier représente une valeur différente de l'ensemble, permettant de stocker une combinaison de valeurs. Une colonne SET peut stocker jusqu'à 64 valeurs différentes, ce qui nécessite jusqu'à 8 octets de stockage.

Il est conseillé de ne pas utiliser des nombres comme valeurs SET pour éviter la confusion entre les valeurs littérales et leurs indices numériques internes.
Exemples d'utilisation

Droits d'accès utilisateurs : dans un système de gestion de contenu, pour définir les droits d'accès d'un utilisateur comme lire, écrire, modifier, supprimer :

droits_acces SET('lire', 'écrire', 'modifier', 'supprimer')

Jours de travail : pour planifier les jours de travail ou les disponibilités d'un employé ou d'une ressource dans une application de planification.

jours_travail SET('lundi', 'mardi', 'mercredi', 'jeudi', 'vendredi')

Bonnes pratiques

Il est conseillé de ne pas utiliser SET dans les cas suivants :

    Grande flexibilité nécessaire : si l'ensemble des valeurs possibles peut changer fréquemment ou si vous avez besoin de plus de 64 valeurs distinctes, considérez l'utilisation d'une table séparée avec des relations many-to-many. Cela facilite les mises à jour et offre une flexibilité accrue.

    Normalisation et intégrité des données : pour maintenir la normalisation de la base de données et garantir l'intégrité des données, il est souvent préférable d'utiliser des tables de relation plutôt que des colonnes SET. Cela simplifie également les requêtes complexes et les jointures.

    Analyse de données complexes : si vous prévoyez de faire des analyses complexes sur les données, les colonnes SET peuvent compliquer les requêtes. Des tables relationnelles séparées peuvent rendre l'analyse de données plus simple et plus directe.

    Internationalisation : si votre application nécessite l'internationalisation et que les valeurs stockées dans SET doivent être traduites, il est préférable d'utiliser des identifiants numériques liés à des tables de traduction.

Dates et temps
Les dates et les temps

Les types de données de date et d'heure permettent de représenter des valeurs temporelles à l'aide de différents formats.

Voyons un premier aperçu de chaque type, puis chaque type plus en détail.
DATE

DATE représente une date sans l'heure, avec une plage prise en charge de '1000-01-01' au '9999-12-31'.

Les valeurs DATE sont affichées au format 'YYYY-MM-DD'.
TIME[(fsp)]

TIME représente une durée ou un moment de la journée, avec une plage de '-838:59:59.000000' à '838:59:59.000000'. Il permet de stocker des secondes fractionnaires jusqu'à une précision de microsecondes (6 chiffres).

La précision des secondes fractionnaires (fsp) peut être spécifiée de 0 à 6. La valeur par défaut est 0 si elle est omise.
DATETIME[(fsp)]

DATETIME combine date et heure, couvrant une plage du '1000-01-01 00:00:00.000000' au '9999-12-31 23:59:59.999999'. Comme TIME, il prend en charge les secondes fractionnaires.

Similaire à TIME, la précision des secondes fractionnaires peut être définie.
TIMESTAMP[(fsp)]

TIMESTAMP représente un instant dans le temps, stocké comme le nombre de secondes écoulées depuis l'époque Unix ('1970-01-01 00:00:00' UTC), avec une plage du '1970-01-01 00:00:01.000000' UTC au '2038-01-19 03:14:07.999999' UTC.

Il permet également la spécification de secondes fractionnaires.
YEAR

YEAR stocke une année en format 4 chiffres. Les valeurs vont de 1901 à 2155, ou 0000.

 
DATE en détail
Syntaxe et format

Les valeurs de type DATE sont stockées et affichées au format 'YYYY-MM-DD', où 'YYYY' représente l'année, 'MM' le mois et 'DD' le jour.

La plage de dates que MySQL permet de stocker dans un champ de type DATE s'étend du '1000-01-01' au '9999-12-31'. Cette large plage garantit que le type DATE est adapté pour stocker des dates historiques ainsi que des dates dans un avenir lointain.

MySQL permet d'affecter des valeurs aux colonnes de type DATE en utilisant des chaînes de caractères, pourvu qu'elles respectent le format 'YYYY-MM-DD'. Par exemple, '2025-03-15' représente le 15 mars 2025.
Bonnes pratiques

Puisque le type DATE ne tient pas compte des fuseaux horaires, il convient pour stocker des dates qui sont indépendantes des heures et des zones géographiques. Pour les événements nécessitant des informations temporelles précises, envisagez d'utiliser DATETIME ou TIMESTAMP.

Autrement dit, l'utilisation du type DATE est recommandée dans tous les cas où l'information temporelle nécessaire se limite à la date. Cela simplifie la manipulation et l'interrogation des données, tout en optimisant l'espace de stockage. Pour les situations nécessitant la prise en compte de l'heure, des minutes et des secondes, les types DATETIME ou TIMESTAMP sont plus appropriés.

 
TIME en détail
Syntaxe et format

Le format TIME est utilisé pour représenter le temps, indépendamment d'une date spécifique.

Ce type de données est idéal pour stocker des durées, des horaires ou l'heure d'une journée, avec ou sans précision des secondes fractionnaires.

Le format de base de TIME est 'HH:MM:SS', mais il peut également stocker des heures au-delà de 24, permettant ainsi de représenter des durées plus longues que la durée d'une journée.

La plage des valeurs TIME va de '-838:59:59.000000' à '838:59:59.000000'. Cette large plage permet de représenter à la fois des heures négatives et positives, utiles pour indiquer des différences ou des durées de temps.

MySQL permet de spécifier une précision des secondes fractionnaires pour le type TIME allant de 0 à 6. Par exemple, TIME(3) permet de stocker le temps avec une précision jusqu'à la milliseconde. Si la précision n'est pas spécifiée, la valeur par défaut est 0, signifiant qu'il n'y a pas de partie fractionnaire.
Bonnes pratiques

Spécifier la précision des secondes fractionnaires uniquement si nécessaire, car cela peut augmenter la taille de stockage et affecter les performances pour les grandes bases de données.

 
DATETIME en détail
Syntaxe et format

Le type de données DATETIME est conçu pour stocker des valeurs de date et d'heure, combinant ainsi les aspects de la date et de l'heure dans un seul champ.

Ce type est particulièrement utile pour enregistrer des moments précis, comme des horodatages d'événements, des moments de création ou de modification de données, et d'autres instances nécessitant à la fois la date et l'heure. 

Le format standard de DATETIME est 'YYYY-MM-DD HH:MM:SS'. Cela permet de représenter la date et l'heure jusqu'à la seconde. MySQL supporte également les secondes fractionnaires, permettant une précision supplémentaire.

La plage de valeurs pour DATETIME va du '1000-01-01 00:00:00' au '9999-12-31 23:59:59'. Avec les secondes fractionnaires, la précision peut aller jusqu'à '9999-12-31 23:59:59.999999', selon la précision spécifiée.

DATETIME peut être configuré pour s'initialiser automatiquement à la date et à l'heure actuelles lors de la création d'un enregistrement et pour se mettre à jour lors de modifications ultérieures. Nous verrons cela lors du chapitre suivant.
Bonnes pratiques

Spécifier la précision des secondes fractionnaires uniquement si nécessaire, car cela peut augmenter la taille de stockage et affecter les performances pour les grandes bases de données.

Soyez conscient de la gestion des fuseaux horaires lors de l'utilisation de DATETIME, surtout si votre application fonctionne à l'échelle mondiale. DATETIME ne stocke pas d'informations sur le fuseau horaire, donc si vous gérez des utilisateurs dans plusieurs fuseaux horaires, envisagez des stratégies pour gérer correctement les conversions.

 
TIMESTAMP en détail
Syntaxe et format

Le type de données TIMESTAMP est spécialement conçu pour enregistrer des instants précis, en associant une date à une heure. Ce type est largement utilisé pour les systèmes qui nécessitent de suivre le moment exact d'un événement, tel que la création ou la mise à jour de données. 

Les valeurs TIMESTAMP sont stockées sous la forme 'YYYY-MM-DD HH:MM:SS', et peuvent optionnellement inclure des fractions de seconde jusqu'à une précision de microsecondes (6 chiffres), par exemple, 'YYYY-MM-DD HH:MM:SS.mmmmmm'.

La plage des valeurs TIMESTAMP s'étend du '1970-01-01 00:00:01' UTC au '2038-01-19 03:14:07' UTC. Cette limitation est due à la représentation interne de TIMESTAMP comme un nombre de secondes écoulées depuis l'époque Unix ('1970-01-01 00:00:00' UTC).

MySQL convertit automatiquement les valeurs TIMESTAMP du fuseau horaire courant en UTC pour le stockage, et inversement d'UTC au fuseau horaire courant lors de la récupération. Cela garantit que les valeurs TIMESTAMP sont toujours enregistrées de manière cohérente, indépendamment du fuseau horaire du serveur ou de celui des clients.

TIMESTAMP peut être configuré pour s'initialiser automatiquement à la date et à l'heure actuelles lors de la création d'un enregistrement et pour se mettre à jour lors de modifications ultérieures. Nous verrons cela lors du chapitre suivant.
Bonnes Pratiques

Il faut être conscient de l'impact des conversions de fuseau horaire sur les valeurs TIMESTAMP et s'assurer que les applications client traitent correctement ces valeurs.

Il faut utiliser la précision des secondes fractionnaires uniquement lorsque la précision temporelle extrême est nécessaire, car cela peut légèrement augmenter la taille de stockage et affecter les performances.

Être conscient de la limite supérieure de l'an 2038 pour les valeurs TIMESTAMP et planifier en conséquence, notamment en envisageant l'utilisation de DATETIME pour les dates postérieures à cette limite.

 
YEAR en détail
Syntaxe et format

Le type de données YEAR dans est utilisé pour stocker une année sur quatre chiffres, offrant une manière compacte et efficace de gérer les informations annuelles dans une base de données.

Le type YEAR permet de stocker des années dans le format YYYY.

Le type YEAR utilise seulement 1 octet de stockage par valeur, ce qui en fait un choix économique pour stocker des informations d'année sans nécessiter beaucoup d'espace disque.
Bonnes pratiques

Éviter la valeur 0000 : bien que 0000 soit une valeur spéciale valide pour YEAR, son utilisation peut être source de confusion. Préférez utiliser un champ NULL pour représenter une année inconnue ou non applicable, si votre schéma de base de données le permet.

Tenez compte de la plage de dates limitée du type YEAR lors de la conception de votre base de données. Pour les données historiques ou futures s'étendant au-delà de 1901 à 2155, envisagez d'utiliser un type de données différent, comme DATE ou INT.

 
Résumé des utilisations des différents formats

    DATE : utilisé pour les dates sans information temporelle, idéal pour les anniversaires, les dates d'expiration, ou les jours fériés. Recommandé quand l'heure de l'événement n'est pas nécessaire.

    TIME : représente des heures du jour ou des durées, parfait pour les horaires de travail, les durées d'activités. Utilisé dans les systèmes de suivi du temps ou quand la date n'est pas pertinente.

    DATETIME : combine date et heure pour les timestamps précis d'événements, comme les créations ou modifications de données. Convient aux planifications d'événements futurs avec date et heure spécifiques.

    TIMESTAMP : horodatage automatique pour suivre la création et modification des enregistrements, avec conversion des heures selon les fuseaux horaires. Idéal pour la synchronisation des données et les environnements nécessitant la précision temporelle avec gestion des fuseaux horaires.

    YEAR : stocke uniquement l'année, utilisé pour des informations comme l'année de fabrication ou de publication. Recommandé pour les analyses annuelles ou quand seule l'année est pertinente.

JSON
Le format JSON

Le format JSON (JavaScript Object Notation) est un format de données textuel léger pour l'échange de données.

Il est facile à lire pour les humains et simple à analyser et à générer pour les machines. JSON est basé sur la notation des objets dans JavaScript, mais il est indépendant du langage et peut être utilisé avec de nombreux langages de programmation. 
Syntaxe

Un objet JSON est entouré par des accolades {} et représente un ensemble de paires clé-valeur. Les clés sont des chaînes de caractères et les valeurs peuvent être de différents types.

Une valeur JSON peut être une chaîne de caractères (entourée de guillemets doubles), un nombre (entier ou flottant), un objet JSON, un tableau JSON, true, false, ou null.

Par exemple : {"nom": "Dupont", "prénom": "Jean"}.

Un tableau JSON est entouré par des crochets [] et contient une liste ordonnée de valeurs (qui peuvent être de n'importe quel type JSON). Par exemple : ["pomme", "banane", "cerise"].

Les objets et les tableaux peuvent être imbriqués, permettant de représenter des données structurées complexes.

Voici les règles syntaxiques :

    Chaînes de caractères : doivent être entourées de guillemets doubles. Les caractères spéciaux doivent être échappés avec un backslash \. Par exemple : "Bonjour, \"monde\"!".
    Nombres : peuvent être entiers ou flottants, sans guillemets. Par exemple : 42 ou 3.14.
    Booléens : représentent true ou false.
    Null : Représente une absence de valeur.

Exemple

Voici un exemple de document JSON représentant des informations sur une personne, incluant un objet et un tableau :

{
  "nom": "Doe",
  "prénom": "John",
  "age": 30,
  "estEmployé": true,
  "adresse": {
    "rue": "123 Rue Principale",
    "ville": "Ville",
    "codePostal": "12345"
  },
  "numérosDeTéléphone": ["0425478774", "0146456796"],
  "languesParlées": ["français", "anglais", "espagnol"],
  "conjoints": null
}

JSON est largement utilisé pour échanger des données entre un serveur et un client web, ainsi que pour stocker des configurations, des options ou des états d'application dans divers logiciels. Sa simplicité et sa facilité de manipulation en font un choix populaire dans de nombreux contextes de développement.

 
Insertion et normalisation

MySQL supporte nativement le type de données JSON, permettant de stocker des documents JSON directement dans les tables.

Cela offre plusieurs avantages, tels que la validation automatique des données JSON et un format de stockage optimisé pour un accès rapide.
Insertion de données JSON

Une fois que vous avez une table avec une colonne JSON, vous pouvez y insérer des données JSON directement.

INSERT INTO utilisateurs (données) VALUES
('{"nom": "Dupont", "prénom": "Jean", "âge": 30}');

Vous pouvez également utiliser des fonctions MySQL pour générer des valeurs JSON :

    JSON_ARRAY([val[, val] ...]) pour créer des tableaux.
    JSON_OBJECT([key, val[, key, val] ...]) pour créer des objets.
    JSON_MERGE_PRESERVE(json_doc, json_doc[, json_doc] ...) pour fusionner des documents JSON.

Par exemple :

JSON_ARRAY('a', 1, NOW());
JSON_OBJECT('clé1', 1, 'clé2', 'abc');

Normalisation des valeurs JSON

La normalisation est un processus appliqué aux documents JSON lors de leur analyse par MySQL.

Ce processus élimine les doublons de clés dans les objets JSON, en conservant uniquement la dernière occurrence de chaque clé. Cette règle, connue sous le nom de "la dernière clé l'emporte".

Par exemple :

SELECT JSON_OBJECT('clé1', 1, 'clé2', 'abc', 'clé1', 'def');
-- Résultat : {"clé1": "def", "clé2": "abc"}

Ce comportement assure que chaque clé dans un objet JSON est unique, simplifiant ainsi l'accès et la manipulation des données.

Ce comportement est différent avec la fonction JSON_MERGE_PRESERVE() qui concatène les valeurs pour les clés en double dans un tableau. Si plusieurs objets contiennent la même clé, les valeurs sont préservées dans un tableau associé à cette clé.

SELECT JSON_MERGE_PRESERVE('{"a": 1}', '{"a": 2}');
-- Résultat : {"a": [1, 2]}

 
Extraire ou modifier une valeur JSON

La recherche et la modification de valeurs JSON utilisent des expressions de chemin JSON pour identifier précisément les éléments à manipuler dans un document JSON.
Expressions de chemin JSON

Les expressions de chemin JSON (JSON path expression) servent à localiser des valeurs spécifiques au sein d'un document JSON.

La syntaxe de base utilise le caractère $ pour représenter le document entier, suivi par une série de sélecteurs qui naviguent vers des parties spécifiques du document.

Les sélecteurs sont les suivants :

    Nom de clé : précédé d'un point ., il désigne un membre dans un objet JSON. Les noms de clés doivent être entre guillemets si leur format n'est pas valide dans les expressions de chemin (par exemple, contenant des espaces).

    Exemple : $.nom sélectionne la valeur associée à la clé nom dans un objet.

    Index d'array : [N] sélectionne la valeur à la position N dans un tableau (commençant par zéro). Si le chemin ne sélectionne pas un tableau, path[0] renvoie la même valeur que path.

    Intervalle : [M to N] désigne un sous-ensemble de valeurs dans un tableau, de la position M à N.

    Élément le plus à droite : last est utilisé comme synonyme pour l'index du dernier élément d'un tableau. L'adressage relatif est également supporté, par exemple last-1.

    Jokers : * et ** permettent de sélectionner des valeurs multiples. .* sélectionne toutes les valeurs d'un objet, [*] toutes celles d'un tableau, et prefix**suffix tous les chemins commençant par prefix et se terminant par suffix.

Extraire une valeur

JSON_EXTRACT() ou l'opérateur -> permet d'extraire une valeur JSON spécifiée par un chemin.

Prenons un document en exemple qui serait dans une colonne informations :

{
  "id": 1,
  "nom": "Dupont",
  "prénom": "Jean",
  "adresse": {
    "rue": "123 rue de Paris",
    "ville": "Paris",
    "codePostal": "75000"
  },
  "contacts": ["email@example.com", "email2@example.com"],
  "preferences": {
    "newsletter": true,
    "couleurs": ["bleu", "vert"]
  }
}

Pour extraire le nom de l'utilisateur :

SELECT JSON_EXTRACT(informations, '$.nom') AS nom FROM utilisateurs;
-- Ou en utilisant l'opérateur ->
SELECT informations->'$.nom' AS nom FROM utilisateurs;

Pour extraire le second email :

SELECT JSON_EXTRACT(informations, '$.contacts[1]') AS deuxieme_email FROM utilisateurs;

Pour extraire la première couleur préférée :

SELECT JSON_EXTRACT(informations, '$.preferences.couleurs[0]') AS premiere_couleur FROM utilisateurs;

Modifier une valeur

JSON_SET(), JSON_INSERT(), JSON_REPLACE(), et JSON_REMOVE() permettent de modifier des documents JSON en ajoutant, insérant, remplaçant ou supprimant des valeurs.

    JSON_SET() ajoute ou remplace une valeur.
    JSON_INSERT() ajoute une nouvelle valeur sans remplacer les existantes.
    JSON_REPLACE() remplace les valeurs existantes sans en ajouter de nouvelles.
    JSON_REMOVE() supprime la valeur à un chemin spécifié.

En reprenant notre exemple, voici comment modifier l'adresse :

UPDATE utilisateurs
SET informations = JSON_SET(informations, '$.adresse.rue', '456 avenue de New York')
WHERE JSON_EXTRACT(informations, '$.id') = 1;

Ou comment ajouter un troisième email :

UPDATE utilisateurs
SET informations = JSON_ARRAY_APPEND(informations, '$.contacts', 'email3@example.com')
WHERE JSON_EXTRACT(informations, '$.id') = 1;

Ou remplacer une valeur :

UPDATE utilisateurs
SET informations = JSON_REPLACE(informations, '$.preferences.newsletter', false)
WHERE JSON_EXTRACT(informations, '$.id') = 1;

Ou supprimer le code postal :

vaUPDATE utilisateurs
SET informations = JSON_REMOVE(informations, '$.adresse.codePostal')
WHERE JSON_EXTRACT(informations, '$.id') = 1;

 
Bonnes pratiques

Le format JSON permet de stocker des données dont la structure peut varier ou évoluer avec le temps, comme des configurations utilisateur personnalisées, des profils avec des attributs variables, ou des métadonnées de produits.

N'utilisez des colonnes JSON que lorsque la flexibilité l'emporte sur les besoins de performance et d'intégrité des données.

Des documents très larges peuvent nuire aux performances, particulièrement pour les opérations d'écriture et de mise à jour.

Ne recourez pas aux colonnes JSON par commodité pour éviter la conception d'un schéma de base de données relationnel approprié. Assurez-vous que l'utilisation de JSON est justifiée par la nature des données.

Pour des données fortement structurées avec des relations claires, le modèle relationnel traditionnel reste le choix optimal.

Pour des applications nécessitant une grande flexibilité schématique ou des performances élevées pour de grandes quantités de données semi-structurées, les bases de données NoSQL comme MongoDB peuvent être plus adaptées.