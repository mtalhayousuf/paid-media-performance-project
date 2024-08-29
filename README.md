# Paid Media Performance Project

### Project Overview
---

This Power BI report provides an in-depth analysis of the effectiveness of various paid media platforms, including Google, Facebook, and LinkedIn, in generating leads and converting them into enrollments for IT programs. Leveraging PostgreSQL for data preparation and analysis.

![channelperformance](https://github.com/user-attachments/assets/687b7bd3-bdb6-4aa4-ba40-64063d9a85ed)

### Data Sources

- CRM Data: (Leads, Opportunities, and Enrollments): Retrieve CRM data to display daily leads and enrollments.
- Facebook/Google/Linkedin Ads Data: Captures advertising performance and lead generation from their campaigns.
- Bridge Table: Links data across different sources for a unified analysis and reporting.

### Tools

- PostgreSQL - Data Cleaning/Data Preparing
- PowerBI - Creating reports

### Data Cleaning/Preparation

In the initial data preparation phase, we performed the following tasks:
- Data loading and inspection.
- Handling missing values.
- Data cleaning and formatting.

### Exploratory Data Analysis

EDA involved exploring the data to answer key questions, such as:

- What is the trend in lead generation across different media platforms?
- Which media platform generates the most leads?
- How does the conversion rate from leads to enrollments vary by platform?
- What is the daily pattern of lead generation and enrollments?
- Are there any seasonal trends or peak periods in enrollments?

### Lead Data Analysis

Created Lead data SQL view and get it into the Power BI report

```
select
	ld.email as Email,
    CASE
        WHEN EXTRACT(DAY FROM (ld.createdreengageddate - ld.leaddate)) > 365 THEN (ld.createdreengageddate - INTERVAL '5 hours')::timestamp
        ELSE (ld.leaddate - INTERVAL '5 hours')::timestamp
    END AS "Lead Date",
    CASE
        WHEN EXTRACT(DAY FROM (ld.createdreengageddate - ld.leaddate)) > 365 THEN date_trunc('month', (ld.createdreengageddate - INTERVAL '5 hours'))::date
    ELSE date_trunc('month', (ld.leaddate - INTERVAL '5 hours'))::date
    END AS "Lead Month",
    COALESCE(NVL(ld.brand, br.brand), 'Unbranded') as Brand,
    ld.program as Program,
    ld.utmmedium as "UTM Medium",
    COALESCE(NVL(ld.brand, br.brand), 'Unbranded') || ' ' || ld.program || ' ' || ld.utmmedium  || ' ' || 
    CASE
        WHEN EXTRACT(DAY FROM (ld.createdreengageddate - ld.leaddate)) > 365 THEN TO_CHAR((ld.createdreengageddate - INTERVAL '5 hours'), 'FMMM/FMDD/YYYY')
        ELSE TO_CHAR((ld.leaddate - INTERVAL '5 hours'), 'FMMM/FMDD/YYYY')
    END AS "BP ID",
	ld.rating as Rating
from
    v1_crm_lead_report ld
left join v1_crm_brand_name_value br on ld.brand_value = br.value
left join v1_crm_lead_owner_name un on ld.ownerguid = un.ownerid 
where
	"Lead Date" >= '2022-01-01'
	and (ld.leadtype like '%B2C%' or ld.leadtype is null)
	and (ld.pagevisited not like '%webinar%' or ld.pagevisited is null)
	and (ld.pagevisited not like '%practice-test%' or ld.pagevisited is null)
	and (un.ownername not like '%Hayden Jones%' or un.ownername is null)
	and (un.ownername not like '%Louie Tran%' or un.ownername is null)
	and (ld.country not like '%Pakistan%' or ld.country is null)
    and ld.email not like '%@%test%'
    and ld.email not like 'testing%@%'
    and ld.email not like '%@%testing%'
    and ld.email not like '%@%qs%'
    and ld.email not like '%@%quickstart%'
    and ld.email not like '%johndoe@%'
    and ld.email not like 'john@%'
    and ld.email not like 'johndoe@%'
    and ld.email not like '%demotest%'
    and ld.email not like '%@ramenpeddler%'
    and ld.email not like '%mailinator%'
order by "Lead Date")
```

### Enrollment Data Analysis

Created Enrollments data SQL view and get it into the Power BI report

```SELECT 
	lower(vpa.email) as Email,
	vpa.createdon - INTERVAL '5 hours' as "Enrolment Date",
	date_trunc('month', vpa.createdon - INTERVAL '5 hours')::date as "Enrolment Month",
    COALESCE(NVL(ld.brand, br.brand), 'Unbranded') as Brand,
    case when vpa.tenure like '%WIOA%' then 'WIOA' 
    	 when vpa.tenure like '%ArmyCOOL%' then 'Army Cool'
   		 when vpa.tenure like '%Ortega Voucher%' then 'WIOA'
    	 when vpa.tenure like '%ArmyIgnitEd%' then 'ArmyIgnitEd'
   		 when vpa.tenure like '%MyCAA%' then 'MYCAA'
    else ld.program end as Program, 
    ld.utmmedium as "UTM Medium", 
    COALESCE(NVL(ld.brand, br.brand), 'Unbranded') || ' ' || 
    case when vpa.tenure like '%WIOA%' then 'WIOA' 
    	 when vpa.tenure like '%ArmyCOOL%' then 'Army Cool'
   		 when vpa.tenure like '%Ortega Voucher%' then 'WIOA'
    	 when vpa.tenure like '%ArmyIgnitEd%' then 'ArmyIgnitEd'
   		 when vpa.tenure like '%MyCAA%' then 'MYCAA'
    else ld.program end || ' ' || ld.utmmedium || ' ' || TO_CHAR(vpa.createdon - INTERVAL '5 hours', 'FMMM/FMDD/YYYY') as "BP ID",    
    vpa.revenue as Revenue, 
    vpa.tenure as Tenure,
    case 
        when ld.createdreengageddate <= vpa.createdon 
        then date_trunc('month', ld.createdreengageddate - INTERVAL '5 hours')::date
        else date_trunc('month', ld.leaddate - INTERVAL '5 hours')::date
    end as "Lead Month",
    case 
        when ld.createdreengageddate <= vpa.createdon 
        then ld.createdreengageddate - INTERVAL '5 hours' 
        else ld.leaddate - INTERVAL '5 hours'
    end as "Lead Date",
    case 
        when ld.createdreengageddate <= vpa.createdon 
        then DATEDIFF(day, ld.createdreengageddate - INTERVAL '5 hours', vpa.createdon - INTERVAL '5 hours') 
        else DATEDIFF(day, ld.leaddate  - INTERVAL '5 hours', vpa.createdon - INTERVAL '5 hours') 
    end as "Sales Cycle",
    case 
	    when ld.createdreengageddate <= vpa.createdon and date_trunc('month', vpa.createdon - INTERVAL '5 hours') = date_trunc('month', ld.createdreengageddate - INTERVAL '5 hours') THEN 1
        when ld.createdreengageddate > vpa.createdon and date_trunc('month', vpa.createdon - INTERVAL '5 hours') = date_trunc('month', ld.leaddate - INTERVAL '5 hours') THEN 1
        else 0
    end as "SM Enrolment"
FROM 
    v1_crm_vpa_data as vpa           
    INNER JOIN v1_crm_lead_report as ld
        ON lower(ld.email) = lower(vpa.email)  
    Left JOIN v1_crm_brand_name_value AS br
        ON ld.brand_value = br.value 
where 
	(vpa.vpatype = 'Bootcamp' or (vpa.vpatype = 'VPA' and vpa.tenure = 'WIOA'))
	and "Enrolment Date" >= '2022-01-01'
	and (vpa.accountname not like '%Ed2go%' or accountname is null)
union 
SELECT 
	lower(er.email) as Email,
	er.createdon - INTERVAL '5 hours' as "Enrolment Date",
	date_trunc('month', er.createdon - INTERVAL '5 hours')::date as "Enrolment Month",
    COALESCE(NVL(opp.brand, br.brand), 'Unbranded') as Brand,
    case when er.tenure like '%WIOA%' then 'WIOA' 
    	 when er.tenure like '%ArmyCOOL%' then 'Army Cool'
   		 when er.tenure like '%Ortega Voucher%' then 'WIOA'
    	 when er.tenure like '%ArmyIgnitEd%' then 'ArmyIgnitEd'
   		 when er.tenure like '%MyCAA%' then 'MYCAA'
    else opp.program end as Program, 
    opp.utmmedium as "UTM Medium", 
    COALESCE(NVL(opp.brand, br.brand), 'Unbranded')  || ' ' ||  
    case when er.tenure like '%WIOA%' then 'WIOA' 
    	 when er.tenure like '%ArmyCOOL%' then 'Army Cool'
   		 when er.tenure like '%Ortega Voucher%' then 'WIOA'
    	 when er.tenure like '%ArmyIgnitEd%' then 'ArmyIgnitEd'
   		 when er.tenure like '%MyCAA%' then 'MYCAA'
    else opp.program end || ' ' || opp.utmmedium || ' ' || TO_CHAR(er.createdon - INTERVAL '5 hours', 'FMMM/FMDD/YYYY') as "BP ID",    
    er.revenue as Revenue, 
    case when opp.program like '%Army Cool%' then 'Army Cool' 
     	 when opp.program like '%WIOA%' then 'WIOA' 
    else er.tenure end as Tenure,
    case 
        when opp.createdreengageddate <= er.createdon
        then date_trunc('month', opp.createdreengageddate - INTERVAL '5 hours')::date
        else date_trunc('month', opp.leaddate - INTERVAL '5 hours')::date
    end as "Lead Month",
    case 
        when opp.createdreengageddate <= er.createdon 
        then opp.createdreengageddate - INTERVAL '5 hours' 
        else opp.leaddate - INTERVAL '5 hours'
    end as "Lead Date",
    case 
        when opp.createdreengageddate <= er.createdon 
        then DATEDIFF(day, opp.createdreengageddate - INTERVAL '5 hours', er.createdon - INTERVAL '5 hours') 
        else DATEDIFF(day, opp.leaddate - INTERVAL '5 hours', er.createdon - INTERVAL '5 hours') 
    end as "Sales Cycle",
    case 
	    when opp.createdreengageddate <= er.createdon and date_trunc('month', er.createdon - INTERVAL '5 hours') = date_trunc('month', opp.createdreengageddate - INTERVAL '5 hours') THEN 1
        when opp.createdreengageddate > er.createdon and date_trunc('month', er.createdon - INTERVAL '5 hours') = date_trunc('month', opp.leaddate - INTERVAL '5 hours') THEN 1
        else 0
    end as "SM Enrolment"
FROM 
    v1_crm_enrollment_data as er           
    INNER JOIN v1_crm_opportunity as opp
        ON opp.oppid = er.opportunityid 
    Left JOIN v1_crm_brand_name_value AS br
        ON opp.brand_value = br.value 
where 
	(opp.leadtype like '%B2C%')
	and "Enrolment Date" >= '2022-01-01'
	and (er.accountname not like '%Ed2go%' or er.accountname is null)
	and lower(er.email) not in (select lower(email) from v1_crm_vpa_data where email is not null)
	and er.email not in (select email from v1_crm_enrollment_data where statecode = 'Inactive'
    and date_trunc('month', createdon - INTERVAL '5 hours') = date_trunc('month', er.createdon - INTERVAL '5 hours'))
order by "Enrolment Date"

```

### Google Data View

Created Google data SQL view and get it into the Power BI report

```
select 
	startdate as "Date",
	date_trunc('month', startdate)::date as "Month",
	brand, 
	program, 
	utmmedium, 
	concat(brand , CONCAT(' ', CONCAT(program , CONCAT(' ', CONCAT(utmmedium , CONCAT(' ', TO_CHAR(startdate, 'FMMM/FMDD/YYYY'))  ) ) ) ) ) as "BP ID",
	coalesce( round(spend, 2), 0) as Spend, 
	coalesce(clicks,0) as clicks, 
	coalesce(impressions,0) as impressions,
	coalesce( round(cpc, 2), 0) as cpc, 
	coalesce( round(ctr, 4), 0) as ctr, 
	coalesce( round(conversions,2), 0) as conversions, 
	leadgoal as "Lead Goal", 
	cplgoal as "CPL Goal"
from dev.public.v1_google_ads_data
where startdate >= '2022-01-01'
order by startdate
```


### Facebook Data View

Created Facebook data SQL view and get it into the Power BI report

```
select 
	startdate as "Date",
	date_trunc('month', startdate)::date as "Month",
	brand,
	program, 
	utmmedium, 
	concat(brand , CONCAT(' ', CONCAT(program , CONCAT(' ', CONCAT(utmmedium , CONCAT(' ', TO_CHAR(startdate, 'FMMM/FMDD/YYYY'))  ) ) ) ) ) as "BP ID",
	coalesce(round(spend, 2), 0) as spend, 
	coalesce(impressions,0) as impressions,
	coalesce(round(cpc, 2), 0) as cpc, 
	coalesce(round(ctr, 4), 0) as ctr, 
	leadgoal as "Lead Goal",
	cplgoal as "CPL Goal"
from dev.public.v1_facebook_ads_data
where startdate >= '2022-01-01'
order by startdate
```

### Bridge Table Data View

Created Bridge table data SQL view for making rrelationship between table 

```
WITH filtered_data AS (
  SELECT 
    "bp id",
    brand,
    program,
    "utm medium",
    DATE_TRUNC('day', "lead date") AS "Date"
  FROM vw_finance_crm_lead_report
  UNION
  SELECT
    "bp id",
    brand,
    program,
    "utm medium",
    DATE_TRUNC('day', "Opp Date") AS "Date"
  FROM vw_finance_crm_opportunity_report
  UNION
  SELECT 
    "bp id",
    brand,
    program,
    "utm medium",
    DATE_TRUNC('day', "Enrolment Date") AS "Date"
  FROM vw_finance_crm_enrollment_report
  UNION
  SELECT
    "bp id",
    brand,
    program,
    "utm medium",
    DATE_TRUNC('day', "Date") AS "Date"
  FROM vw_finance_linkedin_report
  UNION
  SELECT
    "bp id",
    brand,
    program,
    utmmedium as "utm medium",
    DATE_TRUNC('day', "Date") AS "Date" 
  FROM vw_finance_google_ads_report
  UNION
  SELECT
    "bp id",
    brand,
    program,
    utmmedium as "utm medium",
    DATE_TRUNC('day', "Date") AS "Date"
  FROM vw_finance_facebook_ads_report
)
SELECT *
FROM filtered_data
WHERE "Date" >= '2022-01-01'
ORDER BY "Date"

```

### Power BI report pages
---

![er-diagram](https://github.com/user-attachments/assets/1fe5e744-c478-4566-b4f9-1b2ab49dfbb8)

![overview](https://github.com/user-attachments/assets/79669b53-8d77-47d0-8e8c-45b4b1ba5711)

![programs](https://github.com/user-attachments/assets/100af7f3-ca8a-4e97-a214-cceec7b99674)

![programs1](https://github.com/user-attachments/assets/416f9774-05c4-483c-bc30-05f1b544a11b)

![programs2](https://github.com/user-attachments/assets/02a04a35-e35e-43d7-b966-a7a04d4a0a28)


