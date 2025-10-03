# Getting started

This repository is a sample application for users following the getting started guide at https://docs.docker.com/get-started/.

The application is based on the application from the getting started tutorial at https://github.com/docker/getting-started

COMMANDS

docker build -t docker-tut-1 .

docker images

docker run -d -p 3000:3000 docker-tut-1
docker run -d -p 127.0.0.1:3000:3000 docker-tut-1
docker run -dp 127.0.0.1:3000:3000 docker-tut-1

docker ps
docker ps -a

docker stop <CONTAINER-ID>
docker rm <CONTAINER-ID>
docker rm -f <CONTAINER-ID>


VOLUMES
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos docker-tut-1

BIND MOUNTS
docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash

- changes in code updates
docker run -dp 127.0.0.1:3000:3000 
    -w /app --mount type=bind,src="$(pwd)",target=/app 
    node:lts-alpine 
    sh -c "yarn install && yarn run dev"

MULTI-CONTAINER APP - WITH SQL
1. START SQL
docker run -d 
    --network todo-app --network-alias mysql
    -v todo-mysql-data:/var/lib/mysql
    -e MYSQL_ROOT_PASSWORD=secret
    -e MYSQL_DATABASE=todos
    mysql:8.0

docker exec -it <mysql-container-id> mysql -u root -p

2. CONNECT TO MYSQL
docker run -it --network todo-app nicolaka/netshoot
dig mysql

OUTPUT:
; <<>> DiG 9.20.10 <<>> mysql
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1465
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mysql.                         IN      A

;; ANSWER SECTION:
mysql.                  600     IN      A       172.18.0.2

;; Query time: 12 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Fri Oct 03 17:31:11 UTC 2025
;; MSG SIZE  rcvd: 44

EXPECTED OUTPUT:
; <<>> DiG 9.18.8 <<>> mysql
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mysql.				IN	A

;; ANSWER SECTION:
mysql.			600	IN	A	172.23.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Tue Oct 01 23:47:24 UTC 2019
;; MSG SIZE  rcvd: 44


RUN APP WITH MYSQL
docker run -dp 127.0.0.1:3000:3000
  -w /app -v "$(pwd):/app"
  --network todo-app
  -e MYSQL_HOST=mysql
  -e MYSQL_USER=root
  -e MYSQL_PASSWORD=secret
  -e MYSQL_DB=todos
  node:lts-alpine
  sh -c "yarn install && yarn run dev"

INSPECT CONTAINER LOGS (add "-f") 
EXPECTED OUTPUT:

nodemon src/index.js
[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] starting `node src/index.js`
Connected to mysql db at host mysql
Listening on port 3000 

INSPECT MYSQL DB
docker exec -it <mysql-container-id> mysql -p todos

mysql> select * from todo_items;

EXPECTED OUTPUT:
mysql> select * from todo_items;
+--------------------------------------+--------------------+-----------+
| id                                   | name               | completed |
+--------------------------------------+--------------------+-----------+
| c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
| 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
+--------------------------------------+--------------------+-----------+