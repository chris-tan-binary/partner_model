```
CREATE OR REPLACE FUNCTION development.update_partner_fact_myaf_statistics(
	p_start_date date,
	p_end_date date)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$

INSERT INTO development.partner_fact AS p 
(select date as transaction_date, 
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
where date >= p_start_date::date and date < p_end_date::date
group by 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16
 )

ON CONFLICT ON CONSTRAINT "partner_fact_pkey" DO UPDATE SET
    transaction_date			= excluded.transaction_date, 
	partner_account_id			= excluded.partner_account_id, 
	"role"						= excluded."role",
	client_binary_user_id		= excluded.client_binary_user_id, 
	client_loginid				= excluded.client_loginid,
	plan_id						= excluded.plan_id, 
	landing_page_id				= excluded.landing_page_id,
	campaign_id					= excluded.campaign_id,
	media_id					= excluded.media_id, 
	underlying_symbol			= excluded.underlying_symbol, 
	bet_type					= excluded.bet_type, 
	bet_class					= excluded.bet_class,
	payment_type_code			= excluded.payment_type_code,
	payment_gateway_code		= excluded.payment_gateway_code,
	platform					= excluded.platform,
	metric						= excluded.metric,
	"value"						= excluded."value"
    WHERE p IS DISTINCT FROM excluded
    RETURNING *;

$BODY$;
```