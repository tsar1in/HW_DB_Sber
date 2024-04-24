# Presto DB

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/302.png)

## История развития

Presto было создано в Facebook в 2012 году для решения проблем анализа огромных объёмов данных, которые не справлялись другие системы.

Изначально Presto разрабатывалась как внутренний проект Facebook, но вскоре показав свою эффективность, была вынесена в открытый доступ, превратившись в ведущее решение для больших данных с открытым исходным кодом. Развитие Presto продолжилось за пределами Facebook, и, чтобы обеспечить её дальнейшее развитие и поддержку, в 2019 году был создан Presto Foundation под эгидой Linux Foundation.

Presto характеризуется высокой производительностью, эффективностью и гибкой архитектурой, что делает её популярным выбором для анализа больших данных среди многих крупных и малых предприятий по всему миру.

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/301.png)

## Инструменты для взаимодействия

Кроме того, Presto предоставляет веб-интерфейс для мониторинга запросов и управления ими. Веб-интерфейс доступен на Presto coordinator через HTTP, используя номер HTTP-порта, указанный в свойствах конфигурации coordinator.

PrestoDB супер-универсальная база данных, у нее существует огромное множество инструментов взаимодействия, всех перечислить даже не смогу.

Также у Presto усть очень хорошая и полная документация, доступная по [ссылке](https://prestodb.io/docs/0.286/index.html). Большую часть информации я взял чисто оттуда.

1. Presto CLI (command-line interface):
   - Это основной инструмент для работы с PrestoDB. CLI позволяет пользователям подключаться к Presto серверу и выполнять SQL-запросы напрямую из командной строки.
2. JDBC и ODBC драйверы:
   - К Presto можно подключиться из Java с помощью драйвера JDBC или ODBC, которые также используются для интеграции с различными клиентскими приложениями и инструментами аналитики типа Tableau, Power BI, иногда можно использовать оболочку Hadoop
3. Web интерфейс:
   - Presto предоставляет веб-интерфейс, где пользователи могут следить за выполнением запросов, просматривать историю запросов и общую статистику работы кластера. Веб-интерфейс доступен на Presto coordinator через HTTP, используя номер HTTP-порта, указанный в свойствах конфигурации coordinator.
5. Поддержка из IDE:
   - Разработчики могут использовать JDBC драйвер для подключения к Presto из IDE (например, IntelliJ IDEA или Eclipse) для удобной работы с SQL запросами непосредственно из среды разработки.
  
## Database Engine
Presto не использует традиционный database engine в привычном понимании, который хранит данные и управляет транзакциями. Вместо этого, Presto использует так называемый **SQL-query engine**, он разработан для выполнения SQL-запросов практически на любых данных, хранящихся в многочисленных источниках через коннекторы. Разработан как высокопроизводительный механизм для выполнения распределенных запросов, Presto может обращаться к данным, расположенным в системах таких как HDFS, Amazon S3, MySQL, Cassandra, Kafka и многих других, прозрачно обрабатывая и объединяя данные из разных источников.

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/303.png)

## Язык запросов

Presto DB использует собственный язык запросов, который тесно связан с SQL (фактически это и есть стандартный SQL, типо MySQL или PostgreSQL) и известен своей мощностью и гибкостью при работе с большими объемами данных, распределенными по различным источникам. Он позволяет эффективно выполнять различные операции анализа данных, такие как агрегация, соединение и подзапросы.

Весь синтаксис языка очень подробно описан в [документации](https://prestodb.io/docs/0.286/sql.html)

## Развертка БД и некоторые запросы

Инструкцию по установке и разверте БД, я брал с оффициального сайта, со страницы про [докер](https://hub.docker.com/r/prestodb/presto)

Устанавливаем докер-контейнер с базой данных БД, она весит совсем немало:

```bash
docker pull prestodb/presto
```
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/308.png)

Запускаем контейнер:

```bash
docker run -p 8080:8080 -ti prestodb/presto:latest
```

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/309.png)

Ждем пока сервер запустится, информация об этом выведется в терминал, примерно так:

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/310.png)

Теперь переходим на веб-интерфейс БД расположенный на 8080 локалхосте

```
http://localhost:8080/ui/
```

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/311.png)

Посмотрим, как выглядит ui для sql запросов к PrestoDB и cделаем простые запросы для примера работы
![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/313.png)
1) Создаем тестовую схему

```sql
CREATE SCHEMA task_4
```

2) Создаем тестовую таблицу:

```sql
CREATE TABLE IF NOT EXISTS employees (
  id int,
  name varchar,
  surname varchar,
  age int,
  salary double
)
COMMENT 'A table to test PrestoDB'
```

И заполянем ее тестовыми данными:

```sql
INSERT INTO employees (id, name, surname, age, salary) VALUES
(1, 'Иван', 'Иванов', 30, 50000.00),
(2, 'Сергей', 'Петров', 25, 42000.00),
(3, 'Мария', 'Васильева', 28, 45000.00),
(4, 'Елена', 'Сидорова', 35, 55000.00),
(5, 'Александр', 'Морозов', 40, 60000.00),
(6, 'Наталья', 'Сорокина', 24, 39000.00),
(7, 'Дмитрий', 'Кузнецов', 33, 48000.00),
(8, 'Людмила', 'Яковлева', 29, 47000.00),
(9, 'Антон', 'Белов', 31, 51000.00),
(10, 'Юлия', 'Михайлова', 27, 43000.00),
(11, 'Роман', 'Григорьев', 32, 52000.00),
(12, 'Татьяна', 'Романова', 45, 62000.00),
(13, 'Игорь', 'Калинин', 23, 38000.00),
(14, 'Ольга', 'Крылова', 36, 57000.00),
(15, 'Владимир', 'Орлов', 39, 58000.00);
```

Теперь можем поделать различные тестовые запросы. Например выведем всех работников, старше 28 лет и зарабатывающих от 40к до 50к:

```sql
SELECT 
    name, 
    surname
FROM new_catalog.task_4.employees
WHERE age > 28 AND salary BETWEEN 40000.00 AND 50000.00
ORDER BY surname, name
```

Или, например, выведем всех работников и их зарплаты, которые зарабатывают больше среднего опытных сотрудников (работники старше 30 лет) в комппании:

```sql
SELECT 
    name, 
    surname,
    salary
FROM new_catalog.task_4.employees
WHERE salary >= (
    SELECT AVG(salary)
    FROM new_catalog.task_4.employees
    WHERE age > 30
)
ORDER BY salary DESC
```

## Распределение файлов БД по разным носителям

PrestoDB не занимается управлением файлами или их хранением напрямую, поскольку это распределённая **SQL-query engine** система, а не система управления базами данных в традиционном понимании. Presto разрабатывается для выполнения запросов к данным, которые уже хранятся в различных источниках данных, поэтому в рамках Presto говорить о распределении файлов БД по разным носителям нельзя 

## На каком языке/ах программирования написана СУБД

PrestoDB в основном написана на языке программирования **Java**. Разработка на **Java** позволяет PrestoDB гибко интегрироваться с различными системами хранения данных и обеспечивает высокую скорость выполнения запросов на больших объёмах данных благодаря оптимизированной обработке JVM (Java Virtual Machine).

Хотя основная часть Presto написана на Java, встречаются и другие языки программирования в контексте инструментов и библиотек, используемых в экосистеме Presto:
 - **JavaScript**: Используется в веб-интерфейсе для создания интерактивных пользовательских интерфейсов, предназначенных для мониторинга и управления работой Presto.
 - **Python**: Иногда используется для скриптов поддержки, тестирования и анализа данных в рамках разработки и поддержки Presto.

## Индексы

PrestoDB, в отличие от традиционных систем управления базами данных, не поддерживает создание индексов в обычном смысле этого слова, как, например, в MySQL или PostgreSQL. Это связано с тем, что Presto разработан для работы как query engine поверх различных типов данных, хранящихся в разных источниках данных.

Причины отсутствия поддержки традиционных индексов:
1. Распределенные и разнородные источники данных: Данные, с которыми работает Presto, могут быть распределены по различным системам хранения. Каждая из этих систем может иметь свою собственную систему индексации, но Presto сам по себе не управляет хранением данных и не имеет прямого доступа к физической структуре данных, поэтому попросту не может создать традиционные индексы.

2. Оптимизация через другие механизмы: Вместо использования индексов Presto оптимизирует производительность запросов через механизмы такие как эффективное распределение работы по узлам кластера, оптимизация запросов на стороне SQL-парсера и планировщика, и использование специфичных функций коннекторов.

Но есть методы альтернативные индексам в PrestoDB, например:
- Использование индекса страниц в формате Parquet, который позволяет значительно ускорить запросы путем минимизации количества данных, считываемых с диска. Эта оптимизация достигается за счет фильтрации страниц данных еще до их загрузки, что сокращает время обработки запросов и улучшает производительность. Прочитал об этом методе на [форуме](https://prestodb.io/blog/2022/05/10/faster-presto-queries-with-parquet-page-index/).

## Процесс выполнения запросов

**1. Парсинг и планирование:**

   Presto получает запрос SQL, первым делом происходит его парсинг. Система разбирает текст запроса на составные части, проверяет синтаксис и превращает его в абстрактное синтаксическое дерево (AST). Затем базируясь на AST, Presto выделяет таблицу и столбцы, которые упоминаются в запросе, строит внутренний план запроса, оптимизирует его, решая, какие операции и в какой последовательности будут выполнены для получения нужного результата.

**2. Распределение задач:**

   Оптимизированный план затем трансформируется в физический план, который разбивается на задания, которые можно выполнить в распределенной манере. В Presto нет единого места, где хранятся все данные, поэтому оно распределяет эти задания на рабочие узлы (worker nodes), которые будут выполнять их параллельно на различных частях данных, это позволяет системе масштабироваться и обрабатывать очень-очень большие объемы данных быстро и эффективно.

**3. Выполнение запроса:**
   
   Каждый рабочий узел выполняет свою часть работы: чтение данных, фильтрация, агрегация, соединение таблиц и другие операции, определенные в плане запроса. Результаты выполнения каждого узла затем собираются и объединяются. В процессе выполнения Presto может использовать разные алгоритмы выполнения, например, соединение хеш-таблиц или Merge отсортированных листов, выбор алгоритма производится в зависимости от количества данных и доступных вычислеительных ресурсов.

**4. Возврат результатов:**

   После того как все рабочие узлы выполнят свои задачи, и результаты будут собраны и объединены на выходе, окончательный результат возвращается аналитику.

Ниже приложу скрин из лекции про процесс выполнения запросов в Presto

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/304.png)

[Ссылка на видео](https://youtu.be/YmN_WES6o9w?si=kiHNgVF_wOljRyIO)

##  Планы запросов

В PrestoDB, как и в других распределенных SQL-системах, понятие "план запросов" играет критическую роль в оптимизации и выполнении SQL-запросов. Вот примерный план запросов, который я взял из оффициальной документации.

**1. Parsing and Analysis**

**2. Logical Planning**

**3. Optimization**

**4. Physical Planning**

**5. Execution**

## Транзакции

Изначально PrestoDB создавалась как система для выполнения аналитических запросов на больших объемах данных с высокой скоростью. Из-за упора на производительность и масштабируемость, в архитектуре PrestoDB не поддерживались полноценные транзакции, как, например, в традиционных реляционных базах данных.

В PrestoDB поддержка транзакций может быть реализована в контексте определенных коннекторов, которые поддерживают транзакции, то есть PrestoDB может работать с внешними системами хранения данных, которые поддерживают транзакции, например с Hive. Следовательно, если вам нужны транзакции, вы можете выбрать коннектор, который поддерживает транзакции.

В самом PrestoDB также реализованы функции по поддержке транзакций:

 - **START TRANSACTION**: Инициирует новую транзакцию. Этот процесс предполагает, что все следующие операции до команды COMMIT или ROLLBACK будут выполняться в рамках этой транзакции ([документация](https://prestodb.io/docs/0.286/sql/start-transaction.html#start-transaction))

- **COMMIT**: Завершает текущую транзакцию, применяя все изменения, сделанные в рамках транзакции, к целевым данным ([документация](https://prestodb.io/docs/0.286/sql/commit.html#commit))

- **ROLLBACK**: Отменяет все операции, выполненные в рамках текущей транзакции, возвращая данные к состоянию, в котором они находились до начала транзакции ([документация](https://prestodb.io/docs/0.286/sql/rollback.html#rollback))

## Методы восстановления информации

PrestoDB, как и любой SQL-query engine, не управляет напрямую хранением данных и не предоставляет собственные средства для резервного копирования или восстановления данных. Вместо этого, оно работает с данными, которые хранятся в других источниках, таких как HDFS (Hadoop Distributed File System), Amazon S3, MySQL и др. То есть PrestoDB не предлагает прямых механизмов восстановления данных и зависит от инфраструктуры и средствах внешних систем, к которым он подключается для выполнения запросов.

Таким образом, восстановление данных в случае с Presto зависит от механизмов восстановления, которые предоставляют эти внешние системы или платформы хранения данных.

## Шардинг

Аналогично прошлому пункту, PrestoDB сам по себе не управляет шардингом данных, так как это распределённый SQL-query engine, который выполняет обработку данных, лежащих в различных источниках. Шардинг обычно управляется на уровне хранилища данных, к которому подключается Presto. Таким образом, способы и типы шардинга, с которыми работает PrestoDB, зависят от конкретной системы управления базами данных или платформы, из которой Presto извлекает данные.

## Data Mining, Data Warehousing и OLAP

Я считаю, что PrestoDB может применяться в контекстах Data Mining, Data Warehousing и OLAP,учитывая его свойства как распределённого SQL-движка запросов. Рассмотрим как может применяться PrestoDB в:

1) **Data Mining** включает в себя анализ больших объемов данных для выявления различных закономерностей и связей. PrestoDB может поддерживать процессы Data Mining, обеспечивая быстрый и мощный инструмент для выполнения сложных SQL-запросов к данным из разных источников.

   Однако сам по себе Presto не включает специализированные алгоритмы машинного обучения или другие методы анализа данных, которые часто ассоциируются с Data Mining.

2) **Data Warehousing** относится к хранению больших объемов данных на долгое время для их дальнейшего анализа. Хотя Presto сам по себе не управляет физическим хранением данных, он часто используется как инструмент для выполнения запросов в средах, где данные уже хранятся, например, в больших Data Warehouses, таких как Amazon Redshift и Snowflake.

3) **OLAP** — это подход к быстрой обработке многомерных аналитических запросов. Presto поддерживает OLAP-запросы благодаря своим мощным способностям выполнения SQL-запросов и возможности обрабатывать большие объемы данных распределённым образом. Presto особенно хорош в выполнении агрегатных функций, подзапросов и джоинов, которые часто требуются в OLAP-системах.

4) ## Шифрование и другие методы защиты данных

5) PrestoDB поддерживает несколько методов защиты данных, обеспечения безопасности сетевого трафика и доступа к данным. Вот некоторые из важнейших функций безопасности, реализованные в Presto:

**1) Аутентификация**

- Поддержка LDAP: Presto может быть настроено для использования LDAP (Lightweight Directory Access Protocol) для аутентификации пользователей. Это позволяет интегрировать Presto с существующими корпоративными директориями для управления доступом ([документация](https://prestodb.github.io/docs/current/security/ldap.html))
-  Kerberos

**2) Шифрование трафика**

- TLS/SSL: Presto поддерживает шифрование внутреннего и клиентского трафика при помощи TLS (Transport Layer Security). Это обеспечивает защиту данных в процессе передачи между клиентом и сервером, а также между узлами кластера. Администраторы могут настроить Presto на использование SSL для шифрования всех сетевых соединений ([документация](http://prestodb.io/docs/current/security/internal-communication.html))

**3) Управление доступом и авторизация**

- Role-based access control (RBAC): Поддерживает ролевое управление доступом, что позволяет администраторам назначать права доступа на основе ролей пользователям ([документация](https://prestodb.io/docs/current/security/authorization.html))

## Пример запросов

Кажется этот пункт я уже сделал в рамках пункта **Развертка БД и некоторые запросы**.

Приведу еще один запрос к БД, которую я использовал в том пункте. Посчитать среднюю зарплату по возрасту у работников с первой буквой их фамилии или имени "Г", [документация](https://prestodb.io/docs/0.286/functions/comparison.html#like) проверки регулярного выражение:

```sql
SELECT 
    age,
    AVG(salary) AS avg_salary
FROM new_catalog.task_4.employees
WHERE name LIKE "Г%" OR surname LIKE "Г%"
GROUP BY age
```

## Cообщества развивающие PrestoDB

PrestoDB является проектом с открытым исходным кодом, который изначально был разработан в Facebook для анализа больших объемов данных. С тех пор сообщество разработчиков значительно расширилось. Основным ответвлением и продолжением проекта сейчас занимается Presto Foundation, учрежденная Facebook, Uber, Twitter и Alibaba. Presto Foundation работает под эгидой Linux Foundation.

Управление проектом осуществляется через обычный Git. Любой человек может сделать Pull Request, после чего, изменения в коде рассматриваются разработчиками Presto и только после их одобрения изменения могут быть добавлены в основной репозиторий. Это стандартная практика для большинства проектов с открытым исходным кодом. Вот [ссылка](https://prestodb.io/community/) на вступление в комьюнити разработчиков Presto.

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/307.png)

## Документация

Для PrestoDB есть очень хорошая и полная [оффициальная документация](https://prestodb.io/docs/current/index.html#), в которой можно найти абсолютно все: от установки БД, до вариантов подключения к ней других баз данных, синтаксиса языка запросов SQL, методов шифрования и многого другого.

![](https://github.com/tsar1in/HW_DB_Sber/blob/main/images/306.png)

## Как быть в курсе

Следить за [оффициальным сайтом](https://prestodb.io) PrestoDB, [форумом](https://prestodb.io/blog/category/prestodb/) и [документацией](https://prestodb.io/docs/current/index.html#).

Кроме того, про эту базу данных часто рассказывают на различным конференциях, вроде [такой](https://youtu.be/YmN_WES6o9w?si=kiHNgVF_wOljRyIO)
