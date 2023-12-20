## Docker basic commands 
kapsayıcı == container
```cs
    atakan@codebind:~$ sudo apt update;sudo apt upgrade
    atakan@codebind:~$ sudo apt install docker.io
    atakan@codebind:~$ sudo systemctl enable --now docker
    atakan@codebind:~$ docker --version
    atakan@codebind:~$ sudo docker run hello-world
    atakan@codebind:~$ docker ps -a		                // çalışan kapsayıcılar görünür
    atakan@codebind:~$ sudo docker run -it ubuntu bash
    root@212c24d1f5:/# apt-get update
```
```cs
    docker info
    docker ps	                                        // çalışan kapsayıcılar listesi
    docker ps -a	                                    // bütün kapsayıcılar
    docker run centos:7 echo "Hello World"
    docker run centos:7 ps -ef

    docker container ls                                 // çalışan kapsayıcılar listesi
    docker container ls -a                              // bütün kapsayıcılar
```
```cs
    docker container run -it centos:7 bash	            // yeni kapsayıcı ekler
    root@c70f44250d/j# yum -y update
    root@c70f44250d/j# ls -l
    root@c70f44250d/j# echo "I'm here" > test.txt
    root@c70f44250d/j# cat test.txt			            // test.txt'yi oku
    root@c70f44250d/j# exit                             // kapsayıcıdan çıkar

    docker container start containerId		            // kapsayıcıyı başlatır
    docker container exec containerId ps -ef	        // kapsayıcıya bağlanır
    docker container exec -it containerId bash

    docker container ls -a --no-trunc	                // kapsayıcıların detaylı gösterimi
    docker container ls -a -q		                    // kapsayıcıların sadece id'lerini gösterir
    docker container ls -l			                    // son oluşturulmuş kapsayıcı bilgisi verilir
    docker container ls -a --filter ""	                // kapsayıcıları aranan duruma göre filtreler

    docker container run centos:7 ping 127.0.0.1 -c 10	// kapsayıcı 10 ping çalışır kendisini kapatır
    docker container run -d centos:7 ping 127.0.0.1		// kapsayıcı çalışmaya devam eder
    docker container logs containerId			        // kapsayıcının çıktılarını verir
    docker container attach containerId			        // canlı olarak kapsayıcı takibi sağlar - çıkılırsa exit moda düşer
    docker container logs --tail 10 containerId		    // son kaç log görünecekse kullanılır
    docker container logs -f containerId			    // kapsayıcının canlı logları takip edilir
```
```cs
    docker container run -d tomcat		                // tomcat image çalışmaya başlar
    docker container stop containerId	                // kapsayıcıyı durdurur

    docker container start -a containerId	            // kapsayıcıya direkt attack olunur
    docker container start containerId	                // kapsayıcıyı arkada çalıştırır - çıkıldığı durumda çalışmaya devam eder
    docker container kill containerId	                // kapsayıcıyı komple durdurur
    docker container inspect containerId	            // kapsayıcı detayları
    docker container inspect containerId | grep IPAddress
    docker container stop containerId	                // kapsayıcı durdurulur
    docker container rm containerId		                // oluşturulan kapsayıcıyı kaldırır
---
    docker container run -d nginx		
    docker container run -d -p 5000:80 nginx	        // kapsayıcıya port ekler (hostport:containerport)
    docker container port containerId	                // kapsayıcı port bilgileri 	

    vi Dockerfile	                                    // docker file oluşturur
	    FROM nginx
	    EXPOSE 80		^C :wq

    docker image build -t my_nginx
    docker container run -d -p my_nginx

    docker plugin install grafana/loki-docker-driver	        // plugin ekler
    docker plugin ls					                        // plugin listeler
    docker plugin disable containerId			                // plugini disable eder
    docker plugin enable grafana/loki-docker-driver:latest		// plugini enable eder
    docker plugin inspect grafana/loki-docker-driver:latest		// pluginin detayları
    docker plugin rm grafana/loki-docker-driver:latest		    // plugin kaldırır
```
----------
## Dockerfile basic commands

```cs
docker pull nginx		            // image çekilir
docker container run nginx

docker search mysql                 // docker içerisinde arama yapmayı sağlar
docker pull mysql                   // repo'lardan image çekmemizi sağlar

docker login                        // docker'a login olmadan psuh işlemlerini yapamazsın
```

```cs
vi Dockerfile                       // dockerfile içerisine girilir

	FROM centos:7
	LABEL key value
	MAINTAINER AtakanTurgut
	RUN yum -y update
	RUN yum -y install nano
	CMD ping -c 10 127.0.0.1
	CMD ["/bin/echo"]
	EXPOSE 80/TCP
	ENV
	ADD /bin/xyz /bin/xyz
	COPY /bin/yzx bin/yzx
	ENTRYPOINT <>
	VOLUME /MOUNT
	USER
	WORKDIR /www/html	            ^C :wq
```

```cs
-- 1 --
vi Dockerfile

	FROM centos:7
	RUN yum -y update
	RUN yum -y install nano
	CMD ping -c 10 127.0.0.1
	CMD ["/bin/echo"]		        ^C :wq

docker image build --tag learn .		// dockerfile yapılandırılır
docker images

docker container run -it learn bash
```

```cs
-- 2 --
vi Dockerfile

	FROM ubuntu
	RUN apt-get -y update
	RUN apt-get -y install nano

docker image build -t by-ubuntu .
docker images

docker run --name my-ubuntu -p 80:80 -d by-ubuntu
----
vi Dockerfile

	FROM nginx
	RUN rm /etc/nginx/nginx.conf /etc/nginx/conf.d/default.conf
	COPY content /usr/share/nginx/html
	COPY conf /etc/nginx/

docker image build -t by-nginx .
```

```cs
vi Dockerfile

	FROM ubuntu
  	MAINTAINER atakanturgut
  	RUN apt-get -y update
  	RUN apt-get -y upgrade
  	RUN apt-get -y install nano
  	RUN apt-get -y install python3 python3-pip
  	COPY . /app
  	WORKDIR /app
  	EXPOSE 5000
 	ENTRYPOINT echo "Hello World!"

docker build -t py-ubuntu .
docker container run py-ubuntu
```

### Multi Stage

```cs
-- 1 -- Large App --
vi hello.c

	int main(void)
	{
		printf("Hello World!");
		return 0;
 	}

apk add gcc
gcc -Wall hello.c -o hello
./hello

vi Dockerfile

  	FROM alpine:3.5
  	RUN apk update && \
  	apk add --update alpine-sdk
  	RUN mkdir /app
  	WORKDIR /app
  	COPY hello.c /app
	RUN mkdir bin
  	RUN gcc -Wall hello.c -o bin/hello
	CMD /app/bin/hello

docker image build -t my-app-large .
docker images		                    // size!	184MB

-- 2 -- Small App --
vi Dockerfile

    FROM alpine:3.5 AS build
    RUN apk update
    RUN apk add --update alpine-sdk
    RUN mkdir /app
    WORKDIR /app
    COPY hello.c /app
    RUN mkdir bin
    RUN gcc -Wall hello.c -o bin/hello

    FROM alpine:3.5
    COPY --from=build /app/bin/hello /app/hello
    CMD /app/hello

cat Dockerfile
docker image build -t my-app-small .
docker images		                    // size!	4.01MB

docker image rm imageName	            // image'yi kaldırır
docker image rm *		                // image'lerin hepsini kaldırır
```
[dockerhub](https://hub.docker.com/repository/docker/atakanturgut/bydockerlearnpublic/general)
### Pull - Push

```cs
-- 1 --
// Repository push yapacaksak Login olunması lazım
docker login
docker ps -a
```

```cs
!!!
docker push atakanturgut/repoName:imageName
	The push refers to repository [docker.io/atakanturgut/bydockerlearnpublic]
	An image does not exist locally with the tag: atakanturgut/bydockerlearnpublic

docker tag imageId atakanturgut/bydockerlearnpublic:imageName
docker push atakanturgut/repoName:imageName
```

```cs
docker tag 55c3c278336e atakanturgut/bydockerlearnpublic:my-app-large
docker push atakanturgut/bydockerlearnpublic:my-app-large
	The push refers to repository [docker.io/atakanturgut/bydockerlearnpublic]
	327490b9bb21: Pushed
	c6fd5891d749: Pushed
```

```cs
-- 2 --
vi Dockerfile

	FROM ubuntu
  	RUN apt-get -y update
  	RUN apt-get -y upgrade

docker image build -t atakanturgut/bydockerlearnpublic:ubuntu-repo .
docker push atakanturgut/bydockerlearnpublic:ubuntu-repo		// repoya yolla

docker pull atakanturgut/bydockerlearnpublic:ubuntu-repo		// repoydan çek
```

### Docker Registry - Local Repository

```cs
docker pull registery
docker images

mkdir /var/lib/docker/registry
docker run -d -p 5000:5000 -v /var/lib/docker/registry/:/var/lib/registry registry

vi /etc/docker/daemon.jso

  {
      "experimental": true,
      "debug": true,
      "log-level": "info",
      "insecure-registries": [
          "127.0.0.1:5000"          @*<--*@
      ],
      "hosts": [
          "unix:///var/run/docker.sock",
          "tcp://0.0.0.0:2375"
      ],
      "tls": false,
      "tlscacert": "",
      "tlscert": "",
     "tlskey": ""
 }

docker tag nginx 127.0.0.1:5000/nginx:my-registry
docker push 127.0.0.1:5000/nginx:my-registry
```
```cs
[node1] (local) root@192.168.0.8 ~
$ cd /var/lib/docker/registry
[node1] (local) root@192.168.0.8 /var/lib/docker/registry
$ ls
docker
[node1] (local) root@192.168.0.8 /var/lib/docker/registry
$ cd docker/
[node1] (local) root@192.168.0.8 /var/lib/docker/registry/docker
$ cd registry/
[node1] (local) root@192.168.0.8 /var/lib/docker/registry/docker/registry
$ cd v2/
[node1] (local) root@192.168.0.8 /var/lib/docker/registry/docker/registry/v2
$ ls
blobs         repositories
[node1] (local) root@192.168.0.8 /var/lib/docker/registry/docker/registry/v2
$ cd repositories/
[node1] (local) root@192.168.0.8 /var/lib/docker/registry/docker/registry/v2/repositories
$ ls
nginx
```
----------
## Docker Volume basic commands

```cs
docker volume create --name volume1		// volume oluşturur
```

```cs
[node1] (local) root@192.168.0.13 /
$ cd /var/lib/dockervolumes/
[node1] (local) root@192.168.0.13 /var/lib/docker/volumes
$ ls
backingFsBlockDev  metadata.db        volume1

docker container run -it -v volume1:/www/website centos:7 bash

[node1] (local) root@192.168.0.13 /var/lib/docker/volumes
cd /volume1/_data
[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data
$ ls
volume1.txt
[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data
$ cat volume1.txt 
hello there

[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data
$ docker container run --name volume1  -it -v volume1:/www/website centos:7 bash
[root@14b90764ff87 /]# cd /www/website/
[root@14b90764ff87 website]# ls
volume1.txt
[root@14b90764ff87 website]# cat volume1.txt 
hello there
```

```cs
docker volume inspect volume1		//== docker info  -> volumeName detayları gösterir

[
    {
        "CreatedAt": "2023-12-19T19:21:54Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/volume1/_data",
        "Name": "volume1",
        "Options": null,
        "Scope": "local"
    }
]
```

```cs
[node1] (local) root@192.168.0.13 /
$ cd var/lib/docker/volumes/volume1/_data/
[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data
$ mkdir attest
[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data
$ cd attest/

[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data/attest
$ echo 'attest there' > attest.txt 
[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data/attest
$ cd ..

[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/volume1/_data
$ docker container run -it -v volume1:/www/html centos:7 bash

[root@73f877880c56 /]# cd www/html/
[root@73f877880c56 html]# ls
attest  volume1.txt
[root@73f877880c56 html]# cd attest
[root@73f877880c56 attest]# cat attest.txt 
attest there
```

### ReadOnly Volume

```cs
docker volume create nginx_logs

docker container run -it -v nginx_logs:/data/mylogs:ro

[node1] (local) root@192.168.0.13 /
$ docker container run -it -v nginx_logs:/data/mylogs:ro centos:7 bash

[root@e82c98e3551f /]# cd /data/mylogs/
[root@e82c98e3551f mylogs]# touch test.txt
	touch: cannot touch 'test.txt': Read-only file system

[node1] (local) root@192.168.0.13 /
$ cd var/lib/docker/volumes/nginx_logs/_data/

[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/nginx_logs/_data
$ ls
[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/nginx_logs/_data
$ touch test.txt

[node1] (local) root@192.168.0.13 /var/lib/docker/volumes/nginx_logs/_data
$ vi test.txt

	rw
	ro		^C :wq

---
docker container exec -it containerId bash
cd /data/mylogs/
cat test.txt

---
	^ctrl + p + q	            // exit demeden arka planda kapsayıcıyı açık bırakarak çıkış yapılabilir
```
```cs
docker volume ls

docker container run -d -P --name nginx_server -v ~/public_html:/usr/share/nginx/html -v nginx_logs:/var/log/nginx nginx
docker container exec -it nginx_server bash

root@4e603363e490:/# cd var/log/nginx/ 
root@4e603363e490:/var/log/nginx# ls
access.log  error.log  test.txt  text.txt

root@4e603363e490:/var/log/nginx#  
	tail -f access.log	                // log kayıtlarını görebiliriz
	tail -f error.log

tail -f /var/lib/docker/volumes/nginx_logs/_data/access.log
tail -f /var/lib/docker/volumes/nginx_logs/_data/error.log
```

### Volume operation with Dockerfile

```cs
vi Dockerfile
    FROM ubuntu
    RUN apt-get -y update
    RUN apt-get -y upgrade
    RUN apt-get -y install nano
    RUN mkdir /data
    WORKDIR /data
    RUN echo "Dockerfile example volume." > test.txt
    VOLUME /data					^C :wq

docker build -t atakan:1 .
docker volume create image
```
```cs
[node1] (local) root@192.168.0.8 ~
$ docker container run -v image:/data -it atakan:1 bash
root@7cc683429625:/data# ls
test.txt
root@7cc683429625:/data# cat test.txt
Dockerfile example volume.
root@7cc683429625:/data# exit

docker volume inspect image

[
    {
        "CreatedAt": "2023-12-19T21:02:35Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/image/_data",
        "Name": "image",
        "Options": null,
        "Scope": "local"
    }
]
```
```cs
[node1] (local) root@192.168.0.8 ~
$ cd /var/lib/docker/volumes/image/_data/
[node1] (local) root@192.168.0.8 /var/lib/docker/volumes/image/_data
$ ls
test.txt
[node1] (local) root@192.168.0.8 /var/lib/docker/volumes/image/_data
$ cat test.txt 
Dockerfile example volume.
```

### Docker Volume External
```cs
docker volume create --opt type=nfs --opt o=addr=192.168.1.10,rw,nfsserver4 --opt device=:/home/nfsshare nfs-volume
```
```cs
[node1] (local) root@192.168.0.8 /
$ docker volume ls 
DRIVER    VOLUME NAME
local     image
local     nfs-volume

[node1] (local) root@192.168.0.8 /
$ docker volume inspect nfs-volume 

[
    {
        "CreatedAt": "2023-12-19T21:14:20Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/nfs-volume/_data",
        "Name": "nfs-volume",
        "Options": {					@*<--*@
            "device": ":/home/nfsshare",
            "o": "addr=192.168.1.10,rw,nfsserver4",
            "type": "nfs"
        },
        "Scope": "local"
    }
]

[node1] (local) root@192.168.0.8 /
$ docker volume inspect image

[
    {
        "CreatedAt": "2023-12-19T21:02:35Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/image/_data",
        "Name": "image",
        "Options": null,
        "Scope": "local"
    }
]

docker run -it -v nfs-volume:/nfsshare centos
```

### Docker Volume Delete
```cs
// volume'de herhangi bir kapsayıcı bulunuyorsa bu kapsayıcı muhakkak ki silinmelidir
// aksi halde volume silinmez - kapsayıcının aktif ya da pasif olması farketmez

docker volume rm imageName
	Error response from daemon: remove image: volume is in use - [7cc683429625dcb4c581d527b2bc606d2afa1624cc637cbc68a66f2fb561d6ca]
```
```cs
[node1] (local) root@192.168.0.8 /
$ docker container run -it -v image:/data centos bash
[root@58a60f2f4f72 /]# 
                        ^ctrl + p + q


docker container stop containerId           // kapsayıcı durdurulsa da volume silinmez
docker container ls 
docker container rm containerId

docker volume rm imageName
docker volume ls
```
----------
## Docker Network basic commands

```cs
docker network ls			        // network listesi
docker network inspect bridge		// detayları gösterir
```
```cs
docker run centos /usr/sbin/ip route
	default	via
```

```cs
docker run -it centos bash
	ctrl + p + q
docker ps
docker network inspect bridge

docker network create testnetwork
docker network ls
docker network inspect testnetwork

docker container run -it --net testnetwork centos bash
	ctrl + p + q
docker network inspect testnetwork

docker container run -it --net testnetwork centos bash
	ctrl + p + q
docker network inspect testnetwork

docker container exec -it containerId bash
[root@6cb3eeae8fc2 /]# ping 172.19.0.3
	PING 172.19.0.3 (172.19.0.3) 56(84) bytes of data.
	64 bytes from 172.19.0.3: icmp_seq=1 ttl=64 time=0.191 ms
```
```cs
docker network create --subnet 192.168.100.0/24 --gateway 192.168.100.1 test2network
docker network ls
docker network inspect test2network
docker container run --net test2network -it centos bash

[root@5ae600eae010 /]# ctrl + p + q
docker network inspect test2network
```
```
default:
docker network create --driver null test3network
docker network create --driver host test3network
```
```cs
docker container run --name network -it -d centos
docker container run --name network-2 -it -d centos
docker container inspect network

docker container run --name network-0 --network=testnetwork -it -d centos
docker container run --name network-1 --network=testnetwork -it -d centos
docker container inspect network-1

docker container run --name network2-1 --network=test2network -it -d centos
docker container run --name network2-2 --network=test2network -it -d centos
docker container inspect network2-2
```

```cs
docker ps
docker container exec -it network bash
[root@cbb233252119 /]# ping 192.168.100.2
	PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.	// iletişim kuramaz

docker container exec -it network2-1 bash
[root@67f3bb12d6e4 /]# ping 192.168.100.3
	PING 192.168.100.3 (192.168.100.3) 56(84) bytes of data.	// iletişim olur
	64 bytes from 192.168.100.3: icmp_seq=1 ttl=64 time=0.069 ms
```

### Docker Network Delete
```cs
// network'de herhangi bir kapsayıcı bulunuyorsa bu kapsayıcı muhakkak ki silinmelidir
// aksi taktirde network silinmez - kapsayıcının aktif ya da pasif olması farketmez

docker network rm testnetwork
	Error response from daemon: error while removing network: network testnetwork id 8327fd09d09eb1c5e20efe67df4c72c61c714d04e085f934cdc3ed187cd198b0 has active endpoints

docker network inspect testnetwork
```

```cs
docker network create testatnetwork

docker network prune 					            // kullanılmayan networkler'i ortadan kaldırır
	WARNING! This will remove all custom networks not used by at least one container.
	Are you sure you want to continue? [y/N] y
	Deleted Networks:
	testatnetwork

docker network ls
```

### Docker Network Mac Vlan
```cs
docker network create -d macvlan --subnet=192.168.0.0/24 --gateway=192.168.0.1 -o parent=eth0 macvlan-mynet
docker network ls
docker network inspect macvlan-mynet 
```
----------
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
----------

[labs.play-with-docker](https://labs.play-with-docker.com/)