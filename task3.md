# Task 3

## Поднимаю CouchDB с помощью Docker
1) Напишем docker-compose.yaml файл, запоминаем какие пароль и логин задаем
```yaml
version: '3'
services:
  db:
    image: couchdb
    restart: always
    ports:
      - "5984:5984"
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=1234
```
При помощи этого файла поднимаю CouchDB при помощи такой команды:
```bash
docker-compose up
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/201.png)

2) Проверим, что все контейнер запущен
```bash
docker ps
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/202.png)

3) Теперь откроем веб интерфейс перейдя по такой ссылке
```
http://localhost:5984/_utils/#login
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/203.png)

4) Переходим в вкладку Databases
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/204.png)

5) Cоздаем новую БД, с названием test_database при помощи кнопки Create Database.
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/205.png)

6) Подключаем созданную test_database при помощи curl запроса
```
curl -X GET http://admin:1234@127.0.0.1:5984/test_database
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/Images/208.png)

7) Теперь в приложенный в дз [файл](https://github.com/tsar1in/HW_DB_Sber/blob/main/task3_files/ДЗ3.html) html пропишем ссылку и название нашей БД
```
const DBS = {
    // Создаем локальную БД и коннектимся к удаленной 
    Local: new PouchDB('test_database'),
    Remote: new PouchDB('ttp://admin:1234@127.0.0.1:5984/test_database')
}
```
И откроем, пока ничего нет (имени и фамилии):
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/206.png)

8) Теперь при помощи POST-запроса положим в БД мое имя и фамилию
```bash
curl -H "Content-Type: application/json" -X POST http://admin:1234@127.0.0.1:5984/test_database -d '{"name": "Tsarev Ivan"}'
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/209.png)
Проверим, что значение действительно попало в БД в веб-интерфейсе

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/207.png)

9) Опять открываем html-файл и видим, что имя подгрузилось в БД
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/211.png)

10) Теперь стопаем контейнер ```CNTR + C```
    
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/212.png)

12) Перезапустим ДЗ3.html и нажмем опять sync данные (фамилия и имя) сохранились в бд и по прежнему выводятся. Сохраняем полученный [файл](https://github.com/tsar1in/HW_DB_Sber/blob/main/task3_files/WithMyName.html) html
