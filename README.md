## Домашнее задание к занятию «Индексы»  
Инструкция по выполнению домашнего задания  
Сделайте fork репозитория c шаблоном решения к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).  
Выполните клонирование этого репозитория к себе на ПК с помощью команды git clone.  
Выполните домашнее задание и заполните у себя локально этот файл README.md:  
впишите вверху название занятия и ваши фамилию и имя;  
в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;  
для корректного добавления скриншотов воспользуйтесь инструкцией «Как вставить скриншот в шаблон с решением»;  
при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в инструкции по MarkDown.  
После завершения работы над домашним заданием сделайте коммит (git commit -m "comment") и отправьте его на Github (git push origin).  
Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.  
Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.  
Желаем успехов в выполнении домашнего задания.  

## Задание 1  
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц. 

## Ответ 1  
<img width="782" height="147" alt="Снимок306" src="https://github.com/user-attachments/assets/edb25fa1-c5bf-4df5-bb9a-741aa707d810" />




## Задание 2  
Выполните explain analyze следующего запроса:  
 ```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
 ``` 
перечислите узкие места;  
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.  
## Ответ 2
<img width="904" height="568" alt="Снимок307" src="https://github.com/user-attachments/assets/8252cb0e-4a7f-4022-956c-b16ca821f0f9" />

<img width="1556" height="521" alt="Снимок308" src="https://github.com/user-attachments/assets/1aa6d107-39f5-466b-87b1-9bc42cdf2782" />

Время отработки очень большое, много лишних операций

Узкие места следующие: 
использование функции DATE() на столбце payment_date не позволит использовать индекс по payment_date;  
запрос использует неявные соединения таблиц через запятую; 
таблица film не нужна


Надо создать индекс для даты платежа  
 ```
create index day_of_payment on payment(payment_date);
 ```
и не использовать таблицу film  
и используем JOIN  
 ```
SELECT 
    CONCAT(c.last_name, ' ', c.first_name) AS Клиент, 
    SUM(p.amount) AS Платеж
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id 
JOIN payment p ON r.rental_id = p.rental_id  
WHERE p.payment_date >= '2005-07-30' 
  AND p.payment_date < '2005-07-31'  
GROUP BY c.customer_id, c.last_name, c.first_name;  
 ```

и теперь совсем другое дело   
<img width="615" height="163" alt="Снимок311" src="https://github.com/user-attachments/assets/a5c6b7a4-710d-4941-851e-21aa49a526cd" />

<img width="1545" height="323" alt="Снимок312" src="https://github.com/user-attachments/assets/5ce88169-ec08-4545-8a09-809ace51fbcc" />
