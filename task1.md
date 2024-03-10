# Первое домашнее задание

## 1. Установил докер.
Установил Docker с оффициального [сайта](https://docs.docker.com/desktop/install/mac-install/) и открыл его:

![](1.png)

## 2. Скачать нужный контейнер MongoDB.
Скачал нужный контейнер с оффициального [сайта](https://phoenixnap.com/kb/docker-mongodb) MongoDB.
Далее действовал по инструкции:

**a)** Скачиваем контейнер с MongoDB:
```sh
sudo docker pull mongo
```

**б)** Запускаем этот контейнер:
```sh
sudo docker run -it -v mongodata:/data/db --name mongodb -d mongo
```

![](2.png)

## 3. Загрузка датасета.

Перешел на [сайт](https://habr.com/ru/companies/edison/articles/480408/) со слайда и скачал [датасет]( https://www.reddit.com/r/datasets/comments/1uyd0t/200000_jeopardy_questions_in_a_json_file/) jeopardy на $200 \, 000$ строк.

После этого захошел из терминала в директорию, куда я скачал нужный мне датасет и прописываю команду:
```sh
docker cp jeopardy.csv 0f37b59ad913dda185bccafa9c29fed71fe0a3c3bc63b8643750d5e8d33649df:/home/jeopardy.csv
```
где набор букв и цифр перед ```/home``` это номер контейнера, а то что после него отвечает за путь внутри контейнера до места, где будет лежать датасет.

Заходим в виртуальное окружение Shell контейнера при помощи команды:
```sh
sudo docker exec -it mongodb bash
```

Переходим в Shell mongodb, при помощи команды:
```sh
mongosh
```

Далее создаем коллекцию:
```sh
db.createCollection("jeopardy")
```
Далее выхожу в Shell контейнера
```sh
exit
```

Затем, внутри Shell контейнера перехожу в нужное мне место (куда хочу положить датасет):
```sh
root@0f37b59ad913: cd home
```
Внутри этой директории уже импортим нужный датасет при помощи команды:
```sh
root@0f37b59ad913: mongoimport -d jeopardy -c jeopardy  --type csv --file jeopardy.csv  --headerline
```

![](3.png)

## 4. Выполнение различных запросов

Для начала выберем нужную нам таблицу 
```sh
use jeopardy
```
Для начала выведем первые 5 строк датасета и проверим, что все импортировалось корректно:
```sh
db.jeopardy.find().limit(5)
```

![](4.png)

Для начала напишем INSERT-запрос, чтобы проверить, что все работает:
```sh
db.jeopardy.insertMany([
  {
    _id: ObjectId('65ec9e6154c558b3ab3df1e4'),
    'Show Number': 6301,
    'Air Date': '2012-01-28',
    Round: 'Double Jeopardy!',
    Category: 'WORLD CAPITALS',
    Value: "$2000",
    Question: 'This Asian capital is the most densely populated independent country in the world',
    Answer: 'Monaco'
  },
  {
    _id: ObjectId('65ec9e6154c558b3ab3df1e5'),
    'Show Number': 6302,
    'Air Date': '2012-01-29',
    Round: 'Final Jeopardy!',
    Category: 'FAMOUS AUTHORS',
    Value: "$5000",
    Question: 'This author of "Pride and Prejudice" was born in Steventon, England in 1775',
    Answer: 'Jane Austen'
  }
])
```
![](10.png)

Теперь нужно немного привести датасет в порядок. Мы будет анализировать метрику Value, то есть деньги. Но она представлена в неудобном формате строкой с символом "$". Нужно преобразовать Value в целое число.

Для начала удалим все строки со значением Null в Value, то есть реализуем DELETE-запрос:
```sh
db.jeopardy.deleteMany({"Value": NaN})
```

![](5.png)

Далее напишем UPDATE-запрос, для того чтобы поменять строку вида "$1000", нам нужно сначала взять подстроку (все кроме первого символа), за это отвечает функция
```sh
db.jeopardy.find().forEach( function (ch) { 
    var substring = ch.Value.substring(1);
    db.jeopardy.update(
        { "_id": ch._id }, 
        { "$set": { "Value": parseInt(substring) } }
    ); 
});
```
![](6.png)

Теперь мы можем писать различные SELECT-запросы, например давайте выведем все записи, в которых Value меньше чем 9900$ или Category равно HISTORY
```sh
db.jeopardy.find({$or :[{'Value': {$lt: 9900}}, {"Category": "HISTORY"}]})
```
![](7.png)

Теперь хотелось бы посмотреть на диапазон дат в этом датасете, для этого надо вывести максимальную и минимальную даты. Первый способ, через sort:
```sh
db.jeopardy.find().sort({'Air Date': -1 }).limit(1)
db.jeopardy.find().sort({'Air Date': 1}).limit(1)
```
![](8.png)

Либо второй способ через агригирующую функцию max:
```sh
db.jeopardy.aggregate([{$group: {_id: 1, max_w: {$max: "$Air Date"}}}])
db.jeopardy.aggregate([{$group: {_id: 1, max_w: {$min: "$Air Date"}}}])
```
![](9.png)

С помощью агригирующей функции можно также смотреть максимальную и минимальную дату с группировкой по полю, например по полю Round:

![](11.png)

## 5. Создание индекса

В любой базе данных чаще всего возникает необходимость делать SELECT-запросы с использованием среза по датам (например брать записи только конкретного дня или интервала дней), агрегации по датам и т.д. Поэтому, предлагается для оптимизации таких запросов ввести индекс по полю "Air Date".

Выберем какой-нибудь SELECT-запрос использующий дату для сравнения производительности с индексом и без. Я решил взять запрос, который выводит сумму заплаченных денег в зависимости от дня:

```sh
db.jeopardy.aggregate([{$group: {_id: "$Air Date", "sum_by_day" : {$sum: "$Value"}}}])
```

![](12.png)

Так теперь посмотрим за какое время этот запрос выполняется, для этого добавим приписку к запросу:
```sh
db.jeopardy.aggregate([{$group: {_id: "$Air Date", "sum_by_day" : {$sum: "$Value"}}}]).explain("executionStats")
```

![](13.png)

Получаем время выполнения запроса - 1074 мс.

Теперь введем в таблицу индекс по полю "Air Date":
```sh
db.jeopardy.createIndex({"Air Date": 1})
```
![](15.png)

И проверим производительность теперь:
```sh
db.jeopardy.aggregate([{$group: {_id: "$Air Date", "sum_by_day" : {$sum: "$Value"}}}]).explain("executionStats")
```
![](14.png)

Теперь время исполнения запроса уменьшилось до 138 мс, то есть уменьшилось практически в 10 раз!
