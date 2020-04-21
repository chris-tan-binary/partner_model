```sql
--join 2 table to get pa_account and client_account
with deposit as 
(select pp1.payment_ts::date as transaction_date,
	   partner_pa.pa_account as pa_account, 
 	   client.binary_user_id as client_binary_user_id,
	   pp2.login_id as client_loginid, 
	   pp1.currency_code, 
 	   pp1.payment_type_code,
 	   pp1.payment_gateway_code,
	   -1 * sum(pp1.transaction_amount) as deposit
from bo.payment as pp1
join bo.payment as pp2
on pp1.remark = pp2.remark and pp1.login_id != pp2.login_id
left join bo.client 
on pp2.login_id = client.loginid
left join development.partner_pa
on pp1.login_id = partner_pa.loginid 
where  pp1.payment_gateway_code = 'payment_agent_transfer' 
and lower(pp1.remark) like '%from payment agent%' 
and pp1.transaction_amount <0 
and pp1.payment_ts >= '2020-01-01'
and pp1.payment_ts <  '2020-02-01'
group by 1,2,3,4,5,6,7),

fact as
(select transaction_date, 
 		pa_account, 
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
		pa_account as partner_account_id,
	    'pa' as "role",
		coalesce(client_binary_user_id,0), 
		coalesce(client_loginid,'0'),
		0 as plan_id, 
		0 as landing_page_id,
		0 as campaign_id,
		--0 as admin_id,
		0 as media_id, 
		'NA'::varchar as underlying_symbol, 
		'NA'::varchar as bet_type, 
		'NA'::varchar as bett_class ,
		coalesce(payment_type_code,'NA'),
		coalesce(payment_gateway_code,'NA'),
		--null::numeric as probability, 
		'bo' as platform, 
		'pa_deposit_usd' as metric, 
		deposit_usd as "value"
from fact
```