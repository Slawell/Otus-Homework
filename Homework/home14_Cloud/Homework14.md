# Homework14

Необходимо:
- Одну из облачных БД заполнить данными (любыми из предыдущих дз);
- Протестировать скорость запросов.

## Установлена MongoDb v.4.4 

В качестве тестового набора данных взял [Titanic Dataset](https://web.stanford.edu/class/archive/cs/cs109/cs109.1166/problem12.html)


В YaCloud развернут RS по материалам из методички 

yc managed-mongodb cluster list

|          ID          | NAME  |     CREATED AT      | HEALTH | STATUS  |
|----------------------|-------|---------------------|--------|---------|
| c9qisnqurvoa71ebkp1j | mdb-2 | 2023-04-09 16:15:44 | ALIVE  | RUNNING |


 yc managed-mongodb host list --cluster-id c9qisnqurvoa71ebkp1j

|                   NAME                    |      CLUSTER ID      |  TYPE  | SHARD NAME |  ROLE   | HEALTH |    ZONE ID    | PUBLIC IP |
|-------------------------------------------|----------------------|--------|------------|---------|--------|---------------|-----------|
| rc1a-b7yjg7mx47p8h72r.mdb.yandexcloud.net | c9qisnqurvoa71ebkp1j | MONGOD | rs01       | PRIMARY | ALIVE  | ru-central1-a | true      |


Импортируем набор данных:

    mongoimport --ssl 
                --sslCAFile=ya_ca.crt 
                --host  rc1a-b7yjg7mx47p8h72r.mdb.yandexcloud.net:27018 
                -u otus 
                -p otus1234 
                --db db1 
                --collection values 
                --type csv --headerline --file titanic.csv

### Поиск количества документов в наборе:

    rs01 [primary] db1> db.titanic.find().count()
    887

Совпадает с количеством документов указанным в компасе и в развернутом тестовом RS на VM VirtualBox (1CPU,1RAM,HDD).


Выполним запрос:

db.Titanic.find({$and:[{Sex:"female"},{Age:"20"},{Survived:"1"}]},{Name:1,_id:0})


Такой запрос выполнялся  2ms на Standalone в VirtualBox (1CPU,1RAM,HDD) 0ms, на Replicaset из 3 ВМ в VirtualBox (1CPU,1RAM,HDD) и скорее всего 0ms в YaCloud, я не нашел как можно измерить производительность, логи отдают только данные набора реплик но не позволяют читать запросы, вцыставить уровень профилировщика не удалось

    rs01 [primary] db1> db.setProfilingLevel(2)
    MongoServerError: not authorized on db1 to execute command { profile: 2, lsid: { id: UUID("bdcb4f7f-8289-49bb-86fe-67f0a82be0ac") }, $clusterTime: { clusterTime: Timestamp(1681075530, 1), signature: { hash: BinData(0, B36C89C15D3D192A3D571B21D75FC570F9D2B517), keyId: 7220085911453696005 } }, $db: "db1" }