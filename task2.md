# Task 2

## Поднимаю Redis с помощью Docker

Я прочитал информацию по установке Redis с [сайта](https://www.dmosk.ru/miniinstruktions.php?mini=redis-ubuntu&ysclid=luise89idc676136907#install). На данном сайте очень подробно описано как поднимать Redis разными способами, на разных ОС и при помощи разных языков программирования.

1) Скачиваем контейнер Redis-а
```bash
docker pull redis
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/101.png)
2) Запускаем контейнер Redis-а
```bash
docker run --name redis-server -d redis
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/103.png)
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/102.png)
3) Проверяем, что все запустилось коректно
```bash
docker ps
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/104.png)
4) Теперь запускаем cli Redis-а и выполняем простейшие команды set, get для проверки доступности.
```bash
docker exec -it redis-server redis-cli
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/105.png)
5) Без дополнительных настроек, Redis в Docker позволяет сетевое подключение из Docker сети и с локального сервера. Для доступа к контейнеру через хост Docker, необходимо запустить его с параметром -p 6379:6379
```bash
docker container stop redis-server && docker container rm -v redis-server
redis-server
docker run --name redis-server -p 6379:6379 -d redis
```
## Переходим в Python для дальнейшей работы с Redis

1) Устанавливаем библиотеку redis
```bash
pip install redis
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/106.png)
2) 
```python
import redis
r = redis.Redis(host='localhost', port=6379, db=1)
```
3) Проверяем, что все работает
```python
r.set('test_py_key', 'test py value')
redis_get = r.get('test_py_key')
print(redis_get)
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/107.png)
4) Нашел на kaggle [датасет](https://www.kaggle.com/datasets/rmisra/news-category-dataset?resource=download) c разными новостями размером в 88Мб, такого вида:
```json
{
  "link": "https://www.huffpost.com/entry/covid-boosters-uptake-us_n_632d719ee4b087fae6feaac9",
  "headline": "Over 4 Million Americans Roll Up Sleeves For Omicron-Targeted COVID Boosters",
  "category": "U.S. NEWS",
  "short_description": "Health experts said it is too early to predict whether demand would match up with the 171 million doses of the new boosters the U.S. ordered for the fall.",
  "authors": "Carla K. Johnson, AP",
  "date": "2022-09-23"
}
```
5) Импортируем json-файл в Python
```python
import json
data_news = [json.loads(line) for line in open('/Users/tsarevivan/Downloads/News_Category_Dataset_v3.json','r')]
```
6) Загружаем нашу базу данных в Redis и засекаем время выполнения
```python
import time
start_time = time.time()
news_id_counter = 0
pipe = r.pipeline()
for news in data_news:
    news_id = news_id_counter
    news_id_counter += 1
    for field in ('category', 'short_description', 'authors', 'date'):
        pipe.set(f'news:{news_id}:{field}', news[field])
pipe.execute()
execution_time = time.time() - start_time
print(f"Время выполнения: {execution_time} секунд")
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/108.png)
Получаем время загрузки БД в Redis примерно ```12.7``` секунд
7) Теперь проверим время получения по ключу:
```python
start_time = time.time()
r.get('key_56742')
execution_time = time.time() - start_time
print(f"Время выполнения: {execution_time} секунд")
```
Получили время примерно ```8.8ms```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/109.png)

8) Теперь проверим время вставки через hset
Сначала отчистим БД
```python
r.flushdb()
```
Время выполнения: 0.5930590629577637 секунд
Пробуем выгрузить при помощи hset
```python
news_id_counter = 0
pipe = r.pipeline()
for news in data_news:
    news_id = news_id_counter
    news_id_counter += 1
    for field in ('category', 'short_description', 'authors', 'date'):
        pipe.hset(f'news:{news_id}:{field}', news[field])
pipe.execute()
```
Но получаем ошибку (невозможно обработать Null значения)
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/110.png)
Поэтому я решил удалить все строки, где есть хотя бы один Null и заново запустил импорт
```python
data_news = [news for news in data_news if not any(field is None for field in news.values())]
```
Теперь все работает корректно и получаем время выполнения примерно 12.234 секунды, что немного быстрее прошлого способа
9) Теперь проверим время получения по ключу:
```python
start_time = time.time()
r.hget('news_id:56742', 'short_description')
execution_time = time.time() - start_time
```
Этим способом время получения ключа получилось немного лучше, а именно ```6.3ms```
10) Теперь проверим работу Redis Sorted Sets, для определения дат заказов, например, наиболее ранних 
Сделаем функцию для преобразования строковой переменную в timestamp:
```python
def to_timestamp(date_str):
    return time.mktime(datetime.strptime(date_str, '%Y-%m-%d').timetuple())
```
Реализуем вставку:
```python
start_time = time.time()
r.zadd('news_date', {str(index): to_timestamp(user['date']) for index, user in enumerate(data_news)})
execution_time = time.time() - start_time
print(f"Время выполнения: {execution_time} секунд")
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/111.png)

Найдем id новости с наименьшей датой публикации:
```python
start_time = time.time()
print(r.zrange('news_date', 0, 2))
execution_time = time.time() - start_time
print(f"Время выполнения: {execution_time} секунд")
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/112.png)
11) Выполняем аналогичные действия для Redis List:
Вставим туда имена авторов:
```python
for user in data_news:
    r.lpush('news_author', user['authors'])
```
И выведем первые 5 авторов:
```python
r.lrange('users_name', 0, 5)
```
Получаем такой вывод:
```
Out: [b'Carla K. Johnson, AP', b'Mary Papenfuss', b'Nina Golgowski', b'Elyse Wanshel', b'Marina Fang', b'Aamer Madhani']
```

## Поднимаю кластер на 3 нодах и проверяю отказоустойчивость

1) Поднимаю кластер при помощи файла docker-compose.yaml. 
```bash
docker-compose -p my-redis-cluster-nodes up
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/113.png)
2) Мой кластер состоит из трех нод, где одна мастер и две реплики (это все прописывается в файле docker-compose)
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/114.png)
3) Проверим, что все ноды действительно функционируют
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/115.png)
4) Откроем cli первой ноды и положим какое-то тестовое значение по ключу
```bash
docker exec -it my-redis-cluster-nodes-redis-node-1-1 redis-cli
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/116.png)
5) Теперь проверим отказоустойчивость
Выключаем первую ноду (имитируем поломку), тогда одна из дублирующих нод должна ее заменить
```
docker stop my-redis-cluster-nodes-redis-node-1-1
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/117.png)
6) Заходим теперь в cli дублирующей (нового мастера) ноды
```bash
docker exec -it my-redis-cluster-nodes-redis-node-2-1 redis-cli
```
И проверяем доступность ключа

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/118.png)


