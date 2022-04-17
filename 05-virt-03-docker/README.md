
# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"



## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.

***
```
https://hub.docker.com/repository/docker/beketov/mynginx
docker run -d --rm --name nginx-container -p 80:80 beketov/mynginx:1.0
```
```
root@vagrant:/home/vagrant/page# docker exec -it nginx-container bash
root@841ffd3e51dd:/# cat /var/www/html/index.html
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
***

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--
***

Сценарий:

- Высоконагруженное монолитное java веб-приложение;

Физическая машина, т.к. сервис монолитный. Микросервисная архитектура дя которой предназначен Docker не даст преимуществ, а только усложнит.

- Nodejs веб-приложение;

Ввиду того, что это веб-приложение, то докер вполне подойдет для его публикации в контейнере.

- Мобильное приложение c версиями для Android и iOS;

Хорошим вариантом вижу использованиие виртуальных машин. Быстро и как раз будет разделять разные версии используя разные виртуалки.

- Шина данных на базе Apache Kafka;

Тут можно использовать и виртуальную машину и докер. Если докер, то монтируем volume на физ. хост и о потере данных можно не беспокоиться.

- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;

Хорошим вариантом будет docker реализация. Офф документация также рекомендует использования сервисов в docker. Запускать всю систему будем через docker-compose.

- Мониторинг-стек на базе Prometheus и Grafana;

Docker отлично подойдет. Volume прокидываем на хост машину. Лично у меня уже есть подобная реализация, все отлично работает.

- MongoDB, как основное хранилище данных для java-приложения;

Можно поднять под stateless сервис ВМ, но также монгу можно запускать и в докере. Выбор будет сделан в зависимости от требований к высоконагруженности БД.

- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.

Есть реализации под docker, но можно поднять и виртуальную машину. Принципиального отличия в способе реализации не вижу.
***

## Задача 3
***
- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
```
root@vagrant:/data# docker run -d --rm -i -t --name mycentos -v /data:/data centos:6.6 /bin/bash
```
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
```
root@vagrant:/home/vagrant/page# docker run -d -i -t --rm --name mydebian -v /data:/data debian
```
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
```
root@vagrant:/data# docker exec -i -t mycentos /bin/bash
[root@cdf5d35216be /]#
[root@cdf5d35216be /]# cd /data/
[root@cdf5d35216be data]# touch myfile.txt
[root@cdf5d35216be data]# echo "This my file" > myfile.txt
[root@cdf5d35216be data]# cat myfile.txt
This my file
```
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
```
root@vagrant:/data# touch myfile2.txt
root@vagrant:/data# ls -la
total 12
drwxr-xr-x  2 root root 4096 Apr 17 22:03 .
drwxr-xr-x 22 root root 4096 Apr 17 21:28 ..
-rw-r--r--  1 root root    0 Apr 17 22:03 myfile2.txt
-rw-r--r--  1 root root   13 Apr 17 22:02 myfile.txt
```
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.
```
root@vagrant:/data# docker exec -i -t mydebian /bin/bash
root@5115f069f78e:/# cd /data/
root@5115f069f78e:/data# ls -la
total 12
drwxr-xr-x 2 root root 4096 Apr 17 22:03 .
drwxr-xr-x 1 root root 4096 Apr 17 22:01 ..
-rw-r--r-- 1 root root   13 Apr 17 22:02 myfile.txt
-rw-r--r-- 1 root root    0 Apr 17 22:03 myfile2.txt
```
***
