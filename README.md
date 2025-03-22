# Indexing in Databases

## 1. Overview
Indexing is a technique used to improve the performance of database queries by allowing faster retrieval of records. It functions like an index in a book, reducing the need for full table scans. Without indexing, a database would have to scan every row in a table to find relevant data, which can be slow and inefficient for large datasets.

## 2. How Indexing Works
An index is a separate data structure (e.g., B-Tree, Hash Table) that maintains a mapping between indexed columns and the actual location of records. When a query is executed, the database engine first searches the index instead of scanning the entire table, significantly improving performance.

## 3. Types of Indexing
### a. Based on Data Storage & Sorting
- **Clustered Index**: Sorts and stores data physically in order. Only one per table because the data rows themselves are sorted.
  - Example: If you create a clustered index on `order_date`, the database will store rows in ascending order of `order_date`.
  ```sql
  CREATE CLUSTERED INDEX idx_order ON orders(order_date);
  ```
- **Non-Clustered Index**: Stores pointers to actual rows. Multiple non-clustered indexes can exist on a table.
  - Example: This creates an index on `name` without affecting row order.
  ```sql
  CREATE INDEX idx_customer ON customers(name);
  ```

### b. Based on Column Uniqueness
- **Primary Index**: Automatically created for the primary key, ensuring uniqueness.
  ```sql
  CREATE TABLE users (
      id INT PRIMARY KEY,
      name VARCHAR(100)
  );
  ```
  - The `PRIMARY KEY` automatically creates a unique, clustered index.
- **Unique Index**: Prevents duplicate values in a column but does not have to be a primary key.
  ```sql
  CREATE UNIQUE INDEX idx_unique_email ON users(email);
  ```

### c. Based on Multiple Columns
- **Composite Index**: Created on multiple columns to optimize queries that filter using multiple conditions.
  ```sql
  CREATE INDEX idx_name_age ON employees(name, age);
  ```
  - Useful for queries like `SELECT * FROM employees WHERE name = 'Alice' AND age = 30;`

### d. Based on Search Optimization
- **Full-Text Index**: Optimized for searching text-based data, allowing fast lookups of words within text columns.
  ```sql
  CREATE FULLTEXT INDEX idx_description ON products(description);
  ```
  ```sql
  SELECT * FROM products WHERE MATCH(description) AGAINST('laptop');
  ```
- **Spatial Index**: Used for geospatial data (latitude, longitude), enabling fast geographic searches.
  ```sql
  CREATE SPATIAL INDEX idx_location ON locations(coordinates);
  ```

### e. Based on Indexing Algorithm
- **B-Tree Index**: The default index type in most databases, efficient for range queries and sorting.
  ```sql
  CREATE INDEX idx_salary ON employees(salary);
  ```
- **Hash Index**: Provides fast exact matches but is not efficient for range queries.
  ```sql
  CREATE INDEX idx_email_hash ON users USING HASH (email);
  ```
- **Bitmap Index**: Stores bitmap representations of values, useful for low-cardinality columns (e.g., Yes/No, Male/Female).
  ```sql
  CREATE BITMAP INDEX idx_gender ON employees(gender);
  ```

### f. Based on Storage (Special Cases)
- **Covering Index**: Stores all columns needed for a query to avoid accessing the table.
  ```sql
  CREATE INDEX idx_order_details ON orders(customer_id, order_date, total_price);
  ```
  - This speeds up queries that request only `customer_id`, `order_date`, and `total_price`.
- **Bitmap Index**: Uses bitmaps instead of trees for indexing, making it efficient for categorical data in data warehouses.
  ```sql
  CREATE BITMAP INDEX idx_status ON employees(status);
  ```

## 4. When to Use Indexing
- Frequently queried large tables.
- Columns used in `WHERE`, `ORDER BY`, `JOIN` conditions.
- Low-cardinality columns (Bitmap Index).
- Text-based searches (Full-Text Index).
- Geospatial data retrieval (Spatial Index).

## 5. SQL Examples
### Create Index
```sql
CREATE INDEX idx_name ON customers(name);
CREATE UNIQUE INDEX idx_unique_email ON users(email);
CREATE INDEX idx_name_age ON employees(name, age);
```

### Drop Index
```sql
DROP INDEX idx_name;
```

### Full-Text Search (MySQL Example)
```sql
CREATE FULLTEXT INDEX idx_description ON products(description);
SELECT * FROM products WHERE MATCH(description) AGAINST('laptop');
```

## 6. Pros & Cons of Indexing
### ✅ Advantages
- **Faster queries**: Speeds up search and retrieval operations (`SELECT`, `JOIN`, `ORDER BY`).
- **Optimizes large dataset searches**: Reduces the need for full table scans.
- **Reduces I/O operations**: The database reads only indexed records instead of scanning the entire table.

### ❌ Disadvantages
- **Slows down `INSERT`, `UPDATE`, `DELETE` operations**: Every change requires updating the index.
- **Requires extra storage**: Indexes occupy additional disk space.
- **Poorly designed indexes can hurt performance**: Unnecessary indexes can increase overhead without providing much benefit.

## 7. Choosing the Right Index
| **Use Case**         | **Recommended Index** |
|----------------------|----------------------|
| Fast lookups        | B-Tree / Hash Index  |
| Sorting & Range Queries | Clustered Index |
| Text Searching      | Full-Text Index      |
| Multi-column queries | Composite Index     |
| Geospatial Data     | Spatial Index       |
| Categorical data    | Bitmap Index        |

Indexes are a powerful tool for optimizing database performance, but they should be used strategically. Over-indexing can degrade performance instead of improving it. 🚀
