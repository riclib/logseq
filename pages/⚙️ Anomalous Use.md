- #project #doing [[🏛 Tele2]]
-
- Open Tasks
  collapsed:: true
	- {{query (and (todo now later ) [[⚙️ Anomalous Use]]) }}
	  query-table:: false
### Contacts
	- Linus Andersson → Wrote original query for ITM
	- Erica Aakula → ITM
	- Viktor Norberg → snowflake
	- Pekka -> did it before
### The Query
collapsed:: true

```sql
with first_agg as (select
  call_date,
  msisdn,
  ar_key,
  zone,
  count(1) cnt,
  sum(usage_non_foc_mb) vol
from
  access_atlas.rlah_tele2_map
where
  call_date between '{replacement}dateStart{/replacement}' and '{replacement}dateEnd{/replacement}'
  and zone in ('SE','EU/EEA')
group by
  call_date,
  msisdn,
  ar_key,
  zone),

mid_agg as (
select
  call_date,
  msisdn,
  ar_key,
  count(1) cnt,
  sum(case when zone = 'EU/EEA' then vol else 0 end) eu_vol,
  sum(case when zone != 'EU/EEA' then vol else 0 end) nat_vol,
  sum(vol) tot_vol
from
  first_agg
group by
  call_date,
  ar_key,
  msisdn),

fin_agg as (
  select
      msisdn,
      ar_key,
      count(distinct(call_date)) traffic_days,
      sum(case when nat_vol > 0 and eu_vol = 0 then 1 else 0 end) se_days,
      sum(case when nat_vol = 0 and eu_vol > 0 then 1 else 0 end) eu_days,
      sum(case when eu_vol > 0 and nat_vol > 0 then 1 else 0 end) both_days,
      sum(case when eu_vol = 0 and nat_vol > 0 then nat_vol else 0 end) se_only_vol,
      sum(case when eu_vol > 0 and nat_vol = 0 then eu_vol else 0 end) eu_only_vol,
     sum(case when eu_vol > 0 and nat_vol > 0 then nat_vol else 0 end) se_both_vol,
      sum(case when eu_vol > 0 and nat_vol > 0 then eu_vol else 0 end) eu_both_vol,
      sum(nat_vol) se_vol,
      sum(eu_vol) eu_vol,
      sum(nat_vol + eu_vol) tot_vol
  from
      mid_agg
  group by
      msisdn,
      ar_key
  having
      eu_days != 0
      ),

res as(select
  msisdn,
  ar_key,
  se_days,
  eu_days,
  both_days,
  round(eu_vol,2) eu_vol,
  round(se_vol,2) se_vol,
  round(eu_days/(eu_days+se_days+both_days),2) days_ratio,
  round(eu_vol/(eu_vol+se_vol),2) vol_ratio,
  round(tot_vol,2) tot_vol
from
  fin_agg
where
  eu_days/(eu_days+se_days+both_days) > 0.5
  and eu_vol/(eu_vol+se_vol) > 0.5
  and eu_vol > 5000),

ref as (
  select
      ar_key,
      msisdn as master_msisdn,
      cust_type
  from
      access_atlas.se_subs_lookup
  where
      source_date = (select max(source_date) from access_atlas.se_subs_lookup)
      and subs_type = 'MASTER'
      and cstatus = 'ACTIVE'
      and tech_status = 'ACTIVE'
)
select distinct
  master_msisdn as msisdn
From
  res
left join
  ref
on
  res.ar_key = ref.ar_key
where
  res.eu_days>28
  and ref.cust_type = 'RESIDENTIAL'
```
- ### Snow Flake
  collapsed:: true
	- LATER Talk to Nikolaos Paroutsis with Erica Aakula
	  later:: 1628803476686
		- LATER understand how futur data is tagged, can we distinguish migrated and old stack clients?
		  later:: 1628803490924
			- DONE Policyzone → rating system → audit file → → snowflake Viktor Norberg
			  now:: 1628803571437
			  done:: 1628803571991
				- [[👤 Pekka Sjoberg]] talked to him, data is there but we may need a different query
- ### Futur Contact Data
  collapsed:: true
	- Customer contact manager... one of the alo teams. But the team probably doesn't need to do anything. Just for the person interested in the information to connect and get the data
		- Askvig -> Rebeka
### Sending msgs
	- LATER Is MC2 the right way?
	  later:: 1628803709627