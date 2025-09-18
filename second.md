# SQL_Cheatsheet 2

> A handy reference for **SQL commands** and a **Python script** to move `.csv` files into MySQL Workbench.  

---

## 🐍 Python_To_MySQL  

This Python script loads multiple `.csv` files into MySQL by:  
- Connecting to the `ecommerce` database.  
- Reading CSV files with **Pandas**.  
- Creating tables automatically based on CSV columns.  
- Inserting all rows into MySQL tables.  

```python
# Python script to import CSV files into MySQL
import pandas as pd
import mysql.connector
import os

# List of CSV files and their corresponding table names
csv_files = [
    ('customers.csv', 'customers'),
    ('orders.csv', 'orders'),
    ('sellers.csv', 'sellers'),
    ('products.csv', 'products'),
    ('geolocation.csv', 'geolocation'),
    ('payments.csv', 'payments'),
    ('order_items.csv', 'order_items')
]

# Connect to the MySQL database
conn = mysql.connector.connect(
    host='localhost',
    user='root',
    password='Abhi@123',
    database='ecommerce'
)
cursor = conn.cursor()

# Folder containing the CSV files
folder_path = 'D:/Dataset/ecommerce'

# Map pandas datatypes to SQL datatypes
def get_sql_type(dtype):
    if pd.api.types.is_integer_dtype(dtype):
        return 'INT'
    elif pd.api.types.is_float_dtype(dtype):
        return 'FLOAT'
    elif pd.api.types.is_bool_dtype(dtype):
        return 'BOOLEAN'
    elif pd.api.types.is_datetime64_any_dtype(dtype):
        return 'DATETIME'
    else:
        return 'TEXT'

# Process each CSV file
for csv_file, table_name in csv_files:
    file_path = os.path.join(folder_path, csv_file)
    df = pd.read_csv(file_path)

    # Replace NaN with None to handle SQL NULL
    df = df.where(pd.notnull(df), None)

    # Clean column names (no spaces, dashes, or dots)
    df.columns = [col.replace(' ', '_').replace('-', '_').replace('.', '_') for col in df.columns]

    # Create table with correct schema
    columns = ', '.join([f'`{col}` {get_sql_type(df[col].dtype)}' for col in df.columns])
    create_table_query = f'CREATE TABLE IF NOT EXISTS `{table_name}` ({columns})'
    cursor.execute(create_table_query)

    # Insert each row into the table
    for _, row in df.iterrows():
        values = tuple(None if pd.isna(x) else x for x in row)
        sql = f"INSERT INTO `{table_name}` ({', '.join(['`' + col + '`' for col in df.columns])}) VALUES ({', '.join(['%s'] * len(row))})"
        cursor.execute(sql, values)

    conn.commit()  # Save after each CSV

# Close connection
conn.close()
```



---

## 👤 User Management  

| Command | Description |
|---------|-------------|
| `SELECT User, Host FROM mysql.user;` | Show all users |
| `CREATE USER 'someuser'@'localhost' IDENTIFIED BY 'password';` | Create a new user |
| `GRANT ALL PRIVILEGES ON *.* TO 'someuser'@'localhost';` | Grant full access |
| `FLUSH PRIVILEGES;` | Apply permission changes |
| `SHOW GRANTS FOR 'someuser'@'localhost';` | Show user privileges |
| `REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'someuser'@'localhost';` | Remove all permissions |
| `DROP USER 'someuser'@'localhost';` | Delete a user |

---

## 🗄️ Database Management  

| Command | Description |
|---------|-------------|
| `SHOW DATABASES;` | Show all databases |
| `CREATE DATABASE acme;` | Create new database |
| `DROP DATABASE acme;` | Delete database |
| `USE acme;` | Select database |

---

## 📑 Table Management  

| Command | Description |
|---------|-------------|
| `SHOW TABLES;` | List all tables |
| `DESCRIBE users;` | Show table structure |
| `CREATE TABLE users (...);` | Create new table |
| `DROP TABLE users;` | Delete table |
| `ALTER TABLE users ADD age INT;` | Add column |
| `ALTER TABLE users MODIFY age VARCHAR(3);` | Modify column |
| `TRUNCATE TABLE users;` | Delete all rows but keep structure |

---

## 📊 CRUD Operations  

| Command | Description |
|---------|-------------|
| `INSERT INTO users (...) VALUES (...);` | Insert new record |
| `INSERT INTO users (...) VALUES (...), (...), (...);` | Insert multiple records |
| `SELECT * FROM users;` | Select all records |
| `SELECT first_name, last_name FROM users;` | Select specific columns |
| `UPDATE users SET email='new@mail.com' WHERE id=2;` | Update records |
| `DELETE FROM users WHERE id=6;` | Delete a record |

---

## 🔍 Filtering & Sorting  

| Command | Description |
|---------|-------------|
| `SELECT * FROM users WHERE dept='sales';` | Filter results |
| `SELECT * FROM users ORDER BY last_name ASC;` | Sort ascending |
| `SELECT * FROM users LIMIT 5;` | Limit rows |
| `SELECT DISTINCT location FROM users;` | Unique values |
| `SELECT * FROM users WHERE age BETWEEN 20 AND 25;` | Range filter |
| `SELECT * FROM users WHERE dept LIKE 'dev%';` | Pattern matching |

---

## 🔗 Joins & Relationships  

| Command | Description |
|---------|-------------|
| `INNER JOIN` | Return matching rows |
| `LEFT JOIN` | Return all from left + matches |
| `RIGHT JOIN` | Return all from right + matches |
| `CROSS JOIN` | Return all combinations |
| `FOREIGN KEY (...) REFERENCES ...` | Define relationship |

Example:  
```sql
SELECT u.first_name, u.last_name, p.title
FROM users u
INNER JOIN posts p ON u.id = p.user_id;
```

---

## 📈 Aggregate Functions  

| Command | Description |
|---------|-------------|
| `SELECT COUNT(id) FROM users;` | Count rows |
| `SELECT MAX(age) FROM users;` | Maximum value |
| `SELECT MIN(age) FROM users;` | Minimum value |
| `SELECT SUM(age) FROM users;` | Sum values |
| `SELECT AVG(age) FROM users;` | Average value |
| `SELECT dept, COUNT(*) FROM users GROUP BY dept;` | Grouping |
