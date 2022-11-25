# Homework2

Установить MongoDB одним из способов: ВМ, докер;
заполнить данными;
написать несколько запросов на выборку и обновление данных

## Установлена MongoDb v.4.4 

В качестве тестового набора данных взял [Titanic Dataset](https://web.stanford.edu/class/archive/cs/cs109/cs109.1166/problem12.html)

## Запросы:

## find()
***
### Поиск количества документов в наборе:
db.Titanic.find().count()

887

Совпадает с количеством документов указвнным в компасе.
***
### Поиск количества пассажиров мужчин:

db.Titanic.find({Sex:"male"}).count()

573
***
### Поиск пассажиров женщин выживших в катастрофе в возрасте от 20 до 25 лет

db.Titanic.find({$and:[{Sex:"female"},{Age:{$gte:"20",$lte:"25"}}]}).count()

61
***
### Поиск имени пассажрки 20 лет выжившие в катастрофе в возрасте 20 лет.

db.Titanic.find({$and:[{Sex:"female"},{Age:"20"},{Survived:"1"}]},{Name:1,_id:0})

{ Name: 'Miss. Anna Katherine Kelly' }

  ***

  ### Поиск в котором не выполняются условия запроса (Логическая НЕ ИЛИ )

  db.Titanic.find({$nor:[{Age:{$in:[10,15]}},{Survived:"0"},{Fare:{$gt:"0"}}]})

{ _id: ObjectId("637e238611a2081a59c761b2"),
  Survived: '1',
  Pclass: '3',
  Name: 'Mr. William Henry Tornquist',
  Sex: 'male',
  Age: '25',
  'Siblings/Spouses Aboard': '0',
  'Parents/Children Aboard': '0',
  Fare: '0' }
***
### Поиск пассажиров с Именем Anna и иницалом K (regex)

  db.Titanic.find({Name:{$regex:"Anna K"}})

{ _id: ObjectId("637e238611a2081a59c7610e"),
  Survived: '1',
  Pclass: '3',
  ***Name: 'Miss. Anna Kristine Salkjelsvik'***,
  Sex: 'female',
  Age: '21',
  'Siblings/Spouses Aboard': '0',
  'Parents/Children Aboard': '0',
  Fare: '7.65' }
{ _id: ObjectId("637e238611a2081a59c761cf"),
  Survived: '1',
  Pclass: '3',
  ***Name: 'Miss. Anna Katherine Kelly'***,
  Sex: 'female',
  Age: '20',
  'Siblings/Spouses Aboard': '0',
  'Parents/Children Aboard': '0',
  Fare: '7.75' }
***

## insert()
### Добавление выжившего пассажира John Week

  db.Titanic.insert({"Survived":"1", "Pclass":"1", "Name":"Mr.John Week"})

WriteResult({ "nInserted" : 1 })

 db.Titanic.find({Name:"Mr.John Week"})

{ "_id" : ObjectId("637fe7b02cfd78461497b658"), "Survived" : "1", "Pclass" : "1", "Name" : "Mr.John Week" }


## update
### обновление выжившего пассажира John Week

 db.Titanic.updateOne({Name:"Mr.John Week"},{$set:{"Sex":"male", "Age":"33","Fare":"1024"}})

{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

db.Titanic.find({Name:"Mr.John Week"})

{ "_id" : ObjectId("637fe7b02cfd78461497b658"), "Survived" : "1", "Pclass" : "1", "Name" : "Mr.John Week", "Age" : "33", "Fare" : "1024", "Sex" : "male" }

***
## Sort Limit
### Поиск наиболее дорогого билета 

db.Titanic.find().sort({Fare:-1}).limit(1)

{ _id: ObjectId("637e238611a2081a59c762aa"),
  Survived: '1',
  Pclass: '1',
  Name: 'Miss. Anne Perreault',
  Sex: 'female',
  Age: '30',
  'Siblings/Spouses Aboard': '0',
  'Parents/Children Aboard': '0',
   ***Fare: '93.5'*** }

## Skip
db.Titanic.find({}, {Age:1, _id:0}).sort({Age:1}).limit(3)

{ Age: '0.42' }
{ Age: '0.67' }
{ Age: '0.75' }

db.Titanic.find({}, {Age:1, _id:0}).sort({Age:1}).skip(1).limit(3)

{ Age: '0.67' }
{ Age: '0.75' }
{ Age: '0.75' }

## Aggregate

### Match (Выборка мужчин в возрасте 25 лет выживших в катастрофе. Выводим имя , возраст, статус)

db.Titanic.aggregate([{$match:{Sex:"male",Age:"25",Survived:"1"}},{$project:{"Name":1, "Age":1, "Survived":1,"_id":0}}])

{ Survived: '1', Name: 'Mr. Ernst Ulrik Persson', Age: '25' }

{ Survived: '1', Name: 'Mr. William Henry Tornquist', Age: '25' }

{ Survived: '1', Name: 'Mr. George Achilles Harder', Age: '25' }

{ Survived: '1', Name: 'Mr. Dickinson H Bishop', Age: '25' }

### Group (Выборка мужчин в разрезе класса купленного билета)

db.Titanic.aggregate([{$match:{Sex:"male"}},{$group:{_id:"$Pclass", count:{$sum:1}}}])

{ _id: '3', count: 343 }

{ _id: '1', count: 123 }

{ _id: '2', count: 108 }

### Distinct (Выборка массива возрастов пассажиров)
db.Titanic.distinct("Age")

[
  '0.42', '0.67', '0.75', '0.83', '0.92', '1',    '10',   '11',
  '12',   '13',   '14',   '14.5', '15',   '16',   '17',   '18',
  '19',   '2',    '20',   '20.5', '21',   '22',   '23',   '23.5',
  '24',   '24.5', '25',   '26',   '27',   '28',   '28.5', '29',
  '3',    '30',   '30.5', '31',   '32',   '32.5', '33',   '34',
  '34.5', '35',   '36',   '36.5', '37',   '38',   '39',   '4',
  '40',   '40.5', '41',   '42',   '43',   '44',   '45',   '45.5',
  '46',   '47',   '48',   '49',   '5',    '50',   '51',   '52',
  '53',   '54',   '55',   '55.5', '56',   '57',   '58',   '59',
  '6',    '60',   '61',   '62',   '63',   '64',   '65',   '66',
  '69',   '7',    '70',   '70.5', '71',   '74',   '8',    '80',
  '9'
]


# Создание индекса

## Простой индекс. Создадим индекс по полю возраст.

Без индекса запрос выполняется 2ms при этом выполняется COLLSCAN, исследуются 888 документов что на больших объемах данных даст нагрузку на CPU, и операция будет выполнятся за O(n), что неоптимально.

db.Titanic.find({$and:[{Sex:"female"},{Age:"20"},{Survived:"1"}]},{Name:1,_id:0})

После создания индекса
db.Titanic.createIndex({Age:1})

'Age_1'

Операция выполнилась за 0 ms, было исследовано 23 документа, выполнялся IXSCAN


## Cоставной индекс. Создадим индекс по полю возраст и пол. 
db.Titanic.createIndex({Sex:1,Age:1})

'Sex_1_Age_1'

Запрос

db.Titanic.find({$and:[{Sex:"female"},{Age:"20"},{Survived:"1"}]},{Name:1,_id:0})

 Операция выполнилась за 0 ms, было исследовано 3 документа, выполнялся IXSCAN