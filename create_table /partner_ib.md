```
create table development.partner_ib
(	ib_account 		bigint not null,
	mt5_loginid		bigint, 
	"group"			varchar,
	binary_user_id	bigint,
	date_joined		date, 
	phone			varchar,
	username		varchar,
	email			varchar,
	country			varchar,
	currency		varchar,
	parent_ib		bigint,
	is_client		boolean,
	first_activity_date 	date,
	last_activity_date 		date,
CONSTRAINT partner_ib_pkey PRIMARY KEY (ib_account)
)
```