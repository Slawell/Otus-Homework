# Homework4

Необходимо:

1 Развернуть кластер Couchbase

2 Создать БД, наполнить небольшими тестовыми данными

3 Проверить отказоустойчивость

## Развернуть кластер Couchbase


 По инcтрукции [Couchbase Documentation](https://docs.couchbase.com/server/current/manage/manage-nodes/initialize-node.html) развернуто 4 Вм и на каждую установлены пакеты couchbase


Подключаемся на 1 Вм по ее ip адресу и порту и инициализируем кластер + настраиваем первую вм под данные

Идем во вкладку Servers - Add Server и добавляем еще 2 ноды под данные

Нажимаем ребаланс, ждем завершение процесса

    "masterNode": "ns_1@couchbase.ru-central1.internal",
    "startTime": "2022-11-28T18:26:52.814Z",
    "completedTime": "2022-11-28T18:26:53.086Z",
    "timeTaken": 272,
    "completionMessage": "Rebalance completed successfully."

Добавляем последнюю 4 ноду, оставляем ее под все остальные процессы

     "nodesInfo": {
      "active_nodes": [
        "ns_1@couchbase.ru-central1.internal",
        "ns_1@couchbase2.ru-central1.internal",
        "ns_1@couchbase3.ru-central1.internal",
        "ns_1@couchbase4.ru-central1.internal"],
      "keep_nodes": [
        "ns_1@couchbase.ru-central1.internal",
        "ns_1@couchbase2.ru-central1.internal",
        "ns_1@couchbase3.ru-central1.internal",
        "ns_1@couchbase4.ru-central1.internal"


## 2 Создать БД, наполнить небольшими тестовыми данными

Загружены тестовые данные

beer-sample
67MiB / 600MiB
20.5MiB

gamesim-sample
586

56.4MiB / 600MiB
16.7MiB

travel-sample

167MiB / 600MiB
111MiB

## 3 Проверить отказоустойчивость

Выполнил файловер с 1 узла на второй. После завершения процесса потребовалась перебалансировка, был удален второй узел.

Остановил дополнительно еще одну машину, в панели появилась ошибка и кластер еще раз запросил процедуру файловера.

После того как в кластере осталась 1 машина с данными и 1 для бэкапа и тд появилось предупреждение в панели

***Warning: At least two servers with the data service are required to provide replication.***

Вернул удаленную из кластера ВМ. Это плюс Couchbase, в отличии от тарантула, в которрм удаленную машину уже нельзя вернуть в кластер.

После запуска выключенной машины ВМ вернул ее в кластер через GET Back Full Recovery и ребалансировку.


В качестве эксперимента, дополнительно создал группу пользователей users с доступом Vievs Reader, Query Insert, Search Reader и добавил в нее юзера ivan ivanov
