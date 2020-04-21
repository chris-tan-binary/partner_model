bo
```
CREATE OR REPLACE FUNCTION development.update_partner_fact_client_signup(
	p_start_date date,
	p_end_date date)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$

INSERT INTO development.partner_fact AS p 
(with fact as 
( select date_joined::date as transaction_date, coalesce(cm.user_id,0)::bigint as aff_account,loginid,binary_user_id,count(distinct c.loginid) as client_signup
 	from bo.client as c 
 	left join myaffiliate.customers_vw as customer  
 	on c.loginid = customer.client_customer_id
 	left join myaffiliate.customer_map_vw as cm
 	on customer.ma_customer_id = cm.ma_customer_id
	where date_joined >= p_start_date::date
	and date_joined  < p_end_date::date
	group by transaction_date, aff_account,loginid,binary_user_id )  

select transaction_date, 
		aff_account as partner_account_id, 
		(case when aff_account = 0 then 'NA' else 'aff' end ) as "role",
		 coalesce(binary_user_id,0) as client_binary_user_id, 
		 coalesce(loginid,'0') as client_loginid,
			0 as plan_id, 
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
 			'bo' as platform,
			'client_signup' as metric, 
			client_signup as "value"
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

mt5
```
CREATE OR REPLACE FUNCTION development.update_partner_fact_client_signup_mt5(
	p_start_date date,
	p_end_date date)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$

INSERT INTO development.partner_fact AS p 
(with mt5_account as 
(select *, regexp_replace(login_id, '[A-Z]+', '')::bigint as mt5_loginid from bo.user_login_id
			where login_id like 'MT%' and login_id != 'MT' and login_id not like 'MTD%'),

fact as 
( select registration_ts::date as transaction_date, agent::bigint as ib_account,login,binary_user_id,count(distinct login) as client_signup
 	from mt5.user as users 
 	left join mt5_account
 	on users.login = mt5_account.mt5_loginid
	where registration_ts >= p_start_date::date
	and registration_ts  < p_end_date::date
	group by transaction_date, ib_account,login,binary_user_id ) 

select transaction_date, 
		ib_account as partner_account_id, 
		(case when ib_account = 0 then 'NA' else 'ib' end ) as "role",
		 coalesce(binary_user_id,0) as client_binary_user_id, 
		 coalesce("login",'0') as client_loginid,
			0 as plan_id, 
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
			 'mt5' as platform,
			'client_signup' as metric, 
			client_signup as "value"
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
