Second: Write a query that directly answers a predetermined question from a business stakeholder

Written in MSSQL

## What are the top 5 brands by receipts scanned for most recent month?

#assumption – top brands are top used brands in receipt itemlist
With cte as( 
Select Fact_table.barcode, Fact_table.brandcode
from Fact_table 
where Fact_table.datescanned in (select Date_dimension .Timekey
From Date_dimension where Date_dimension.month = month(getdate())-1 and Date_dimension .year=year(getdate())  )
)

Select brands_dimension.brandname, count(cte.barcode)
From cte join brands_dimension 
on brands_dimension.barcode= cte. barcode, brands_dimension.brandcode= cte. brandcode groupby cte.brandname
order by count(cte. barcode) desc limit 5


## How does the ranking of the top 5 brands by receipts scanned for the recent month compare to the ranking for the previous month?

#to store recent month items sale
With rec_cte as( 
Select Fact_table.barcode, Fact_table.brandcode
from Fact_table 
where Fact_table.datescanned in (select Date_dimension .Timekey
From Date_dimension where Date_dimension.month = month(getdate())-1 and Date_dimension .year=year(getdate())  )
)

#to store previous month items sale
With prev_cte as( 
Select Fact_table.barcode, Fact_table.brandcode
 from Fact_table 
where Fact_table.datescanned in (select Date_dimension .Timekey
 From Date_dimension where Date_dimension.month = month(getdate())-2 and Date_dimension .year=year(getdate())  )
)

#joining brands and count of their brand name for both previous and recent months
With rec_cte_2 as(
Select rec_cte.brandname, count(rec_cte.barcode) as brand_count 
From rec_cte join brands_dimension 
on brands_dimension .barcode= rec_cte. Barcode and 
brands_dimension .brandcode= rec_cte.brandcode 
groupby rec_cte.brandname order by count(rec_cte. barcode) desc limit 5)

With prev_cte_2 as(
Select prev_cte.brandname, count(prev_cte. barcode) as brand_count 
From prev_cte join brands_dimension 
on brands_dimension .barcode= prev_cte. barcode, brands_dimension.brandcode= prev_cte.brandcode
groupby prev_cte.brandname order by count(prev_cte. barcode) desc limit 5)

#ranking the result based on count of usage 
With rec_cte_3 as(
Select *, rank() over(partition by rec_cte_2.brandname order by rec_cte_2.brand_count desc) as brandrank from rec_cte_2 )
With prev_cte_3 as(
Select *, rank() over(partition by prev_cte_2.brandname order by prev_cte_2.brand_count desc) as brandrank from prev_cte_2)

#joining 2 cte (results based on count) on rank to show them side by side
Select rec_cte_3.brandrank, rec_cte_3. brandname as recent_brand, prev_cte_3.brand.brandname  as previous_brand
from rec_cte_3 join prev_cte_3 
on rec_cte_3.Brandrank= prev_cte_3.brandrank


## When considering average spend from receipts with 'rewardsReceiptStatus' of 'Accepted' or 'Rejected', which is greater?
#storing cte with the barcode and brandcode based on status accepted/rejected
With cte as(
Select receipt_dimension.brandcode, receipt_dimension .barcode, receipt_dimension.receipt_id, receipt_dimension.rewardreceiptstatus
 from receipt_dimension 
where receipt_dimension.rewardreceiptstatus=’Accepted' or receipt_dimension .rewardreceiptstatus=’Rejected'
)

#joining fact table and cte results 

Select cte.Rewardreceiptstatus, avg(fact_table.totalspent) as average_spend 
From fact_table join cte
on fact_table.barcode=cte.barcode and fact_table.brandcode=cte.brandcode and fact_table. receipt_id = cte. receipt_id order by avg(fact_table.totalspent) desc limit 1

## When considering total number of items purchased from receipts with 'rewardsReceiptStatus' of 'Accepted' or 'Rejected', which is greater?
#storing cte with the barcode and brandcode based on status accepted/rejected

With cte as(
Select receipt_dimension.brandcode, receipt_dimension.barcode, receipt_dimension. receipt_id, receipt_dimension.rewardreceiptstatus
from receipt_dimension where receipt_dimension.rewardreceiptstatus=’Accepted' or receipt_dimension .rewardreceiptstatus=’Rejected'
)
#joining fact table and cte results

Select cte.rewardreceiptstatus,count(fact_table.barcode) as total_number_of_items_purchased 
From fact_table 
join cte on fact_table.barcode = cte.barcode and fact_table.brandcode = cte.brandcode and fact_table. receipt_id = cte. receipt_id order by count(fact_table.barcode) desc limit 1

## Which brand has the most spend among users who were created within the past 6 months?
With cte as(
Select Fact_table.barcode, Fact_table.brandcode, sum(Fact_table.finalprice) as brandspend as  from Fact_table where datescanned in (select Timekey from Date_dimension where month in (month(getdate())-1, month(getdate())-2,month(getdate())-3,month(getdate())-4,month(getdate())-5,month(getdate())-6) and year=year(getdate())  )
 group by Fact_table.barcode, Fact_table.brandcode order by sum(Fact_table .finalprice) desc limit 1 )

Select brands_dimension.name From cte join brands_dimension on cte.barcode= brands_dimension.barcode and cte.brandcode= brands_dimension.barcode 

## Which brand has the most transactions among users who were created within the past 6 months?
With cte as(
Select Fact_table.barcode, Fact_table.brandcode, count(Fact_table.barcode) as brandspend as  from Fact_table where datescanned in (select Timekey from Date_dimension where month in ( month(getdate())-1, month(getdate())-2, month(getdate())-3, month(getdate())-4, month(getdate())-5, month(getdate())-6) and year=year(getdate())  ) group by Fact_table.barcode, Fact_table.brandcode order by Fact_table.count(Fact_table.barcode) desc limit 1)

Select brands_dimension.name From cte join brands_dimension on cte.barcode= brands_dimension.barcode and cte.brandcode= brands_dimension.barcode


These queries can be better written and evaluated if we load data into a real time OLAP database and execute with data.
