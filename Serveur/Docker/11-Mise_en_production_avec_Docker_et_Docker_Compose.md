# Mise en production avec Docker et Docker compose

## Introduction à la mise en production

### Objectifs du chapitre

Dans ce chapitre, nous allons voir comment mettre en production l'application que nous avons créée dans le chapitre précédent sur un serveur.

Nous allons pour ce faire devoir nous connecter au serveur en *SSH* et créer un certificat *TLS* pour pouvoir servir l'application de manière sécurisée.

Pour la partie *API*, nous allons voir comment utiliser *pm2* avec *Docker* afin de lancer plusieurs processus *Node.js* et augmenter les performances de notre microservice *API*.

Il s'agit d'un chapitre avancé qui demande d'avoir vu les cours *Linux*, *Git* et *Node.js*. Nous ne détaillerons pas toutes les explications ici car le chapitre serait bien trop long, aussi si vous ne comprenez pas une partie tournez vous vers le cours correspondant !

### Utilisation d'un *VPS*

Un serveur dédié virtuel (*VPS* en anglais) est une méthode de division d'un serveur en plusieurs serveurs virtuels indépendants. Cela permet aux hébergeurs de proposer des offres à bas coûts en louant des "bouts de serveur", plutôt que des serveurs dédiés, qui sont beaucoup plus onéreux.

Chaque serveur virtuel peut fonctionner avec un système d'exploitation différent et redémarrer indépendamment.

Les détails techniques de l'isolation sont totalement différents suivant les hébergeurs, ils reposent sur des technologies de virtualisation (machines virtuelles).

### Gestion du signal *SIGINT*

Nous allons modifier *api/src/index.js* pour mettre en place ce qu'on appelle un *graceful shutdown*.

Cela signifie que lorsque l'application *Node.js* reçoit un signal d'arrêt (*cf le cours Linux pour les signaux*), nous voulons couper le serveur pour ne plus qu'il accepte de requête entrante, puis une fois qu'il est coupé, couper la connexion à la base de données.

De cette manière, nous sommes sûr que lorsque nous ferons *docker compose stop* par exemple, notre application *Node.js* sera coupée proprement sans que des requêtes soient en cours de traitement, aussi bien au niveau du serveur que de la base de données.

```js
const express = require('express');
const { MongoClient } = require('mongodb');
const client = new MongoClient(uri);
let count;

const uri =
  process.env.NODE_ENV === 'production'
    ? `mongodb://${process.env.MONGO_USERNAME}:${process.env.MONGO_PWD}@db`
    : `mongodb://db`;

async function run() {
  try {
    await client.connect();
    await client.db('admin').command({ ping: 1 });
    console.log('CONNEXION DB OK !');
    count = client.db('test').collection('count');
  } catch (err) {
    console.log(err.stack);
  }
}
run().catch(console.dir);


const app = express();

app.get('/api/count', (req, res) => {
  count
    .findOneAndUpdate({}, { $inc: { count: 1 } }, { returnNewDocument: true, upsert: true })
    .then((doc) => {
      res.status(200).json(doc ? doc.count : 0);
    });
});

app.all('*', (req, res) => {
  res.status(404).end();
});

const server = app.listen(80);

// Graceful shutdown.
// On empêche les nouvelles connexions sur le serveur
// Ensuite on close proprement la connexion DB
process.on('SIGINT', () => {
  server.close((err) => {
    if (err) {
      process.exit(1);
    } else {
      if (client) {
        client.close((err) => {
          process.exit(err ? 1 : 0);
        });
      } else {
        process.exit(0);
      }
    }
  });
});
```

### Utilisation d'une version *LTS*

Pour la production il vaut mieux toujours utiliser les versions stables.

Pour l'image de *Node.js*, dans sa version *Alpine*, la version stable est *node:lts-alpine*.

Aussi, nous modifions le *Dockerfile* dans le dossier *api* :

```dockerfile
FROM node:lts-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
RUN npm install pm2 -g
COPY . .
CMD ["pm2-runtime", "ecosystem.config.js"]
EXPOSE 80
```

## Utilisation de pm2

### Rappels sur PM2

*PM2* est un gestionnaire de processus pour *Node.js*.

Il permet notamment de faciliter l'utilisation du mode *cluster*, c'est-à-dire l'utilisation de plusieurs instances d'une même application pour optimiser les performances.

En effet, *Node.js* utilise un *thread* uniquement, et pour être sûr d'utiliser toute la puissance disponible de votre serveur il vaut mieux avoir une instance de *Node.js* par *thread* disponible.

Vous trouverez tous les détails dans la formation *Node.js*. Ce qui nous intéresse ici est uniquement son utilisation avec *Docker*.

### Création d'un *Dockerfile* pour la production

Nous créons un fichier *Dockerfile.prod* dans le dossier *api* :

```dockerfile
FROM node:lts-alpine
WORKDIR "/app"
COPY ./package.json ./
RUN npm install
RUN npm install pm2 -g
COPY . .
CMD ["npm", "run", "prod"]
EXPOSE 80
```

Nous renommons également le *Dockerfile* existant en lui ajoutant l'extension *.dev*.

Nous modifions en conséquence les fichiers *docker-compose.dev.yml* :

```yaml
version: "3.9"
services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.dev
    volumes:
      - type: bind
        source: ./client
        target: /home/node
      - type: volume
        target: /home/node/node_modules
    ports:
      - 3000:3000
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.dev
    volumes:
      - type: bind
        source: ./api/src
        target: /app/src
    ports:
      - 3001:80
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbtest
        target: /data/db
  reverse-proxy:
    build:
      context: ./reverse-proxy
      dockerfile: Dockerfile.dev
    ports:
      - 80:80
    depends_on:
      - api
      - db
volumes:
  dbtest:
```

Et *docker-compose.prod.yml* :

```yaml
version: "3.9"
services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.prod
    restart: unless-stopped
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
    env_file:
      - ./api/.env
    environment:
      NODE_ENV: production
    restart: unless-stopped
    depends_on:
      - db
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbprod
        target: /data/db
    env_file:
      - ./db/.env
    restart: unless-stopped
  reverse-proxy:
    build:
      context: ./reverse-proxy
      dockerfile: Dockerfile.prod
    ports:
      - 80:80
    restart: unless-stopped
    depends_on:
      - api
      - db
      - client
volumes:
  dbprod:
    external: true
```

### Modification du *package.json*

Nous ajoutons le script pour le lancement de la production qui est lancé par *npm*.

Nous modifions donc *api/package.json* :

```json
{
  "name": "api",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "nodemon ./src/index.js",
    "prod": "pm2-runtime ecosystem.config.js",
    "test": "echo "Error: no test specified" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^6.1.0",
    "nodemon": "^3.0.1"
}
```

### Création du fichier *ecosystem.config.js*

Le fichier *ecosystem.config.js* est un fichier de configuration permettant de configurer les options de lancement d'une ou plusieurs applications *Node.js* par *PM2*.

Nous plaçons le code suivant :

```js
module.exports = [
  {
    script: 'src/index.js',
    name: 'api',
    exec_mode: 'cluster',
    instances: max,
  },
];
```

Grâce à cette configuration, *PM2* lancera *max* instances de notre application *api* en mode *cluster*.

La valeur *max* sera déterminée par votre *CPU*, et plus précisément le nombre de *threads* disponibles.

Nous pouvons tester notre setup :

```sh
docker compose -f docker-compose.prod.yml up --build
```

## Environnement de production

### Enlever les fichiers d'environnement

Nous allons commencer par enlever les fichiers d'environnement car ce n'est pas une bonne pratique de les laisser en production.

Nous modifions *docker-compose.prod.yml* :

```yaml
version: "3.9"
services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.prod
    restart: unless-stopped
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
    environment:
      - MONGO_USERNAME
      - MONGO_PWD
      - NODE_ENV=production
    restart: unless-stopped
    depends_on:
      - db
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbprod
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
    restart: unless-stopped
  reverse-proxy:
    build:
      context: ./reverse-proxy
      dockerfile: Dockerfile.prod
    ports:
      - 80:80
    restart: unless-stopped
    depends_on:
      - api
      - db
      - client
volumes:
  dbprod:
    external: true
```

Nous déclarons les variables d'environnement mais nous ne passerons les valeurs que lors du lancement du processus *Docker*.

Nous cachons également le fichier *.env* en créant un fichier *.dockerignore* dans lequel nous mettons :

```
.env
```

De cette manière, les fichiers *.env* ne seront pas copiés par *Docker* dans les images.
### Passer les variables d'environnement lors du lancement

Pour passer les variables d'environnement à *Docker Compose*, il suffira de faire :

```sh
MONGO_USERNAME=paul MONGO_PWD=123 docker compose -f docker-compose.prod.yml up --build
```

## Mise du projet sur Gitlab

### Initialisation du répertoire *Git*

Nous allons placer notre application sur *Gitlab*.

Pour voir tous les détails nous vous invitons à voir la formation Git.

Nous initialisons le répertoire :

```sh
git init
```

Ensuite, nous créons un fichier *.gitignore* pour ne pas mettre non plus nos variables d'environnement sur *Gitlab*. Même si le répertoire est privé, c'est une mauvaise pratique de laisser des secrets (clés privées, mot de passe etc) en clair sur un répertoire.

Dans ce fichier nous mettons simplement :

```
.env
**/node_modules
```

### Premier *commit*

Nous ajoutons ensuite tous les éléments, à la racine de notre projet nous faisons :

```sh
git add *
```

Puis :

```sh
git commit -m "premier commit"
```

### Créer un compte sur Gitlab

Une fois votre compte créé et votre email validé, allez tout d'abord sur l'icône de profil tout en haut à droite.

Cliquez ensuite sur *Settings*.

Dans la colonne de gauche, vers le bas, cliquez sur *Preferences*.

Dans *Localization* sélectionnez *language Français* et *First day of the week Monday* puis sauvegardez.

En haut à gauche cliquez sur *Projets > Vos projets*.

Ensuite cliquez sur *Create Project*.

Donnez n'importe quel nom au projet, laissez la visibilité par défaut à privé et cliquez sur *create project*.

Vous arriverez alors sur une nouvelle interface *Project overview > Détails*.

*Gitlab* vous indiquera que *Le dépôt de ce projet est vide*. Ce qui est tout à fait normal !

### Générer une paire de clés et l'utiliser avec *Gitlab*

Vous pouvez commencer par vérifier si une paire de clé existe sur votre machine :

```sh
cd ~/.ssh
ls
```

Si vous avez un fichier *id_rsa* et un fichier *id_rsa.pub*, vous avez une paire de clé publique / privée.

Nous allons utiliser la librairie *ssh-keygen* qui est disponible sur tous les environnements (*Windows*, *Linux* et *MacOS*) et installée par défaut..

Il suffit de faire :

```sh
ssh-keygen
```

Cela générera par défaut une paire de clés de longueur de *2048 bits* en utilisant l'algorithme *RSA*.

Tapez entrée à toutes les questions, sauf si vous utilisez un ordinateur partagé dans ce cas entrez un mot de passe lorsque cela vous sera demandé. Ce mot de passe sera demandé à chaque fois que la paire de clé sera utilisée.

Par défaut les clés seront enregistrées dans */home/utilisateur/.ssh/*.

Où *utilisateur* est le nom de votre utilisateur courant.

Les options courantes sont :
- `-t` permettant de spécifier le type d'algorithme utilisé pour la génération de clés. Laissez le par défaut, *RSA* est le standard.
- `-C` pour *comment* : cela permet d'insérer un commentaire dans la clé. Le plus souvent on précise un email ou un identifiant pour permettre plus facilement de connaitre le propriétaire de la clé (attention ces informations sont incluses dans la clé publique est donc accessibles).
- `-b` permet de préciser la longueur de la clé en *bits*. Par défaut, suivant votre environnement ce sera *2048 bits* et parfois *3072 bits*. Vous pouvez préciser `-b 4096` pour allonger la clé et la rendre plus sécurisée, mais cela ne sert à rien en l'état actuelle des puissances de calcul disponible.

Pour retrouver vos clés rendez vous dans :

```sh
cd /home/utilisateur/.ssh
```

Ou, le raccourci :

```sh
cd ~/.ssh
```

Vous devrez copier la clé publique pour la mettre sur *Gitlab*.

Affichez là en faisant :

```sh
cat ~/.ssh/id_rsa.pub
```

Elle doit ressembler à :

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC+OAb59N+WS68D/xiYdw
  PPqAOqbrhFAney4jJVMiKMVrt0rYC9V+80mjKB9WA1jpf4e+3AtrXMDqiq2rbdMuQbkjzbjSCMz
  l57vJlXkixNpAgzdF3HU13RhMvi+PVMrbKwPIMMQPgdJIemRm85HI4GPz0WaTDezBwLOvTbmiPz
  Jl/LbfmcCGD8ViciPOhlLFcj3nPLzmU3PtE8UC57SoiYaXYlGuc+p0tQN7TjvITQHNvSvukzOQT
  ZjC/w+52v6tmjULej6nLXRN4wTEpB21lnYw+UYRY2egd385yGVx13kIs/4DR9apD/rW4BqwOHuV
  DHwur5f41tlys3Rg0hAgysSWjhQaT+qt7sy3PgkM7p0IAXeGuOst7IKXt+vTTgUrhBMVWiOb1dH
  vCcdjaCASdQ/3U1JE5Anqe+eoimzSr3vJQfpZfZwS34lQqqRnQgIjwiHXPPJxpFxd7mctt7AnVC
  LMpOnKZLpaRRVei3FjQ6mwlqOXpanVyivvIVx60MHItKbw1ZV4/NOHri5B+3qC4OQKIVB0iUmmj
  xEqxXXw84EV5vAmasWrztShBID+WCezvYIG3Bw3NHiVxv+cnBmrRSLLToZGkAFbjoDLE7mckLVX
  tUmA+/wWTuznI7O//7jxgm7eowbyM9JmLIrtx1hvuIBg9KsTkAYhj59s5knd37eQ
  == erwan@erwan-XPS-8930
```

Vous pouvez ensuite la copier et vous rendre sur *Gitlab*.

Allez sur *Profil* (en haut à droite), puis *Paramètres*.

Ensuite dans la colonne de gauche allez dans *Clefs SSH*.

Copier la clé dans l'espace et cliquez sur *Ajouter une clef*.

Vous pouvez maintenant vous connecter en utilisant *SSH* !

### *Push* du projet sur *Gitlab*

**De retour sur le projet dans *Gitlab* copiez la commande qui commence par `git remote add origin git@gitlab` et tapez là dans votre terminal.**

Vous pourrez ensuite faire :

```sh
git push -u origin master
```

Vous avez maintenant le projet disponible sur *Gitlab* et votre connexion en *SSH* paramétrée ! 

## Création du VPS
### Création d'un serveur

Il vous faut un serveur pour suivre ce chapitre. Cela peut être n'importe quel serveur *GNU/Linux* sur n'importe quel fournisseur : OVH, Scaleway, LWS, AWS, Azure, Google Cloud ou un des centaines d'autres existant.

Nous recommandons OVH avec une tarification claire et leader en Europe.

Nous vous conseillons un *VPS* (*Virtual Private Server*), c'est-à-dire un environnement virtuel isolé sur une machine physique partagée avec de nombreux autres utilisateurs. L'avantage des *VPS* est bien sûr leur coût et leur flexibilité car vous pouvez tout faire comme si vous étiez sur un serveur dédié.

Sur OVH créez un compte puis allez dans Serveur puis *VPS*.

### Connexion au serveur

Quel que soit le fournisseur, vous recevrez un email avec les identifiants pour vous connecter en *SSH* au serveur.

Cela peut être par exemple sur *OVH* quelque chose comme :

```sh
Bonjour,


Votre VPS vient d'être installé sous le système d'exploitation / distribution
Ubuntu 20.04 Server (en version 64 bits)


PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 164.132.230.167
L'adresse IPv6 du VPS est : 2001:41d0:0401:3200:0000:0000:0000:1be5

Le nom du VPS est : vps790870.ovh.net

Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : root
Mot de passe :      NAzz1CCE
```

Pour vous connecter en *SSH* il faut utiliser la syntaxe suivante :

```sh
ssh utilisateur@hôte
```

L'hôte du serveur est l'adresse *IP* ou son adresse qui sera résolue en *IP*.

Dans notre exemple il faut donc faire :

```sh
ssh root@164.132.230.167
```

Il faut ensuite entrer le nom d'utilisateur et le *password* envoyés par email.

Une fois connecté, nous allons créer un nouvel utilisateur :

```sh
adduser jean
```

Entrez un mot de passe.

Nous l'ajoutons ensuite au groupe des *sudoers* :

```sh
usermod -aG sudo jean
```

Nous nous connectons enfin à *jean* :

```sh
su jean -
```

*Pour plus de détails il faut suivre la formation **Linux**.*

### Configuration de l'hôte distant

*De retour sur votre machine*, après avoir quitté la connexion *SSH* au serveur, nous allons configurer l'hôte distant pour se connecter facilement en *SSH*.

**Le fichier *~/.ssh/config* est extrêmement utile pour configurer la connexion à ses hôtes distants.**

Nous modifions ce fichier avec *nano* :

```sh
nano ~/.ssh/config
```

Et nous mettons les valeurs suivantes (il faut bien sûr que vous mettiez l'*IP* et l'utilisateur créé sur votre serveur) :

```
Host dockerprod
      Hostname 164.132.230.167
      User jean
      Port 22
```

Nous ajoutons enfin notre clé *ssh* sur le serveur pour se connecter au serveur sans avoir à entrer de mot de passe :

```sh
ssh-copy-id jean@164.132.230.167
```

Cette commande va utiliser le protocole *SSH* pour se connecter à l'hôte et y copier toutes vos clés publiques situées dans *~/.ssh/*.

Vous verrez alors :

```
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
jean@164.132.230.167's password:
```

Il faut alors entrer le mot de passe de l'utilisateur que vous avez créé sur le serveur.

Vous pouvez maintenant vous connecter au serveur en faisant simplement :

```sh
ssh dockerprod
```

### Configuration sécurisée de *SSH* sur le serveur

*Sur votre serveur*, une fois connecté en *SSH*, nous allons modifier la configuration du démon *SSH* pour avoir un minimum de sécurité :

```sh
sudo nano /etc/ssh/sshd_config
```

Modifiez les valeurs suivantes :

```
PasswordAuthentication no
X11Forwarding no
PermitRootLogin no
Protocol 2
```

Une fois la configuration terminée, il ne reste plus qu'à recharger la configuration avec le démon *sshd* :

```
systemctl reload sshd
```

**Vous ne pouvez plus vous connecter en *SSH* que depuis votre machine sur l'utilisateur que vous avez créé.** C'est ce que nous voulions pour une sécurité minimale.

Enfin, nous modifions les permissions du dossier *home* pour plus de sécurité :

```sh
sudo chmod 700 ~
```

### Achat du nom de domaine

Etape 1 : Allez dans *Web*.

Etape 2 : *Order*.

Etape 3 : *Domains*.

Choisissez ensuite votre nom de domaine, faites attention au prix d'achat, mais aussi au **prix du renouvellement** qui peut fortement augmenter pour certains domaines.

Ensuite procédez au paiement. Le nom de domaine est prêt au bout d'environ 10 minutes.

### Configuration de la zone *DNS* sur *OVH*

Ensuite retournez sur *OVH* sur la page de votre domaine et allez dans l'onglet *Zone DNS*.

Sur l'entrée de type *A* cliquez sur "Modifier l'entrée" et entrez l'*IP* de votre serveur, qui est *164.132.230.167* dans notre exemple.

## Mise en place du certificat TLS

### Qu'est-ce que *letsencrypt* ?

*Let's Encrypt* est une autorité de certification en 2015. Elle fournit gratuitement des certificats *X509* pour pouvoir utiliser le protocole *TLS*.

Les principaux sponsors de cette initiative sont Cisco, Mozilla, EFF, OVH, Google, l'Internet Society et Facebook.

Ils permettent à 150 millions de sites web de pouvoir servir gratuitement leur application en *HTTPS*.

Ils ont développé un protocole *ACME* permettant de gérer les certificats de manière totalement automatisée (attribution, révocation et mise à jour) qui est standardisé par l'*IETF* depuis début 2019.

### Qu'est-ce que *certbot* ?

*Certbot* est le client officiel de *Let's Encrypt* permettant d'utiliser le protocole *ACME* de manière automatisée.

Il permet ainsi de gérer ses certificats de manière vraiment rapide et efficace.

### Installation de *certbot*

Connectez-vous en *ssh* à votre serveur.

Puis lancez l'installation de *certbot*, sur *Uubntu* faites :

```sh
sudo snap install certbot --classic
```

Sur une autre distribution *Debian* qu'*Ubuntu* faites :

```sh
sudo apt install certbot
```

Pour créer un certificat avec *certbot*, il suffit de faire :

```sh
sudo certbot certonly -d DOMAIN1 -d DOMAIN2 -d DOMAIN3
```

Il faut que le domaine passé match exactement l'*URL* racine de votre site.

Par exemple si l'utilisateur tape *www.votresite.fr* ou *votresite.fr* il faudra ajouter les deux domaines à la demande de certificat.

Ensuite sélectionnez l'option 1 : *standalone*. Il faut que votre application soit coupée car le port *80* doit être disponible pour le mini-serveur que va lancer *certbot*.

Ensuite, vous allez devoir créer un compte auprès de *letsencrypt*, vous devrez fournir l'email de l'administrateur du domaine (le vôtre probablement), vous devez ensuite accepter les conditions d'utilisation avec *A* et enfin indiquer si vous voulez partager votre email à une *ONG* pour la liberté de l'*Internet* (qui défend la neutralité du net entre autre).

*certbot* vous a automatiquement généré une clé privée et obtenu un certificat valide auprès de *letsencrypt*.

Les certificats se trouvent dans :

*/etc/letsencrypt/live/NOM_DOMAINE*

### Utilisation du certificat TLS

Nous devons modifier *docker-compose.prod.yml* pour ouvrir le port *443* utilisé par *HTTPS* et *HTTP2* pour notre *reverse proxy* :

```yaml
version: "3.9"
services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.prod
    restart: unless-stopped
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
    environment:
      - MONGO_USERNAME
      - MONGO_PWD
      - NODE_ENV=production
    restart: unless-stopped
    depends_on:
      - db
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbprod
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
    restart: unless-stopped
  reverse-proxy:
    build:
      context: ./reverse-proxy
      dockerfile: Dockerfile.prod
    ports:
      - 80:80
      - 443:443
    restart: unless-stopped
    depends_on:
      - api
      - db
      - client
volumes:
  dbprod:
    external: true
```

### Modification de *reverse-proxy/conf/prod.conf*

Nous devons maintenant utiliser *HTTP2* avec *nginx*.

```sh
server {
    listen 80;
    return 301 https://sandbox-dyma.ovh$request_uri;
}

server {
    listen 443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/www.sandbox-dyma.ovh/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.sandbox-dyma.ovh/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/www.sandbox-dyma.ovh/chain.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://client;
    }

    location /api {
        proxy_pass http://api;
    }
}
```

Nous forçons la redirection des requêtes sur le port 80 (requêtes *HTTP*) sur le port 443 (*HTTPS* / *HTTP2*).

Pour le serveur virtuel écoutant sur le port *443*, nous utilisons les certificats créés pour pouvoir utiliser *HTTP2* (qui utilise *TLS*).

Nous spécifions que nous voulons les deux dernières versions de *TLS* pour les connexions client / serveur pour plus de sécurité.

Nous utilisons ce qu'on appelle une terminaison *TLS* : notre serveur *proxy* est un intermédiaire entre le client et nos autres services. La connexion entre le client et le *reverse proxy* est une connexion *HTTPS* sécurisée. Mais les connexions entre le *reverse proxy* et les autres parties de l'application n'ont pas à être sécurisées car elles utilisent un réseau privée mis en place par *Docker*. Seul le *reverseproxy* peut s'y connecter.

### Création d'un *bind mount*

Pour que le *reverse proxy* puisse accéder aux certificats *TLS*, nous allons créer un *bind mount* en modifiant *docker-compose.prod.yml* :

```yaml
version: "3.9"
services:
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.prod
    restart: unless-stopped
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
    environment:
      - MONGO_USERNAME
      - MONGO_PWD
      - NODE_ENV=production
    restart: unless-stopped
    depends_on:
      - db
  db:
    image: mongo:7
    volumes:
      - type: volume
        source: dbprod
        target: /data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD
    restart: unless-stopped
  reverse-proxy:
    build:
      context: ./reverse-proxy
      dockerfile: Dockerfile.prod
    ports:
      - 80:80
      - 443:443
    volumes:
      - type: bind
        source: /etc/letsencrypt
        target: /etc/letsencrypt
    restart: unless-stopped
    depends_on:
      - api
      - db
      - client
volumes:
  dbprod:
    external: true
```

Nous montons simplement le dossier */etc/letsencrypt* qui contient nos certificats *letsencrypt*. 

## Lancement de la production

### *Push* du code sur *Git*

Nous ajoutons nos changements :

```sh
git add *
```

Nous créons un *commit* :

```sh
git commit -m "prod ready"
```

Puis nous pushons sur le serveur :

```sh
git push origin
```

### Clonage depuis le serveur

Connectez vous de nouveau en *SSH* sur votre serveur.

Ensuite récupérez le lien pour cloner votre projet qui est disponible sur *Gitlab* en cliquant sur le bouton bleu *Clone*.

Récupérez le lien qui commence par *https* car nous ne copierons pas la clé *SSH* du serveur sur *Gitlab* pour qu'il se connecte en *SSH*.

Collez la commande fournie qui commence par `git clone https://gittlab.com/` sur le terminal connecté à votre serveur en *SSH*.

Si le répertoire est privé il faudra bien sûr vous connecter avec vos identifiants *Gitlab*.

### Installation de *Docker*

Si votre serveur est sous *Ubuntu* faites simplement :

```sh
sudo snap install docker
```

Sinon, suivez les instructions comme indiqué au début du cours.

Pour vérifier l'installation, faites simplement :

```sh
sudo docker
```

### Mise en place de la base de données

Sur le serveur, nous devons créer le volume de production :

```sh
sudo docker volume create dbprod
```

Nous initialisons ensuite notre service de base de données :

```sh
sudo MONGO_INITDB_ROOT_PASSWORD=password MONGO_INITDB_ROOT_USERNAME=root docker compose -f docker-compose.prod.yml run -d db
```

Il faut préciser les identifiants de l'administrateur de la base de données qui a les permissions *root* (il peut tout faire).

Nous nous connectons sur le client de *MongoDB* :

```sh
sudo docker compose exec -it db mongosh
```

Nous pouvons alors nous authentifier dans la console :

```sql
use admin
db.auth({user: 'root', pwd: 'password'})
```

La console retourne alors `1`, ce qui signifie que l'authentification s'est bien déroulée.

Nous créons ensuite l'utilisateur qui sera utilisé pour la connexion de l'*API* à la base de données :

```sql
db.createUser({user: 'jean', pwd: '123', roles:[{role: 'readWrite', db: 'test'}]})
```

Nous passons sur la base de données *test* :

```sql
use test
```

Nous insérons un document :

```sql
db.count.insertOne({count: 0});
```

Notre base de données est maintenant prête !

Nous pouvons maintenant quitter :

```sh
sudo docker compose -f docker-compose.prod.yml down
```

### Lancement de l'application

Nous n'avons plus qu'à lancer notre application :

```sh
sudo MONGO_USERNAME=jean MONGO_PWD=123 docker compose -f docker-compose.prod.yml up -d --build
```

Nous vérifions que tout est bien lancé :

```sh
sudo docker compose -f docker-compose.prod.yml logs -f
```

## Renouvellement automatique des certificats TLS

Un certificat *TLS* de *letsencrypt* est valable pendant 90 jours. Il faut donc reprouver que vous êtes bien propriétaire du domaine tous les 90 jours.

Pour ce faire, il faut taper une commande, mais si l'on oublie l'application ne fonctionnera plus car le certificat *TLS* ne sera plus valide et les navigateurs refuseront la connexion en *HTTPS* au site.

Nous allons donc automatiser la commande avec une tâche *cron*.

### Tester la commande de renouvellement

Avant de créer la tâche *cron*, nous allons vérifier que la commande de renouvellement fonctionne bien :

```sh
sudo certbot --standalone renew --force-renewal --pre-hook "/snap/bin/docker compose -f ~/docker-production/docker-compose.prod.yml stop reverseproxy" --post-hook "/snap/bin/docker compose -f ~/docker-production/docker-compose.prod.yml restart reverseproxy"
```

Il faut bien sûr adapter le dossier de votre projet */docker-production*.

Et vérifier où est installé *Docker Compose* en faisant `which docker-compose`.

### Ajouter la tâche *cron*

Nous ajoutons la tâche *cron* :

```sh
crontab -e
```

Nous plaçons :

```sh
0 0 * * * certbot --standalone renew --pre-hook "/snap/bin/docker compose -f ~/docker-production/docker-compose.prod.yml stop reverseproxy" --post-hook "/snap/bin/docker compose -f ~/docker-production/docker-compose.prod.yml restart reverseproxy"
```

Nous sauvegardons et nous quittons l'éditeur : le certificat sera renouvelé au bon moment par *certbot* automatiquement. Vous aurez un *downtime* de quelques secondes un jour par mois tous les 3 mois environ.

### Code de l'exemple

Vous pouvez également retrouver le code du projet sur [Github](https://github.com/dymafr/docker-chapitre11-server-prod). 