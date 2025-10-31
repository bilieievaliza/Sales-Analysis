# Sales-Analysis#ForPirtfolio
with revenue_usd as (
select
s.date,
sum(p.price) as revenue
from `DA.order` o
join `DA.product` p
on o.item_id=p.item_id
join `DA.session` s 
on o.ga_session_id = s.ga_session_id
group by s.date),

email_metrics as (
select
date_add(s.date, INTERVAL ems.sent_date DAY) as sent_date,
count(distinct ems.id_message) as sent_msg,
count(distinct eo.id_message) as open_msg,
count(distinct ev.id_message) as click_msg
from `DA.email_sent` ems
join `DA.account_session` acs 
on ems.id_account = acs.account_id
join `DA.session` s 
on acs.ga_session_id = s.ga_session_id
left join `DA.email_open` eo
on ems.id_message = eo.id_message
left join `DA.email_visit` ev
on ems.id_message = ev.id_message
group by date_add(s.date, INTERVAL ems.sent_date DAY)
),

registrations as (
select
s.date,
count (ac.account_id) as account_cnt
from `DA.account_session` ac 
join `DA.session` s 
on ac.ga_session_id = s.ga_session_id
group by s.date 
),

final as (
select date,
revenue,
0 as cost,
0 sent_msg,
0 as open_msg,
0 as click_msg,
0 as account_cnt
from revenue_usd
union all 
select
date,
0 as revenue,
cost,
0 as sent_msg,
0 as open_msg,
0 as click_msg,
0 as account_cnt
from `DA.paid_search_cost`
union all 
select
sent_date as date,
0 as revenue,
0 as cost,
sent_msg,
open_msg ,
click_msg,
0 as account_cnt
from email_metrics
union all 
select
date,
0 as revenue,
0 as cost,
0 as sent_msg,
0 as open_msg,
0 as click_msg,
account_cnt
from registrations
)

select
date,
sum(revenue) as revenue,
sum(cost) as cost,
sum(sent_msg) as sent_msg,
case 
  when sum(sent_msg) > 0 then sum(open_msg) / sum(sent_msg) 
  else 0 
end as open_rate,
case 
  when sum(sent_msg) > 0 then sum(click_msg) / sum(sent_msg) 
  else 0 
end as click_rate,
sum(account_cnt) as registrations
from final
group by date
order by date;
