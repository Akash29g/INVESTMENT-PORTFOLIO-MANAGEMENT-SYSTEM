# Investment Portfolio Management System

A SQL database for tracking investment portfolios across stocks, mutual funds (with SIPs), fixed deposits, and government bonds. Built as a DBMS course project by Akash Goswami and Suhaas Karthikeyan.

## What it does

- Multi-user support, identified by PAN ID, with several portfolios per user
- Stock buy and sell tracking with FIFO cost basis
- Mutual funds with SIP installments and NAV updates
- Fixed deposit and government bond returns with compound interest
- Per-user reports: unrealised gains, total returns, holdings count

## Design

- 13 tables, mapped from an extended ER model (see `ER Model.jpg`)
- Normalization: 11 tables are in BCNF, `MUTUAL_FUNDS` and `MF_UNITS_SELL` are in 3NF. Full functional dependency analysis is in the documentation PDF.
- Tables: `USER`, `PORTFOLIO`, `TRANSACTIONS`, `STOCK_BUY`, `STOCK_SELL`, `STOCKS`, `STOCK_INFO`, `SCHEMES`, `MUTUAL_FUNDS`, `SIP`, `MF_UNITS_SELL`, `FIXED_DEPOSITS`, `GOVT_BONDS`

## How FIFO works here

Each buy keeps a remaining quantity in `STOCK_BUY.DUMMY_QUANTITY`. A sell reduces the oldest buy first, then the holding is recomputed:

```sql
-- weighted average cost of the remaining shares
UPDATE STOCKS SET average_price_per_share = (
  SELECT ROUND(SUM(PRICE_PER_SHARE * dummy_quantity) / total_quantity, 2)
  FROM (
    SELECT AMOUNT / QUANTITY AS PRICE_PER_SHARE, dummy_quantity
    FROM TRANSACTIONS JOIN STOCK_BUY
      ON STOCK_BUY.TRANSACTION_ID = TRANSACTIONS.TRANSACTION_ID
    WHERE ISIN = 'INE009A01021' AND portfolio_id = 'AG101'))
WHERE ISIN = 'INE009A01021' AND portfolio_id = 'AG101';
```

Example: buy 10 Infosys, buy 10 more, sell 13. The first buy drops to 0 remaining, the second to 7, and the holding shows 7 shares.

## Run it

Needs SQLite 3 only (PowerShell):

```powershell
sqlite3 portfolio.db ".read Investment_Portfolio_Management_System_Code.sql"
sqlite3 portfolio.db "SELECT * FROM STOCKS;"
```

The script creates all tables, loads sample users, portfolios, trades, FDs, bonds, and SIPs, then runs the report queries.

## Files

- `Investment_Portfolio_Management_System_Code.sql`: schema, sample data, queries
- `ER Model.jpg`: ER diagram
- `(Documentation) Investment Portfolio Management System.pdf`: ER to relational mapping and normalization
