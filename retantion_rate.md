### Retention Rate.

Показывает, насколько долго клиент остаётся с компанией.

Чтобы рассчитать *Retention Rate*, количество активных пользователей на текущий день делят на количество активных пользователей на первый день.




**Дата первого действия пользователя** — это может быть посещение, регистрация или любое другое событие, для которого мы хотим посчитать Retention Rate.
**Лайфтайм** — время «жизни» пользователя относительно даты старта. Лайфтайм можно указать в виде порядкового номера когорты или даты её старта.
**Значение метрики** значения, которые пригодятся для расчёта метрик, например количество пользователей или выручка. В запросе можно указать как абсолютные, так и относительные значения.



В зависимости от выбранной точки отсчёта значение *Retention Rate* может отличаться. 
```
15 июня на сайт зашли восемь с половиной тысяч новых пользователей. 
Из них чуть больше четырёх тысяч пользователей вернулись на сайт на следующий день, а около трёх тысяч пользователей — ещё через день. 
Retention Rate в зависимости от дня будет меняться.
```
![some_img](https://pictures.s3.yandex.net/resources/3.1.4_2880border_1637146266.png)


### Пример расчетов:

Каждый пользователя, старт когорты и ко-во пользователей в ег группе.  
```
WITH profile AS
  (SELECT user_id,
          dt
     FROM online_store.profiles
   WHERE channel = 'Organic'),
cohort_users_cnt AS 
  (SELECT dt,
          COUNT(user_id) AS cohort_users_cnt
   FROM online_store.profiles
   WHERE channel = 'Organic'
   GROUP BY dt)
SELECT p.user_id,
       p.dt,
       cohort_users_cnt
FROM profile p
JOIN cohort_users_cnt cuc ON p.dt = cuc.dt; 
```

Определяем дни посещения(возврата) на площадку:

```
WITH profile AS
  (SELECT user_id,
          dt,
          COUNT(*) OVER (PARTITION BY dt) AS cohort_users_cnt
   FROM online_store.profiles 
   WHERE channel = 'Organic'),
sessions AS 
(SELECT user_id,
        session_start::date AS session_date
FROM online_store.sessions 
GROUP BY 1,
         2)
SELECT *
FROM profile p JOIN sessions s ON p.user_id = s.user_id;  
```

Обьединяем и расчитываем retanton rate:

```
WITH profile AS
  (SELECT user_id,
          dt,
          COUNT(*) OVER (PARTITION BY dt) AS cohort_users_cnt
   FROM online_store.profiles 
   WHERE channel = 'Organic'),
sessions AS 
(SELECT user_id,
        session_start::date AS session_date
FROM online_store.sessions 
GROUP BY 1,
         2)
SELECT p.dt cohort_dt,
       session_date,
       COUNT(p.user_id) AS users_cnt,
       cohort_users_cnt,
       ROUND(COUNT(p.user_id) * 100.0 / cohort_users_cnt, 2) AS retention_rate
FROM profile p
JOIN sessions s ON p.user_id = s.user_id
GROUP BY 1,
         2,
         4; 
```


![ret_range](https://i.ibb.co/XSr4QbX/retantion-rate.png)
