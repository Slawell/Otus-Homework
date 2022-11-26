# Homework3

Необходимо:

1 Построить шардированный кластер из 3 кластерных нод( по 3 инстанса с репликацией) и с кластером конфига(3 инстанса);
2 Добавить балансировку, нагрузить данными, выбрать хороший ключ шардирования, посмотреть как данные перебалансируются между шардами;
3 Поронять разные инстансы, посмотреть, что будет происходить, поднять обратно. Описать что произошло.
4 Настроить аутентификацию и многоролевой доступ;

## Построить шардированный кластер

## Установлена MongoDb v.4.4 
## Создано 14 ВМ
### 3 ВМ - 1 шард "otus1"
### 3 ВМ - 2 шард "otus2"
### 3 ВМ - 3 шард "otus3"
### 3 ВМ - Конфиг сервера
### 2 ВМ - Mongos (Роутер)

### В кластер через монгос добавлены шарды otus1,otus2,otus3

sh.status()

--- Sharding Status ---

  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("63811f03a27b08cd621fb259")
  }

  shards:

        {  "_id" : "otus1",  "host" : "otus1/192.168.56.10:27017,192.168.56.11:27017,192.168.56.12:27017",  "state" : 1 }
        {  "_id" : "otus2",  "host" : "otus2/192.168.56.13:27017,192.168.56.14:27017,192.168.56.15:27017",  "state" : 1 }
        {  "_id" : "otus3",  "host" : "otus3/192.168.56.16:27017,192.168.56.17:27017,192.168.56.18:27017",  "state" : 1 }
  
## Добавить балансировку, нагрузить данными, выбрать хороший ключ шардирования, посмотреть как данные перебалансируются между шардами;

После создания кластера произошла перебалансировка чанков

 chunks:

      otus1   743
      otus2   183
      otus3   98

 chunks:

      otus1   342
      otus2   341
      otus3   341

Добавил данные. Все данные хранятся во 2 шарде otus2

 databases:
        {  "_id" : "Companies",  "primary" : "otus2",  ***"partitioned" : false,***  "version" : {  "uuid" : UUID("e64390aa-b146-4dea-b1ce-e39a4d0d1ed0"),  "lastMod" : 1 } }

Включим шардирование для Бд Companies

sh.enableSharding("Companies")
{
  ok: 1,}

**В качестве ключа шардирования я решил использовать хэш от поля _id (objectID)**

Создаю индекс: 

db.Companies.ensureIndex({_id:"hashed"})

[ '_id_hashed' ]

Запускаем шардирование по коллекциям.

sh.shardCollection("Companies.Companies",{_id:"hashed"})


databases:

        {  "_id" : "Companies",  "primary" : "otus2",  "partitioned" : true,  "version" : {  "uuid" : UUID("e64390aa-b146-4dea-b1ce-e39a4d0d1ed0"),  "lastMod" : 1 } }
                Companies.Companies
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                otus2   1
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : otus2 Timestamp(1, 0)

        {  "_id" : "Wine",  "primary" : "otus3",  "partitioned" : true,  "version" : {  "uuid" : UUID("54f9c16d-dae2-4057-9483-83efde93f530"),  "lastMod" : 1 } }
                Wine.RedWine
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                otus3   1
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : otus3 Timestamp(1, 0)

                Wine.WhiteWine
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                otus3   1
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : otus3 Timestamp(1, 0)
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }

                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                otus1   342
                                otus2   341
                                otus3   341



## 3 Поронять разные инстансы, посмотреть, что будет происходить, поднять обратно. Описать что произошло.

При отключении Slave node в репликасете, работоспособность кластера не нарушается. При отключении Primary происходят перевыборы и работа продолжается в штатном режиме. После восстановления Primary переключение на него не происходит автоматиески если у него не указан повышенный приоритет. Можно переключать ноды командой ***rs.stepdown()***

## 4 Настроить аутентификацию и многоролевой доступ

После создания репликасета используя localhost exeption создал пользователя с root доступом. 

"db" : "admin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"


Добавил простую роль с доступом на чтение и запись, роль clusterMonitor для возможности подлючения мониторинга, и создал кастомную роль.

### clusterMonitor

mongos> db.createUser({user:"mongomonitor", pwd:"password",roles:["clusterMonitor","dbAdmin"]})

Successfully added user: { "user" : "mongomonitor", "roles" : [ "clusterMonitor", "dbAdmin" ] }

### Чтение запись ДБ Companies

mongos> db.createUser({user:"readwrite", pwd:"passwd",roles:[{role:"readWrite",db:"Companies"}]})

Successfully added user: {
        "user" : "readwrite",
        "roles" : [{ "role" : "readWrite","db" : "Companies" }]}

### Кастомная роль update+insert

mongos> db.createRole({role:"InsertAndUpdate",privileges: [{actions: [ "update", "insert" ], resource: { db: "Companies", collection: "Companies" }}],roles: [] })

{ "role" : "InsertAndUpdate",
    "privileges" : [{"actions" : ["update","insert"],
                        "resource" : {"db" : "Companies",
                          "collection" : "Companies"}}],
                             "roles" : [ ]}
