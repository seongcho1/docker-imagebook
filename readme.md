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

## 






# ch08 kubernetes



