```
CREATE OR REPLACE FUNCTION development.update_partner_aff(
	)
    RETURNS void
    LANGUAGE 'sql'

    COST 100
    VOLATILE 
AS $BODY$
	
insert into development.partner_aff as p 
(select users.user_id::bigint as aff_account,
		login."value"::varchar as loginid,
	    uli.binary_user_id::bigint,
		join_date::date as date_joined, 
		ib."value"::bigint as ib_account,
		website."value"::varchar as website,
		phone."value"::varchar as phone,
		skype."value"::varchar as skype,
		users.username::varchar,
		business_name."value"::varchar as business_name, 
		users.email::varchar,
		lower(country)::varchar as country,
		admin_id,
		currency::varchar,
		language_code::varchar as "language",
		payment_eligibility::varchar,
		payment_type_id::varchar,
		payment_type_name::varchar,
		status::varchar, 
		parent_user_id::bigint,
		(case when tas.binary_user_id is null then false
			else true
			end)::boolean as is_client,
		activity.first_activity_date::date,
		activity.last_activity_date::date 
from myaffiliate.users_vw as users
left join (select user_id,"value" from myaffiliate.user_variables_vw 
					where trim("name") = 'affiliates_client_loginid') as login
on users.user_id = login.user_id
left join bo.user_login_id as uli
on login."value" = uli.login_id 
left join (select * from myaffiliate.user_variables_vw
		   where name = 'mt5_account') as ib
on users.user_id = ib.user_id
left join (select * from myaffiliate.user_details_vw
					 where name = 'website') as website 
on users.user_id = website.user_id 
left join (select * from myaffiliate.user_details_vw
					 where name = 'business') as business_name
on users.user_id = business_name.user_id 
left join (select * from myaffiliate.user_details_vw
					 where name = 'phone_number') as phone
on users.user_id = phone.user_id 
left join (select * from myaffiliate.user_details_vw
					 where name = 'skype') as skype
on users.user_id = skype.user_id 
left join (select distinct binary_user_id from summary.trading_activity) as tas 
on uli.binary_user_id = tas.binary_user_id
left join (select user_id, min(from_date) as first_activity_date, max(from_date) as last_activity_date
				   from myaffiliate.customer_map_vw
					 group by user_id) as activity 
on users.user_id = activity.user_id  )

ON CONFLICT ON CONSTRAINT "partner_aff_pkey" DO UPDATE SET
        aff_account 		= excluded.aff_account,	
		loginid 			= excluded.loginid ,
		binary_user_id		= excluded.binary_user_id,	
		date_joined			= excluded.date_joined,	
		ib_account			= excluded.ib_account,
		website				= excluded.website,	
		phone				= excluded.phone,	
		skype				= excluded.skype,	
		username			= excluded.username,
		business_name		= excluded.business_name,
		email				= excluded.email,
		country				= excluded.country,
		admin_id			= excluded.admin_id,
		currency			= excluded.currency	,
		"language"			= excluded."language",
		payment_eligibility = excluded.payment_eligibility,
		payment_type_id     = excluded.payment_type_id ,
		payment_type_name   = excluded.payment_type_name,
		status				= excluded.status,
		parent_user_id		= excluded.parent_user_id,
 		is_client			= excluded.is_client,
		first_activity_date	= excluded.first_activity_date,
		last_activity_date	= excluded.last_activity_date
    WHERE p IS DISTINCT FROM excluded
    RETURNING *;

$BODY$;
```