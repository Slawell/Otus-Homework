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