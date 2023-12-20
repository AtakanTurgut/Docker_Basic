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
### .yml
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
```cs
mkdir docker-compose/build
cd docker-compose/build/
```
```yml
vi Dockerfile

  FROM nginx:alpine
  RUN echo "Welcome to Docker Wrokshop!" > /usr/share/nginx/html/index.html
  CMD ["nginx", "-g", "daemon off;"]

vi docker-compose.yml

  version: "3.7"
  services:
    webapp:
      build:
        context: .
        dockerfile: Dockerfile
      image: webapp:v1
```
```cs
docker-compose build
docker image ls
```
-----
```yml
vi docker-compose.yml

  version: "3"
  services:
    elasticsearch:
      image: docker.elastic.co/elasticsearch/elasticsearch:7.0.0
      environment:
        - discovery.type=single-node
      ports:
        - 9200:9200
  
    kibana:
      image: docker.elastic.co/kibana/kibana:7.0.0
      ports:
        - 5601:5601    

docker-compose up
```

[labs.play-with-docker](https://labs.play-with-docker.com/)
