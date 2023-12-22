## Docker Swarm basic commands
```cs
// 4 tane instance mevcut
docker swarm init
	Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces (192.168.0.28 on eth0 and 172.18.0.17 on eth1) - specify one with --advertise-addr

// instance'lerden bir tanesinin managmant olarak belirlenmesi gerekir.
docker swarm init --advertise-addr 192.168.0.28
	docker swarm join --token SWMTKN-1-5ai0znmsmd3qncwthsu20u4ir8psfpbueqjaysv6kxj1jpn0e6x-7604dledf79axvsu8yx03xyha 192.168.0.28:2377

docker info
docker swarm
```
```cs
docker swarm join-token manager
	docker swarm join --token SWMTKN-1-5ai0znmsmd3qncwthsu20u4idr8pspbueqjaysv6kxj1jpn0e6x-ex04ydu92o58w9hk6hvl7ameq 192.168.0.28:2377
	// node'ler manager olarak eklenebilir
docker swarm join-token worker
	docker swarm join --token SWMTKN-1-5ai0znmsmd3qncwtdhsu20u4ir8pspbueqjaysv6kxj1jpn0e6x-7604dledf79axvsu8yx03xyha 192.168.0.28:2377
	// node'ler worker olarak eklenebilir

docker swarm leave	            // mevcut ortam kaldırılır

docker node ls		              // node listesi

docker node promote to node2	  // workerları manager olarak eklemek için kullanılır	- Reachable

docker network ls 		          // ingress - swarm tarafından kullanılır
docker network inspect ingress	// ortama ekli bütün worker ve managerlerin ip bilgileri
```

### Swarm Service Task
```cs
docker service create --name WebService --publish 8080:80 --replicas 4 nginx

docker service scale WebService=6

docker service ls
docker service ps WebService
docker service update --replicas 4 WebService			  // service güncelleme
```
```cs
docker service create --name service --constraint "node.hostname==node2" --replicas 4 nginx
  // host adresini belirlemek için
docker service ps service 

docker service create --name servicenot --constraint "node.hostname!=node2" --replicas 4 nginx
  // host adresinde service çalıştırılmaz
docker service ps servicenot 

docker service ls
```
-----------
## Docker Swarm Tools
#### [Docker Swarm Visualizer](https://github.com/dockersamples/docker-swarm-visualizer)
```cs
docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer

docker service ls
docker service ps viz

> 8080 port click
```
#### [Docker Swarm Portainer Web UI](https://docs.portainer.io/start/install-ce/server/swarm/linux)
```cs
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
---
curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml

vi portainer-agent-stack.yml 
docker stack deploy --compose-file=portainer-agent-stack.yml portainer

> 9000 port click
  username: admin
  passwor : 1-9
```
#### [Docker Swarm Sematext Monitoring](https://sematext.com/container-monitoring/)
```cs
docker pull sematext/sematext-agent-docker
> //../apps.eu.sematext.com/ui/..

docker ps
```
------------
### Docker Swarm Stack
```yml
vi docker_stack.yml 

  version: "3"
  services:
    redis:
      image: redis:alpine
      networks:
        - frontend
      deploy:
        replicas: 1
        update_config:
          parallelism: 2
          delay: 10s
        restart_policy:
          condition: on-failure
    db:
      image: postgres:9.4
      environment:
        POSTGRES_USER: "postgres"
        POSTGRES_PASSWORD: "postgres"
      volumes:
        - db-data:/var/lib/postgresql/data
      networks:
        -backend
      deploy:
        placement:
          constraints: [node.role == manager]
    vote:
      image: dockersamples/examplevotingapp_vote:before
      ports:
        - 5000:80
      networks:
        - frontend
      depends_on:
        - redis
      deploy:
        replicas: 2
        update_config:
          paralellism: 2
        restart_policy:
          condition: on-failure
    result:
      image: dockersamples/examlevotingapp_result:before
      ports:
        - 5001:80
      networks:
        - backend
      depends_on:
        - db
      deploy:
        replicas: 1
        update_config:
          paralellism: 2
          delay: 10s
        restart_policy:
          condition: on-failure
    worker:
      image: dockersamples/examplevotingapp_worker
      networks:
        - frontend
        - backend
      depends_on:
        - db
        - redis
      deploy:
        mode: replicated
        replicas: 1
        labels: [APP=VOTING]
        restart_policy:
          condition: on-failure
          delay: 10s
          max_attempts: 3
          window: 120s
        placement:
          constraints: [node.role == manager]
    visualizer:
      image: dockersamples/visualizer:stable
      ports:
        - "8080:8080"
      stop_grace_period: 1m30s
      volumes:
        - "/var/run/docker.sock:/var/run/docker.sock"
      deploy:
        placement:
          constraints: [node.role == manager]
  networks:
    frontend:
    backend:
  volumes:
    db-data:        ^C :wq
```
```cs
docker stack deploy --compose-file docker_stack.yml voteapplication

docker stack ls
docker stack ps voteapplication 				
docker stack services voteapplication 				// service'ler
```

### Docker Swarm Secret
```
```

[labs.play-with-docker](https://labs.play-with-docker.com/)
