# WordPressDockerAWS

amazon AMI

## Lancement putty winscp

Avec puttygen load private key "*" et save private key

User : ec2-user

## Installer docker et docker-compose

```
sudo yum update
sudo yum install -y docker
sudo usermod -a -G docker ec2-user
sudo curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-`uname -s`-`uname -m` | sudo tee /usr/local/bin/docker-compose > /dev/null
sudo chmod +x /usr/local/bin/docker-compose
sudo service docker start
sudo chkconfig docker on
```
### Mettre docker-compose comme executable partout

```
export PATH=$PATH:/usr/local/bin/
```

## Ouvrir les port TCP dans aws
