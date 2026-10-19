# Online Book Store – Sales & Inventory Dashboard (Power BI)

A single-page Power BI report that analyses sales, customers and stock for an online bookstore. It reads from a SQL Server database (`OnlineBookstore`) and shows revenue, order volume, genre performance, top-selling books, customer loyalty and inventory health.

**File:** `ONLINE_BOOK_STORE.pbix`

## Report page

One page (1920 × 1080), titled "Page 2".

| Area | Visual | What it shows |
|---|---|---|
| KPI cards | Total Revenue, Total Stock, Books Sold, MoM Revenue Change, Repeat Customers | Headline numbers for the current filter selection |
| Revenue trend | Stacked area chart | Revenue by month |
| Genre analysis | Bar chart | Revenue by genre |
| Genre analysis | Column chart | Total orders by genre |
| Revenue split | Donut chart | Total revenue breakdown |
| Top sellers | Table "Top 10 Books by Revenue" | Title, Author, Books Sold, Total Revenue |
| Inventory | Card | Low Stock, Out of Stock and Total Stock counts |
| Slicers | Country, Order Date, Year, Genre | Filter the whole page |

## Data source

SQL Server, imported with Power Query:

- Server: `localhost\SQLEXPRESS`
- Database: `OnlineBookstore`
- Tables: `dbo.Books`, `dbo.Customers`, `dbo.Orders`

To refresh the report you need access to that database, or you can repoint the source under **Transform data → Data source settings**.

## Data model

Star-style schema with `Orders` as the fact table.

```
Customers (1) ──< Orders >── (1) Books
                    
DateTable   (calendar dimension, 730 dates)
```

| Table | Rows (approx.) | Columns |
|---|---|---|
| Books | 500 | Book_ID, Title, Author, Genre (7 genres), Published_Year, Price, Stock |
| Customers | 500 | Customer_ID, Name, Email, Phone, City, Country |
| Orders | 500 | Order_ID, Customer_ID, Book_ID, Order_Date, Quantity, Total_Amount |
| DateTable | 730 | Date, Year, Month Number, Month, Year Month |

Relationships (many-to-one, single direction, active):

- `Orders[Customer_ID]` → `Customers[Customer_ID]`
- `Orders[Book_ID]` → `Books[Book_ID]`

The file also contains auto-generated date tables (`LocalDateTable_…`, `DateTableTemplate_…`) that Power BI creates for date hierarchies.

## DAX measures

| Measure | Table | Definition |
|---|---|---|
| Total Revenue | Orders | `SUM(Orders[Total_Amount])` |
| Total Orders | Orders | `DISTINCTCOUNT(Orders[Order_ID])` |
| Books Sold | Orders | `SUM(Orders[Quantity])` |
| Average Order_Value | Orders | `DIVIDE([Total Orders],[Total Revenue],0)` ⚠️ see notes |
| Revenue per Book Sold | Orders | `DIVIDE([Total Revenue],[Books Sold],0)` |
| Previous Month Revenue | DateTable | `CALCULATE([Total Revenue], DATEADD(DateTable[Date], -1, MONTH))` |
| MoM Revenue Change | DateTable | `[Total Revenue] - [Previous Month Revenue]` |
| MoM Revenue Growth % | DateTable | `DIVIDE([MoM Revenue Change],[Previous Month Revenue],0)` |
| Total Stock | Books | `SUM(Books[Stock])` |
| Books in Catalog | Books | `DISTINCTCOUNT(Books[Book_ID])` |
| Low Stock Books | Books | Books with stock < 10 |
| Out of Stock Books | Books | Books with stock ≤ 0 |
| Repeat Customers | Customers | Customers with 2 or more distinct orders |
| Total Customer | Customers | `COUNT(Customers[Customer_ID])` |
| Repeat Customer % | Customers | `DIVIDE([Repeat Customers],[Total Customer],0)` |

## How to open it

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (Windows).
2. Open `ONLINE_BOOK_STORE.pbix`.
3. If prompted, supply credentials for the SQL Server source, or choose **Cancel** to browse the data already cached in the file.

## Known issues / suggested fixes

These were found while reading the model. They are worth fixing before sharing the report.

1. **Average Order_Value is inverted.** It divides orders by revenue. It should be:
   ```DAX
   Average Order Value = DIVIDE([Total Revenue], [Total Orders], 0)
   ```
2. **`DateTable` is not related to `Orders`.** No relationship links `DateTable[Date]` to `Orders[Order_Date]`, so the Date/Year slicers, the monthly revenue chart and the MoM measures may not filter or compute against order dates as intended. Add a relationship from `DateTable[Date]` (one) to `Orders[Order_Date]` (many) and mark `DateTable` as the date table.
3. **Page name.** The page is called "Page 2"; consider renaming it, for example "Sales Overview".
4. **Personal data.** `Customers` holds names, emails and phone numbers. Remove or mask them before publishing or sharing the file.

## Possible extensions

- Add a Quarter column to `DateTable` and a quarterly view.
- Add pages for customer geography (City/Country) and author performance.
- Add Average Order Value and Repeat Customer % cards to the existing page.
- Publish to Power BI Service with an on-premises data gateway for scheduled refresh.
