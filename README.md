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


[labs.play-with-docker](https://labs.play-with-docker.com/)