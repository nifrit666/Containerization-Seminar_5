## Урок 10. Семинар5. Docker Compose и Docker Swarm

Задание:
Задание 1:
1) создать сервис, состоящий из 2 различных контейнеров: 1 — веб, 2 — БД
2) далее необходимо создать 3 сервиса в каждом окружении (dev, prod, lab)
3) по итогу, на каждой ноде должно быть по 2 работающих контейнера
4) выводы зафиксировать

Задание 2*:
1) нужно создать 2 ДК-файла, в которых будут описываться сервисы
2) повторить задание 1 для двух окружений: lab, dev
3) обязательно проверить и зафиксировать результаты, чтобы можно было выслать преподавателю для проверки.

**Выполнение**
# Создаем файл docker-composer.yaml

root@gb-test1:~/compose# nano compose.yaml

# Посмотрим содержимое файла

root@gb-test1:~/compose# cat compose.yaml
version: ‘3.9’

services:

  db:
    image: mariadb:10.10.2
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 12345

  adminer:
    image: adminer:4.8.1
    restart: always
    ports:
      - 6080:8080

# Поднимаем сервис на основе этого файла

root@gb-test1:~/compose# docker compose up -d
[+] Building 0.0s (0/0)
[+] Running 2/0
 ✔ Container compose-adminer-1  Running                                                                            0.0s
 ✔ Container compose-db-1       Running                                                                            0.0s

# Проверяем

root@gb-test1:~/compose# docker ps -a

CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                    PORTS                                       NAMES

b2123d294118   adminer:4.8.1            "entrypoint.sh php -…"   40 minutes ago   Up 39 minutes             0.0.0.0:6080->8080/tcp, :::6080->8080/tcp   compose-adminer-1

e34ce20dab62   mariadb:10.10.2          "docker-entrypoint.s…"   40 minutes ago   Up 39 minutes             3306/tcp                                    compose-db-1

f4eb4f40143f   image_for_task4:latest   "python ./task4.py"      5 days ago       Exited (0) 5 days ago                                                 funny_engelbart

92a23be60b01   ubuntu:22.10             "/bin/bash"              5 days ago       Exited (0) 5 days ago                                                 gb-test

5d389821236b   ubuntu:22.10             "bash"                   5 days ago       Exited (127) 5 days ago                                               gracious_benz

09e77c329d00   ubuntu:22.10             "bash"                   5 days ago       Exited (127) 5 days ago                                               dazzling_buck

b196a4319211   mysql                    "docker-entrypoint.s…"   5 days ago       Exited (130) 5 days ago                                               happy_tu

850e46b8182b   mysql                    "docker-entrypoint.s…"   5 days ago       Created                                                               boring_mccarthy

446fe5a5a560   mysql                    "docker-entrypoint.s…"   5 days ago       Exited (130) 5 days ago                                               vigilant_lalande

dcb67e16c779   mysql                    "docker-entrypoint.s…"   5 days ago       Exited (130) 5 days ago                                               awesome_bartik

f75eda6d5115   mysql                    "docker-entrypoint.s…"   5 days ago       Created                                                               silly_blackwell

c62fe5c5a706   hello-world              "/hello"                 3 months ago     Exited (0) 3 months ago                                               pensive_galileo


# Подключаемся к adminer  в браузере по адресу http://localhost:6080

# Инициализируем swarm

root@gb-test1:~/compose# docker swarm init

Swarm initialized: current node (jcjydag44eyfamnyvntmdkiot) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1z1e48okt5gcw176rka6ffb2r3kszsv14ycwcn0oisqohju3x3-7arvj0n2qnser1pcoo0b43smt 192.168.1.103:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

# Добавляем вторую ноду на хосте gb-test2

root@gb-test2:~$ docker swarm join --token SWMTKN-1-1z1e48okt5gcw176rka6ffb2r3kszsv14ycwcn0oisqohju3x3-7arvj0n2qnser1pcoo0b43smt 192.168.1.103:2377

[sudo] password for gb-test: 

This node joined a swarm as a worker.

# Добавляем третью ноду на хосте gb-test3

root@gb-test3:~# docker swarm join --token SWMTKN-1-1z1e48okt5gcw176rka6ffb2r3kszsv14ycwcn0oisqohju3x3-7arvj0n2qnser1pcoo0b43smt 192.168.1.103:2377

[sudo] password for gb-test: 

This node joined a swarm as a worker.

# Проверяем наличие нод и их состояние (на хосте мастер-ноды)

root@gb-test1:~/compose# docker node ls

ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION

jcjydag44eyfamnyvntmdkiot *   gb-test      Ready     Active         Leader           24.0.4

latu4gl026bleo789vc6vpyk1     gb-test2     Ready     Active                          24.0.4

pn1c1mwrfleovi5nulkrsclox     gb-test3     Ready     Active                          24.0.4

# Добавляем нодам метки(labels) и проверяем их наличие

root@gb-test1:~/compose# docker node update --label-add env=lab pn1c1mwrfleovi5nulkrsclox

pn1c1mwrfleovi5nulkrsclox

root@gb-test1:~/compose# docker inspect --format='{{json .Spec}}' pn1c1mwrfleovi5nulkrsclox

{"Labels":{"env":"lab"},"Role":"worker","Availability":"active"}


root@gb-test1:~/compose# docker node update --label-add env=dev jb5fd4hyolcmmkgjwj1en6w56

jb5fd4hyolcmmkgjwj1en6w56

root@gb-test1:~/compose# docker inspect --format='{{json .Spec}}' jb5fd4hyolcmmkgjwj1en6w56

{"Labels":{"env":"dev"},"Role":"manager","Availability":"active"}


root@gb-test1:~/compose# docker node update --label-add env=prod latu4gl026bleo789vc6vpyk1

latu4gl026bleo789vc6vpyk1

root@gb-test1:~/compose# docker inspect --format='{{json .Spec}}' latu4gl026bleo789vc6vpyk1

{"Labels":{"env":"prod"},"Role":"worker","Availability":"active"}


# Создаем сервис sql-service на двух контейнерах

root@gb-test1:~/compose#   docker service create --name sql-service --replicas 2 -e MYSQL_ROOT_PASSWORD=12345 mariadb:10.10.2

permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/services/create": dial unix /var/run/docker.sock: connect: permission denied

root@gb-test1:~/compose#  sudo !!

sudo docker service create --name sql-service --replicas 2 -e MYSQL_ROOT_PASSWORD=tst123 mysql:8.0

9tq1jv2hoyjedv06muhj5zeko

overall progress: 2 out of 2 tasks

1/2: running   [==================================================>]

2/2: running   [==================================================>]

verify: Service converged

# Проверяем наличие и распределение контейнеров сервиса на всех нодах

root@gb-test1:~/compose# docker ps

CONTAINER ID   IMAGE             COMMAND                  CREATED             STATUS          PORTS                                       NAMES

91752a2de435   mariadb:10.10.2   "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes    3306/tcp                                    sql-service.2.mt9s4evdlfb2d52qxjogdiyac

b2123d294118   adminer:4.8.1     "entrypoint.sh php -…"   About an hour ago   Up 10 minutes   0.0.0.0:6080->8080/tcp, :::6080->8080/tcp   compose-adminer-1

e34ce20dab62   mariadb:10.10.2   "docker-entrypoint.s…"   About an hour ago   Up 10 minutes   3306/tcp                                    compose-db-1

root@gb-test2:~/compose# docker ps

[sudo] password for gb-test: 

CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS          PORTS                                       NAMES

a36c100bd6ce   mariadb:10.10.2 "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes    3306/tcp                                    sql-service.1.obz5vkb0qbr1btwotml2dhmd4

e7e722624b4c   mariadb:10.10.2 "docker-entrypoint.s…"   2 hours ago     Up 25 minutes   3306/tcp                                    compose-db-1

b485bed9c421   adminer:4.8.1   "entrypoint.sh php -…"   2 hours ago     Up 25 minutes   0.0.0.0:6080->8080/tcp, :::8085->8080/tcp   compose-adminer-1

root@gb-test3:~/compose# docker ps

[sudo] password for gb-test: 

CONTAINER ID   IMAGE           COMMAND                  CREATED       STATUS          PORTS                                       NAMES

e7e722624b4c   mariadb:10.10.2 "docker-entrypoint.s…"   2 hours ago   Up 28 minutes   3306/tcp                                    compose-db-1

b485bed9c421   adminer:4.8.1   "entrypoint.sh php -…"   2 hours ago   Up 28 minutes   0.0.0.0:6080->8080/tcp, :::8085->8080/tcp   compose-adminer-1


# Масштабируем сервис в 0 (снимаем нагрузку)

root@gb-test1:~/compose# docker service scale sql-service=0

# Создаем новый сервис из 2-х контейнеров на ноде "prod"

root@gb-test1:~/compose# docker service create --name prod_sql_service --constraint node.labels.env==prod --replicas 2 -e MYSQL_ROOT_PASSWORD=12345 mariadb:10.10.2

lyomxjig6ljaiw32r4wapmkpn

overall progress: 2 out of 2 tasks 

1/2: running   [==================================================>]

2/2: running   [==================================================>] 

verify: Service converged

# Проверяем сервисы

root@gb-test1:~/compose# docker service ls

ID             NAME               MODE         REPLICAS   IMAGE             PORTS

lyomxjig6lja   prod_sql_service   replicated   2/2        mariadb:10.10.2 

9tq1jv2hoyje   sql-service        replicated   0/0        mariadb:10.10.2 

# Проверяем контейнеры на ноде "prod" 

root@gb-test1:~/compose# docker ps 

CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                 NAMES

6035769196d3   mariadb:10.10.2   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   3306/tcp              prod_sql_service.1.nqjpr6mjcjjv36wkbcy4cfu5k

124c74d8a0b5   mariadb:10.10.2   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   3306/tcp              prod_sql_service.2.meih5kv70brskmtvjvtg08uxj
