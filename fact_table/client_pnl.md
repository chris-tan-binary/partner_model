--mapping the transaction with current affiliate, to improve, map to client within the range
```sql bo
with fact as 
(select transaction_date,
 		coalesce(user_id,0) as aff_account,
 		login_id as client_loginid, 
 		--source::bigint as app_account,
 		underlying_symbol,
 		contract_type as bet_type, 
 		contract_class as bet_class, 
 		round(sum(coalesce(total_sell_usd,0) - coalesce(total_buy_usd,0))::numeric,2) as pnl
from summary.bo_transaction as ts 
left join myaffiliate.customers_vw as c 
on ts.login_id = c.client_customer_id 
left join myaffiliate.customer_map_vw as cm 
on c.ma_customer_id = cm.ma_customer_id 
where transaction_date >= '2019-09-01' and transaction_date < '2019-09-02'
and (total_buy_usd is not null or total_sell_usd is not null)
group by transaction_date, aff_account, client_loginid, underlying_symbol, contract_type, contract_class
)

select transaction_date::date, 
			aff_account::bigint as partner_account_id, 
			(case when aff_account = 0 then 'NA' else 'aff' end )::varchar as "role",
			coalesce(c.binary_user_id::bigint,0) as client_binary_user_id, 
			coalesce(client_loginid,'0')::varchar,
			0::int as plan_id, 
			0::int as landing_page_id,
			0::int as campaign_id,
			--0 as admin_id,
			0::int as media_id, 
			coalesce(underlying_symbol,'NA')::varchar, 
			coalesce(bet_type,'NA')::varchar , 
			coalesce(bet_class,'NA')::varchar ,
			'NA'::varchar as payment_type_code,
			'NA'::varchar as payment_gateway_code,
			--null::numeric as probability, 
			'bo'::varchar as platform,
			'client_pnl_usd'::varchar as metric, 
			pnl::numeric as "value"
from fact 
left join bo.client as c
on fact.client_loginid = c.loginid 
```

--should I put MT infront of client_loginid?
```sql mt5 
with pnl as 
(select  deals.transaction_date, 
 		 deals.login_id as client_loginid,
		 deals.symbol, 
		 users.agent as ib_account, 
		 "group".currency, 
		 sum(deals.profit)  as pnl
 from summary.mt5_deal as deals
 left join mt5.user as users 
	on deals.login_id = users.login
 left join mt5.trading_group as "group"
	on users."group" = "group"."group"	 
 where "action" in (0,1) 
	and transaction_date >= '2019-09-01'
	and transaction_date < '2019-10-01'
 group by transaction_date, deals.login_id , symbol, agent, currency),
		
fact as
(select transaction_date, 
 		ib_account, 
 		binary_user_id as client_binary_user_id, 
 		client_loginid, 
 		symbol as underlying_symbol,
 		round((pnl  * r.rate)::numeric,2) as pnl_usd 
from pnl
left join (select *, regexp_replace(login_id, '[A-Z]+', '')::bigint as mt5_loginid from bo.user_login_id
			where login_id like 'MT%' and login_id != 'MT' and login_id not like 'MTD%' ) as uli
on pnl.client_loginid = uli.mt5_loginid 
join LATERAL (
    SELECT rate
          FROM bo.exchange_rate
         WHERE source_currency = pnl.currency
           AND target_currency = 'USD'
           AND date <= pnl.transaction_date
      ORDER BY date DESC
         LIMIT 1
) as r on true
)

insert into development.partner_fact
(select transaction_date, 
		ib_account as partner_account_id,
	    (case when ib_account = 0 then null else 'ib' end) as "role",
		client_binary_user_id, 
		client_loginid,
		0 as plan_id, 
		0 as landing_page_id,
		0 as campaign_id,
		--0 as admin_id,
		0 as media_id, 
		underlying_symbol, 
		null::varchar as contract_type, 
		null::varchar as contract_class ,
		null::varchar as payment_type_code,
		null::varchar as payment_gateway_code,
		--null::numeric as probability, 
		'mt5' as platform, 
		'client_pnl_usd' as metric, 
		pnl_usd as "value"
from fact )
```