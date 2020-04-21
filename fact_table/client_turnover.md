--same problem with client_pnl
--dont need for mt5?
```sql bo
with fact as 
(select transaction_date,
 		coalesce(user_id,0) as aff_account,
 		login_id as client_loginid, 
 		--source::bigint as app_account,
 		underlying_symbol,
 		bet_type, 
 		bet_class, 
 		round(sum(total_buy_usd)::numeric,2) as turnover
from summary.bo_transaction as ts 
left join myaffiliate.customers_vw as c 
on ts.login_id = c.client_customer_id 
left join myaffiliate.customer_map_vw as cm 
on c.ma_customer_id = cm.ma_customer_id 
where transaction_date >= '2019-09-01' and transaction_date < '2019-09-02'
and total_buy_usd is not null
group by transaction_date, aff_account, client_loginid, underlying_symbol, contract_type, contract_class
)

select transaction_date, 
			aff_account as partner_account_id, 
			(case when aff_account = 0 then n'NA' else 'aff' end ) as "role",
			coalesce(c.binary_user_id,0) as client_binary_user_id, 
			coalesce(client_loginid,'0'),
			0 as plan_id, 
			0 as landing_page_id,
			0 as campaign_id,
			-- 0 as admin_id,
			0 as media_id, 
			coalesce(underlying_symbol,'NA'), 
			coalesce(bet_type,'NA'), 
			coalesce(bet_class,'NA'),
			'NA'::varchar as payment_type_code,
			'NA'::varchar as payment_gateway_code,
            --null::numeric as probability, 
			'bo' as platform,
			'client_turnover_usd' as metric, 
			turnover as "value"
from fact 
left join bo.client as c
on fact.client_loginid = c.loginid
```