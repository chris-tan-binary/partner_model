```
CREATE OR REPLACE FUNCTION development.update_partner_app(
	)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$
	
insert into development.partner_app as p 
(with date_joined as 
(select app."id", least(app.stamp,trade.min_trade)::date as date_joined
 from bo.oauth_apps as app
 left join (select min(trade_date) as min_trade, app_id
			from summary.trading_activity
 			group by app_id) as trade
 on app."id"::varchar = trade.app_id)
 
 
select app."id"::bigint as app_account,
	app.binary_user_id::bigint, 
	date_joined::date, 
	app.name::varchar as app_name, 
	homepage::varchar,
	github::varchar,
	appstore::varchar,
	googleplay::varchar,
	redirect_uri::varchar,
	bypass_verification::varchar,
	email::varchar,
	app_markup_percentage::varchar,
	lower(country)::varchar as country, 
	(case when tas.binary_user_id is null then 'false'
	else 'true'
	end)::boolean as is_client,
    first_activity_date::date,
	last_activity_date::date
from bo.oauth_apps as app 
--get unique binary_user_id details
left join (select email,country,binary_user_id from
	  			(select row_number () over(partition by binary_user_id order by date_joined) as rn, email,residence as country,binary_user_id 
				 from bo.client)as tmp
	 	   where rn = 1 ) as details
on app.binary_user_id = details.binary_user_id
left join (select distinct binary_user_id from summary.trading_activity) as tas 
on app.binary_user_id = tas.binary_user_id
left join (select app_id, min(trade_date) as first_activity_date, max(trade_date) as last_activity_date
					 from summary.trading_activity
					group by app_id) as activity
on app."id"::varchar = activity.app_id
left join date_joined
on app."id" = date_joined."id" )

ON CONFLICT ON CONSTRAINT "partner_app_pkey" DO UPDATE SET
        app_account				= excluded.app_account,	
		binary_user_id			= excluded.binary_user_id,
		date_joined				= excluded.date_joined,
		app_name				= excluded.app_name,
		homepage				= excluded.homepage,
		github					= excluded.github,	
		appstore				= excluded.appstore,
		googleplay				= excluded.googleplay,
		redirect_uri			= excluded.redirect_uri,
		bypass_verification		= excluded.bypass_verification,
		email					= excluded.email,		
		app_markup_percentage 	= excluded.app_markup_percentage,
		country					= excluded.country,	 
		is_client				= excluded.is_client,
   		first_activity_date		= excluded.first_activity_date,	
		last_activity_date		= excluded.last_activity_date
    WHERE p IS DISTINCT FROM excluded
    RETURNING *;

$BODY$;
```