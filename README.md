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

[labs.play-with-docker](https://labs.play-with-docker.com/)
