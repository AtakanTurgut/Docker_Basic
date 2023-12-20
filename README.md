## Docker Compose basic commands

#### Install the Docker Compose plugin
```cs
sudo apt-get update
sudo apt-get upgrade

sudo apt-get install \
> apt-transport-https \
> ca-certificates \
> curl \
> gnupg-agent \
> software-properties-common -y

sudo curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
docker-compose --help
```

-----
#### .yml
```yml
vi docker-compose.yml

    version: '2'
  	services:
  	    nginx-service:
  	     build: ./WebSite
  	     depends_on:
  	     - ruby-service
  	     
  	   ruby-service:
 	     build: ./WebApp
 	     depends_on:
 	     - redis-service
 	     
 	   redis-service:
 	     image: redis               ^C :wq
```
```cs
mkdir WebSite
mkdir WebApp

cd WebSite
vi Dockerfile
	 FROM nginx:latest
cd ..
cd WebApp
vi Dockerfile
	FROM ruby

cd ..
docker-compose up
```

-----

[labs.play-with-docker](https://labs.play-with-docker.com/)
