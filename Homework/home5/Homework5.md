# Homework5

ClickHouse

Описание/Пошаговая инструкция выполнения домашнего задания:
Необходимо, используя туториал https://clickhouse.tech/docs/ru/getting-started/tutorial/ :

- Развернуть БД;
выполнить импорт тестовой БД;
выполнить несколько запросов и оценить скорость выполнения.

- Развернуть дополнительно одну из тестовых БД https://clickhouse.com/docs/en/getting-started/example-datasets , протестировать скорость запросов

- Развернуть Кликхаус в кластерном исполнении, создать распределенную таблицу, заполнить данными и протестировать скорость по сравнению с 1 инстансом

## 1 Развернуть БД
Используя туториал https://clickhouse.tech/docs/ru/getting-started/tutorial/ был развернут standalon сервер clickhouse на платформе ya cloud

## 2 Выполнить импорт тестовой БД;

**Загружаем тестовые данные**
curl https://datasets.clickhouse.com/hits/tsv/hits_v1.tsv.xz | unxz --threads=`nproc` > hits_v1.tsv

curl https://datasets.clickhouse.com/visits/tsv/visits_v1.tsv.xz | unxz --threads=`nproc` > visits_v1.tsv

**Создаем таблицы** tutorial.hits_v1, tutorial.visits_v1

**Импортируем данные**
clickhouse-client --query "INSERT INTO tutorial.hits_v1 FORMAT TSV" --max_insert_block_size=100000 < hits_v1.tsv

clickhouse-client --query "INSERT INTO tutorial.visits_v1 FORMAT TSV" --max_insert_block_size=100000 < visits_v1.tsv

**Оптимизируем таблицы**

clickhouse-client --query "OPTIMIZE TABLE tutorial.hits_v1 FINAL"
clickhouse-client --query "OPTIMIZE TABLE tutorial.visits_v1 FINAL"

**Проверяем что данные импортировались**
clickhouse-client --query "SELECT COUNT(*) FROM tutorial.hits_v1"
8873898

clickhouse-client --query "SELECT COUNT(*) FROM tutorial.visits_v1"
1679791

**Выполняем тестовые запросы**

SELECT
    StartURL AS URL,
    AVG(Duration) AS AvgDuration
FROM tutorial.visits_v1
WHERE StartDate BETWEEN '2014-03-23' AND '2014-03-30'
GROUP BY URL
ORDER BY AvgDuration DESC
LIMIT 10

*10 rows in set. Elapsed: 0.123 sec. Processed 1.45 million rows, 114.59 MB (11.80 million rows/s., 931.12 MB/s.)*

SELECT
    sum(Sign) AS visits,
    sumIf(Sign, has(Goals.ID, 1105530)) AS goal_visits,
    (100. * goal_visits) / visits AS goal_percent
FROM tutorial.visits_v1
WHERE (CounterID = 912887) AND (toYYYYMM(StartDate) = 201403) AND (domain(StartURL) = 'yandex.ru')

*1 row in set. Elapsed: 0.018 sec. Processed 40.05 thousand rows, 4.94 MB (2.24 million rows/s., 275.91 MB/s.)*


## 3 Развернуть дополнительно одну из тестовых БД, выполнить несколько запросов и оценить скорость выполнения

Для примера был выбран Набор данных меню https://clickhouse.com/docs/ru/getting-started/example-datasets/menus

Создаем по инструкции таблицы Menu,Dish,MenuPage,MenuItem

Импортируем данные в Clickhouse
clickhouse-client --format_csv_allow_single_quotes 0 --input_format_null_as_default 0 --query "INSERT INTO dish FORMAT CSVWithNames" < Dish.csv

clickhouse-client --format_csv_allow_single_quotes 0 --input_format_null_as_default 0 --query "INSERT INTO menu FORMAT CSVWithNames" < Menu.csv

clickhouse-client --format_csv_allow_single_quotes 0 --input_format_null_as_default 0 --query "INSERT INTO menu_page FORMAT CSVWithNames" < MenuPage.csv

clickhouse-client --format_csv_allow_single_quotes 0 --input_format_null_as_default 0 --date_time_input_format best_effort --query "INSERT INTO menu_item FORMAT CSVWithNames" < MenuItem.csv

Денормализуем данные, затем проверяем 
SELECT count() FROM menu_item_denorm;

count()─┐
│ 1329175 

Получаем верный результат согласующийся с примером в инструкции.


Усредненные исторические цены на блюда

Query id: e516087a-5dc7-4ff0-861a-42b19485a789
17 rows in set. Elapsed: 0.213 sec. Processed 1.33 million rows, 54.62 MB (6.25 million rows/s., 256.97 MB/s.)

Цены на бургеры

Query id: 421b478a-320e-49bb-af16-f59496930a3a
14 rows in set. Elapsed: 0.249 sec. Processed 1.33 million rows, 94.15 MB (5.34 million rows/s., 378.45 MB/s.)
