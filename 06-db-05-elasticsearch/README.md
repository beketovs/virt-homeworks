# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [elasticsearch:7](https://hub.docker.com/_/elasticsearch) как базовый:

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib` 
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения
- обратите внимание на настройки безопасности такие как `xpack.security.enabled` 
- если докер образ не запускается и падает с ошибкой 137 в этом случае может помочь настройка `-e ES_HEAP_SIZE`
- при настройке `path` возможно потребуется настройка прав доступа на директорию

Далее мы будем работать с данным экземпляром elasticsearch.

***
```
beketov@beketovs-MacBook-Pro elasticsearch % cat Dockerfile 
FROM centos:7

ENV PATH=/usr/lib:/usr/lib/jvm/jre-11/bin:$PATH

RUN yum install java-11-openjdk -y 
RUN yum install wget -y 

RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz \
    && wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz.sha512 
RUN yum install perl-Digest-SHA -y 
RUN shasum -a 512 -c elasticsearch-7.11.1-linux-x86_64.tar.gz.sha512 \ 
    && tar -xzf elasticsearch-7.11.1-linux-x86_64.tar.gz \
    && yum upgrade -y
    
ADD elasticsearch.yml /elasticsearch-7.11.1/config/
ENV JAVA_HOME=/elasticsearch-7.11.1/jdk/
ENV ES_HOME=/elasticsearch-7.11.1
RUN groupadd elasticsearch \
    && useradd -g elasticsearch elasticsearch
    
RUN mkdir /var/lib/logs \
    && chown elasticsearch:elasticsearch /var/lib/logs \
    && mkdir /var/lib/data \
    && chown elasticsearch:elasticsearch /var/lib/data \
    && chown -R elasticsearch:elasticsearch /elasticsearch-7.11.1/
RUN mkdir /elasticsearch-7.11.1/snapshots &&\
    chown elasticsearch:elasticsearch /elasticsearch-7.11.1/snapshots
    
USER elasticsearch
CMD ["/usr/sbin/init"]
CMD ["/elasticsearch-7.11.1/bin/elasticsearch"]
```
Ссылка на образ https://hub.docker.com/repository/docker/beketov/elastic

```
beketov@beketovs-MacBook-Pro elasticsearch % curl localhost:9200
{
  "name" : "2e8fdb5f637d",
  "cluster_name" : "netology_test",
  "cluster_uuid" : "zQrpROL0Tqi3OoMj7JLDWQ",
  "version" : {
    "number" : "7.11.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "ff17057114c2199c9c1bbecc727003a907c0db7a",
    "build_date" : "2021-02-15T13:44:09.394032Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
***

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

***
```
Создание индексов:
beketov@beketovs-MacBook-Pro elasticsearch % curl -X PUT localhost:9200/ind-1 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-1"}%                                                                            
beketov@beketovs-MacBook-Pro elasticsearch % curl -X PUT localhost:9200/ind-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-2"}%                                                                            
beketov@beketovs-MacBook-Pro elasticsearch % curl -X PUT localhost:9200/ind-3 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-3"}%

Просмотр статусов:
beketov@beketovs-MacBook-Pro elasticsearch % curl -X GET 'http://localhost:9200/_cat/indices?v' 
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 QE4sRq8cR62BTdANo5-trw   1   0          0            0       208b           208b
yellow open   ind-3 V_9opFzvR-e9ohu9Umk23w   4   2          0            0       208b           208b
yellow open   ind-2 PmmgREG3RzeF0aeeObcGCg   2   1          0            0       416b           416b

Удаление индексов:
beketov@beketovs-MacBook-Pro elasticsearch % curl -XDELETE http://127.0.0.1:9200/ind-1
{"acknowledged":true}%                                                                                                                       beketov@beketovs-MacBook-Pro elasticsearch % curl -XDELETE http://127.0.0.1:9200/ind-2
{"acknowledged":true}%                                                                                                                       beketov@beketovs-MacBook-Pro elasticsearch % curl -XDELETE http://127.0.0.1:9200/ind-3
{"acknowledged":true}%                                                                                                                       beketov@beketovs-MacBook-Pro elasticsearch % curl 'localhost:9200/_cat/indices?v'     
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```
Некоторые индексы в состоянии yellow, т.к. мы задавали количетсво реплик, но реплицировать некуда, т.к. у нас только один сервер.
***

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

***
```
Регистрация snapshots:
beketov@beketovs-MacBook-Pro elasticsearch % curl -X PUT "localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/elasticsearch-7.11.1/snapshots"   
  }
}
'


{
  "acknowledged" : true
}

Создаем индекс test:
beketov@beketovs-MacBook-Pro elasticsearch % curl -X PUT localhost:9200/test -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"test"}%

Создаем snapshot:
beketov@beketovs-MacBook-Pro elasticsearch % curl -X PUT "localhost:9200/_snapshot/netology_backup/elasticsearch?wait_for_completion=true&pretty"


{
  "snapshot" : {
    "snapshot" : "elasticsearch",
    "uuid" : "UViacEpvRxSfJjvOx7gt2g",
    "version_id" : 7110199,
    "version" : "7.11.1",
    "indices" : [
      "test"
    ],
    "data_streams" : [ ],
    "include_global_state" : true,
    "state" : "SUCCESS",
    "start_time" : "2022-06-29T14:00:16.413Z",
    "start_time_in_millis" : 1656511216413,
    "end_time" : "2022-06-29T14:00:16.413Z",
    "end_time_in_millis" : 1656511216413,
    "duration_in_millis" : 0,
    "failures" : [ ],
    "shards" : {
      "total" : 1,
      "failed" : 0,
      "successful" : 1
    }
  }
}

Смотрим, что snapshot создался:
beketov@beketovs-MacBook-Pro elasticsearch % docker exec -it elastic /bin/bash                                                                
[elasticsearch@2e8fdb5f637d /]$ cd elasticsearch-7.11.1/snapshots/
[elasticsearch@2e8fdb5f637d snapshots]$ ls -la
total 60
drwxr-xr-x 1 elasticsearch elasticsearch  4096 Jun 29 14:00 .
drwxr-xr-x 1 elasticsearch elasticsearch  4096 Jun 27 23:45 ..
-rw-r--r-- 1 elasticsearch elasticsearch   437 Jun 29 14:00 index-2
-rw-r--r-- 1 elasticsearch elasticsearch     8 Jun 29 14:00 index.latest
drwxr-xr-x 3 elasticsearch elasticsearch  4096 Jun 29 14:00 indices
-rw-r--r-- 1 elasticsearch elasticsearch 30935 Jun 29 14:00 meta-UViacEpvRxSfJjvOx7gt2g.dat
-rw-r--r-- 1 elasticsearch elasticsearch   269 Jun 29 14:00 snap-UViacEpvRxSfJjvOx7gt2g.dat

Удаляем индекс test и создаем test-2:
[elasticsearch@2e8fdb5f637d snapshots]$ curl -X DELETE 'http://localhost:9200/test?pretty'
{
  "acknowledged" : true
}
[elasticsearch@2e8fdb5f637d snapshots]$ curl -X PUT localhost:9200/test-2?pretty -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
[elasticsearch@2e8fdb5f637d snapshots]$ curl -X GET 'http://localhost:9200/_cat/indices?v' 
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 vLmJMbGFTI-s1O_P5DNDbg   1   0          0            0       208b           208b

Восстанавливаем состояние кластера из snapshot:
[elasticsearch@2e8fdb5f637d snapshots]$ curl -X POST localhost:9200/_snapshot/netology_backup/elasticsearch/_restore?pretty -H 'Content-Type: application/json' -d'{"include_global_state":true}'
{
  "accepted" : true
}

Смотрим индексы:
[elasticsearch@2e8fdb5f637d snapshots]$ curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 vLmJMbGFTI-s1O_P5DNDbg   1   0          0            0       208b           208b
green  open   test   mdz7LkZVRDytUCUCSK43Dw   1   0          0            0       208b           208b

```
***
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
