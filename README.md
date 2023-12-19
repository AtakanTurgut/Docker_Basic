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
docker volume inspect volume1		//== docker info  -> volumeName

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
```

```cs
	ctrl + p + q	// exit demeden arka planda kapsayıcıyı açık bırakarak çıkış yapılabilir
```
[labs.play-with-docker](https://labs.play-with-docker.com/)
