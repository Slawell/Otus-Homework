# Homework9

Необходимо:

1 Cохранить большой жсон (~20МБ) в виде разных структур - строка, hset, zset, list;
2 Протестировать скорость сохранения и чтения;
3 Предоставить отчет.

4 Задание повышенной сложности*
  настроить редис кластер на 3х нодах с отказоусточивостью, затюнить таймоуты

## Настроить редис кластер на 3х нодах с отказоусточивостью, затюнить таймоуты

Характеристики ВМ: 1 CPU 1 Gb RAM CentOS8 развернуты в Virtual Box

## Установлен Redis 6.0.9 подключен модуль rejson
## Создано 6 ВМ
### 3 ВМ - Redis Master
### 3 ВМ - Redis Replica

Собран кластер с 1 репликой на мастера

 redis-cli --user admin --pass password --cluster create 192.168.56.104:6379 192.168.56.105:6379 192.168.56.106:6379 192.168.56.107:6379 192.168.56.108:6379 192.168.56.109:6379 --cluster-replicas 1
Master:  A 192.168.56.104:6379 B 192.168.56.105:6379 C 192.168.56.106:6379
Replica: D 192.168.56.107:6379 E 192.168.56.108:6379 F 192.168.56.109:6379


## Тест записи большого json файла

Был взят файл по ссылке https://github.com/zemirco/sf-city-lots-json/blob/master/citylots.json и урезан до 65 мб. Файл приложен в папке с ДЗ.

1 String

time redis-cli  -c --user admin --pass password -p 6379 -x set test-db  < citylots_edited.json

real    0m0.847s

2 HSET

time redis-cli  -c --user admin --pass password -p 6379 -x hset test test-db  < citylots_edited.json

real    0m2.168s

3 ZSET

time redis-cli  -c --user admin --pass password -p 6379 -x zadd ztest 0  < citylots_edited.json

real    0m0.262s

4 LIST

 time redis-cli  -c --user admin --pass password -p 6379 -x LPUSH ltest-db  < citylots_edited.json

real    0m1.261s

### Итого по записи

Zset     0m0.262s

String   0m0.847s

List     0m0.847s

Hset     0m0.847s

C запросами у меня плохо, попробовал сделать по аналогии 
time redis-cli  -c --user admin --pass password -p 6379 -x get test-db  

Запрос висел более 4 минут дальше продолжать не стал