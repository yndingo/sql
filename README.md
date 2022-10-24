

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
