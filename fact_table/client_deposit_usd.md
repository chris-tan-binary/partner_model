```sql
with deposit as 
(select payment_ts::date as transaction_date,
	   coalesce(user_id,0) as aff_account, 
 	   client.binary_user_id as client_binary_user_id,
	   pp.login_id as client_loginid, 
	   pp.currency_code, 
 	   pp.payment_type_code,
 	   pp.payment_gateway_code,
	   sum(pp.transaction_amount) as deposit
from bo.payment as pp
left join bo.client 
on pp.login_id = client.loginid
left join myaffiliate.customers_vw as c 
on pp.login_id = c.client_customer_id 
left join myaffiliate.customer_map_vw as cm 
on c.ma_customer_id = cm.ma_customer_id 
where  pp.payment_type_code != 'internal_transfer' 
and pp.payment_type_code != 'mt5_transfer' 
and pp.transaction_amount > 0 
and pp.payment_ts >= '2020-01-01'
and pp.payment_ts <  '2020-01-02'
group by 1,2,3,4,5,6,7),

fact as
(select transaction_date, 
 		aff_account, 
 		client_binary_user_id, 
 		client_loginid, 
 		payment_type_code,
 	    payment_gateway_code,
 		round((deposit  * r.rate)::numeric,2) as deposit_usd 
from deposit
join LATERAL (
    SELECT rate
          FROM bo.exchange_rate
         WHERE source_currency = deposit.currency_code
           AND target_currency = 'USD'
           AND date <= deposit.transaction_date
      ORDER BY date DESC
         LIMIT 1
) as r on true
)

select transaction_date, 
		aff_account as partner_account_id,
	    (case when aff_account = 0 then 'NA' else 'aff' end )::varchar as "role",
		coalesce(client_binary_user_id,0), 
		coalesce(client_loginid,'o'),
		0 as plan_id, 
		0 as landing_page_id,
		0 as campaign_id,
		--0 as admin_id,
		0 as media_id, 
		'NA'::varchar as underlying_symbol, 
		'NA'::varchar as bet_type, 
		'NA'::varchar as bet_class ,
		coalesce(payment_type_code,'NA'),
		coalesce(payment_gateway_code,'NA'),
		--null::numeric as probability, 
		'bo' as platform, 
		'client_deposit_usd' as metric, 
		deposit_usd as "value"
from fact
```