### Когортный анализ Churn Rate
----
**Churn Rate**, Обратная _Retention Rate_. 
Эта метрика показывает, какой процент пользователей прекращает использовать сервис с течением времени.


Расчитываем во временной таблице 
_дату_, когда пользователь впервые посетил сайт.
сколько _уникальных пользователей_ посетили сайт, в зависимости от даты старта когорты и даты самого посещения

```
WITH sessions AS
  (SELECT p.dt AS start_dt,
          s.session_start::date AS session_dt,
          COUNT(DISTINCT p.user_id) AS users_cnt
   FROM online_store.profiles p
   JOIN online_store.sessions s ON p.user_id = s.user_id
   WHERE p.channel = 'Organic'
   GROUP BY 1,
            2) 
```

Чтобы рассчитать отток для каждого дня, нужно знать количество пользователей в предыдущий день. 
**LAG()** — с её помощью можно найти количество пользователей в предыдущий день для каждой когорты. 
**Churn Rate** — количество пользователей за текущий период разделить на количество пользователей за предшествующий период и результат вычесть из единицы, умножив на 100. 

```
WITH sessions AS
  (SELECT p.dt AS start_dt,
          s.session_start::date AS session_dt,
          COUNT(DISTINCT p.user_id) AS users_cnt
   FROM online_store.profiles p
   JOIN online_store.sessions s ON p.user_id = s.user_id
   WHERE p.channel = 'Organic'
   GROUP BY 1,
            2)
SELECT *,
       LAG(users_cnt) OVER (PARTITION BY start_dt ORDER BY session_dt) AS previous_day_users_cnt,
       ROUND((1 - (users_cnt::numeric/ LAG(users_cnt) OVER (PARTITION BY start_dt ORDER BY session_dt))) * 100, 2) AS churn_rate
FROM sessions; 
```
