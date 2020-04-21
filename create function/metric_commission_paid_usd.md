```
CREATE OR REPLACE FUNCTION development.update_partner_fact_commission_paid_usd(
	p_start_date date,
	p_end_date date)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$

INSERT INTO development.partner_fact AS p 
(select date as transaction_date,
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
       'commission_paid_usd' as metric,  
	   sum(amount*-1) as "value"
from myaffiliate.user_transactions_vw as trans
where "type" = 'Payment Issued' 
and date >= p_start_date::date 
and date < p_end_date::date
group by transaction_date,partner_account_id,plan_id
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