```
create table development.partner_app
(	app_account			bigint,
	binary_user_id		bigint, 
	date_joined			date, 
	app_name			varchar, 
	homepage			varchar,
	github				varchar,
	appstore			varchar,
	googleplay			varchar,
	redirect_uri		varchar,
	bypass_verification	varchar,
	email				varchar,
	app_markup_percentage varchar,
	country				varchar, 
	is_client			boolean,
    first_activity_date	date,
	last_activity_date	date,
CONSTRAINT partner_app_pkey PRIMARY KEY (app_account)
)
```