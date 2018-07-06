# WordPressDockerAWS

amazon AMI : 35.180.98.45

## Lancement putty winscp

Avec puttygen load private key "*" et save private key

User : ec2-user

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

```
export PATH=$PATH:/usr/local/bin/
```

## Creer docker-compose.yml

```
version: '3.3'

services:
  db:
    image: mysql:5
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress

  traefik:
    image: traefik
    restart: always
    command: --api --docker # Enables the web UI and tells Træfik to listen to docker
    ports:
      - '443:443'
      - '8080:8080'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik:/etc/traefik
volumes:
  db_data:
```

## Ouvrir les port TCP dans aws

## Traefik

https://docs.traefik.io/#the-trfik-quickstart-using-docker
