# 그림과 실습으로 배우는 도커 & 쿠버네티스

# ch05 docker network

## network, db, program

commands 
```
docker network create wordpress000net3

docker run --name mariadb000ex15 -dit --net=redmine000net3 -e MYSQL_ROOT_PASSWORD=myrootpass -e MYSQL_DATABASE=redmine000db -e MYSQL_USER=redmine000kun -e MYSQL_PASSWORD=rkunpass mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password

docker run -dit --name redmine000ex16 --network redmine000net3 -p 8087:3000 -e REDMINE_DB_MYSQL=mariadb000ex15 -e REDMINE_DB_DATABASE=redmine000db -e REDMINE_DB_USERNAME=redmine000kun -e REDMINE_DB_PASSWORD=rkunpass redmine
```

commands
```
docker network create wordpress000net4

docker run --name mariadb000ex17 -dit --net=wordpress000net4 -e MYSQL_ROOT_PASSWORD=mariarootpass -e MYSQL_DATABASE=wordpress000db -e MYSQL_USER=wordpress000kun -e MYSQL_PASSWORD=wkunpass mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --default-authentication-plugin=mysql_native_password

docker run --name wordpress000ex18  -dit --net=wordpress000net4 -p 8088:80 -e WORDPRESS_DB_HOST=mariadb000ex17 -e WORDPRESS_DB_NAME=wordpress000db -e WORDPRESS_DB_USER=wordpress000kun -e WORDPRESS_DB_PASSWORD=wkunpass wordpress
```

# ch06 docker volume

## docker cp

index.html
```
<html>
<meta charset="utf-8"/>
<body>
<div>hello</div>
</body>
</html>
```

commands
```
docker run --name apa000ex19 -d -p 8089:80 httpd

docker cp ch06/index.html apa000ex19:/usr/local/apache2/htdocs/

docker cp apa000ex19:/usr/local/apache2/htdocs/index.html ch06/index2.html
```

## volume mount vs bind mount

bind mount
```
md ch06/apa_folder

docker run --name apa000ex20 -d -p 8090:80 -v /Users/seongcho/42seoul/7Outer/docker-imagebook/ch06/apa_folder:/usr/local/apache2/htdocs httpd
```

volume mount
```
docker volume create apa000vol1
docker volume inspect apa000vol1
docker volume rm apa000vol1


docker volume create apa000vol1

docker run --name apa000ex21 -d -p 8091:80 -v apa000vol1:/usr/local/apache2/htdocs httpd

docker volume inspect apa000vol1

docker container inspect apa000ex21

docker cp ch06/index.html apa000ex21:/usr/local/apache2/htdocs/
```

rm volume
```
docker stop apa000ex21
docker rm apa000ex21
docker volume rm apa000vol1
```

backup volume
```
docker volume ls

docker run --rm -v apa000vol1:/source -v /Users/seongcho/42seoul/7Outer/docker-imagebook/ch06/backup_vol:/target busybox tar cvzf /target/backup_vol1.tar.gz -C /source .
```

restore volume
```
docker run --rm -v apa000vol1:/source -v /Users/seongcho/42seoul/7Outer/docker-imagebook/ch06/backup_vol:/target busybox tar xzvf /target/backup_vol1.tar.gz -C /source
```

## build an image

commit
```
docker run --name apa000ex22 -d -p 8092:80 httpd

docker commit apa000ex22 ex22_original1
```

ch06/apa_folder/Dockerfile
```
FROM httpd
COPY index.html /usr/local/apache2/htdocs

```

build
```
docker build -t ex22_original2 ch06/apa_folder

docker image ls
```

make it movable
```
docker save -o ex22.tar ex22_original2

ls -al
rm ex22.tar
```

## modify a container

```
docker exec -it apa000ex23 /bin/bash
echo "hello world"
apt install apache2
apt install mysql-server
exit
```

## hub.docker.com

docker registry
```
docker tag apa000ex22 zoozoo.coomm/nyapacchi:13
docker push zoozoo.coomm/nyapacchi:13
```

docker hub https://hub.docker.com



# ch07 docker compose

```
docker-compose --version
```

## how to compose

```
version: "3"		# 버전 기재

services:			# 컨테이너 관련 정보
  container_name1:
    image: httpd
    networks:
      - network_name1:
    ports:
      - 8080:80

  container_name2:

networks:			# 네트워크 관련 정보
  network_name_1:

volumes:			# 볼륨 관련 정보
  volume_name1:
  volume_name2:
```


ch07/docker-compose-test1.yml
```
version: "3"

services:
  
  # docker run --name apa000ex2 -d -p 8080:80 httpd
  apa000ex2:
    image: httpd
    ports:
      - 8080:80
    restart: always


  # docker run --name wordpress000ex12 -dit --net=wordpress000net1 -p 8085:80 
  #   -e WORDPRESS_DB_HOST=mysql000ex11 -e WORDPRESS_DB_NAME=wordpress000db 
  #   -e WORDPRESS_DB_USER=wordpress000kun -e WORDPRESS_DB_PASSWORD=wkunpass wordpress
  wordpress000ex12:
    depends_on:
      - mysql000ex11
    image: wordpress
    networks:
      - wordpress000net1
    ports:
      - 8085:80
    restart: always
    environment:
      WORDPRESS_DB_HOST=mysql000ex11
      WORDPRESS_DB_NAME=wordpress000db 
      WORDPRESS_DB_USER=wordpress000kun
      WORDPRESS_DB_PASSWORD=wkunpass 
```




## execute

```
docker-compose -f ch07/com_folder/docker-compose.yml up -d
docker-compose -f ch07/com_folder/docker-compose.yml down
```

```
cd ch07/com_folder

docker-compose up -d #--scale wordpress000ex12=3
docker-compose down
```



# ch08 kubernetes


## master node and worker node

master node + worker nodes = a cluster
kuberctl -> master node: kube-scheduler, etcd, ... -> worker node: kube-let, kube-proxy
kube-scheduler -> kube-let -> pod
클러스터 상태 유지: 컨테이너, 네트워크, 볼륨의 생성, 모니터링, 유지 + 로드 밸런서
쿠버네티스의 정의 파일 내용 -> etcd에 오브젝트로 등록 -> pod, service 인스턴스 생성

## 구성과 용어

resources:
	pod, service, deployment, replicaset, ...

pod: 컨테이너 + 볼륨
service: 
	pod에 서비스 요청을 배분하는 반장 (cluster ip)
	같은 종류의 pod를 하나의 service가 관리 
	내부 ip를 할당받은 여러개의 pod가 있어도 외부에서는 하나의 ip (cluster ip)로만 접근 가능
	여러 worker node 간의 분배는 실제 로드 밸런서 또는 ingress가 담당함 (별도의 node 또는 물리적 하드웨어)
	로드 밸런스 또는 ingress가 분배 -> service가 pod로 분배

ReplicaSet:
	pod의 수를 관리하는 반장
	동일한 구성의 pod: replica
	deployment와 함께 쓰임

deployment:
	pod의 deploy(배포)를 관리하는 요소: 개념적으로 ReplicatSet 상위 개념
	pod가 사용하는 image 등 pod info를 가지고 있음


## manifest

매니페스트 파일(.yml)은 리소스 단위로 작성 (```---```로 구분)

```
apiVersion: core/v1, apps/v1
kind:		Pod, Service, Deployment, ReplicaSet
metadata:	name, uid, labels, anotation
spec:		containers
```

ch08/kube_folder/apa000dep.yml
ch08/kube_folder/apa000ser.yml

## kubectl commands

```
kubectl create
kubectl delete
kubectl get
kubectl set
kubectl apply
kubectl scale
kubectl logs
```


deployment
```
kubectl apply -f ch08/kube_folder/apa000dep.yml
kubectl get pods
```

```
kubectl get pods
kubectl delete pod apa000dep-56858455dc-4hh2p
```

service
```
kubectl apply -f ch08/kube_folder/apa000ser.yml
kubectl get services
```

delete deployment
```
kubectl delete -f ch08/kube_folder/apa000dep.yml
kubectl get deployment
```

delete service
```
kubectl delete -f ch08/kube_folder/apa000ser.yml
kubectl get services
```


