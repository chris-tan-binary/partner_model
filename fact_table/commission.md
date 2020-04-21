```sql aff
--myaffiliate calculated the commission in usd
select date as transaction_date,
	   user_id::bigint as partner_account_id,
 	   'aff' as "role",
	   0::bigint as client_binary_user_id, 
	   '0'::varchar as client_loginid,
	   coalesce(plan_id,0), 
	   0 as landing_page_id,
	   0 as campaign_id,
	   --0 as admin_id,
	   0 as media_id, 
	   'NA'::varchar as underlying_symbol, 
	   'NA'::varchar as bet_type, 
	   'NA'::varchar as bet_class,
	   'NA'::varchar as payment_type_code,
	   'NA'::varchar as payment_gateway_code,
       --null::numeric as probability,
	   'NA'::varchar as platform, 
       'commission_usd' as metric,  
	   sum(amount) as "value"
from myaffiliate.user_transactions_vw as trans
where "type" = 'Commission Earned' 
and date >= '2019-09-01' 
and date < '2019-10-01' 
group by transaction_date,partner_account_id,plan_id
```

```sql ib
with commission as 
(select transaction_date,
				login_id as partner_account_id,
		 		currency,
				sum(profit)  as commission
		from summary.mt5_deal as deals 
		left join mt5.user as users 
		on deals.login_id = users.login 
		left join mt5.trading_group as "group"
		on users."group" = "group"."group"
		where "action" = 10
		and transaction_date >= '2019-09-01' 
		and transaction_date < '2019-10-01'
		group by transaction_date,partner_account_id,currency)
		
select transaction_date,
	   partner_account_id, 
	   'ib' as "role",
	   0::bigint as client_binary_user_id, 
	   '0'::varchar as client_loginid,
	   0 as plan_id, 
	   0 as landing_page_id,
	   0 as campaign_id	,			
	   --0 as admin_id,
	   0 as media_id, 		
	   'NA'::varchar as underlying_symbol, 
	   'NA'::varchar as bet_type, 		
	   'NA'::varchar as bet_class,
	   'NA'::varchar as payment_type_code,
	   'NA'::varchar as payment_gateway_code,
	   -- null::numeric as probability,
	   'NA'::varchar as platform,
	   'commission_usd' as metric, 
	   round((commission  * r.rate)::numeric,2) as "value"
from commission
join LATERAL (
    SELECT rate
          FROM bo.exchange_rate
         WHERE source_currency = commission.currency
           AND target_currency = 'USD'
           AND date <= commission.transaction_date
      ORDER BY date DESC
         LIMIT 1
) as r on true
```
