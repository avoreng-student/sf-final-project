Задание 1
with r_retention as (
select u2.id as user_id, to_char(u2.date_joined, 'YYYY-MM') as cohort,
extract (days from entry_at - date_joined) as diff
from userentry u 
join users u2 
on u.user_id = u2.id
where to_char(u2.date_joined, 'YYYY-MM') >= '2022-01')
select cohort,
	round(count (distinct case when diff >= 0 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end), 2) as "0 day",
	round(count (distinct case when diff >= 1 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end),2) as "1 day",
	round(count (distinct case when diff >= 3 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end),2) as "3 day",
	round(count (distinct case when diff >= 7 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end),2) as "7 day",
	round(count (distinct case when diff >= 14 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end),2) as "14 day",
	round(count (distinct case when diff >= 30 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end),2) as "30 day",
	round(count (distinct case when diff >= 60 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end),2) as "60 day",
	round(count (distinct case when diff >= 90 then user_id end)*100.0/count (distinct case when diff >= 0 then user_id end),2) as "90 day"
from r_retention 
group by cohort 

Задание 2
with a as(
select 
user_id, round(avg(value)*1.0, 2) as avg_out 
from transaction
where type_id in (1, 23, 24, 25, 26, 27, 28, 30)
group by user_id
order by user_id),
b as (
select t.user_id, avg_out, round(avg(value)*1.0) as avg_in
from a
join "transaction" t 
on a.user_id = t.user_id 
where t.type_id not in (1, 23, 24, 25, 26, 27, 28, 30)
group by t.user_id, a.avg_out ),
c as (
select b.user_id, b.avg_out, b.avg_in, round(avg(t.value)*1.0, 2) as avg_general
from b
join "transaction" t 
on b.user_id = t.user_id 
group by b.user_id, b.avg_out, b.avg_in),
d as (
select c.user_id, c.avg_out, c.avg_in, c.avg_general,  sum(case when type_id in (1, 23, 24, 25, 26, 27, 28, 30) then -value else value end) as balance
from c 
join "transaction" t  
on c.user_id = t.user_id
group by c.user_id, c.avg_out, c.avg_in, c.avg_general
)
select PERCENTILE_CONT(0.5) within group (order by balance) as median 
from d

*Не совсем поняла, как вывести все требуемые столбцы в один запрос, тогда медиана очень сильно скачет. Еще почему-то значения, полученные через функции PERCENTILE_CONT и MODE разные, решила оставить ту функцию, которая была в разборе. 


