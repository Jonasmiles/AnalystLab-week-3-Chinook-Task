# Chinook Database SQL Analysis

SQL Server analysis of the Chinook sample database — a fully normalized digital music store dataset covering artists, albums, tracks, customers, employees, and invoices.

## Overview

| Metric | Value |
|---|---|
| Database | Chinook (SQL Server) |
| Tables | 11, fully normalized |
| Domain | Digital music store |
| Source Script | `Chinook_SqlServer.sql` |

## Schema

```
ARTIST ──< ALBUM ──< TRACK >── GENRE
                       │
                       └──< INVOICELINE >── INVOICE >── CUSTOMER
                                                              │
                                                         EMPLOYEE (self-referencing via ReportsTo)
```

| Table | Primary Key | Relationship |
|---|---|---|
| Artist | `ArtistId` | 1-to-many → Album |
| Album | `AlbumId` | 1-to-many → Track |
| Track | `TrackId` | Linked to Album, Genre, MediaType |
| Genre | `GenreId` | Referenced by Track |
| Customer | `CustomerId` | 1-to-many → Invoice |
| Employee | `EmployeeId` | Self-referencing (manager hierarchy) |
| Invoice | `InvoiceId` | 1-to-many → InvoiceLine |
| InvoiceLine | `InvoiceLineId` | many-to-1 → Invoice & Track |
| Playlist / PlaylistTrack | `PlaylistId` | many-to-many with Track |

Revenue per line = `UnitPrice * Quantity`. Revenue chain: Customer → Invoice → InvoiceLine → Track → Album → Artist/Genre.

## Setup

1. Open `Chinook_SqlServer.sql` in SSMS or Azure Data Studio
2. Connect to your SQL Server instance
3. Execute the script (F5) — it creates the database, tables, and seed data

```bash
sqlcmd -S your_server_name -i Chinook_SqlServer.sql
```

## What's Covered

- **Core SQL** — `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, `HAVING`, aggregate functions (`SUM`, `AVG`, `COUNT`)
- **Advanced SQL** — `INNER`/`LEFT`/`RIGHT JOIN`, self-joins, subqueries, window functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `PARTITION BY`, `LAG`)
- **Business analysis** — top tracks/artists/genres, top customers, revenue trends, customer lifetime value, support rep performance
- **Optimization** — indexing strategy, clean/readable query style

## Key Insights

1. **Revenue concentrates around top customers and genres** — a small tier of customers and genres (led by Rock) drives a disproportionate share of revenue.
2. **Customer purchasing is mostly repeat behavior** — most customers return for multiple purchases rather than buying once.
3. **Support rep revenue contribution is uneven** — revenue generated per employee (via assigned customers) varies meaningfully.
4. **Revenue trends are trackable over time** — year-over-year comparison via `LAG()` clarifies growth, flat, or declining periods.
5. **Genre and artist performance is heavily skewed** — a handful of artists/genres account for the bulk of track sales.

## Repo Structure

```
├── scripts/
│   ├── 01_core_queries.sql
│   ├── 02_joins.sql
│   ├── 03_subqueries_window_functions.sql
│   ├── 04_business_questions.sql
│   └── 05_indexing_optimization.sql
├── reports/
│   └── Chinook_SQL_Queries_and_Insights.docx
├── Chinook_SqlServer.sql
└── README.md
```

## Sample Query

```sql
-- Top 10 customers by total spend, ranked
SELECT
    c.FirstName + ' ' + c.LastName AS customer,
    SUM(i.Total) AS total_spent,
    RANK() OVER (ORDER BY SUM(i.Total) DESC) AS customer_rank
FROM Customer c
INNER JOIN Invoice i ON c.CustomerId = i.CustomerId
GROUP BY c.FirstName, c.LastName
ORDER BY total_spent DESC;
```

Full scripts: [`scripts/`](./scripts) · Full report: [`reports/Chinook_SQL_Queries_and_Insights.docx`](./reports/Chinook_SQL_Queries_and_Insights.docx)

## Tools Used
SQL Server
