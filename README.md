# SQL- BigQuery - Sales Analysis
## Introduction
This project aims to conduct an in-depth analysis of sales performance for AdventureWorld, leveraging comprehensive SQL analytics on BigQuery to extract critical business insights across multiple dimensions.
## Key Analysis Areas
The analysis is divided into the following focus areas:
### Sales and Growth Analysis
- Calculate the total quantity of items, sales value, and order quantity by subcategory for the last 12 months (L12M).
- Determine the YoY growth rate (%) by subcategory and identify the top three categories with the highest growth rates based on quantity sold.
### Territory Performance
- Rank the top three TerritoryIDs by yearly order quantity, ensuring no ranking is skipped for ties.
### Discount Costs
- Calculate the total discount cost for seasonal discounts by subcategory.
### Customer Retention
Perform a cohort analysis to calculate the retention rate of customers in 2014 who achieved the "Successfully Shipped" status.
### Stock Trends
- Analyze stock levels in 2011, identifying month-over-month (MoM) differences (%) for all products. If the growth rate is null, default to 0.
- Calculate the stock-to-sales ratio by product and month in 2011, ordering results by descending month and ratio.
### Order Metrics
- Summarize the number of orders and total value for orders in "Pending" status during 2014.
### Employee Performance
- Identify the top employee with the highest monthly pay over the last six months.
## Business Value
This analysis empowers AdventureWorld to:
- Monitor sales performance and identify high-growth subcategories to drive revenue.
- Recognize top-performing territories for strategic focus.
- Optimize discount strategies by evaluating seasonal discount costs.
- Retain customers through actionable insights into shipping success and retention metrics.
- Manage inventory effectively using stock-level trends and stock-to-sales ratios.
- Improve operational efficiency by understanding pending orders and employee performance.
## Data Access & Structure
### Dataset Information
- Platform: Google BigQuery
- Dataset: AdventureWorks (Specific database for analysis)
- Time Period: Full historical data, with focus on 2011-2014
### Data Schema
For detailed information about the Google Analytics dataset schema, please refer to the official Google Analytics documentation, available at https://drive.google.com/file/d/1bwwsS3cRJYOg1cvNppc1K_8dQLELN16T/view ↩
## Technical Implementation
The project leverages BigQuery's powerful features including:
- Advanced SQL aggregations and window functions
- Cohort analysis using date-based grouping
- Comparative growth calculations (YoY, MoM)
## Exploring the Dataset
### Query 1: Calc Quantity of items, Sales value & Order quantity by each Subcategory in L12M
#### Syntax 
```sql
with latest_date as (
      select 
            max (ModifiedDate) as latest_date 
      from `adventureworks2019.Sales.SalesOrderDetail` 
)

select 
      format_date ('%b %Y', a.ModifiedDate) as period,
      c.name as name,
      sum (a.OrderQty) as qty_item,
      sum (a.LineTotal) as total_sales,
      count (distinct a.SalesOrderID) as order_cnt 
from `adventureworks2019.Sales.SalesOrderDetail` a 
left join `adventureworks2019.Production.Product` b 
      on a.ProductID = b.ProductID
left join `adventureworks2019.Production.ProductSubcategory` c
      on cast (b.ProductSubcategoryID as int) = c.ProductSubcategoryID
where date(a.ModifiedDate) between (date_sub('2014-06-30', interval 12 month)) and '2014-06-30'
group by period, name 
order by name;
```
#### Result
![image](https://github.com/user-attachments/assets/b9e70755-4641-46fb-9a5a-1c648804351c)

The sales data reveals distinct patterns across product categories, with ___bikes (Mountain, Road, and Touring) driving the highest revenue___ despite lower order counts due to their premium pricing. Accessories like ___Tires and Tubes, Bottles and Cages, and Helmets demonstrate the highest order frequency___, indicating strong recurring purchase behavior. There's a clear seasonal trend with ___peak sales occurring in March 2014___ across multiple categories, while ___November-December 2013 shows a noticeable dip___. ___Small accessories___ consistently generate ___high order volumes but lower total sales values___, while ___bikes___ show the opposite pattern with ___fewer orders but higher revenue impact___. Most categories exhibit ___stronger performance during warmer months (March-July)___, with core products maintaining steady sales throughout the year and seasonal items showing more pronounced fluctuations in demand.
### Query 2: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate
#### Syntax
``` sql 
with 
sale_info as (
  SELECT 
      FORMAT_TIMESTAMP("%Y", a.ModifiedDate) as yr
      , c.Name
      , sum(a.OrderQty) as qty_item

  FROM `adventureworks2019.Sales.SalesOrderDetail` a 
  LEFT JOIN `adventureworks2019.Production.Product` b on a.ProductID = b.ProductID
  LEFT JOIN `adventureworks2019.Production.ProductSubcategory` c on cast(b.ProductSubcategoryID as int) = c.ProductSubcategoryID

  GROUP BY 1,2
  ORDER BY 2 asc , 1 desc
),

sale_diff as (
  select *
  , lead (qty_item) over (partition by Name order by yr desc) as prv_qty
  , round(qty_item / (lead (qty_item) over (partition by Name order by yr desc)) -1,2) as qty_diff
  from sale_info
  order by 5 desc 
),

rk_qty_diff as (
  select *
      ,dense_rank() over( order by qty_diff desc) dk
  from sale_diff
)

select distinct Name
      , qty_item
      , prv_qty
      , qty_diff
from rk_qty_diff 
where dk <=3
order by dk ;
```
#### Result 
![image](https://github.com/user-attachments/assets/0f018b06-39ca-4f6d-ad10-20ae2328a1d1)

The table calculates the Year-over-Year (YoY) growth rate by subcategory, revealing the top 3 subcategories with the highest growth rates. ___Mountain Frames___ leads with a growth rate of ___5.21%___, followed by ___Socks at 4.21%___, and ___Road Frames at 3.89%___. These subcategories show ___strong performance___, indicating successful strategies in boosting sales. Monitoring these categories can provide insights into ___effective approaches that could be applied to other subcategories to drive growth___.
### Query3: Ranking Top 3 TeritoryID with biggest Order quantity of every year 
#### Syntax 
```sql 
with order_cnt_data as ( 
      select 
            extract (year from a.ModifiedDate) as yr,
            TerritoryID, 
            sum(a.OrderQty) as order_cnt,
      from `adventureworks2019.Sales.SalesOrderDetail` a 
      left join `adventureworks2019.Sales.SalesOrderHeader` b
            on a.SalesOrderID = b.SalesOrderID
      group by yr, TerritoryID
)

select * 
from ( 
      select 
            yr,
            TerritoryID,
            order_cnt,
            dense_rank () over (partition by yr order by order_cnt desc) as rk
      from order_cnt_data
      ) 
where rk < 4 
order by yr desc; 
```
#### Result
![image](https://github.com/user-attachments/assets/684ddbe1-3cbf-4fd5-9d37-c9e7bc4444fb)

The table ranks the top 3 Territory IDs with the biggest order quantities for each year. ___TerritoryID 4 consistently ranks first, showing its dominant position across all years___.
### Query4: Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory
#### Syntax
```sql
select 
    FORMAT_TIMESTAMP("%Y", ModifiedDate) as year
    , Name
    , sum(disc_cost) as total_cost
from (
      select distinct a.*
      , c.Name
      , d.DiscountPct, d.Type
      , a.OrderQty * d.DiscountPct * UnitPrice as disc_cost 
      from `adventureworks2019.Sales.SalesOrderDetail` a
      LEFT JOIN `adventureworks2019.Production.Product` b on a.ProductID = b.ProductID
      LEFT JOIN `adventureworks2019.Production.ProductSubcategory` c on cast(b.ProductSubcategoryID as int) = c.ProductSubcategoryID
      LEFT JOIN `adventureworks2019.Sales.SpecialOffer` d on a.SpecialOfferID = d.SpecialOfferID
      WHERE lower(d.Type) like '%seasonal discount%' 
)
group by 1,2;
```
#### Result 
![image](https://github.com/user-attachments/assets/e02ec6eb-6bc1-461c-8890-a12b327fdd90)

The query calculates the total discount cost related to the Seasonal Discount for each SubCategory, and in this case, ___"Helmets" is the only product that receives the discount___. For 2012, the total discount cost for Helmets is 827.65, while ___for 2013, the cost increases___ to 1606.04. This shows a ___significant increase___ in the total seasonal discount cost for Helmets from 2012 to 2013, highlighting the growing investment in discounts for this particular product.
### Query 5: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)
#### Syntax
```sql 
with 
info as (
  select  
      extract(month from ModifiedDate) as month_no
      , extract(year from ModifiedDate) as year_no
      , CustomerID
      , count(Distinct SalesOrderID) as order_cnt
  from `adventureworks2019.Sales.SalesOrderHeader`
  where FORMAT_TIMESTAMP("%Y", ModifiedDate) = '2014'
  and Status = 5
  group by 1,2,3
  order by 3,1 
),

row_num as (--đánh số thứ tự các tháng họ mua hàng
  select *
      , row_number() over (partition by CustomerID order by month_no) as row_numb
  from info 
), 

first_order as (   --lấy ra tháng đầu tiên của từng khách
  select *
  from row_num
  where row_numb = 1
), 

month_gap as (
  select 
      a.CustomerID
      , b.month_no as month_join
      , a.month_no as month_order
      , a.order_cnt
      , concat('M - ',a.month_no - b.month_no) as month_diff
  from info a 
  left join first_order b 
  on a.CustomerID = b.CustomerID
  order by 1,3
)

select month_join
      , month_diff 
      , count(distinct CustomerID) as customer_cnt
from month_gap
group by 1,2
order by 1,2;
```
#### Result 
![image](https://github.com/user-attachments/assets/d1c7b413-4015-4d49-b982-4e44cd3c2131)

The table shows a ___significant decline in customer counts as the months since joining increase___. Month M-0 has the highest number of customers, while Month M-1 and beyond see sharp drops, with Month M-6 having the lowest count. This suggests ___a need for retention strategies___ to keep customers engaged beyond their first few months.
### Query 6: Trend of Stock level & MoM diff % by all product in 2011
#### Syntax 
```sql 
with stock_current_data as (
      select
            a.Name as name, 
            extract (month from b.ModifiedDate) as month,
            extract (year from b.ModifiedDate) as year,
            sum(StockedQty) as stock_current
      from `adventureworks2019.Production.Product` a 
      left join `adventureworks2019.Production.WorkOrder` b
           on a.ProductID = b.ProductID
      where extract (year from b.ModifiedDate) = 2011 
      group by name, month, year 
), 

stock_prev_data as ( 
      select
            name,
            month,
            year,
            stock_current,
            lag (stock_current, 1) over (partition by name order by month) as stock_prev 
      from stock_current_data
)

select 
      name,
      month,
      year,
      stock_current,
      stock_prev,
      round ((stock_current - stock_prev) / stock_prev * 100, 1) as diff
from stock_prev_data
order by name, month desc; 
```
#### Result 
![image](https://github.com/user-attachments/assets/5aded4ef-8619-4686-a65c-694219470fbc)

The data reveals ___fluctuating stock levels___ and ___significant month-over-month changes in 2011___, with ___most products bearings showing sharp declines in stock towards the end of the year___. ___Seasonal demand___ appears to influence these variations, as many items experienced large swings in stock, indicating unpredictable demand. To improve, it's recommended to ___enhance demand forecasting models___ to align stock levels with actual trends, ___focus on better inventory management___ for seasonal fluctuations, and ___prioritize replenishing___ high-demand items before peak periods to avoid shortages.
### Query 7: Calc Ratio of Stock / Sales in 2011 by product name, by month
#### Syntax
``` sql 
with 
sale_info as (
  select 
      extract(month from a.ModifiedDate) as mth 
     , extract(year from a.ModifiedDate) as yr 
     , a.ProductId
     , b.Name
     , sum(a.OrderQty) as sales
  from `adventureworks2019.Sales.SalesOrderDetail` a 
  left join `adventureworks2019.Production.Product` b 
    on a.ProductID = b.ProductID
  where FORMAT_TIMESTAMP("%Y", a.ModifiedDate) = '2011'
  group by 1,2,3,4
), 

stock_info as (
  select
      extract(month from ModifiedDate) as mth 
      , extract(year from ModifiedDate) as yr 
      , ProductId
      , sum(StockedQty) as stock_cnt
  from 'adventureworks2019.Production.WorkOrder'
  where FORMAT_TIMESTAMP("%Y", ModifiedDate) = '2011'
  group by 1,2,3
)

select
      a.*
    , b.stock_cnt as stock  --(*)
    , round(coalesce(b.stock_cnt,0) / sales,2) as ratio
from sale_info a 
full join stock_info b 
  on a.ProductId = b.ProductId
and a.mth = b.mth 
and a.yr = b.yr
order by 1 desc, 7 desc;
```
#### Result

![image](https://github.com/user-attachments/assets/9a314043-f3e1-4077-9e0d-e2fd13fb9ada)
___The data shows varying stock-to-sales ratios in 2011___. In December, products like HL Mountain Frame - Black, 48 had high ratios (27), indicating overstocking, while items such as Road-150 Red, 52 had balanced ratios (~1). October saw a mix, with some products overstocked (e.g., HL Mountain Frame - Black, 48, ratio 2.91) and others understocked (e.g., Road-150 Red, 52, ratio < 1). ___Improved demand forecasting is needed to optimize inventory levels.___





























`
