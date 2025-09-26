# SQL Databases 🗃️

[![SQL Databases](https://img.shields.io/badge/SQL-Relational%20Databases-blue)](https://en.wikipedia.org/wiki/SQL)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Advanced%20SQL-green)](https://postgresql.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Relational Database Management Systems** - SQL databases are relational databases that store data in structured tables with predefined schemas, using SQL (Structured Query Language) for data manipulation and retrieval.

## 🎯 Overview

SQL databases are relational database management systems that:
- **Store Structured Data** - Organize data in tables with rows and columns
- **Ensure ACID Properties** - Atomicity, Consistency, Isolation, Durability
- **Use SQL Language** - Standardized query language for data operations
- **Maintain Data Integrity** - Enforce constraints and relationships
- **Support Transactions** - Ensure data consistency across operations
- **Provide Scalability** - Vertical and horizontal scaling options

## 🚀 Key Features

### Core Capabilities
- **Relational Model** - Data organized in tables with relationships
- **ACID Compliance** - Ensures data reliability and consistency
- **SQL Standard** - Standardized query language across vendors
- **Data Integrity** - Constraints, foreign keys, and validation rules
- **Transaction Support** - Multi-statement transactions with rollback
- **Concurrent Access** - Multiple users accessing data simultaneously

### Advanced Features
- **Indexing** - Optimize query performance with indexes
- **Views** - Virtual tables for simplified data access
- **Stored Procedures** - Pre-compiled SQL code for better performance
- **Triggers** - Automated actions on data changes
- **Partitioning** - Divide large tables for better performance
- **Replication** - Copy data across multiple servers

## 🏗️ Architecture

### SQL Database Components
```
SQL Database Architecture
├── 🗃️ Storage Engine
│   ├── Data Files
│   ├── Index Files
│   ├── Transaction Logs
│   └── Buffer Pool
├── 🔍 Query Engine
│   ├── Parser
│   ├── Optimizer
│   ├── Executor
│   └── Cache Manager
├── 🔐 Security Layer
│   ├── Authentication
│   ├── Authorization
│   ├── Encryption
│   └── Audit Logging
├── 🌐 Network Layer
│   ├── Connection Pool
│   ├── Protocol Handler
│   └── Session Management
└── 🔧 Management
    ├── Backup/Restore
    ├── Monitoring
    └── Maintenance
```

## 🛠️ Popular SQL Databases

### PostgreSQL
```bash
# Install PostgreSQL
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# macOS
brew install postgresql

# Start PostgreSQL
sudo systemctl start postgresql
```

```python
import psycopg2
from psycopg2.extras import RealDictCursor

# Connect to PostgreSQL
conn = psycopg2.connect(
    host="localhost",
    database="myapp",
    user="postgres",
    password="password"
)

# Create cursor
cur = conn.cursor(cursor_factory=RealDictCursor)

# Create table
cur.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        email VARCHAR(100) UNIQUE NOT NULL,
        age INTEGER,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
""")

# Insert data
cur.execute("""
    INSERT INTO users (name, email, age)
    VALUES (%s, %s, %s)
""", ("Alice", "alice@example.com", 30))

# Query data
cur.execute("SELECT * FROM users WHERE age > %s", (25,))
users = cur.fetchall()
for user in users:
    print(f"Name: {user['name']}, Email: {user['email']}")

# Update data
cur.execute("""
    UPDATE users SET age = %s WHERE email = %s
""", (31, "alice@example.com"))

# Delete data
cur.execute("DELETE FROM users WHERE id = %s", (1,))

# Commit changes
conn.commit()

# Close connection
cur.close()
conn.close()
```

### MySQL
```python
import mysql.connector
from mysql.connector import Error

# Connect to MySQL
try:
    connection = mysql.connector.connect(
        host='localhost',
        database='myapp',
        user='root',
        password='password'
    )
    
    if connection.is_connected():
        cursor = connection.cursor()
        
        # Create table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                email VARCHAR(100) UNIQUE NOT NULL,
                age INT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        # Insert data
        cursor.execute("""
            INSERT INTO users (name, email, age)
            VALUES (%s, %s, %s)
        """, ("Alice", "alice@example.com", 30))
        
        # Query data
        cursor.execute("SELECT * FROM users WHERE age > %s", (25,))
        users = cursor.fetchall()
        for user in users:
            print(f"Name: {user[1]}, Email: {user[2]}")
        
        # Update data
        cursor.execute("""
            UPDATE users SET age = %s WHERE email = %s
        """, (31, "alice@example.com"))
        
        # Delete data
        cursor.execute("DELETE FROM users WHERE id = %s", (1,))
        
        connection.commit()
        
except Error as e:
    print(f"Error: {e}")
finally:
    if connection.is_connected():
        cursor.close()
        connection.close()
```

### SQLite
```python
import sqlite3

# Connect to SQLite
conn = sqlite3.connect('myapp.db')
cursor = conn.cursor()

# Create table
cursor.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        age INTEGER,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
""")

# Insert data
cursor.execute("""
    INSERT INTO users (name, email, age)
    VALUES (?, ?, ?)
""", ("Alice", "alice@example.com", 30))

# Query data
cursor.execute("SELECT * FROM users WHERE age > ?", (25,))
users = cursor.fetchall()
for user in users:
    print(f"Name: {user[1]}, Email: {user[2]}")

# Update data
cursor.execute("""
    UPDATE users SET age = ? WHERE email = ?
""", (31, "alice@example.com"))

# Delete data
cursor.execute("DELETE FROM users WHERE id = ?", (1,))

# Commit changes
conn.commit()

# Close connection
conn.close()
```

## 📊 SQL Fundamentals

### Data Types
```sql
-- Numeric types
INTEGER, BIGINT, SMALLINT
DECIMAL(10,2), NUMERIC(10,2)
REAL, DOUBLE PRECISION
FLOAT(24), FLOAT(53)

-- Character types
CHAR(10), VARCHAR(255), TEXT
NCHAR(10), NVARCHAR(255), NTEXT

-- Date and time types
DATE, TIME, TIMESTAMP, DATETIME
INTERVAL, YEAR, MONTH

-- Boolean type
BOOLEAN, BOOL

-- Binary types
BLOB, BINARY(255), VARBINARY(255)

-- JSON type (PostgreSQL, MySQL 5.7+)
JSON, JSONB
```

### Table Operations
```sql
-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INTEGER CHECK (age >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create table with foreign key
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    product_name VARCHAR(100) NOT NULL,
    quantity INTEGER NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Alter table
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users ALTER COLUMN age SET NOT NULL;

-- Drop table
DROP TABLE IF EXISTS users;
```

### Indexes
```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_age ON users(age);

-- Create unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Create composite index
CREATE INDEX idx_users_name_age ON users(name, age);

-- Create partial index
CREATE INDEX idx_users_active ON users(email) WHERE active = true;

-- Drop index
DROP INDEX idx_users_email;
```

## 🔍 SQL Queries

### Basic Queries
```sql
-- Select all columns
SELECT * FROM users;

-- Select specific columns
SELECT name, email FROM users;

-- Select with alias
SELECT name AS user_name, email AS user_email FROM users;

-- Select with expressions
SELECT name, age, age * 2 AS double_age FROM users;

-- Select with functions
SELECT UPPER(name) AS name_upper, LENGTH(email) AS email_length FROM users;
```

### Filtering and Sorting
```sql
-- Where clause
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE name LIKE 'A%';
SELECT * FROM users WHERE email IN ('alice@example.com', 'bob@example.com');
SELECT * FROM users WHERE age BETWEEN 20 AND 30;

-- Multiple conditions
SELECT * FROM users WHERE age > 25 AND name LIKE 'A%';
SELECT * FROM users WHERE age > 25 OR name LIKE 'B%';

-- Order by
SELECT * FROM users ORDER BY name ASC;
SELECT * FROM users ORDER BY age DESC, name ASC;

-- Limit and offset
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
SELECT * FROM users ORDER BY created_at DESC LIMIT 10 OFFSET 20;
```

### Joins
```sql
-- Inner join
SELECT u.name, o.product_name, o.quantity
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Left join
SELECT u.name, o.product_name, o.quantity
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- Right join
SELECT u.name, o.product_name, o.quantity
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- Full outer join
SELECT u.name, o.product_name, o.quantity
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- Cross join
SELECT u.name, p.name
FROM users u
CROSS JOIN products p;
```

### Aggregation
```sql
-- Count
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT email) FROM users;

-- Sum, average, min, max
SELECT SUM(age) FROM users;
SELECT AVG(age) FROM users;
SELECT MIN(age) FROM users;
SELECT MAX(age) FROM users;

-- Group by
SELECT age, COUNT(*) FROM users GROUP BY age;
SELECT age, COUNT(*) FROM users GROUP BY age HAVING COUNT(*) > 1;

-- Group by with multiple columns
SELECT age, name, COUNT(*) FROM users GROUP BY age, name;
```

### Subqueries
```sql
-- Subquery in WHERE clause
SELECT * FROM users WHERE age > (SELECT AVG(age) FROM users);

-- Subquery in FROM clause
SELECT * FROM (
    SELECT name, age, ROW_NUMBER() OVER (ORDER BY age DESC) as rn
    FROM users
) ranked_users WHERE rn <= 5;

-- Subquery in SELECT clause
SELECT name, (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM users;

-- EXISTS subquery
SELECT * FROM users WHERE EXISTS (
    SELECT 1 FROM orders WHERE user_id = users.id
);
```

## 🚀 Advanced Features

### Window Functions
```sql
-- Row number
SELECT name, age, ROW_NUMBER() OVER (ORDER BY age DESC) as rank
FROM users;

-- Rank and dense rank
SELECT name, age, RANK() OVER (ORDER BY age DESC) as rank,
       DENSE_RANK() OVER (ORDER BY age DESC) as dense_rank
FROM users;

-- Partition by
SELECT name, age, ROW_NUMBER() OVER (PARTITION BY age ORDER BY name) as rank
FROM users;

-- Lag and lead
SELECT name, age, LAG(age) OVER (ORDER BY age) as prev_age,
       LEAD(age) OVER (ORDER BY age) as next_age
FROM users;

-- Cumulative sum
SELECT name, age, SUM(age) OVER (ORDER BY age) as cumulative_sum
FROM users;
```

### Common Table Expressions (CTEs)
```sql
-- Simple CTE
WITH user_stats AS (
    SELECT age, COUNT(*) as user_count
    FROM users
    GROUP BY age
)
SELECT * FROM user_stats WHERE user_count > 1;

-- Recursive CTE
WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id, 1 as level
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    SELECT c.id, c.name, c.parent_id, ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree;
```

### Stored Procedures
```sql
-- PostgreSQL stored procedure
CREATE OR REPLACE FUNCTION get_user_orders(user_id INTEGER)
RETURNS TABLE(order_id INTEGER, product_name VARCHAR, quantity INTEGER, price DECIMAL)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT o.id, o.product_name, o.quantity, o.price
    FROM orders o
    WHERE o.user_id = $1;
END;
$$;

-- Call stored procedure
SELECT * FROM get_user_orders(1);

-- MySQL stored procedure
DELIMITER //
CREATE PROCEDURE GetUserOrders(IN user_id INT)
BEGIN
    SELECT o.id, o.product_name, o.quantity, o.price
    FROM orders o
    WHERE o.user_id = user_id;
END //
DELIMITER ;

-- Call stored procedure
CALL GetUserOrders(1);
```

### Triggers
```sql
-- PostgreSQL trigger
CREATE OR REPLACE FUNCTION update_modified_time()
RETURNS TRIGGER AS $$
BEGIN
    NEW.modified_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_modified_time
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_time();

-- MySQL trigger
DELIMITER //
CREATE TRIGGER update_users_modified_time
    BEFORE UPDATE ON users
    FOR EACH ROW
BEGIN
    SET NEW.modified_at = CURRENT_TIMESTAMP;
END //
DELIMITER ;
```

## 🔧 Best Practices

### Database Design
1. **Normalize Data** - Follow normalization rules to reduce redundancy
2. **Use Appropriate Data Types** - Choose right data types for your data
3. **Create Indexes** - Index frequently queried columns
4. **Use Constraints** - Enforce data integrity with constraints
5. **Plan for Growth** - Design for future data growth

### Query Optimization
1. **Use EXPLAIN** - Analyze query execution plans
2. **Avoid SELECT *** - Select only needed columns
3. **Use LIMIT** - Limit result sets when possible
4. **Optimize Joins** - Use appropriate join types
5. **Use Prepared Statements** - Prevent SQL injection and improve performance

### Performance
1. **Monitor Performance** - Track slow queries and optimize
2. **Use Connection Pooling** - Reuse database connections
3. **Implement Caching** - Cache frequently accessed data
4. **Regular Maintenance** - Update statistics and rebuild indexes
5. **Backup Strategy** - Implement regular backups

## 🚀 Use Cases

### E-commerce Platform
```sql
-- Product catalog
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    category_id INTEGER REFERENCES categories(id),
    stock_quantity INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User orders
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    total_amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Order items
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER NOT NULL,
    price DECIMAL(10,2) NOT NULL
);

-- Query: Get user's order history
SELECT o.id, o.total_amount, o.status, o.order_date,
       COUNT(oi.id) as item_count
FROM orders o
LEFT JOIN order_items oi ON o.id = oi.order_id
WHERE o.user_id = 1
GROUP BY o.id, o.total_amount, o.status, o.order_date
ORDER BY o.order_date DESC;
```

### Content Management System
```sql
-- Articles
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    author_id INTEGER REFERENCES users(id),
    category_id INTEGER REFERENCES categories(id),
    status VARCHAR(20) DEFAULT 'draft',
    published_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Comments
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    article_id INTEGER REFERENCES articles(id),
    user_id INTEGER REFERENCES users(id),
    content TEXT NOT NULL,
    parent_id INTEGER REFERENCES comments(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Query: Get article with comment count
SELECT a.id, a.title, a.content, u.name as author_name,
       COUNT(c.id) as comment_count
FROM articles a
JOIN users u ON a.author_id = u.id
LEFT JOIN comments c ON a.id = c.article_id
WHERE a.status = 'published'
GROUP BY a.id, a.title, a.content, u.name
ORDER BY a.published_at DESC;
```

### Analytics Platform
```sql
-- User events
CREATE TABLE user_events (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    event_type VARCHAR(50) NOT NULL,
    event_data JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Daily statistics
CREATE TABLE daily_stats (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL,
    total_users INTEGER DEFAULT 0,
    active_users INTEGER DEFAULT 0,
    total_events INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Query: Get daily active users
SELECT DATE(created_at) as date,
       COUNT(DISTINCT user_id) as active_users
FROM user_events
WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY DATE(created_at)
ORDER BY date DESC;
```

## 🔍 Monitoring and Debugging

### Performance Monitoring
```sql
-- PostgreSQL: Check slow queries
SELECT query, mean_time, calls, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- MySQL: Check slow queries
SELECT * FROM mysql.slow_log
ORDER BY start_time DESC
LIMIT 10;

-- Check table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### Database Health
```sql
-- Check database connections
SELECT count(*) as active_connections
FROM pg_stat_activity
WHERE state = 'active';

-- Check table statistics
SELECT schemaname, tablename, n_tup_ins, n_tup_upd, n_tup_del
FROM pg_stat_user_tables
ORDER BY n_tup_ins DESC;

-- Check index usage
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

## 🐛 Troubleshooting

### Common Issues
1. **Slow Queries** - Use EXPLAIN to analyze query plans
2. **Connection Issues** - Check connection pool settings
3. **Lock Contention** - Monitor for deadlocks and long-running transactions
4. **Disk Space** - Monitor database size and disk usage
5. **Memory Issues** - Check buffer pool and cache settings

### Debugging Tools
```sql
-- Check current queries
SELECT pid, state, query, query_start
FROM pg_stat_activity
WHERE state = 'active';

-- Check locks
SELECT locktype, database, relation, mode, granted
FROM pg_locks
WHERE NOT granted;

-- Check table bloat
SELECT schemaname, tablename, n_dead_tup, n_live_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000;
```

## 📚 Learning Resources

### Official Documentation
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [SQLite Documentation](https://www.sqlite.org/docs.html)

### Community Resources
- [SQL Tutorial](https://www.w3schools.com/sql/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [MySQL Tutorial](https://www.mysqltutorial.org/)

## 🔄 Updates and Roadmap

### Recent Updates
- **PostgreSQL 15** - Enhanced performance and new features
- **MySQL 8.0** - Improved security and performance
- **SQLite 3.40** - Better JSON support and performance

### Upcoming Features
- **PostgreSQL 16** - Enhanced partitioning and performance
- **MySQL 8.1** - Better cloud integration
- **SQLite 3.41** - Improved query optimization

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow SQL coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://opensource.org/licenses/MIT) file for details.

## 🙏 Acknowledgments

- **SQL Database Teams** for creating and maintaining these systems
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements
- **Contributors** for advancing SQL technology

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#sql-databases-)

</div>
