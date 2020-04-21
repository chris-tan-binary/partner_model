```create sequence
CREATE SEQUENCE development.partner_pa_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
```


```
create table development.partner_pa
(pa_account bigint DEFAULT nextval('development.partner_pa_id_seq'::regclass) NOT NULL,
binary_user_id			bigint, 
loginid					varchar,
date_joined				date,
payment_agent_name		varchar,
url						varchar,
email					varchar,
phone					varchar,
currency_code			varchar,
commission_deposit		varchar,
commission_withdrawal	varchar,
is_authenticated		boolean,
is_listed				boolean,
target_country			varchar,
country					varchar,
is_client				boolean,
first_activity_date		date,
last_activity_date		date,
CONSTRAINT partner_pa_pkey PRIMARY KEY (loginid)
)
```
