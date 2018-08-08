# WordPressDockerAWS

amazon AMI : 35.180.98.45

## Lancement putty winscp

Convertir la clé pem vers ppk : https://my.bluehost.com/hosting/help/putty. Résumé : 
 - ouvrir puttygen.
 - load private key, dans la fenetre de recherche selectionner `*` comme extension de fichier (à la place de .ppk).  
 - save private key.

Dans putty : 
 - Connection -> SSH -> Auth : ajouter la clé ppk.
 - Connection -> Data : ajouter "ec2-user" dans le champs username.

User : ec2-user

Commande pour passer root : sudo bash.

## Installer docker et docker-compose

```
sudo yum update
sudo yum install -y docker
sudo usermod -a -G docker ec2-user
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo service docker start
sudo chkconfig docker on
```
### Mettre docker-compose comme executable partout

https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-linux.html

Solution temporaire :
```
export PATH=$PATH:/usr/local/bin/
```

## Fabriquer les dockers de A à Z

Choisir une de ces 3 versions :

### Version http

Créer le fichier `docker-compose.yml` :

```
version: '3.3'

services:
  db:
    image: mysql:5.7.22
    volumes:
      - ./db-data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ChangeMeRoot
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: ChangeMe

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "80:80"
    restart: always
    volumes:
      - ./wp-content:/var/www/html/wp-content
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ChangeMe
```

### Version https caddy

Passer en https nécessite obligatoirement de disposer d'un nom de domaine.
Utiliser une IP ne fonctionnera pas.

Créer le fichier `docker-compose.yml` :

```
version: '3.3'

services:
  db:
    image: mysql:5.7.22
    volumes:
      - ./db-data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ChangeMeRoot
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: ChangeMe

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    expose:
      - "80"
    restart: always
    volumes:
      - ./wp-content:/var/www/html/wp-content
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ChangeMe

  proxy-https:
    image: abiosoft/caddy
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./Caddyfile:/etc/Caddyfile
      - ./caddy-data:/root/.caddy
    environment:
      - ACME_AGREE=true
    networks:
      - web
    logging:
      driver: json-file
      options:
        max-size: 10m
        max-file: "3"
```

Créer le fichier `Caddyfile` et éditer l'email et le nom de domaine :

```
mon.site.com {
  tls mon@email.com

#  nom et port du container à recuperer
  proxy / wordpress:80 {
    transparent
  }
}
```

### Version https traefik (plus compliqué que caddy)

Créer le fichier `docker-compose.yml` et éditer le nom de domaine du site :

```
version: '3.3'

services:
  db:
    image: mysql:5.7.22
    volumes:
      - ./db-data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ChangeMeRoot
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: ChangeMe

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    expose:
      - "80"
    labels:
      - "traefik.frontend.rule=Host:mon.site.fr"    # A personnaliser, renommage de route sur nom de domaine
      - "traefik.enable=true"                       # activation de traefik sur ce container
    restart: always
    volumes:
      - ./wp-content:/var/www/html/wp-content
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ChangeMe

  proxy-https:
    image: traefik
    restart: always
    command: --api
    ports:
      - 80:80
      - 443:443
      - 9090:8080
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.toml:/traefik.toml
      - ./traefik-data/acme.json:/acme.json
```

Créer le fichier `traefik.toml`, éditer le nom de domaine du site et l'adresse email :

```
debug = true

logLevel = "ERROR"
defaultEntryPoints = ["https","http"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
    entryPoint = "https"
  [entryPoints.https]
  address = ":443"
  [entryPoints.https.tls]

[retry]

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "mon.site.fr"     # A personnaliser
watch = true
exposedByDefault = false

[acme]
email = "mon@email.fr"     # A personnaliser
storage = "acme.json"
entryPoint = "https"
onHostRule = true
[acme.httpChallenge]
entryPoint = "http"
```

Executer ces lignes de commande depuis la racine du fichier docker-compose.yml (traefik requiert des droits limités sur le fichier acme.json) :

```
mkdir traefik-data
touch traefik-data/acme.json
chmod 600 traefik-data/acme.json
```

## Ouvrir les port TCP dans AWS

## Lancer les containers docker

```
docker-compose up
```

sources : https://docs.traefik.io/#the-trfik-quickstart-using-docker
