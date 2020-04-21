```sql
-- get first stamp or first trade activity
with date_joined as 
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
--into development.partner_app
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
on app."id" = date_joined."id"
```