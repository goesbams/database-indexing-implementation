# Indexing in Databases

## 1. Overview
Indexing is a technique used to improve the performance of database queries by allowing faster retrieval of records. It functions like an index in a book, reducing the need for full table scans. Without indexing, a database would have to scan every row in a table to find relevant data, which can be slow and inefficient for large datasets.

## 2. How Indexing Works
An index is a separate data structure (e.g., B-Tree, Hash Table) that maintains a mapping between indexed columns and the actual location of records. When a query is executed, the database engine first searches the index instead of scanning the entire table, significantly improving performance.

### Why Does Indexing Improve Performance?
Without an index, when a query searches for a specific value (e.g., `SELECT * FROM users WHERE name = 'Budi';`), the database must perform a **full table scan**, meaning it checks every row in the table. This is slow for large datasets.

However, with an index, the database can use efficient lookup mechanisms such as B-Trees or Hash Tables, which work as follows:

### Step-by-Step Example:
#### Without Index (Full Table Scan)
1. Suppose we have a `users` table with 1,000,000 records.
2. Running `SELECT * FROM users WHERE name = 'Budi';` means the database has to check all 1,000,000 rows.
3. The process involves reading each row from disk, comparing the `name` column, and collecting matching records.
4. This is inefficient and slow, especially for large datasets.

#### With Index (Optimized Search)
1. We create an index on the `name` column:
   ```sql
   CREATE INDEX idx_name ON users(name);
   ```
2. The database builds a B-Tree (or Hash Table) structure for the `name` column, storing pointers to actual records.
3. When we run `SELECT * FROM users WHERE name = 'Budi';`, the database:
   - Searches the B-Tree, which is structured like a binary search tree.
   - Finds the location of `Budi` in `O(log N)` time instead of `O(N)`.
   - Directly retrieves the records instead of scanning the whole table.
4. This reduces disk I/O and speeds up query execution dramatically.

#### How a B-Tree Index is Built
1. **Sorting the Data:**
   - The database sorts indexed values (e.g., names) in ascending order.
   - Example: `Budi, Ucok, Adam, Panjul, Andi, Dita, Zara`.

2. **Creating the Root Node:**
   - The middle value (`Panjul`) becomes the root.
   - Left subtree: `Adam, Budi, Ucok`
   - Right subtree: `Andi, Dita, Zara`

3. **Building Subtrees:**
   - Each subtree follows the same principle:
```
        Panjul
      /        \
  Adam          David
  /    \        /     \
Budi   Ucok  Dita     Zara
```
4. **Searching in the B-Tree:**
   - To find `Budi`, start from `Panjul`.
   - Since `Budi < Panjul`, go left to `Adam`.
   - Since `Budi < Adam`, go left again.
   - Found `Budi` quickly instead of scanning all rows.

#### Example: Hash Index Process
- If a Hash Index is used, the database hashes the `name` values:
  ```
  Hash(Budi) -> Bucket 7
  Hash(Panjul) -> Bucket 3
  Hash(Zara)  -> Bucket 5
  ```
- The database directly accesses **Bucket 7**, retrieving `Budi` instantly, let's say the query retreives rows (ID=2, ID=4).
- A hash index lookup is O(1) time complexity
- hash index donot support range queries (e.g `BETWEEN name 'A' AND 'C'`)

### Conclusion
Indexing improves performance by reducing the number of rows that need to be scanned. Instead of searching the entire table, the database leverages optimized data structures like B-Trees and Hash Tables to find records faster, reducing query execution time from **O(N) (linear search)** to **O(log N) (B-Tree search) or O(1) (Hash lookup)**.

## 3. Types of Indexing
### a. Based on Data Storage & Sorting
- **Clustered Index**: Sorts and stores data physically in order. Only one per table because the data rows themselves are sorted.
  - Example: If you create a clustered index on `order_date`, the database will store rows in ascending order of `order_date`.
  ```sql
  CREATE CLUSTERED INDEX idx_order ON orders(order_date);
  ```
  - **How It Works:**
    - The database stores rows **physically** in sorted order.
    - When searching for a specific `order_date`, the database **navigates the B-Tree** to find the row quickly.
    - Range queries (`BETWEEN`, `<`, `>` conditions) are optimized since data is already sorted.

  - **How Clustered Index Works (Finding a Row)**:
    A clustered index sorts and stores data physically in order. When a query searches for a specific row, the database can efficiently locate it using a B-Tree structure.

  - **Step-by-Step Example:**
    Assume we have an `orders` table with a clustered index on `order_date`:
    ```sql
    CREATE CLUSTERED INDEX idx_order ON orders(order_date);
    ```

    1. **How Data is Stored:**
      With a clustered index, rows are stored physically in the order of `order_date`:

        | order_id | order_date  | customer_name |
        |----------|------------|---------------|
        | 101      | 2024-01-02 | Budi         |
        | 102      | 2024-01-05 | Panjul           |
        | 103      | 2024-01-08 | Dita       |
        | 104      | 2024-01-12 | Andi         |
        | 105      | 2024-01-15 | Adam          |

          The database organizes this in a B-Tree structure:
          ```
                  2024-01-08
                /           \
          2024-01-02     2024-01-12
              \            /
          2024-01-05   2024-01-15
          ```

    2. **Query Execution (Finding a Row)**
      ```sql
      SELECT * FROM orders WHERE order_date = '2024-01-08';
      ```
      - The query starts at the root node (`2024-01-08`).
      - Since `2024-01-08` matches the search value, the database immediately finds the row.
      - The query returns the corresponding row **without scanning the full table**.

    3. **Query Execution (Range Search)**
      ```sql
      SELECT * FROM orders WHERE order_date BETWEEN '2024-01-05' AND '2024-01-12';
      ```
      - The search starts at the root node (`2024-01-08`).
      - It moves left to `2024-01-05` and right to `2024-01-12`.
      - The database retrieves the range `[2024-01-05, 2024-01-08, 2024-01-12]` efficiently.
      

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
  - Useful for queries like `SELECT * FROM employees WHERE name = 'Budi' AND age = 30;`
  - Also effective for order query like `SELECT * FROM employees ORDER BY name, age;`
  - Best Practice:
    - Use a composite index only if queries frequently filter using multiple columns in order.
    - Ensure that the leading column in the index is frequently used in queries.
    - Avoid indexing too many columns, as it increases index size and affects write performance.
    - If different query patterns exist, consider separate indexes on individual columns or index reordering.

### d. Based on Search Optimization
- **Full-Text Index**: Optimized for searching text-based data, allowing fast lookups of words within text columns.
  ```sql
  CREATE FULLTEXT INDEX idx_description ON products(description);
  ```
  ```sql
  SELECT * FROM products WHERE MATCH(description) AGAINST('laptop');
  ```
  **Example:**

  | id | description                                |
  |----|--------------------------------------------|
  | 1  | High-performance gaming laptop with SSD.    |
  | 2  | Affordable business laptop with long battery life. |
  | 3  | Gaming keyboard with RGB lighting.          |

  When we create a **Full-Text Index**:
  ```sql
  CREATE FULLTEXT INDEX idx_description ON products(description);
  ```
  👉 The database tokenizes the text into words and builds an inverted index like this:
  | Word       | Document IDs (Product IDs) |
  |------------|----------------------------|
  | gaming     | 1, 3                       |
  | laptop     | 1, 2                       |
  | performance| 1                          |
  | affordable | 2                          |
  | keyboard   | 3                          |

  ```sql
  SELECT * FROM products WHERE MATCH(description) AGAINST('laptop');
  ```
    - The query engine checks the inverted index for laptop.
    - It finds Product IDs: 1, 2.
    - The database retrieves only those rows from the table.

- **Spatial Index** is designed to optimize geospatial queries—such as searching for places near a point (latitude, longitude) or finding all locations within a certain area. Instead of using a traditional B-tree or inverted index, it uses special data structures like R-tree (Region Tree) or QuadTree, which are optimized for multidimensional data.

  ```sql
  CREATE SPATIAL INDEX idx_location ON locations(coordinates);
  ```
  **How data is stored in a Spatial Index?**
  - Consider a `locations` table with geospatial coordinates (e.g., longitude, latitude):

  | id | name          | coordinates (POINT)       |
  |----|---------------|---------------------------|
  | 1  | Jakarta Mall  | POINT(106.8456, -6.2088)   |
  | 2  | Bandung Cafe  | POINT(107.6191, -6.9175)   |
  | 3  | Surabaya Park | POINT(112.7508, -7.2575)   |

  When we create a spatial index:
  ```sql
  CREATE SPATIAL INDEX idx_location ON locations(coordinates);
  ```
  👉 The database organizes the points using an R-tree (a tree-like structure that groups nearby locations).

  **How does R-Tree works?**
  - Unlike B-trees (which work well for sorted data), R-trees store bounding boxes around points.
A simplified R-tree might look like:

  ```scss
    Root
  ├── [106.8, -6.2] (Jakarta)
  ├── [107.6, -6.9] (Bandung)
  ├── [112.7, -7.2] (Surabaya)
  ```
  - Instead of scanning all rows, queries only check relevant branches.
  - Searching for nearby places is much faster than using a WHERE condition without an index.

  **How to use spatial index?**
  - Find the nearest locations. Example: Find places within 10 km of Jakarta (106.8456, -6.2088).
  ```sql
    SELECT * FROM locations 
    WHERE ST_Distance_Sphere(coordinates, POINT(106.8456, -6.2088)) < 10000;
  ```
  - `ST_Distance_Sphere()` calculates the distance in meters (10,000m = 10km).
  

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
