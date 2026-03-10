# Power BI MCP — Prompt Library
**Dataset:** Northwind Traders (Kaggle — jeetahirwar/northwind-traders)
**Tested with:** Claude Desktop + Power BI Modeling MCP
**Column names:** Exact match to CSV files

> Always run the **Connect** prompt first before anything else.

---

## 🔌 CONNECTION

```
Connect to northwind in Power BI Desktop
```

---

## 📅 DATE TABLE

### Basic Date Table
```
Build a date table for this model. The orders table has an orderDate column.
The date range in this dataset is 2013-07-04 to 2015-05-06.
Use those as the start and end dates.
Create the relationship between the new date table and orders[orderDate].
```

### Full Date Table with Fiscal Year
```
Build a complete date dimension table called DimDate with these columns:
- Date (primary key, used for relationship)
- Year
- Quarter (text: Q1, Q2, Q3, Q4)
- QuarterNumber (integer 1-4 for sorting)
- MonthNumber (integer 1-12)
- MonthName (full name: January, February...)
- MonthShort (Jan, Feb, Mar...)
- MonthYearLabel (e.g. Jul 2013)
- WeekNumber (ISO, starts Monday)
- DayOfMonth
- DayOfWeekNumber (1=Monday, 7=Sunday)
- DayName (Monday, Tuesday...)
- IsWeekend (TRUE/FALSE)
- FiscalYear (fiscal year starts July — so Jul 2013 is FY2014)
- FiscalQuarter
- MonthSort (YYYYMM integer for sorting MonthYearLabel)
- QuarterSort (YYYYQ integer)

Date range: 2013-07-04 to 2015-12-31.
Mark as date table using the Date column.
Create relationship to orders[orderDate].
```

---

## 📊 BASE MEASURES

### Auto-Build All Measures
```
Look at the Northwind tables and create a [Measures] table with appropriate
aggregations. The tables are: orders, order_details, products, customers,
employees, categories, shippers.

Revenue formula: order_details[unitPrice] * order_details[quantity] * (1 - order_details[discount])

Create measures for:
- Total Revenue (using the formula above)
- Total Orders (distinct count of orders[orderID])
- Average Order Value (Total Revenue / Total Orders)
- Total Units Sold (sum of order_details[quantity])
- Total Freight (sum of orders[freight])
- Average Discount % (average of order_details[discount])
- Total Products (distinct count of order_details[productID])
- Total Customers (distinct count of orders[customerID])

Organise into folders: Sales, Customers, Products.
Add a DAX block comment to every measure explaining what it calculates.
```

### Revenue Measures Only
```
In the [Measures] table, create these revenue measures in a folder called "Revenue":
- Total Revenue = SUMX(order_details, order_details[unitPrice] * order_details[quantity] * (1 - order_details[discount]))
- Total Orders = DISTINCTCOUNT(orders[orderID])
- Average Order Value = DIVIDE([Total Revenue], [Total Orders], 0)
- Total Units Sold = SUM(order_details[quantity])
- Total Freight = SUM(orders[freight])
- Revenue per Unit = DIVIDE([Total Revenue], [Total Units Sold], 0)
Add a comment to each.
```

### Customer Measures
```
In [Measures], create customer analytics measures in a folder called "Customers":
- Total Customers = DISTINCTCOUNT(orders[customerID])
- Avg Orders per Customer = DIVIDE([Total Orders], [Total Customers], 0)
- Avg Revenue per Customer = DIVIDE([Total Revenue], [Total Customers], 0)
- Customers in Period = DISTINCTCOUNT of customerID in current filter context
Add comments to each measure.
```

### Product Measures
```
In [Measures], create product measures in a folder called "Products":
- Products Sold = DISTINCTCOUNT(order_details[productID])
- Avg Unit Price = AVERAGE(order_details[unitPrice])
- Avg Discount % = AVERAGE(order_details[discount])
- Revenue per Product = DIVIDE([Total Revenue], [Products Sold], 0)
- Discontinued Products = CALCULATE(COUNTROWS(products), products[discontinued] = 1)
- Active Products = CALCULATE(COUNTROWS(products), products[discontinued] = 0)
Add comments.
```

### Employee Measures
```
In [Measures], create employee performance measures in a folder called "Employees":
- Total Employees = COUNTROWS(employees)
- Revenue per Employee = DIVIDE([Total Revenue], [Total Employees], 0)
- Orders per Employee = DIVIDE([Total Orders], [Total Employees], 0)
Add comments.
```

---

## ⏱ TIME INTELLIGENCE

### Full Time Intelligence Suite
```
Using [Measures] and the DimDate table (connected to orders[orderDate]),
create time intelligence measures for Total Revenue in a folder "Time Intelligence":

- Revenue MTD = TOTALMTD of Total Revenue using DimDate[Date]
- Revenue QTD = TOTALQTD of Total Revenue
- Revenue YTD = TOTALYTD of Total Revenue
- Revenue Previous Month = CALCULATE with DATEADD -1 MONTH
- Revenue Previous Quarter = CALCULATE with DATEADD -1 QUARTER
- Revenue Previous Year = CALCULATE with SAMEPERIODLASTYEAR
- Revenue vs Prev Month = [Total Revenue] minus [Revenue Previous Month]
- Revenue vs Prev Year = [Total Revenue] minus [Revenue Previous Year]
- MoM Growth % = DIVIDE([Revenue vs Prev Month], [Revenue Previous Month], BLANK())
- YoY Growth % = DIVIDE([Revenue vs Prev Year], [Revenue Previous Year], BLANK())
- Rolling 3 Month Revenue = DATESINPERIOD last 3 months
- Rolling 12 Month Revenue = DATESINPERIOD last 12 months
- 3 Month Moving Average = [Rolling 3 Month Revenue] / 3

Add a DAX comment to every single measure.
```

### Orders Time Intelligence
```
In [Measures], create time intelligence for Total Orders in a folder "Orders - Time Intelligence":
- Orders MTD
- Orders YTD
- Orders Previous Month
- Orders Previous Year
- Orders YoY Growth %
Use DimDate[Date] as the date axis.
```

---

## 🪣 CALCULATED COLUMNS

### Freight Band on orders table
```
On the orders table, create a calculated column called FreightBand that
categorises the freight column into three bands:
- "Low" if freight < 25
- "Medium" if freight is between 25 and 75
- "High" if freight > 75
```

### Delivery Status on orders table
```
On the orders table, create a calculated column called DeliveryStatus:
- "On Time" if shippedDate <= requiredDate
- "Late" if shippedDate > requiredDate
- "Not Shipped" if shippedDate is blank/null
```

### Revenue Band on order_details table
```
On the order_details table, create a calculated column called LineTotalBand.
First calculate the line total as unitPrice * quantity * (1 - discount).
Then band it as:
- "Small" if line total < 100
- "Medium" if 100 to 500
- "Large" if over 500
```

### Product Price Tier on products table
```
On the products table, create a calculated column called PriceTier:
- "Budget" if unitPrice < 15
- "Standard" if unitPrice 15 to 50
- "Premium" if unitPrice > 50
```

---

## 📝 DOCUMENTATION & METADATA

### Add Descriptions to All Columns
```
Add business-friendly descriptions to every column in all 7 tables:
orders, order_details, products, customers, employees, categories, shippers.

Use the exact column names from the data:
- orders: orderID, customerID, employeeID, orderDate, requiredDate, shippedDate, shipperID, freight
- order_details: orderID, productID, unitPrice, quantity, discount
- products: productID, productName, quantityPerUnit, unitPrice, discontinued, categoryID
- customers: customerID, companyName, contactName, contactTitle, city, country
- employees: employeeID, employeeName, title, city, country, reportsTo
- categories: categoryID, categoryName, description
- shippers: shipperID, companyName

Descriptions should be in plain business English, not just restate the column name.
```

### Add Comments to All Measures
```
Go through every measure in [Measures] and add a DAX block comment at the top
explaining: what it calculates, which tables/columns it uses, and any assumptions.
```

### Generate Full Model Documentation
```
Generate complete markdown documentation for the Northwind Power BI model.

Structure it as:
1. Model Summary (tables, row counts, date range, total measures)
2. Relationships (from → to, cardinality, filter direction)
3. Table Details — for each of the 7 tables: column name, data type, description
4. Measures — grouped by folder: name, formula summary, plain English description
5. Calculated Columns — table, name, logic summary

The 7 tables are: orders (830 rows), order_details (2155 rows), products (77 rows),
customers (91 rows), employees (9 rows), categories (8 rows), shippers (3 rows).
Date range: 2013-07-04 to 2015-05-06.
```

---

## 🔗 RELATIONSHIPS

### Verify and Fix the Data Model
```
Check all relationships in this model. The expected relationships are:
- order_details[orderID] → orders[orderID] (Many-to-One)
- order_details[productID] → products[productID] (Many-to-One)
- orders[customerID] → customers[customerID] (Many-to-One)
- orders[employeeID] → employees[employeeID] (Many-to-One)
- orders[shipperID] → shippers[shipperID] (Many-to-One)
- products[categoryID] → categories[categoryID] (Many-to-One)
- DimDate[Date] → orders[orderDate] (One-to-Many)

List any that are missing or have wrong cardinality and fix them.
All filter directions should be Single (dimension → fact).
```

---

## 🧪 VALIDATION PROMPTS

Use these to verify Claude's output is correct.

```
What is the total revenue for the entire Northwind dataset?
Calculate it as SUM of (unitPrice * quantity * (1 - discount)) from order_details.
Show the expected result so I can verify it in a card visual.
```

```
List every measure currently in the [Measures] table,
grouped by folder, with a one-line description of each.
```

```
Check all measures for common DAX mistakes:
missing CALCULATE context, using SUM where SUMX is needed,
using division without DIVIDE(), incorrect filter direction issues.
```

---

## 💡 Tips

1. **Always connect first** — run the connect prompt every new session
2. **Use exact column names** — match the CSV column names exactly (camelCase: orderID, customerID etc.)
3. **One feature at a time** — date table first, then measures, then time intelligence
4. **Always ask for comments** — saves hours when you revisit the model
5. **Test every measure** — build a matrix or card visual to verify numbers
6. **Save .pbix after each step** — Claude changes are live and immediate
7. **Note:** This dataset has no Suppliers table (unlike the original Microsoft Northwind)
