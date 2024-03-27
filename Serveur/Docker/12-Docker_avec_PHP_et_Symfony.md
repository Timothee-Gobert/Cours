# Docker avec PHP et Symfony

## Introduction aux environnements PHP

### Objectifs

L'objectif est de mettre en place *Docker* sur le projet final du cours *Symfony*.

C'est un projet intéressant car il est courant de vouloir "dockeriser" une application existante.

Nous allons donc mettre en place *Docker* pour obtenir un environnement de développement et un environnement de production grâce à deux fichiers *docker-compose* différents.

### Environnement de développement

L'environnement de développement sera : *Symfony* exécuté par le serveur de développement *PHP*, une base de donnés locale *MySQL* et le serveur de développement *Node.js* de *Webpack* pour utiliser *Webpack Encore*.

En développement, nous utiliserons le certificat *TLS* installé par le *CLI Symfony*.

### Environnement de production

L'environnement de production sera : *Symfony* exécuté par l'interface *PHP-FPM*, une base de donnés *MySQL*, *Webpack* pour *bundler* le code de production et le serveur *Web NGINX*.

En production, nous utiliserons un certificat *TLS* en utilisant le client *Certbot*.

### Code sur Github

Le code est disponible sur [Github](https://github.com/dymafr/docker-chapitre12-environnement-symfony/tree/master). Il y a une branche pour Windows / Mac et une branche pour Linux. 

## Mise en place du service MySQL

### Récupération du projet

Récupérez le code du projet sur [Github](https://github.com/dymafr/symfony-chapitre21-mise-en-production).

### Création du fichier *docker-compose.dev.yml*

Créez le fichier *docker-compose.dev.yml* pour l'environnement de développement :

```yaml
version: '3.9'
services:
  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: Pomme123;
    volumes:
      - type: volume
        source: dbtest
        target: /var/lib/mysql
    ports:
      - 3306:3306

volumes:
  dbtest:
```

Lancez ensuite *docker-compose* avec :

```sh
docker compose -f docker-compose.dev.yml  up
```

L'image officielle *MySQL* sera automatiquement téléchargée et construite. Ensuite, un utilisateur *root* sera automatiquement créé avec le mot de passe que nous avons spécifié.

Un volume *dbtest* sera également créé et lié avec le chemin */var/lib/mysql* du conteneur. Ainsi, vous ne perdrez pas vos données entre chaque lancement. Vous pourrez également créer une sauvegarde du volume et la restaurer.

Nous n'isolons pas les conteneurs en local pendant le développement et publions le *port 3306*. Ainsi, vous pouvez accéder directement à la base de données très facilement en utilisant *Workbench*.

### Attention sur les Mac M1

Il faut ajouter *platform: linux/x86_64* au fichier *docker-compose.dev.yml* juste en dessous de image

## Mise en place du service PHP avec Symfony

### Création du fichier *Dockerfile.php*

Nous allons créer une image pour l'environnement de développement *PHP* :

```dockerfile
FROM php:latest
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
RUN curl -sS https://getcomposer.org/installer | php -- --filename=composer --install-dir=/usr/local/bin
RUN curl -sS https://get.symfony.com/cli/installer | bash
RUN apt update && apt install -y zip git libicu-dev locales
RUN docker-php-ext-install opcache intl pdo_mysql
RUN mv /root/.symfony5/bin/symfony /usr/local/bin/symfony
RUN locale-gen fr_FR.UTF-8
WORKDIR /app
COPY ./composer.json .
RUN composer install
COPY . .
RUN symfony server:ca:install
CMD ["symfony", "serve"]
```

Nous partons de la dernière image *PHP* officielle disponible.

Ensuite, nous déplaçons le fichier de configuration de développement de *PHP* au bon endroit, suivant les instructions de l'image *PHP*.

Nous installons *Composer* au bon endroit.

Nous installons le *CLI Symfony*.

Nous installons *zip git libicu-dev locales* qui sont nécessaires pour *PHP* et *Symfony*.

Nous installons les extensions *PHP opcache intl pdo_mysql* qui sont nécessaires à *Symfony* et à *Doctrine* pour utiliser *MySQL*.

Nous déplaçons l'exécutable du *CLI Symfony* pour pouvoir le lancer de n'importe où.

Nous générons les locales de langue française.

Nous définissons le répertoire de travail pour les commandes suivantes dans le conteneur comme */app*.

Nous copions le fichier *composer.json* pour obtenir la liste des dépendances nécessaires à installer et nous les installons.

Nous copions le reste de l'application.

Nous générons le certificat *TLS* local pour pouvoir utiliser *HTTPS*.

Enfin, nous définissons la commande par défaut de l'image : *symfony serve*.

### Modification de *docker-compose.dev.yml*

Voici maintenant notre fichier *docker-compose.dev.yml* :

```yaml
version: '3.9'

services:
  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 'Pomme123123'
      MYSQL_DATABASE: 'quora'
      MYSQL_USER: 'test'
      MYSQL_PASSWORD: 'Pomme123123'
      MYSQL_ROOT_HOST: '%'
    volumes:
      - type: volume
        source: dbtest
        target: /var/lib/mysql
    ports:
      - 3306:3306
  php:
    build:
      context: .
      dockerfile: Dockerfile.php
    volumes:
      - type: volume
        source: tls
        target: /root/.symfony5/certs
      - type: bind
        source: .
        target: /app
      - type: volume
        target: /app/vendor
    ports:
      - 8000:8000

volumes:
  dbtest:
  tls:
```

Nous avons trois volumes pour notre service *php*.

Le premier permet de récupérer le certificat *TLS* généré par le *CLI Symfony* pour que le serveur *Webpack* puisse l'utiliser. Comme il s'agit d'un autre conteneur, il faut obligatoirement créer un volume.

Le second est un *bind mount* permettant de propager les modifications effectuées localement dans le dossier courant dans le conteneur.

Le dernier est un volume anonyme qui permet d'empêcher que le dossier *vendor* du conteneur soit écrasé par le dossier *vendor* local à cause du *bind* précédent.

Lancez ensuite *docker-compose* avec :

```sh
docker compose -f docker-compose.dev.yml  up
```

### Création de la base de données locale

Pour créer la base de données locale, faites simplement :

```sh
docker exec -it NOM_CONTENEUR_PHP bash
```

Ensuite faites :

```sh
symfony console doctrine:database:create
```

Et lancez les migrations :

```sh
symfony console doctrine:migration:migrate
```

## Mise en place du service Node.js

### Création du fichier *Dockerfile.encore.dev*

Créez le fichier *Dockerfile.encore.dev* pour l'image de développement que nous allons créer pour *Webpack Encore* :

```dockerfile
FROM node:16
WORKDIR /app
COPY ./package.json .
RUN yarn
COPY . .
CMD ["yarn", "dev-server"]
```

### Modification de *docker-compose.dev.yml*

Voici maintenant notre fichier *docker-compose.dev.yml* :

```yaml
version: '3.9'

services:
  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 'Pomme123123'
      MYSQL_DATABASE: 'quora'
      MYSQL_USER: 'test'
      MYSQL_PASSWORD: 'Pomme123123'
      MYSQL_ROOT_HOST: '%'
    volumes:
      - type: volume
        source: dbtest
        target: /var/lib/mysql
    ports:
      - 3306:3306
  php:
    build:
      context: .
      dockerfile: Dockerfile.php
    volumes:
      - type: volume
        source: tls
        target: /root/.symfony5/certs
      - type: bind
        source: .
        target: /app
      - type: volume
        target: /app/vendor
    ports:
      - 8000:8000
  node:
    build:
      context: .
      dockerfile: Dockerfile.encore.dev
    volumes:
      - type: volume
        source: tls
        target: /root/.symfony5/certs
      - type: volume
        target: /app/node_modules
      - type: bind
        target: /app
        source: .
    ports:
      - 8080:8080
volumes:
  dbtest:
  tls:
```

Pour le service *node* qui est utilisé pour *Webpack Encore*, nous aurons également besoin de trois volumes.

Le premier, permet de récupérer le certificat *TLS* généré par le *CLI Symfony*.

Le second, permet d'éviter que le dossier *node_modules* du conteneur ne soit écrasé par le *bind*.

Le dernier, permet de propager les changements dans l'application au conteneur. De cette manière, *Webpack Encore* a bien accès aux changements.

L'environnement de développement est prêt.

## Mise en place des services PHP-FPM et Webpack Encore

### Création du fichier *Dockerfile.fpm*

Nous allons créer une image pour la production, pour le service *PHP-FPM* que nous allons utiliser :

```dockerfile
FROM php:8.0-fpm
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
RUN curl -sS https://getcomposer.org/installer | php -- --filename=composer --install-dir=/usr/local/bin
RUN curl -sS https://get.symfony.com/cli/installer | bash
RUN apt update && apt install -y zip git libicu-dev locales
RUN docker-php-ext-install opcache intl pdo_mysql
RUN mv /root/.symfony5/bin/symfony /usr/local/bin/symfony
RUN locale-gen fr_FR.UTF-8
WORKDIR /app
COPY ./composer.json .
RUN composer install --no-dev --optimize-autoloader
COPY . .
RUN APP_ENV=prod APP_DEBUG=0 php bin/console cache:clear
RUN usermod -u 1000 www-data
```

L'image est proche de celle de développement. La principale différence est que nous partons de l'image officielle *php:8.0-fpm* car nous voulons utiliser une version fixe et en version *PHP-FPM* pour la production.

L'autre différence est que nous utilisons les commandes recommandées par *Symfony* pour la production pour optimiser l'*opcache* et l'*autoloader*.

### Création du fichier *Dockerfile.encore.prod*

Pour *Webpack Encore*, l'image va nous permettre de lancer le *build* des assets.

```dockerfile
FROM node:16
WORKDIR /app
COPY ./package.json .
RUN yarn
COPY . .
CMD ["yarn", "build"]
```

### Création du fichier *docker-compose.prod.yml*

Nous passons au fichier *docker-compose* pour mettre en place les services en production :

```yaml
version: '3.9'

services:
  mysql:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: Pomme123;
    volumes:
      - type: volume
        source: dbprod
        target: /var/lib/mysql
  fpm:
    build:
      context: .
      dockerfile: Dockerfile.fpm
    volumes:
      - type: volume
        target: /app/vendor
      - type: bind
        target: /app
        source: .

  node:
    build:
      context: .
      dockerfile: Dockerfile.encore.prod
    volumes:
      - type: volume
        target: /app/node_modules
      - type: bind
        target: /app
        source: .

volumes:
  dbprod:
  ```

## Mise en place du service NGINX

### Création du fichier *Dockerfile.nginx*

Nous créons une image à partir de l'image officielle *NGINX* et nous y copions notre configuration :

```dockerfile
FROM nginx:latest
WORKDIR /app
COPY ./nginx/wonder.conf /etc/nginx/conf.d/default.conf
COPY . .
```

### Modification du fichier *nginx/wonder.conf*

Il faut effectuer quelques modifications à la configuration de *NGINX* :

```sh
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    return 301 https://dymawonder.fr$request_uri;
}


server {
    listen 443 ssl default_server http2;
    listen [::]:443 ssl default_server http2;

    # ssl_certificate /certs/fullchain.cer;
    # ssl_certificate_key /certs/dymawonder.fr.key;

    server_name dymawonder.fr www.dymawonder.fr;
    root /app/public; # Modifier ici !

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/index.php(/|$) {
        fastcgi_pass fpm:9000; # Modifier ici !
        fastcgi_split_path_info ^(.+.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
    }

    location ~ .php$ {
        return 404;
    }
    error_log /var/log/nginx/project_error.log;
    access_log /var/log/nginx/project_access.log;
}
```

### Modification du fichier *docker-compose.prod.yml*

Nous passons à la mise en place du service :

```yaml
version: '3.9'

services:
  nginx:
    build:
      context: .
      dockerfile: Dockerfile.nginx
    volumes:
      - type: bind
        source: .
        target: /app
    ports:
      - 80:80
      - 443:443
  mysql:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: Pomme123;
    volumes:
      - type: volume
        source: dbprod
        target: /var/lib/mysql
  fpm:
    build:
      context: .
      dockerfile: Dockerfile.fpm
    volumes:
      - type: volume
        target: /app/vendor
      - type: bind
        target: /app
        source: .
  node:
    build:
      context: .
      dockerfile: Dockerfile.encore.prod
    volumes:
      - type: volume
        target: /app/node_modules
      - type: bind
        target: /app
        source: .

volumes:
  dbprod:
```

## Mise en production

### Récupération du code depuis *Github*

Sur votre *VPS* ou sur votre serveur dédié, il faut commencer par récupérer le code, par exemple :

```sh
git clone https://github.com/dymafr/docker-chapitre12-environnement-symfony
```

### Installation de *Docker* et *Docker-Compose* sur le serveur

Pour installer *Docker* sur le serveur, commencez par mettre à jour les dépendances disponibles :

```sh
sudo apt-get update
```

Ensuite installez les dépendances nécessaires :

```sh
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```

Ajoutez la clé GPG officielle :

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Enfin, ajoutez le répertoire stable *Docker* :

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Récupérez la dernière version disponible :

```sh
sudo apt-get update
```

Installez là :

```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Ensuite créez le groupe *Docker* :

```sh
sudo groupadd docker
```

Et ajoutez votre utilisateur courant à celui-ci :

```sh
sudo usermod -aG docker $USER
```

Rafraîchissez le groupe :

```sh
newgrp docker
```

Cela permet de lancer les commandes *Docker* sans *sudo* (ou sans être *root*).

Nous passons à l'installation de *Docker-Compose* :

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Puis nous changeons les permissions du binaire pour l'exécuter :

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

Vérifiez l'installation :

```sh
docker compose --version
```

### Utilisation d'*acme.sh* pour le certificat *TLS*

Nous allons utiliser le client *acme.sh* qui est un autre client que *certbot* permettant de récupérer un certificat *TLS* auprès de l'autorité de certification *Let's Encrypt*.

Commencez par l'installer :

```sh
sudo curl https://get.acme.sh | sh -s email=VOTRE@EMAIL
```

Puis installez la seule dépendance nécessaire :

```sh
sudo apt install socat
```

Ensuite, **lorsque tout est coupé sur votre serveur**, exécutez la commande pour récupérer votre certificat *TLS* et mettre en place son renouvellement automatique :

```sh
sudo sh /root/.acme.sh/acme.sh --issue -d VOTRE_DOMAINE.FR  --standalone --pre-hook "docker compose -f /home/ubuntu/docker-chapitre12-environnement-symfony/docker-compose.prod.yml stop" --post-hook "docker compose -f /home/ubuntu/docker-chapitre12-environnement-symfony/docker-compose.prod.yml start"
```

`--pre-hook` permet d'exécuter une commande avant de tenter d'obtenir ou de renouveler un certificat. Ici nous voulons couper nos services et plus particulièrement *NGINX* car le port 80 doit être libre pour cette procédure.

`--post-hook` permet d'exécuter une commande après avoir tenté d'obtenir ou de renouveler un certificat. Ici nous voulons relancer nos services.

**Adaptez bien le chemin vers le dossier contenant votre application**, cela ne sera pas forcément `/home/ubuntu/docker-chapitre12-environnement-symfony/`. Cela dépend bien entendu de là où vous l'avez placée.

### Modification de *nginx/wonder.conf*

Maintenant que vous avez votre certificat *TLS* sur votre serveur, il faut modifier la configuration *NGINX* pour la préciser :

```sh
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    return 301 https://dymawonder.fr$request_uri; # Modifier ici
}

server {
    listen 443 ssl default_server http2;
    listen [::]:443 ssl default_server http2;

    ssl_certificate /certs/fullchain.cer;
    ssl_certificate_key /certs/dymawonder.fr.key; # Modifier ici

    server_name dymawonder.fr www.dymawonder.fr; # Modifier ici
    root /app/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/index.php(/|$) {
        fastcgi_pass fpm:9000;
        fastcgi_split_path_info ^(.+.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        internal;
    }
    location ~ .php$ {
        return 404;
    }
    error_log /var/log/nginx/project_error.log;
    access_log /var/log/nginx/project_access.log;
}
```

Modifiez la directive *ssl_certificate_key* en utilisant le nom donné par *acme.sh* après l'obtention de votre certificat *TLS*.

Modifiez également la directive *server_name* et le *return 301* suivant votre nom de domaine.

### Modification du fichier *docker-compose.prod.yml*

Voici nos services pour la production :

```yaml
version: '3.9'

services:
  nginx:
    build:
      context: .
      dockerfile: Dockerfile.nginx
    volumes:
      - type: bind
        source: /root/.acme.sh/dymawonder.fr
        target: /certs
      - type: bind
        source: .
        target: /app
    depends_on:
      - fpm
    ports:
      - 80:80
      - 443:443
  mysql:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: Pomme123;
    volumes:
      - type: volume
        source: dbprod
        target: /var/lib/mysql
  fpm:
    build:
      context: .
      dockerfile: Dockerfile.fpm
    volumes:
      - type: volume
        target: /app/vendor
      - type: bind
        target: /app
        source: .
    depends_on:
      - mysql

  node:
    build:
      context: .
      dockerfile: Dockerfile.encore.prod
    volumes:
      - type: volume
        target: /app/node_modules
      - type: bind
        target: /app
        source: .

volumes:
  dbprod:
```

N'oubliez pas de modifier *source: /root/.acme.sh/dymawonder.fr* suivant le nom de votre certificat donné par *acme.sh*.

### Création du fichier *.env.local*

**Sur le serveur de production uniquement**, créez un fichier *.env.local*.

Il doit contenir notamment le mot de passe de la base de données de production, le secret pour la clé d'application *Symfony* et l'environnement :

```sh
APP_ENV=dev
APP_SECRET=f5488b891dfa4540efaa1c19890b8036
DATABASE_URL="mysql://root:Pomme123;@mysql/quora?serverVersion=8.0.26"
```

Générez un secret long pour *APP_SECRET* comme nous l'avons vu dans le cours *Symfony*.

N'oubliez pas de *dumper* l'environnement, *toujours sur le serveur en utilisant une connexion SSH* :

```sh
composer dump-env prod
```

### Création de la base de données de production

**Sur le serveur**, en utilisant une clé *SSH*, connectez-vous au conteneur du service *fpm* :

```sh
docker exec -it NOM_CONTENEUR_PHP_FPM bash
```

Ensuite faites :

```sh
symfony console doctrine:database:create
```

Et lancez les migrations :

```sh
symfony console doctrine:migration:migrate
```