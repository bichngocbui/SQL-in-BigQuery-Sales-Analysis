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
