# DAX Measures Documentation
This document contains key DAX measures used in the Power BI Dashboard.

## Date Table
A dedicated calendar table used for time intelligence calculations such as Year-over-Year (YoY) growth, monthly trends, and date-based filtering.
The table ensures consistent time analysis across the model and enables functions like SAMEPERIODLASTYEAR.
```DAX
Date Table = 
VAR mindate = MIN('Sales fact'[Order Date])
VAR maxdate = MAX('Sales fact'[Order Date])
 RETURN 
 CALENDAR(mindate, maxdate)
```

## Year-Over-Year Revenue Growth
Calculates the percentage change in revenue compared to the previous year.
```DAX
RevenueLY = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date Table'[Date]))
YOY Revenue% = DIVIDE([Total Revenue] - [RevenueLY],[RevenueLY])
```

## Year-Over-Year Profit Growth
Calculates the percentage change in profit compared to the previous year.
```DAX
ProfitLY = CALCULATE([Profit], SAMEPERIODLASTYEAR('Date Table'[Date]))
ProfitYOY = DIVIDE([Profit] - [ProfitLY],[ProfitLY])
```

## Product by Profit Rank
Ranks products based on total profit in descending/ascending order to identify the top/bottom performing products.
Used in the dashboard to display the Top N products by profit.
```DAX
Rank ProductbyProfit = 
       VAR Topproductbyprofit = RANKX(ALL('Products'[Product Name]),[Profit], ,DESC,Dense)
       VAR Bottomproductbyprofit = RANKX(ALL('Products'[Product Name]), [Profit], , ASC,Dense)
       VAR _ranking = IF(SELECTEDVALUE(TopBottom[Value]) ="TOP",Topproductbyprofit, Bottomproductbyprofit)
RETURN 
       IF(_ranking <= 'TopN Parameter'[TopN Parameter Value], [Profit])
```

## Product by Revenue Rank
Ranks products based on total revenue in descending/ascending order to identify the top/bottom performing products.
Used in the dashboard to display the Top N products by revenue.
```DAX
Rank ProductbyRevenue = 
       VAR Topproductbyrevenue = RANKX(ALL('Products'[Product Name]),[Total Revenue], ,DESC,Dense)
       VAR Bottomproductbyrevenue = RANKX(ALL('Products'[Product Name]), [Total Revenue], , ASC,Dense)
       VAR _ranking = IF(SELECTEDVALUE(TopBottom[Value]) ="TOP",Topproductbyrevenue, Bottomproductbyrevenue)
RETURN 
       IF(_ranking <= 'TopN Parameter'[TopN Parameter Value], [Total Revenue])
 ```

## New Customers
Counts customers who made their first purchase within the selected time period.
```DAX
New customers = 
       CALCULATE(DISTINCTCOUNT('Sales fact'[CustomerKey]),FILTER('Sales fact',
       'Sales fact'[First Purchase Date]  >= MIN('Date Table'[Date]) &&
        'Sales fact'[First Purchase Date] <= MAX('Date Table'[Date])))
```

## First Purchase Date
Identifies the earliest purchase date for each customer.  
This measure is used to determine when a customer first engaged with the business and is essential for classifying customers as new or returning.
```DAX
First Purchase Date =
       CALCULATE(MIN('Sales fact'[Order Date]),
       ALLEXCEPT('Sales fact','Sales fact'[CustomerKey]))
 ```

## Store Age
Calculates the number of years since each store was opened.  
This metric is used to analyze store performance based on store maturity and operational experience.
```DAX
Store Age = 
        SWITCH(
            TRUE(),
            Stores[Age] <= 10, "Under 10 years",
            Stores[Age] <= 14, "11-14 years",
            Stores[Age] <= 17, "15-17 years", 
            "> 17 years")
```
