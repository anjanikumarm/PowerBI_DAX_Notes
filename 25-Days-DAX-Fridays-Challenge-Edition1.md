# 25 Days of DAX Fridays - Challenge Edition 1 by Curbal 
[Refer to more challeges](https://curbal.com/25-days-of-dax-fridays-challenge-ed1-northwind-company)


**Day 1 Question: How many curent products cost less than $20 ?**

```
CALCULATE(DISTINCTCOUNT(Products[ProductID]), FILTER(Products, Products[Unit Price]>20))
  
```
  
**Day 2 Question: Which product is most expensive ?**

```
CALCULATE(SELECTEDVALUE(Products[ProductName]), TOPN(1, Products, [Unit Price], DESC))
  
```
**Note: [Unit Price] is a measure**

_Alternate approach_
```
FIRSTNONBLANK(TOPN(1, VALUES(Products([ProductName]), [Unit Price], 1)
  
```

**Day 3 Question: What is the average unit price for products ?**

```
AVERAGE(Products[UnitPrice])
  
```

**Day 4 Question: How mnay products are above the average unit price ?**

```
CALCULATE(DISTINCTCOUNT(Products[ProductID]), FILTER(Products, Products[Unit Price] > Products[Avg_Unit_Price]))
  
```

**Day 5 Question: How many products cost between $15 and $25 ?**

```
CALCULATE(DISTINCTCOUNT(Products[ProductID]), 
  FILTER(Products, 
  Products[Unit Price] >= 15 &&
  Products[Unit Price] <= 25
  )
)
  
```

**Day 6 Question: What is the average number of produts per order ?**

```
AVERAGEX(VALUES(Orders[OrderID]), [order_cnt])
  
```

**Day 7 Question: What is the order value in $ of open orders (Not shipped yet) ?**

```
IF(ISBLANK(Orders[ShippedDate]), [Total sales], 0)
  
```

**Day 8 Question: How many orders are Single Item (one product per order) ?**

```
IF(
    HASONEVALUE(Orders[OrderID]), [Single_Orders],
    SUMX(VALUES(Orders[OrderID]), [Single_Orders]))
  
```

_Alternate approach_
```
COUNTROWS(
    FILTER(
        SUMMARIZE(Orders, Orders[OrderID], "num_prds", COUNTA(Orders[ProductID])), [num_prds] = 1))
  
```

**Day 9 Question: Average sales per transaction (OrderID) for "Romero y Tomillo" ?**

```
CALCULATE(
AVERAGEX(
    VALUES(Orders[OrderID]), [Total sales]), Customers[CompanyName] = "Romero y Tomillo"
    )
  
```


**Day 10 Question: How many days since "North/South" last purchase ?**

```
var lastpurchase = CALCULATE(LASTDATE(Orders[OrderDate]), Customers[CompanyName] = "North/South")
var days_diff = DATEDIFF(lastpurchase, TODAY(), DAY)
RETURN days_diff
  
```

**Day 11 Question: How many Customers have ordered only once ?**

```
COUNTROWS(
    FILTER(
        SUMMARIZE(Orders, Orders[CustomerID], "order_cnt", DISTINCTCOUNT(Orders[OrderID])), [order_cnt] = 1
        )
)
  
```

**Day 12 Question: How many New Customers in current year ?**

```
var curr_year = YEAR(TODAY())
var new_customers = 
COUNTROWS(
    FILTER(
        SUMMARIZE(Orders, Orders[CustomerID], "first_purchase_year", YEAR(MIN(Orders[OrderDate]))), [first_purchase_year] = curr_year)
)
RETURN new_customers
  
```

**Day 13 Question: How many lost customers in current year ?**

```
VAR curr_yr = YEAR(TODAY())
VAR prev_yr = curr_yr - 1
RETURN
COUNTROWS(
    FILTER(
        SUMMARIZE( FILTER(Orders, YEAR(Orders[OrderDate]) = prev_yr), Orders[CustomerID] ),
        NOT(CONTAINS(
        SUMMARIZE( FILTER(Orders, YEAR(Orders[OrderDate]) = curr_yr), Orders[CustomerID]), Orders[CustomerID], Orders[CustomerID]) )
    )
)
  
```

_Alternate approach_
```
VAR curr_yr = YEAR(TODAY())
VAR prev_yr = curr_yr - 1
var curr_yr_cust = SUMMARIZE(FILTER(Orders, YEAR(Orders[OrderDate]) = prev_yr), Orders[CustomerID])
var prev_yr_cust = SUMMARIZE(FILTER(Orders, YEAR(Orders[OrderDate]) = curr_yr), Orders[CustomerID])
RETURN
COUNTROWS(EXCEPT(curr_yr_cust, prev_yr_cust))
  
```

**Day 14 Question: How many Customers have never ordered "Queso Cabrales" ?**

```
VAR SelectedProduct = SELECTEDVALUE(Orders[ProductID], "All")
VAR TotalCustomers = CALCULATE(DISTINCTCOUNT(Orders[CustomerID]), ALL(Orders))
VAR CustomersBoughtProduct = CALCULATE(DISTINCTCOUNT(Orders[CustomerID]), FILTER(ALL(Orders), Orders[ProductID] = SelectedProduct))

RETURN IF(ISBLANK(SelectedProduct), BLANK(), TotalCustomers - CustomersBoughtProduct)
  
```

**Day 15 Question: How many Customers have ordered only "Queso Cabrales" ?**

```
VAR SelectedProduct = SELECTEDVALUE(Orders[ProductID], "All")
VAR TotalCustomers = CALCULATE(DISTINCTCOUNT(Orders[CustomerID]), ALL(Orders))

VAR cust_selected_product = CALCULATE(DISTINCTCOUNT(Orders[CustomerID]), FILTER(ALL(Orders), Orders[ProductID] = SelectedProduct))

RETURN IF(ISBLANK(SelectedProduct), TotalCustomers, cust_selected_product)
  
```

**Day 16 Question: How many products out of stock ?**

```
COUNTROWS(
    FILTER(
        SUMMARIZE(
            Products, Products[ProductID], "stock", SUM(Products[UnitsInStock])), 
            [stock] = 0)
  
```

**Day 17 Question: How many products need to be restocked ?**

```
CALCULATE(
    COUNTA(Products[ProductName]), 
    FILTER(Products, Products[UnitsInStock] < [Restock level]))
  
```

**Day 18 Question: How many products on order to restock ?**

```
CALCULATE(COUNT(Products[ProductName]), FILTER(Products, [Stocked units] < [Orderec units]))
  
```

**Day 19 Question: What is the stocked value of the discontinued products ?**

```
CALCULATE(
    SUMX(Products, 
    [Unit Price]*[Stocked units]), Products[Discontinued]=TRUE())
  
```

**Day 20 Question: Which vendor has the highest stock value ?**

```
CALCULATE(
    SELECTEDVALUE(Suppliers[CompanyName]),
    TOPN(1, 
    SUMMARIZE(Products, Suppliers[CompanyName], "stock_value", SUMX(Products, [Unit Price]*[Stocked units])), [stock_value], DESC))
  
```

**Day 21 Question: How many Employees are female % ?**

```
VAR fml_emp = CALCULATE(COUNTA(Employees[EmployeeID]), Employees[Gender] = "Female")
VAR all_emp = COUNTA(Employees[EmployeeID])

RETURN DIVIDE(fml_emp, all_emp)
  
```

**Day 22 Question: How many employees are 60 years old or over ?**

```
VAR snr_emp = CALCULATE(COUNTA(Employees[EmployeeID]), Employees[Emp_Age] > 60)
RETURN snr_emp
  
```

**Day 23 Question: Which employee had the highest sales in current year ?**

```
VAR curr_yr = YEAR(TODAY())

RETURN
CALCULATE(SELECTEDVALUE(Employees[Full Name]), 
TOPN(1, Employees, CALCULATE([Total sales], 'Calendar'[Year] = curr_yr), DESC))
  
```

**Day 24 Question: How many employees sold over $100k in current year ?**

```
var curr_yr = YEAR(TODAY())

RETURN
CALCULATE(
    COUNTA(Orders[EmployeeID]), 
    FILTER(Orders, [Total sales] > 10000), 
    'Calendar'[Year] = curr_yr )
  
```

_Alternate approach_
```
VAR curr_yr = YEAR(TODAY())
RETURN
CALCULATE(
    COUNTA(Orders[EmployeeID]),
    FILTER(ALL('Calendar'), 'Calendar'[Year] = curr_yr),
    FILTER(ALL(Orders), [Total sales] > 10000)
)
  
```

**Day 25 Question: How many employees got hired in 1994 ?**

```
CALCULATE(
    COUNTROWS(Employees), 
    YEAR(Employees[HireDate]) = 1994)
  
```



