```sql
--pay lesser than earned because of payment eligibility / below threshold
select date as transaction_date,
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
and date >= '2019-09-01' 
and date < '2019-10-01'
group by transaction_date,partner_account_id,plan_id
```