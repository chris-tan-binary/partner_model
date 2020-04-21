```sql
select date as transaction_date, 
	   user_id as partner_account_id ,
	   'aff' as roles,
	   coalesce(client.binary_user_id,0) as client_binary_user_id, 
	   coalesce(c.client_customer_id,'0') as client_loginid,
	   plan_id, 
	   landing_page_id, 
	   campaign_id,
	   media_id,
	   'NA'::varchar as underlying_symbol, 
	   'NA'::varchar as bet_type, 
	   'NA'::varchar as bet_class,
	   'NA'::varchar as payment_type_code,
	   'NA'::varchar as payment_gateway_code,  
	   'bo' as platform,
	   'myaf_'||operation_name as metric, 
	   sum("count") as "value"
from myaffiliate.statistics as s
left join myaffiliate.operations_vw as o 
on s.operation_id =  o.operation_id 
left join myaffiliate.customers_vw as c
on s.ma_customer_id = c.ma_customer_id
left join bo.client 
on c.client_customer_id = client.loginid 
where date >= '2019-09-01' and date < '2019-10-01' 
group by 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16
```
