В торговой Компании эксплуатируется база данных следующей структуры (скрипты по ее созданию
приведены далее):

![Alt text](/sql-select.png?raw=true "structure")

Свое решение Вы можете проверить на сайте http://sqlfiddle.com/#!4/ (Oracle) или 
https://www.db-fiddle.com/ (PostgreSQL).
Для создания тестовой базы данных (схемы) можно использовать следующий скрипт:
```
CREATE TABLE Client (ID INTEGER NOT NULL PRIMARY KEY, Name varchar(50), Surname Varchar(50), Firm_ID INTEGER NULL,
Gender Varchar(1), Birthday Date, EMail Varchar(100) NULL);
INSERT INTO Client (ID, Name, Surname, Firm_ID, Gender, Birthday, EMail) VALUES (1,'Елена', 'Сидорова', NULL, 'F',
TIMESTAMP '1993-04-05', 'E.Sidorova@mail.ru');
INSERT INTO Client (ID, Name, Surname, Firm_ID, Gender, Birthday, EMail) VALUES (2,'Петр', 'Иванов', 1, 'M',
TIMESTAMP '1990-12-02', 'P.Ivanov@mail.ru');
INSERT INTO Client (ID, Name, Surname, Firm_ID, Gender, Birthday, EMail) VALUES (3,'Anna', 'Петрова', 1, 'F',
TIMESTAMP '1993-05-13', 'A.Petrova@mail.ru');
INSERT INTO Client (ID, Name, Surname, Firm_ID, Gender, Birthday, EMail) VALUES (4,'Иван', 'Иванов', 1, 'M',
TIMESTAMP '1990-12-02', NULL);
INSERT INTO Client (ID, Name, Surname, Firm_ID, Gender, Birthday, EMail) VALUES (5,'Даниил','Пупкин', 2, 'M',
TIMESTAMP '1991-12-05', NULL);
INSERT INTO Client (ID, Name, Surname, Firm_ID, Gender, Birthday, EMail) VALUES (6,'Янина', 'Котикова', NULL, 'F',
TIMESTAMP '1991-10-20', 'Koteiko@mail.ru');
INSERT INTO Client (ID, Name, Surname, Firm_ID, Gender, Birthday, EMail) VALUES (7,'Ян', 'Шнайдмиллер', 2, 'M',
TIMESTAMP '1990-12-02', 'ya.shnaid@mail.ru');
CREATE TABLE Firm (ID INTEGER NOT NULL PRIMARY KEY, Name varchar(100), Address Varchar(100));
INSERT INTO Firm (ID, Name, Address) VALUES(1, 'ООО "Рога & Копыта"', 'Барнаул, ул. Кривая, 2');
INSERT INTO Firm (ID, Name, Address) VALUES(2, 'ПАО "Копыта & Рога"', 'Барнаул, ул. Ровная, 5');
CREATE TABLE Orders (ID INTEGER NOT NULL PRIMARY KEY, Ord_Time Date NOT NULL, Amount INTEGER NOT NULL, Client_ID
INTEGER NOT NULL);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (1, TIMESTAMP '2021-12-01 11:35:45', 115000, 1);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (2, TIMESTAMP '2021-12-02 12:25:56', 307000, 1);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (3, TIMESTAMP '2021-12-02 13:15:07', 350000, 2);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (4, TIMESTAMP '2021-12-03 14:05:18', 670000, 1);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (5, TIMESTAMP '2021-12-04 15:55:29', 70000, 3);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (6, TIMESTAMP '2021-12-04 16:45:30', 150000, 4);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (7, TIMESTAMP '2021-12-05 17:35:41', 250000, 5);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (8, TIMESTAMP '2021-12-06 18:25:52', 650000, 2);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (9, TIMESTAMP '2021-12-06 19:15:03', 450000, 6);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (10, TIMESTAMP '2022-01-10 19:15:03', 37000, 6);
INSERT INTO Orders (ID, Ord_Time, Amount, Client_ID) VALUES (11, TIMESTAMP '2022-01-11 10:15:03', 29000, 7);
COMMIT;
```
Планируется отослать поздравительные сообщения по электронной почте всем клиентам Компании на 23 Февраля и 8 Марта. Требуется одним запросом определить количество сообщений, которые нужно будет отправить 23 Февраля и 8 Марта.
```
select count(*) from client
where gender like 'F' or gender like 'M'
```
Вывести список заказов клиентов: [Фамилия + имя, название фирмы, дата заказа, сумма заказа],
отсортированный по фамилии, имени, дате заказа.
```
select client.Surname, client.Name, Firm.Name, Orders.Ord_Time
from client
left join orders on client.id = orders.id
left join firm on client.id = firm.id
 order by client.Surname, client.Name, Orders.Ord_Time
 ```
Компания намеревается ввести скидку в размере 10% от суммы заказа, если заказ выполнен в день
рождения клиента. Требуется определить общую сумму скидки по всем клиентам за 2021 год, как
если бы скидка действовала.
```
with query
as
(
  select client.id,
  to_char(client.Birthday,'2021-MM-DD') as Birthday
  from client
)
select sum(orders.Amount)*0.1 as discount
from orders, query
where query.id = orders.Client_ID and CAST(orders.Ord_Time as text) = query.Birthday
```
Перед Новым 2022-м Годом решено отправить ценные подарки руководителям фирм, сотрудники
которых за год в общей сложности сделали заказов более чем на 1 000 000 руб. Необходимо
подготовить такой список: [Название фирмы, адрес, сумма заказов].
```
with query as
(
  select sum(orders.amount), client.firm_id
  from orders
  inner join client on orders.Client_ID = client.ID
  group by client.firm_id
  having sum(amount)>1000000 and client.firm_id is not null
)
select firm.name, firm.address, query.sum
from firm, query
where firm.id = query.firm_id
```
С помощью одного SQL запроса подготовить список мужчин - потенциальных близнецов среди
клиентов (с совпадением фамилии и даты рождения): [ID клиента, фамилия, имя, день рождения]
```
with query as
(
  select surname, birthday
  from client
  group by surname, birthday
  having count(surname)>1 and count(birthday)>1
)
select client.id, client.surname, client.birthday
from client, query
where client.birthday = query.birthday and client.surname = query.surname
```
Выбрать топ-3 покупателей среди фирм (просуммировав заказы всех представителей фирмы) и
клиентов-физических лиц (у которых Firm_ID не указан) с самым большим объемом заказов за
декабрь 2021 года: [Фамилия+имя или название фирмы, сумма заказов].
```
with result as
(
  with q0 as
  (
    select *
    from orders
    where ord_time between '2021-12-01' and '2021-12-31'
  ),
  q1 as
  (
    select sum(q0.amount), Firm.Name
    from q0
    inner join client on q0.Client_ID = client.ID
    inner join firm on client.firm_id = firm.id
    group by client.firm_id, Firm.Name
    having client.firm_id is not null
  ),
  q2 as
  (
    select sum(q0.amount), concat(client.Surname,' ',client.Name)
    from q0
    inner join client on q0.Client_ID = client.ID
    group by client.firm_id, client.Surname, client.Name
    having client.firm_id is null
  )
  select * from q1
  union
  select * from q2
  order by sum desc
)
select *
from result
limit 3
```
