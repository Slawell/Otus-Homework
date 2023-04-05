# Homework 7

Необходимо:

- Развернуть Instance ES – желательно в AWS
- Создать в ES индекс, в нём должно быть обязательное поле text типа string
- Создать для индекса pattern
- Добавить в индекс как минимум 3 документа желательно со следующим содержанием:

«моя мама мыла посуду а кот жевал сосиски»
«рама была отмыта и вылизана котом»
«мама мыла раму»
- Написать запрос нечеткого поиска к этой коллекции документов ко ключу «мама ела сосиски»
- Расшарить коллекцию postman (желательно сдавать в таком формате)
- Прислать ссылку на коллекцию

### - Развернуть Instance ES – желательно в AWS

Не реализуемо на текущий момент, у меня в Ycloud не получилось создать кластер. Процесс бесконечно висит Поднял ВМ на Virtual Box, отдельно скачал пакеты, установил из пакетов. ver. 8.7

### Создать в ES индекс, в нём должно быть обязательное поле text типа string

PUT https://192.168.18.100:9200/otus-index/otus-index
{
  "mappings": {
    "properties": {
      "phrase": { "type": "text" }
    }
  }
}

Ответ Postman

{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "otus-index"
}

### - Создать для индекса pattern

В версии 8.7 индекс паттерн был переименован в data viev https://www.elastic.co/guide/en/kibana/8.7/data-views.html

Подключился в web интерфейсу kibans и согласно инструкции по ссылке выше создал data viev

![Pattern](https://github.com/Slawell/Otus-Homework/blob/main/Homework/home7_elastic/Pattern.png)

### - Добавить в индекс как минимум 3 документа:

POST https://192.168.18.100:9200/otus-index/_doc
{
  "phrase": "моя мама мыла посуду а кот жевал сосиски"
}

Ответ Postman:

    {
        "_index": "otus-index",
        "_id": "tsYRT4cByNGi60UN93Vh",
        "_version": 1,
        "result": "created",
        "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1
    }

POST https://192.168.18.100:9200/otus-index/_doc
{
  "phrase": "рама была отмыта и вылизана котом"
}

Ответ Postman:

    {
        "_index": "otus-index",
        "_id": "t8YUT4cByNGi60UNUHXZ",
        "_version": 1,
        "result": "created",
        "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1
    }

POST https://192.168.18.100:9200/otus-index/_doc
{
  "phrase": "мама мыла раму"
}

Ответ Postman:

    {
        "_index": "otus-index",
        "_id": "uMYVT4cByNGi60UNYnU_",
        "_version": 1,
        "result": "created",
        "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1
    }

Проверяем данные

GET https://192.168.18.100:9200/otus-index/_search

{
  "query":{
      "match_all":{}
  }
}

Ответ Postman:

    {
        "took": 39,
        "timed_out": false,
        "_shards": {
            "total": 1,
            "successful": 1,
            "skipped": 0,
            "failed": 0
        },
        "hits": {
            "total": {
                "value": 3,
                "relation": "eq"
            },
            "max_score": 1.0,
            "hits": [
                {
                    "_index": "otus-index",
                    "_id": "tsYRT4cByNGi60UN93Vh",
                    "_score": 1.0,
                    "_source": {
                        "phrase": "моя мама мыла посуду а кот жевал сосиски"
                    }
                },
                {
                    "_index": "otus-index",
                    "_id": "t8YUT4cByNGi60UNUHXZ",
                    "_score": 1.0,
                    "_source": {
                        "phrase": "рама была отмыта и вылизана котом"
                    }
                },
                {
                    "_index": "otus-index",
                    "_id": "uMYVT4cByNGi60UNYnU_",
                    "_score": 1.0,
                    "_source": {
                        "phrase": "мама мыла раму"
                    }
                }
            ]
        }
    }


### - Написать запрос нечеткого поиска к этой коллекции документов ко ключу «мама ела сосиски»

GET https://192.168.18.100:9200/otus-index/_search

{
  "query": {
    "match": {
      "phrase": 
        "мама ела сосиски"
    }
  }
}

Ответ Postman:

    {
        "took": 45,
        "timed_out": false,
        "_shards": {
            "total": 1,
            "successful": 1,
            "skipped": 0,
            "failed": 0
        },
        "hits": {
            "total": {
                "value": 2,
                "relation": "eq"
            },
            "max_score": 1.241674,
            "hits": [
                {
                    "_index": "otus-index",
                    "_id": "tsYRT4cByNGi60UN93Vh",
                    "_score": 1.241674,
                    "_source": {
                        "phrase": "моя мама мыла посуду а кот жевал сосиски"
                    }
                },
                {
                    "_index": "otus-index",
                    "_id": "uMYVT4cByNGi60UNYnU_",
                    "_score": 0.5820575,
                    "_source": {
                        "phrase": "мама мыла раму"
                    }
                }
            ]
        }
    }
    
    
    Не уверен что правильно экспортировал коллекцию, прикладываю скрин postmann
    ![Postmann_Collection](https://github.com/Slawell/Otus-Homework/blob/main/Homework/home7_elastic/Collection.png)

    
