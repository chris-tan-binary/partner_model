```
CREATE OR REPLACE FUNCTION development.update_partner_fact_client_withdrawal_usd(
	p_start_date date,
	p_end_date date)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$

INSERT INTO development.partner_fact AS p 
(with withdrawal as 
(select payment_ts::date as transaction_date,
	   coalesce(user_id,0) as aff_account, 
 	   client.binary_user_id as client_binary_user_id,
	   pp.login_id as client_loginid, 
	   pp.currency_code, 
 	   pp.payment_type_code,
 	   pp.payment_gateway_code,
	   -1 * sum(pp.transaction_amount) as withdrawal
from bo.payment as pp
left join bo.client 
on pp.login_id = client.loginid
left join myaffiliate.customers_vw as c 
on pp.login_id = c.client_customer_id 
left join myaffiliate.customer_map_vw as cm 
on c.ma_customer_id = cm.ma_customer_id 
where  pp.payment_type_code != 'internal_transfer' 
and pp.payment_type_code != 'mt5_transfer' 
and pp.transaction_amount < 0 
and pp.payment_ts >= p_start_date::date
and pp.payment_ts <  p_end_date::date
group by 1,2,3,4,5,6,7),

fact as
(select transaction_date, 
 		aff_account, 
 		client_binary_user_id, 
 		client_loginid, 
 		payment_type_code,
 	    payment_gateway_code,
 		round((withdrawal  * r.rate)::numeric,2) as withdrawal_usd 
from withdrawal
join LATERAL (
    SELECT rate
          FROM bo.exchange_rate
         WHERE source_currency = withdrawal.currency_code
           AND target_currency = 'USD'
           AND date <= withdrawal.transaction_date
      ORDER BY date DESC
         LIMIT 1
) as r on true
)

select transaction_date, 
		aff_account as partner_account_id,
	    (case when aff_account = 0 then 'NA' else 'aff' end )::varchar as "role",
		coalesce(client_binary_user_id,0), 
		coalesce(client_loginid,'0'),
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
		'client_withdrawal_usd' as metric, 
		withdrawal_usd as "value"
from fact
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