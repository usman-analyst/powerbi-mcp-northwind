# ⚡ Power BI MCP + Claude — Northwind Traders

> Connect Claude Desktop directly to Power BI Desktop using the Microsoft Power BI Modeling MCP Server.
> Built and tested on the **Northwind Traders dataset from Kaggle** (jeetahirwar/northwind-traders).

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Claude](https://img.shields.io/badge/Claude-Desktop-cc785c?style=for-the-badge)
![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-00b4d8?style=for-the-badge)
![Windows](https://img.shields.io/badge/Windows-Only-0078D6?style=for-the-badge&logo=windows)
![Status](https://img.shields.io/badge/Status-Preview-orange?style=for-the-badge)

---

## 🎯 What This Project Is

This repo documents how to connect **Claude Desktop** to **Power BI Desktop** via Microsoft's
**Power BI Modeling MCP Server** — so you can build and modify your semantic model
using plain English instead of writing DAX manually.

| Task | Without MCP | With Claude + MCP |
|------|------------|-------------------|
| Build a date table | Write 50+ lines of DAX | *"Build a date table"* |
| Create 10 KPI measures | 1–2 hours manually | Under 2 minutes |
| Add column descriptions to all 7 tables | Click every column one by one | *"Add descriptions to all columns"* |
| Generate model documentation | Hours of copy-paste | *"Document my entire model"* |
| Time intelligence (MTD, YTD, YoY) | Complex DAX knowledge | One prompt |

---

## 📊 Dataset — Northwind Traders (Kaggle)

**Source:** [kaggle.com/datasets/jeetahirwar/northwind-traders](https://www.kaggle.com/datasets/jeetahirwar/northwind-traders)

A fictional gourmet food trading company. 7 CSV tables, imported into Power BI Desktop.

| Table | Rows | Description |
|-------|------|-------------|
| orders | 830 | Sales order headers |
| order_details | 2,155 | Line items per order |
| products | 77 | Product catalogue |
| customers | 91 | Customer master data |
| employees | 9 | Employee records |
| categories | 8 | Product categories |
| shippers | 3 | Shipping carriers |

**Date range:** 4 July 2013 → 6 May 2015

---

## 🗂 Exact Schema (from your CSV files)

### orders
| Column | Type | Sample Value |
|--------|------|-------------|
| orderID | Integer | 10248 |
| customerID | Text | VINET |
| employeeID | Integer | 5 |
| orderDate | Date | 2013-07-04 |
| requiredDate | Date | 2013-08-01 |
| shippedDate | Date | 2013-07-16 |
| shipperID | Integer | 3 |
| freight | Decimal | 32.38 |

### order_details
| Column | Type | Sample Value |
|--------|------|-------------|
| orderID | Integer | 10248 |
| productID | Integer | 11 |
| unitPrice | Decimal | 14.00 |
| quantity | Integer | 12 |
| discount | Decimal | 0 |

### products
| Column | Type | Sample Value |
|--------|------|-------------|
| productID | Integer | 1 |
| productName | Text | Chai |
| quantityPerUnit | Text | 10 boxes x 20 bags |
| unitPrice | Decimal | 18.00 |
| discontinued | Integer | 0 (false) / 1 (true) |
| categoryID | Integer | 1 |

### customers
| Column | Type | Sample Value |
|--------|------|-------------|
| customerID | Text | ALFKI |
| companyName | Text | Alfreds Futterkiste |
| contactName | Text | Maria Anders |
| contactTitle | Text | Sales Representative |
| city | Text | Berlin |
| country | Text | Germany |

### employees
| Column | Type | Sample Value |
|--------|------|-------------|
| employeeID | Integer | 1 |
| employeeName | Text | Nancy Davolio |
| title | Text | Sales Representative |
| city | Text | New York |
| country | Text | USA |
| reportsTo | Integer | 8 (manager's employeeID) |

### categories
| Column | Type | Sample Value |
|--------|------|-------------|
| categoryID | Integer | 1 |
| categoryName | Text | Beverages |
| description | Text | Soft drinks, coffees, teas... |

### shippers
| Column | Type | Sample Value |
|--------|------|-------------|
| shipperID | Integer | 1 |
| companyName | Text | Speedy Express |

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        YOUR MACHINE                          │
│                                                             │
│  ┌──────────────┐    ┌─────────────────┐    ┌───────────┐  │
│  │ Power BI     │◄──►│ powerbi-         │◄──►│  Claude   │  │
│  │ Desktop      │    │ modeling-mcp     │    │  Desktop  │  │
│  │ (.pbix open) │    │ (VS Code ext)    │    │  (app)    │  │
│  └──────────────┘    └─────────────────┘    └─────┬─────┘  │
│                                                     │        │
│                                              Anthropic API   │
└────────────────────────────────────────────────────┼────────┘
                                                     ▼
                                          ┌─────────────────┐
                                          │  Claude API      │
                                          │  (internet)      │
                                          └─────────────────┘
```

**What leaves your machine:** Table/column names, measure formulas, relationships, descriptions.
**What stays local:** Your actual data rows — never sent externally.

---

## ✅ Prerequisites

- [ ] **Power BI Desktop** — [Microsoft Store](https://aka.ms/pbidesktopstore)
- [ ] **VS Code** — [code.visualstudio.com](https://code.visualstudio.com)
- [ ] **Claude Desktop** — [claude.ai/download](https://claude.ai/download)
- [ ] **Windows 10/11** — MCP extension is Windows only
- [ ] **Northwind CSV files** — [Kaggle dataset](https://www.kaggle.com/datasets/jeetahirwar/northwind-traders)

---

## 🚀 Setup — Step by Step

### Step 1 — Load Northwind into Power BI

1. Open **Power BI Desktop**
2. **Get Data → Text/CSV** → load each of the 7 CSV files:
   `categories`, `customers`, `employees`, `order_details`, `orders`, `products`, `shippers`
3. In **Model view**, create these relationships manually:

| From | To | Cardinality |
|------|----|-------------|
| order_details[orderID] | orders[orderID] | Many → One |
| order_details[productID] | products[productID] | Many → One |
| orders[customerID] | customers[customerID] | Many → One |
| orders[employeeID] | employees[employeeID] | Many → One |
| orders[shipperID] | shippers[shipperID] | Many → One |
| products[categoryID] | categories[categoryID] | Many → One |

4. Save as `northwind.pbix`

---

### Step 2 — Install the MCP Extension in VS Code

1. Open **VS Code**
2. Press `Ctrl+Shift+X` to open Extensions
3. Search: `PowerBI Modeling MCP`
4. Install the one by **Microsoft** (blue verified tick)

---

### Step 3 — Find the MCP Server File Path

1. Open File Explorer, paste in address bar and press Enter:
   ```
   %USERPROFILE%\.vscode\extensions
   ```
2. Find the folder starting with:
   ```
   ms-analysisservices.powerbi-modeling-mcp-...
   ```
3. Open it → open `server` subfolder
4. **Copy the full path** to `PowerBIModelingMCP.exe`

---

### Step 4 — Configure Claude Desktop

1. Open **Claude Desktop** → `File → Settings → Developer → Edit Config`
2. Open `claude_desktop_config.json` in VS Code
3. Replace contents with the template from [`setup/claude_desktop_config.json`](setup/claude_desktop_config.json)
4. Replace the file path with your actual path from Step 3
5. **Every `\` must be doubled to `\\`** in JSON

---

### Step 5 — Restart & Verify

1. Fully close Claude Desktop (check system tray)
2. Reopen Claude Desktop
3. Go to `File → Settings → Developer`
4. Confirm **powerbi-modeling** shows as connected (green)

---

### Step 6 — Connect Claude to Your Model

Make sure `northwind.pbix` is open in Power BI Desktop, then in Claude Desktop type:

```
Connect to northwind in Power BI Desktop
```

Claude responds:
> *"Connected to the northwind model in Power BI Desktop"*

**You're live.** Claude can now read and write your semantic model.

---

## 💬 Prompt Library

See the full tested collection: [`prompts/prompts_library.md`](prompts/prompts_library.md)

### Quick Start Prompts

**Build the date table:**
```
Build a date table. Use the orderDate column from the orders table to detect
the date range (2013-07-04 to 2015-05-06). Include year, quarter, month number,
month name, week number, day of week, IsWeekend flag, and sort order columns.
Mark it as a date table and create a relationship to orders[orderDate].
```

**Create base measures:**
```
Create a [Measures] table with basic sales aggregations using the Northwind tables.
For revenue, calculate: unitPrice * quantity * (1 - discount) from order_details.
Include: Total Revenue, Total Orders, Average Order Value, Total Units Sold,
Total Freight. Organise into folders and add a DAX comment to each measure.
```

**Time intelligence:**
```
Using the [Measures] table and the date table connected to orders[orderDate],
create time intelligence measures for Total Revenue: MTD, QTD, YTD,
Previous Month, Previous Year, MoM Growth %, YoY Growth %, Rolling 3 Month.
```

**Document everything:**
```
Generate complete markdown documentation for this Power BI model.
Include all 7 tables (orders, order_details, products, customers, employees,
categories, shippers), all columns with descriptions, all measures with formulas,
and all relationships.
```

---

## 📁 Project Structure

```
powerbi-mcp-northwind/
├── README.md
├── setup/
│   └── claude_desktop_config.json      ← Edit this with your file path
├── prompts/
│   └── prompts_library.md              ← Full prompt collection
└── outputs/
    ├── generated_dax_measures.md       ← Base measures DAX
    ├── time_intelligence_dax.md        ← Time intel measures
    ├── calendar_table_dax.md           ← Date dimension DAX
    └── model_documentation.md         ← Auto-generated model docs
```

---

## 🔐 Security

| Data | Sent to API? | Notes |
|------|-------------|-------|
| Column/table names | ✅ Yes | Structural metadata only |
| Measure DAX formulas | ✅ Yes | Business logic |
| **Actual row data** | ❌ Never | Stays on your machine |

| Scenario | Use |
|----------|-----|
| Test data (this project) | ✅ Claude Desktop Free |
| Client work | ⚠️ Check NDA first |
| Corporate / sensitive | 🔒 Azure OpenAI in your tenant |
| Healthcare / Finance | 🔒 Local LLM (Ollama) only |

---

## ⚠️ Warnings

1. **Preview feature** — do not use on production reports
2. **Back up your .pbix** before every AI session
3. **Verify all DAX** — test measures against known values
4. **Free plan rate limits** — space out large requests

---

## 📚 References

- [Power BI Modeling MCP — Microsoft](https://learn.microsoft.com/en-us/power-bi)
- [Model Context Protocol — Anthropic](https://www.anthropic.com/news/model-context-protocol)
- [Northwind Traders Dataset — Kaggle](https://www.kaggle.com/datasets/jeetahirwar/northwind-traders)

---

*Community learning project. Not affiliated with Microsoft or Anthropic.*
