# Generated DAX — Base Measures
**Dataset:** Northwind Traders (Kaggle — jeetahirwar)
**Column names:** Exact match to CSV files
**Generated via:** Claude Desktop + Power BI Modeling MCP

Paste any measure into Power BI Desktop: **Modeling tab → New Measure**

---

## Revenue Measures

```dax
-- ─────────────────────────────────────────────────────
-- Total Revenue
-- Net revenue after applying discount on each line item.
-- Formula: unitPrice × quantity × (1 - discount)
-- Source: order_details table
-- ─────────────────────────────────────────────────────
Total Revenue =
SUMX(
    order_details,
    order_details[unitPrice]
        * order_details[quantity]
        * ( 1 - order_details[discount] )
)
```

```dax
-- ─────────────────────────────────────────────────────
-- Total Orders
-- Distinct count of unique orders placed.
-- Source: orders table (830 total orders in dataset)
-- ─────────────────────────────────────────────────────
Total Orders =
DISTINCTCOUNT( orders[orderID] )
```

```dax
-- ─────────────────────────────────────────────────────
-- Average Order Value
-- Mean revenue per order.
-- Uses DIVIDE to safely return 0 if no orders exist.
-- ─────────────────────────────────────────────────────
Average Order Value =
DIVIDE(
    [Total Revenue],
    [Total Orders],
    0
)
```

```dax
-- ─────────────────────────────────────────────────────
-- Total Units Sold
-- Sum of all quantities across all order lines.
-- Source: order_details[quantity]
-- ─────────────────────────────────────────────────────
Total Units Sold =
SUM( order_details[quantity] )
```

```dax
-- ─────────────────────────────────────────────────────
-- Total Freight
-- Total shipping cost across all orders.
-- Source: orders[freight]
-- ─────────────────────────────────────────────────────
Total Freight =
SUM( orders[freight] )
```

```dax
-- ─────────────────────────────────────────────────────
-- Average Discount %
-- Average discount rate applied across all order lines.
-- Source: order_details[discount] — stored as 0.0 to 1.0
-- Format this measure as Percentage in Power BI.
-- ─────────────────────────────────────────────────────
Average Discount % =
AVERAGE( order_details[discount] )
```

```dax
-- ─────────────────────────────────────────────────────
-- Revenue per Unit
-- Average revenue generated per unit sold.
-- ─────────────────────────────────────────────────────
Revenue per Unit =
DIVIDE(
    [Total Revenue],
    [Total Units Sold],
    0
)
```

---

## Customer Measures

```dax
-- ─────────────────────────────────────────────────────
-- Total Customers
-- Distinct customers who placed at least one order.
-- Counts from orders table, not customers master table.
-- This means only active customers are counted.
-- ─────────────────────────────────────────────────────
Total Customers =
DISTINCTCOUNT( orders[customerID] )
```

```dax
-- ─────────────────────────────────────────────────────
-- Avg Orders per Customer
-- How frequently a customer places orders on average.
-- ─────────────────────────────────────────────────────
Avg Orders per Customer =
DIVIDE(
    [Total Orders],
    [Total Customers],
    0
)
```

```dax
-- ─────────────────────────────────────────────────────
-- Avg Revenue per Customer
-- Average revenue generated per unique customer.
-- ─────────────────────────────────────────────────────
Avg Revenue per Customer =
DIVIDE(
    [Total Revenue],
    [Total Customers],
    0
)
```

---

## Product Measures

```dax
-- ─────────────────────────────────────────────────────
-- Products Sold
-- Number of distinct products that appear in order lines.
-- Source: order_details[productID]
-- ─────────────────────────────────────────────────────
Products Sold =
DISTINCTCOUNT( order_details[productID] )
```

```dax
-- ─────────────────────────────────────────────────────
-- Active Products
-- Products where discontinued = 0 (still available).
-- Source: products[discontinued] (0=active, 1=discontinued)
-- ─────────────────────────────────────────────────────
Active Products =
CALCULATE(
    COUNTROWS( products ),
    products[discontinued] = 0
)
```

```dax
-- ─────────────────────────────────────────────────────
-- Discontinued Products
-- Products where discontinued = 1 (no longer for sale).
-- ─────────────────────────────────────────────────────
Discontinued Products =
CALCULATE(
    COUNTROWS( products ),
    products[discontinued] = 1
)
```

```dax
-- ─────────────────────────────────────────────────────
-- Avg Unit Price
-- Average unit price across all order lines (at time of order).
-- Note: uses order_details[unitPrice], not products[unitPrice],
-- as prices may differ between current catalogue and past orders.
-- ─────────────────────────────────────────────────────
Avg Unit Price =
AVERAGE( order_details[unitPrice] )
```

---

## Employee Measures

```dax
-- ─────────────────────────────────────────────────────
-- Total Employees
-- Count of all employees in the employees table.
-- Total in dataset: 9 employees across New York and London.
-- ─────────────────────────────────────────────────────
Total Employees =
COUNTROWS( employees )
```

```dax
-- ─────────────────────────────────────────────────────
-- Revenue per Employee
-- Average revenue attributed to each employee.
-- Useful for comparing sales rep performance overall.
-- ─────────────────────────────────────────────────────
Revenue per Employee =
DIVIDE(
    [Total Revenue],
    [Total Employees],
    0
)
```

```dax
-- ─────────────────────────────────────────────────────
-- Orders per Employee
-- Average number of orders handled per employee.
-- ─────────────────────────────────────────────────────
Orders per Employee =
DIVIDE(
    [Total Orders],
    [Total Employees],
    0
)
```
