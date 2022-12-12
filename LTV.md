### Когортный анализ LTV
-----

Чаще всего *LTV* считают в динамике лайфтайма — в разные периоды «жизни» пользователя.
В примере будет показан *LTV* в разрезе по каналу в первые семь дней с момента первого посещения пользователя

Для каждого пользователя нужно определить дату первого посещения, от которой будет рассчитываться лайфтайм.
```
SELECT p.user_id,
       MIN(session_start)::date AS first_session_dt
FROM online_store.profiles p
JOIN online_store.orders o ON p.user_id = o.user_id
JOIN online_store.sessions s ON p.user_id = s.user_id
GROUP BY 1; 
```

Для каждой покупки можно рассчитать лайфтайм, чтобы понять, в какой день с момента первого посещения пользователь её совершил, и рассчитать LTV этого дня. 
```
WITH profile AS
  (SELECT p.user_id,
          MIN(session_start)::date AS first_session_dt
   FROM online_store.profiles p
   JOIN online_store.orders o ON p.user_id = o.user_id
   JOIN online_store.sessions s ON p.user_id = s.user_id
   GROUP BY 1)
SELECT *,
       EXTRACT(DAY FROM AGE(o.event_dt, p.first_session_dt)) AS lifetime
FROM profile p
JOIN online_store.orders o ON p.user_id = o.user_id; 
```

Чаще в выборках для макетинга необходим канал привлечения пользователя,
и суммарная стоимость заказов с накопением.


```
WITH profile AS
  (SELECT p.user_id,
          p.channel,
          MIN(session_start)::date AS first_session_dt
   FROM online_store.profiles p
   JOIN online_store.orders o ON p.user_id = o.user_id
   JOIN online_store.sessions s ON p.user_id = s.user_id
   GROUP BY 1,
            2)
SELECT p.user_id,
       p.channel,
       o.order_amt,
       o.event_dt,
       p.first_session_dt,
       EXTRACT(DAY FROM AGE(o.event_dt, p.first_session_dt)) AS livetime,
       SUM(o.revenue) OVER (PARTITION BY p.user_id ORDER BY o.event_dt) AS ltv
FROM profile p
JOIN online_store.orders o ON p.user_id = o.user_id; 
```


