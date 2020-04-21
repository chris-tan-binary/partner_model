```
CREATE OR REPLACE FUNCTION development.update_partner_ib(
	)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$
	
insert into development.partner_ib as p 
(select ib.value::bigint as ib_account,
	   mt5_users.login::bigint as mt5_loginid, 
	   mt5_users.group::varchar as "group",
	   uli.binary_user_id::bigint,
	   ib.updated_at::date as date_joined, 
	   mt5_users.phone::varchar as phone,
	   mt5_users.name::varchar as username,
	   mt5_users.email::varchar,
	   mt5_users.country::varchar as country,
	   "group".currency::varchar,
	   mt5_users.agent::bigint as parent_ib,
	  (case when tas.binary_user_id is null then 'false'
   	     else 'true'
	     end)::boolean as is_client,
	   activity.first_activity_date::date,
	   activity.last_activity_date::date
--min because multiple affiliate user_id may have the same ib account
from (select "value", min(updated_at) as updated_at
	  from myaffiliate.user_variables_vw
			where name = 'mt5_account'
	 group by "value") as ib
left join mt5.user as mt5_users
		on ib."value"::bigint = mt5_users."login"
left join (select *, regexp_replace(login_id, '[A-Z]+', '')::bigint as mt5_loginid from bo.user_login_id
			where login_id like 'MT%' and login_id != 'MT' and login_id not like 'MTD%' ) as uli
		on ib."value"::bigint = uli.mt5_loginid
left join mt5.trading_group as "group"
		on mt5_users."group" = "group"."group"
left join (select distinct binary_user_id from summary.trading_activity) as tas 
		on uli.binary_user_id = tas.binary_user_id
left join (select agent,min(registration_ts::date) as first_activity_date,max(registration_ts::date) as last_activity_date
					from mt5.user 
					group by agent) as activity 
		on ib.value::bigint = activity.agent )

ON CONFLICT ON CONSTRAINT "partner_ib_pkey" DO UPDATE SET
        ib_account 		 = excluded.ib_account,
		mt5_loginid		 = excluded.mt5_loginid, 
		"group"			 = excluded."group"	,
		binary_user_id	 = excluded.binary_user_id,
		date_joined		 = excluded.date_joined,
		phone			 = excluded.phone,	
		username		 = excluded.username,
		email			 = excluded.email,	
		country			 = excluded.country,
		currency		 = excluded.currency,
		parent_ib		 = excluded.parent_ib,
		is_client		 = excluded.is_client,
		first_activity_date = excluded.first_activity_date,	
		last_activity_date 	= excluded.last_activity_date	
    WHERE p IS DISTINCT FROM excluded
    RETURNING *;

$BODY$;
```