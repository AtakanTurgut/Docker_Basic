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

### .yml
```yml
-- 1 --
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
-- 2 --
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
-- 3 --
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
-----
```cs
-- 4 --
mkdir MyApp
cd MyApp

openssl rand -base64 32 > db_password.txt
openssl rand -base64 32 > db_root_password.txt
cat db_password.txt
```
```yml
vi docker-compose.yml

  version: '3.1'
  services:
    #Nginx Service
      webserver:
        image: nginx:alpine
        container_name: webserver
        restart: unless-stopped
        ports:
          - "80:80"
          - "443:443"
    #Mysql DB
      db:
        image: mysql:5.7
        container_name: Mysqldb
        restart: unless-stopped
        volumes:
          - db_data:/var/lib/mysql
        ports:
          - "3306:3306"
        environment:
          MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
          MYSQL_DATABASE: wordpress
          MYSQL_USER: wordpress
          MYSQL_PASSWORD_FILE: /run/secrets/db_password
        secrets:
          - db_root_password
          - db_password
  secrets:
    db_password:
      file: db_password.txt
    db_root_password:
      file: db_root_password.txt
  
  volumes:
    db_data:					^C :wq
```
```cs
docker-compose up
docker-compose ps -a
docker-compose logs

docker-compose start db 	// SERVICE Name
docker-compose start webserver
```

[labs.play-with-docker](https://labs.play-with-docker.com/)
