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

docker-compose start db 	    // SERVICE Name
docker-compose start webserver
```
-----
```cs
-- 5 --
mkdir -p wordpress-compose 
touch docker-compose.yml

mkdir -p nginx/
mkdir -p db-data/
mkdir -p logs/nginx/
mkdir -p wordpress/
```
```cs
vim nginx/wordpress.conf

  server {
    listen 80;
    server_name localhost;
    root /var/www/html;
    index index.php;
   
    access_log /var/log/nginx/localhost-access.log;
    error_log /var/log/nginx/localhost-error.log;
   
    location /{
      try_files $uri $uri/ /index.php?$args;
    }
   
    location ~ \.php$ {
      try_files $uri = 404;
      fastcgi_split_path_info ^(.+\.php)(/.+)$;
      fastcgi_pass wordpress:9000;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PATH_INFO $fastcgi_path_info;
    }
  }
```
```yml
vim docker-compose.yml

  version: '2.6'
  services:
    nginx:
      container_name: webserver
      image: nginx:latest
      ports:
        - '80:80'
      volumes:
        - ./nginx:/etc/nginx/conf.d
        - ./logs/nginx:/var/log/nginx
        - ./wordpress:/var/www/html
      links:
        - wordpress
      restart: always
    mysql:
      container_name: dbserver
      image: mariadb
      ports:
        - '3306:3306'
      volumes:
        - ./db-data:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=aqwe123
      restart: always
    wordpress:
      container_name: wpserver
      image: wordpress:4.7.1-php7.0-fpm
      ports:
        - '9000:9000'
      volumes:
        - ./wordpress:/var/www/html
      environment:
        - WORDPRESS_DB_NAME=wpdb
        - WORDPRESS_TABLE_PREFIX=wp_
        - WORDPRESS_DB_HOST=mysql
        - WORDPRESS_DB_PASSWORD=aqwe123
      links:
        - mysql
      restart: always
```
```cs
cd ~/wordpress-compose/

docker-compose up -d
docker-compose ps
docker-compose logs nginx       // SERVICE Name
docker-compose logs mysql
docker-compose logs wordpress

netstat -plntu	                // hareketli portlar kontrol edilir

docker-compose ls
docker container ls
```

### Docker Scale
```cs
docker-compose up -d --scale mysql=1	    // docker'da scale mimarisi oluşturur
docker-compose ps
docker-compose logs mysql

// scale edilmeden önce kapsayıcı host aralığı belirtilmelidir  !!!
docker-compose up -d --scale nginx=3	    // 3 replika sayısı
```
```yml
vim docker-compose.yml

  version: '2.6'
  services:
    nginx:
      container_name: webserver
      image: nginx:latest
      ports:
        - '85-87:80'				    # 85 86 87 - 3 replika ^
      volumes:
        - ./nginx:/etc/nginx/conf.d
        - ./logs/nginx:/var/log/nginx
        - ./wordpress:/var/www/html
      links:
        - wordpress
      restart: always
    mysql:
      container_name: dbserver
      image: mariadb
      ports:
        - '3310-3312:3306'			    # 10 11 12 - 3 replika ^
      volumes:
        - ./db-data:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=aqwe123
      restart: always
    wordpress:
      container_name: wpserver
      image: wordpress:4.7.1-php7.0-fpm
      ports:
        - '9010-9012:9000'			    # 10 11 12 - 3 replika ^
      volumes:
        - ./wordpress:/var/www/html
      environment:
        - WORDPRESS_DB_NAME=wpdb
        - WORDPRESS_TABLE_PREFIX=wp_
        - WORDPRESS_DB_HOST=mysql
        - WORDPRESS_DB_PASSWORD=aqwe123
      links:
        - mysql
      restart: always
```
```cs
docker-compose up -d
docker-compose up -d --scale mysql=3
// belirtilen aralıktan fazla sayıda replika oluşturulamaz
docker-compose up -d --scale nginx=3

// bir scale işleminden sonra tekrar scale yapılırsa önceki replikalar sıfırlanır
// bu durumda birbiriyle entegre sistemlerde aynı anda scale edilmelidir
docker-compose up -d --scale nginx=3 --scale mysql=3 --scale wordpress=3
```
```yml
# .yml içerisinde scale işlemi gerçekleştirmek istersek    !!!
vim docker-compose.yml

  version: '2.6'
  services:
    nginx:
      container_name: webserver
      image: nginx:latest
      ports:
        - '85-87:80'				
      volumes:
        - ./nginx:/etc/nginx/conf.d
        - ./logs/nginx:/var/log/nginx
        - ./wordpress:/var/www/html
      links:
        - wordpress
      restart: always
      deploy: 
        replicas: 3
    mysql:
      container_name: dbserver
      image: mariadb
      ports:
        - '3310-3312:3306'			
      volumes:
        - ./db-data:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=aqwe123
      restart: always
      deploy: 
        replicas: 3
    wordpress:
      container_name: wpserver
      image: wordpress:4.7.1-php7.0-fpm
      ports:
        - '9010-9012:9000'			
      volumes:
        - ./wordpress:/var/www/html
      environment:
        - WORDPRESS_DB_NAME=wpdb
        - WORDPRESS_TABLE_PREFIX=wp_
        - WORDPRESS_DB_HOST=mysql
        - WORDPRESS_DB_PASSWORD=aqwe123
      links:
        - mysql
      restart: always
      deploy: 
        replicas: 3
```
```cs
docker-compose up -d
docker-compose ps
```

[labs.play-with-docker](https://labs.play-with-docker.com/)
