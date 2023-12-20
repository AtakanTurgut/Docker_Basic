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

[labs.play-with-docker](https://labs.play-with-docker.com/)
