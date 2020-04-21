```sql
with audit as
	(select audit.* 
	from bo.audit_payment_agent as audit
	where audit.is_authenticated = True),

join_date as
	(select client_loginid,min(stamp)::date as pa_joined_date
	from audit
	group by client_loginid),
					
activity as 
	(select pa.client_loginid, min(payment_ts)::date as first_activity_date , max(payment_ts)::date as last_activity_date
	from 
	bo.betonmarkets_payment_agent  as pa 
	left join bo.payment as pp  
	on pa.client_loginid = pp.login_id
	where payment_gateway_code = 'payment_agent_transfer'
	group by 1)
										
										
select c.binary_user_id::bigint, 
		pa.client_loginid::varchar as loginid ,
		pa_joined_date::date as date_joined,
	    pa.payment_agent_name::varchar,
		pa.url::varchar,
		pa.email::varchar,
		pa.phone::varchar,
		pa.currency_code::varchar,
		pa.commission_deposit::varchar,
	 	pa.commission_withdrawal::varchar,
	 	pa.is_authenticated::boolean,
		pa.is_listed::boolean,
		pa.target_country::varchar,
		c.residence::varchar as country,
		(case when tas.binary_user_id is null then 'false'
			else 'true'
			end)::boolean as is_client,
		activity.first_activity_date::date,
		activity.last_activity_date::date	
--into development.partner_pa
from bo.betonmarkets_payment_agent as pa
left join bo.client as c 
on pa.client_loginid = c.loginid 
left join join_date 
on pa.client_loginid = join_date.client_loginid
left join activity
on pa.client_loginid = activity.client_loginid
left join (select distinct binary_user_id from summary.trading_activity) as tas 
on c.binary_user_id = tas.binary_user_id  
order by loginid
```