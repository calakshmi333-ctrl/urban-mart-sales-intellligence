# UrbanMart Sales Intelligence Pipeline

A small SQL pipeline for the fictional **UrbanMart Retail Group**. It builds a SQLite database from raw setup scripts, validates the data, and runs the queries behind the weekly sales dashboard.

## What the Pipeline Does

1. **Setup:** drops and recreates all six tables, then loads the sample data.
2. **Validate:** checks keys, relationships and known data quirks.
3. **Analyse:** runs the sales intelligence queries (orders, revenue, average order value, return rate).
4. **Publish:** shares the final queries as a GitHub Gist.

The setup script is idempotent. Re-running it rebuilds the database from scratch with no duplicates.

## Data Tables

| Table | Description | Key Columns |
|---|---|---|
| `customers` | Customer profiles (8 rows) | `customer_id` (PK), `name`, `city`, `tier`, `signup_date` |
| `products` | Product catalogue with cost and sale price (8 rows) | `product_id` (PK), `name`, `category`, `cost_price`, `sale_price` |
| `orders` | Order headers (12 rows) | `order_id` (PK), `customer_id`, `order_date`, `channel`, `status` |
| `order_items` | Line items for each order (19 rows) | `item_id` (PK), `order_id`, `product_id`, `qty`, `unit_price` |
| `returns` | Refunds issued against orders (2 rows) | `return_id` (PK), `order_id`, `reason`, `refund_amount` |
| `api_events` | Web/app events with JSON metadata (5 rows) | `event_id` (PK), `event_type`, `timestamp`, `user_id`, `metadata` |

### Relationships

- `orders.customer_id` → `customers.customer_id`
- `order_items.order_id` → `orders.order_id`
- `order_items.product_id` → `products.product_id`
- `returns.order_id` → `orders.order_id`
- `api_events.user_id` → `customers.customer_id`

- ## How to Run

### Prerequisites

- SQLite 3.38 or later (the `sqlite3` command-line tool)
- Git

### Steps

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd urbanmart-sales-intelligence

# 2. Build the database (drops and recreates all tables)
sqlite3 urbanmart.db < setup.sql

# 3. Confirm the tables and row counts
sqlite3 urbanmart.db "SELECT 'customers', COUNT(*) FROM customers
UNION ALL SELECT 'products', COUNT(*) FROM products
UNION ALL SELECT 'orders', COUNT(*) FROM orders
UNION ALL SELECT 'order_items', COUNT(*) FROM order_items
UNION ALL SELECT 'returns', COUNT(*) FROM returns
UNION ALL SELECT 'api_events', COUNT(*) FROM api_events;"

# 4. Run the analysis queries
sqlite3 -header -column urbanmart.db < queries.sql
```

### Expected Row Counts

| Table | Rows |
|---|---|
| `customers` | 8 |
| `products` | 8 |
| `orders` | 12 |
| `order_items` | 19 |
| `returns` | 2 |
| `api_events` | 5 |

### Validation Checks

Run these after setup. Both should return no rows.

