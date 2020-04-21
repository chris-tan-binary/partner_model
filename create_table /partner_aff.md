```
create table development.partner_aff
(	aff_account 		bigint not null,
	loginid 			varchar,
	binary_user_id		bigint,
	date_joined			date, 
	ib_account			bigint,
	website				varchar,
	phone				varchar,
	skype				varchar,
	username			varchar,
	business_name		varchar, 
	email				varchar,
	country				varchar,
	admin_id			bigint,
	currency			varchar,
	"language"			varchar,
	payment_eligibility varchar,
	payment_type_id     varchar,
	payment_type_name   varchar,
	status				varchar, 
	parent_user_id		bigint,
 	is_client			boolean,
	first_activity_date	date,
	last_activity_date	date,
CONSTRAINT partner_aff_pkey PRIMARY KEY (aff_account)
)
```