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
docker volume inspect volume1		//== docker info  -> volumeName deyaları gösterir

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

[labs.play-with-docker](https://labs.play-with-docker.com/)
