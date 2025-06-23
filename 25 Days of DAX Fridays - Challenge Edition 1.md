# 25 Days of DAX Fridays - Challenge Edition 1 by Curbal 
[Refer to more challeges](https://curbal.com/25-days-of-dax-fridays-challenge-ed1-northwind-company)


** Day 1 Question: How many curent products cost less than $20**
```

CALCULATE(DISTINCTCOUNT(Products[ProductID]), FILTER(Products, Products[Unit Price]>20))

```
