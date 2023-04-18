# Homework 13

Необходимо:

- Написать на тарантуле биллинг реального времени облачной системы. 

Должны быть хранимые процедуры:
 - добавление денег на баланс;
 - списание денег.

Когда баланс становится равным нулю, тарантул по http должен сделать GET-запрос на какой-либо внешний урл, где передать userID пользователя, у которого кончились деньги (запрос на отключение виртуальных машин). Этот вызов должен происходить как можно быстрее после окончания денег на счете.

Для реализации рекомендуется использовать библиотеку expirationd.

Использовать шардинг на основе vshard.


## Устанавливаем TDG

curl -L https://tarantool.io/HuWVKXj/release/2/installer.sh | bash

sudo dnf -y install tarantool

## Устанавливаем cartridge

sudo dnf install cartridge-cli

## Создаем приложение

[root@localhost tdg_otus]# cartridge create --name otus

    • Create application otus
    • Generate application files
    • Initialize application git repository
    • Application "otus" created successfully

[root@localhost /]# cd otus/
[root@localhost otus]# cartridge build

    • Build application in /otus
    • Running `cartridge.pre-build`
    • Running `tarantoolctl rocks make`
    • Application was successfully built

[root@localhost otus]# cartridge start

Подключаемся на web страницу тарантула:

Далее для проверки я попробовал воспроизвести код со странички Try Tarantool https://try.tarantool.io/ru?_ga=2.216687766.1245521301.1681818716-1313280701.1681582769

но на этапе получения запроса по http получаю 404 ответ

[root@localhost /]# curl -v -X POST --data "description=My first tiktok" http://192.168.18.51:8082/add_video                                            

    Note: Unnecessary use of -X or --request, POST is already inferred.
    *   Trying 192.168.18.51...
    * TCP_NODELAY set
    * Connected to 192.168.18.51 (192.168.18.51) port 8082 (#0)
    > POST /add_video HTTP/1.1
    > Host: 192.168.18.51:8082
    > User-Agent: curl/7.61.1
    > Accept: */*
    > Content-Length: 27
    > Content-Type: application/x-www-form-urlencoded
    >
    * upload completely sent off: 27 out of 27 bytes
    < HTTP/1.1 404 Not found
    < Content-length: 0
    < Server: Tarantool http (tarantool v2.10.6-0-g3990f976b)
    < Content-type: text/plain; charset=utf-8
    < Connection: keep-alive
    <
    * Connection #0 to host 192.168.18.51 left intact

Http модуль в rocks присутвует

![Http_module](https://github.com/Slawell/Otus-Homework/blob/main/Homework/home13_TDG/http_module.png)   

Далее чтобы что то поделать собираю стандартный кластер с двумя шардами и 1 роутером

настраиваю 1 репликасет и роутер

![Http_module](https://github.com/Slawell/Otus-Homework/blob/main/Homework/home13_TDG/1storage_1shard.png)   

Подключаю реплику в репликасет:

![Http_module](https://github.com/Slawell/Otus-Homework/blob/main/Homework/home13_TDG/join%20replicaset.png)  

Включаю файловер на основе raft:

![Http_module](https://github.com/Slawell/Otus-Homework/blob/main/Homework/home13_TDG/Failover.png)  

C raft не сработало переключение лидера, необходимо больше нод в реплике. Получилось переключить только исключив из голосования мастер

![Http_module](https://github.com/Slawell/Otus-Homework/blob/main/Homework/home13_TDG/Promote%20leader.png)







