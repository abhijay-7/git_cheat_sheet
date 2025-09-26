# MySQL Complete Cheat Sheet

## Database Operations

### Create & Drop Database
```sql
CREATE DATABASE database_name;
DROP DATABASE database_name;
USE database_name;
SHOW DATABASES;
```

### Database Information
```sql
SELECT DATABASE();           -- Current database
SHOW TABLES;                -- List all tables
DESCRIBE table_name;         -- Table structure
SHOW COLUMNS FROM table_name;
```

## Table Operations

### Create Table
```sql
CREATE TABLE table_name (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE,
    age INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Alter Table
```sql
ALTER TABLE table_name ADD COLUMN column_name datatype;
ALTER TABLE table_name DROP COLUMN column_name;
ALTER TABLE table_name MODIFY COLUMN column_name new_datatype;
ALTER TABLE table_name CHANGE old_name new_name datatype;
ALTER TABLE table_name RENAME TO new_table_name;
```

### Drop & Truncate
```sql
DROP TABLE table_name;       -- Delete table completely
TRUNCATE TABLE table_name;   -- Delete all data, keep structure
```

## Data Types

### Numeric Types
```sql
TINYINT      -- 1 byte (-128 to 127)
SMALLINT     -- 2 bytes (-32,768 to 32,767)
MEDIUMINT    -- 3 bytes
INT          -- 4 bytes
BIGINT       -- 8 bytes
DECIMAL(M,D) -- Fixed-point number
FLOAT        -- Single precision floating point
DOUBLE       -- Double precision floating point
```

### String Types
```sql
CHAR(n)      -- Fixed length string
VARCHAR(n)   -- Variable length string
TEXT         -- Large text data
TINYTEXT     -- Up to 255 characters
MEDIUMTEXT   -- Up to 16,777,215 characters
LONGTEXT     -- Up to 4,294,967,295 characters
```

### Date & Time Types
```sql
DATE         -- YYYY-MM-DD
TIME         -- HH:MM:SS
DATETIME     -- YYYY-MM-DD HH:MM:SS
TIMESTAMP    -- YYYY-MM-DD HH:MM:SS (with timezone)
YEAR         -- YYYY
```

## CRUD Operations

### INSERT
```sql
INSERT INTO table_name (column1, column2) VALUES (value1, value2);
INSERT INTO table_name VALUES (value1, value2, value3);
INSERT INTO table_name (column1, column2) VALUES 
    (value1, value2),
    (value3, value4),
    (value5, value6);
```

### SELECT
```sql
SELECT * FROM table_name;
SELECT column1, column2 FROM table_name;
SELECT DISTINCT column_name FROM table_name;
SELECT column_name AS alias_name FROM table_name;
```

### UPDATE
```sql
UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;
UPDATE table_name SET column1 = value1;  -- Updates all rows
```

### DELETE
```sql
DELETE FROM table_name WHERE condition;
DELETE FROM table_name;  -- Deletes all rows
```

## WHERE Clause & Operators

### Comparison Operators
```sql
SELECT * FROM table_name WHERE column = value;
SELECT * FROM table_name WHERE column != value;
SELECT * FROM table_name WHERE column <> value;  -- Not equal
SELECT * FROM table_name WHERE column > value;
SELECT * FROM table_name WHERE column < value;
SELECT * FROM table_name WHERE column >= value;
SELECT * FROM table_name WHERE column <= value;
```

### Logical Operators
```sql
SELECT * FROM table_name WHERE condition1 AND condition2;
SELECT * FROM table_name WHERE condition1 OR condition2;
SELECT * FROM table_name WHERE NOT condition;
```

### Pattern Matching
```sql
SELECT * FROM table_name WHERE column LIKE 'pattern%';    -- Starts with
SELECT * FROM table_name WHERE column LIKE '%pattern';    -- Ends with
SELECT * FROM table_name WHERE column LIKE '%pattern%';   -- Contains
SELECT * FROM table_name WHERE column LIKE 'p_ttern';     -- Single character wildcard
```

### Range & List Operations
```sql
SELECT * FROM table_name WHERE column BETWEEN value1 AND value2;
SELECT * FROM table_name WHERE column IN (value1, value2, value3);
SELECT * FROM table_name WHERE column NOT IN (value1, value2);
SELECT * FROM table_name WHERE column IS NULL;
SELECT * FROM table_name WHERE column IS NOT NULL;
```

## Sorting & Limiting

### ORDER BY
```sql
SELECT * FROM table_name ORDER BY column ASC;   -- Ascending (default)
SELECT * FROM table_name ORDER BY column DESC;  -- Descending
SELECT * FROM table_name ORDER BY column1 ASC, column2 DESC;
```

### LIMIT
```sql
SELECT * FROM table_name LIMIT 10;              -- First 10 rows
SELECT * FROM table_name LIMIT 10 OFFSET 20;    -- Skip 20, then 10 rows
SELECT * FROM table_name LIMIT 20, 10;          -- MySQL specific syntax
```

## Aggregate Functions

### Basic Aggregates
```sql
SELECT COUNT(*) FROM table_name;
SELECT COUNT(column_name) FROM table_name;
SELECT SUM(column_name) FROM table_name;
SELECT AVG(column_name) FROM table_name;
SELECT MIN(column_name) FROM table_name;
SELECT MAX(column_name) FROM table_name;
```

### GROUP BY & HAVING
```sql
SELECT column, COUNT(*) FROM table_name GROUP BY column;
SELECT column, COUNT(*) FROM table_name GROUP BY column HAVING COUNT(*) > 1;
```

## Joins

### INNER JOIN
```sql
SELECT t1.column, t2.column 
FROM table1 t1 
INNER JOIN table2 t2 ON t1.id = t2.foreign_id;
```

### LEFT JOIN
```sql
SELECT t1.column, t2.column 
FROM table1 t1 
LEFT JOIN table2 t2 ON t1.id = t2.foreign_id;
```

### RIGHT JOIN
```sql
SELECT t1.column, t2.column 
FROM table1 t1 
RIGHT JOIN table2 t2 ON t1.id = t2.foreign_id;
```

### FULL OUTER JOIN (MySQL workaround)
```sql
SELECT t1.column, t2.column FROM table1 t1 LEFT JOIN table2 t2 ON t1.id = t2.foreign_id
UNION
SELECT t1.column, t2.column FROM table1 t1 RIGHT JOIN table2 t2 ON t1.id = t2.foreign_id;
```

### CROSS JOIN
```sql
SELECT t1.column, t2.column FROM table1 t1 CROSS JOIN table2 t2;
```

## Subqueries

### Basic Subqueries
```sql
SELECT * FROM table1 WHERE column IN (SELECT column FROM table2);
SELECT * FROM table1 WHERE column = (SELECT MAX(column) FROM table1);
```

### EXISTS
```sql
SELECT * FROM table1 t1 WHERE EXISTS (
    SELECT 1 FROM table2 t2 WHERE t1.id = t2.foreign_id
);
```

## String Functions

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
SELECT LENGTH(column_name) FROM table_name;
SELECT UPPER(column_name) FROM table_name;
SELECT LOWER(column_name) FROM table_name;
SELECT SUBSTRING(column_name, start, length) FROM table_name;
SELECT TRIM(column_name) FROM table_name;
SELECT REPLACE(column_name, 'old', 'new') FROM table_name;
```

## Date Functions

```sql
SELECT NOW();                           -- Current datetime
SELECT CURDATE();                       -- Current date
SELECT CURTIME();                       -- Current time
SELECT DATE(datetime_column) FROM table_name;
SELECT YEAR(date_column) FROM table_name;
SELECT MONTH(date_column) FROM table_name;
SELECT DAY(date_column) FROM table_name;
SELECT DATE_ADD(date_column, INTERVAL 1 DAY) FROM table_name;
SELECT DATE_SUB(date_column, INTERVAL 1 MONTH) FROM table_name;
SELECT DATEDIFF(date1, date2) FROM table_name;
```

## Indexes

### Create Index
```sql
CREATE INDEX index_name ON table_name (column_name);
CREATE UNIQUE INDEX index_name ON table_name (column_name);
CREATE INDEX index_name ON table_name (column1, column2);  -- Composite index
```

### Drop Index
```sql
DROP INDEX index_name ON table_name;
ALTER TABLE table_name DROP INDEX index_name;
```

### Show Indexes
```sql
SHOW INDEX FROM table_name;
```

## Constraints

### Primary Key
```sql
ALTER TABLE table_name ADD PRIMARY KEY (column_name);
ALTER TABLE table_name DROP PRIMARY KEY;
```

### Foreign Key
```sql
ALTER TABLE table_name 
ADD CONSTRAINT fk_name 
FOREIGN KEY (column_name) REFERENCES other_table(column_name);

ALTER TABLE table_name DROP FOREIGN KEY fk_name;
```

### Other Constraints
```sql
ALTER TABLE table_name ADD CONSTRAINT UNIQUE (column_name);
ALTER TABLE table_name ADD CONSTRAINT CHECK (condition);
ALTER TABLE table_name MODIFY column_name datatype NOT NULL;
```

## Views

```sql
CREATE VIEW view_name AS SELECT column1, column2 FROM table_name WHERE condition;
DROP VIEW view_name;
SELECT * FROM view_name;
```

## Stored Procedures

### Create Procedure
```sql
DELIMITER //
CREATE PROCEDURE procedure_name(IN param1 INT, OUT param2 VARCHAR(255))
BEGIN
    -- Procedure body
    SELECT column INTO param2 FROM table_name WHERE id = param1;
END //
DELIMITER ;
```

### Call Procedure
```sql
CALL procedure_name(value1, @output_var);
SELECT @output_var;
```

### Drop Procedure
```sql
DROP PROCEDURE procedure_name;
```

## User Management

### Create User
```sql
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
```

### Grant Privileges
```sql
GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';
GRANT SELECT, INSERT, UPDATE ON table_name TO 'username'@'localhost';
FLUSH PRIVILEGES;
```

### Revoke Privileges
```sql
REVOKE ALL PRIVILEGES ON database_name.* FROM 'username'@'localhost';
```

### Drop User
```sql
DROP USER 'username'@'localhost';
```

## Backup & Restore

### Backup (Command Line)
```bash
mysqldump -u username -p database_name > backup.sql
mysqldump -u username -p --all-databases > all_databases.sql
```

### Restore (Command Line)
```bash
mysql -u username -p database_name < backup.sql
```

## Common System Commands

```sql
SHOW PROCESSLIST;               -- Show running processes
SHOW STATUS;                    -- Server status
SHOW VARIABLES;                 -- Server variables
SHOW ENGINES;                   -- Available storage engines
SELECT VERSION();               -- MySQL version
```

---

# MySQL Keywords Reference

## Data Definition Language (DDL)

### Database Operations
- `CREATE` - Create database or table
- `DROP` - Delete database or table
- `ALTER` - Modify database or table structure
- `TRUNCATE` - Delete all data from table
- `RENAME` - Rename table or column

### Table Structure
- `TABLE` - Specify table operations
- `DATABASE` - Specify database operations
- `SCHEMA` - Synonym for database
- `COLUMN` - Specify column operations
- `INDEX` - Create or manage indexes
- `VIEW` - Create or manage views
- `TRIGGER` - Create or manage triggers
- `PROCEDURE` - Create or manage stored procedures
- `FUNCTION` - Create or manage functions

## Data Manipulation Language (DML)

### Basic Operations
- `SELECT` - Query data
- `INSERT` - Add new data
- `UPDATE` - Modify existing data
- `DELETE` - Remove data
- `REPLACE` - Insert or update data

### Query Modifiers
- `FROM` - Specify source table
- `WHERE` - Filter conditions
- `ORDER BY` - Sort results
- `GROUP BY` - Group rows
- `HAVING` - Filter grouped results
- `LIMIT` - Limit number of results
- `OFFSET` - Skip rows before returning results
- `DISTINCT` - Remove duplicates
- `ALL` - Include all rows (opposite of DISTINCT)

## Data Types

### Numeric Keywords
- `INT` / `INTEGER` - Integer data type
- `TINYINT` - Small integer
- `SMALLINT` - Small integer
- `MEDIUMINT` - Medium integer
- `BIGINT` - Large integer
- `DECIMAL` - Fixed-point number
- `NUMERIC` - Synonym for DECIMAL
- `FLOAT` - Floating-point number
- `DOUBLE` - Double precision floating-point
- `BIT` - Bit field
- `BOOLEAN` / `BOOL` - Boolean data type

### String Keywords
- `CHAR` - Fixed-length string
- `VARCHAR` - Variable-length string
- `TEXT` - Large text data
- `TINYTEXT` - Small text data
- `MEDIUMTEXT` - Medium text data
- `LONGTEXT` - Large text data
- `BINARY` - Fixed-length binary string
- `VARBINARY` - Variable-length binary string
- `BLOB` - Binary large object
- `TINYBLOB` - Small binary data
- `MEDIUMBLOB` - Medium binary data
- `LONGBLOB` - Large binary data

### Date/Time Keywords
- `DATE` - Date data type
- `TIME` - Time data type
- `DATETIME` - Date and time
- `TIMESTAMP` - Timestamp with timezone
- `YEAR` - Year data type

### JSON Keywords
- `JSON` - JSON data type

## Constraints

### Constraint Types
- `PRIMARY KEY` - Primary key constraint
- `FOREIGN KEY` - Foreign key constraint
- `UNIQUE` - Unique constraint
- `CHECK` - Check constraint
- `NOT NULL` - Not null constraint
- `DEFAULT` - Default value constraint
- `AUTO_INCREMENT` - Auto-incrementing values

### Constraint Actions
- `REFERENCES` - Reference another table
- `ON DELETE` - Action on delete
- `ON UPDATE` - Action on update
- `CASCADE` - Cascade action
- `SET NULL` - Set to null action
- `RESTRICT` - Restrict action
- `NO ACTION` - No action

## Operators

### Comparison Operators
- `=` - Equal to
- `!=` / `<>` - Not equal to
- `<` - Less than
- `<=` - Less than or equal to
- `>` - Greater than
- `>=` - Greater than or equal to
- `<=>` - NULL-safe equal to

### Logical Operators
- `AND` / `&&` - Logical AND
- `OR` / `||` - Logical OR
- `NOT` / `!` - Logical NOT
- `XOR` - Exclusive OR

### Pattern Matching
- `LIKE` - Pattern matching with wildcards
- `REGEXP` / `RLIKE` - Regular expression matching
- `SOUNDS LIKE` - Phonetic matching

### Set Operations
- `IN` - Value in list
- `NOT IN` - Value not in list
- `EXISTS` - Subquery exists
- `NOT EXISTS` - Subquery does not exist
- `BETWEEN` - Value between range
- `NOT BETWEEN` - Value not between range

### NULL Operations
- `IS NULL` - Check for null value
- `IS NOT NULL` - Check for non-null value
- `ISNULL()` - Function to check null
- `COALESCE()` - Return first non-null value

## Join Operations

### Join Types
- `INNER JOIN` - Inner join
- `LEFT JOIN` - Left outer join
- `RIGHT JOIN` - Right outer join
- `FULL OUTER JOIN` - Full outer join
- `CROSS JOIN` - Cross join
- `NATURAL JOIN` - Natural join
- `STRAIGHT_JOIN` - Force join order

### Join Conditions
- `ON` - Join condition
- `USING` - Join using columns

## Aggregate Functions

### Basic Aggregates
- `COUNT()` - Count rows
- `SUM()` - Sum values
- `AVG()` - Average value
- `MIN()` - Minimum value
- `MAX()` - Maximum value
- `GROUP_CONCAT()` - Concatenate group values

### Statistical Functions
- `STD()` / `STDDEV()` - Standard deviation
- `VARIANCE()` - Variance

## String Functions Keywords
- `CONCAT()` - Concatenate strings
- `LENGTH()` - String length
- `CHAR_LENGTH()` - Character length
- `UPPER()` / `UCASE()` - Convert to uppercase
- `LOWER()` / `LCASE()` - Convert to lowercase
- `SUBSTRING()` / `MID()` - Extract substring
- `LEFT()` - Extract from left
- `RIGHT()` - Extract from right
- `TRIM()` - Remove spaces
- `LTRIM()` - Remove left spaces
- `RTRIM()` - Remove right spaces
- `REPLACE()` - Replace substring
- `REVERSE()` - Reverse string

## Date/Time Functions Keywords
- `NOW()` - Current datetime
- `CURDATE()` - Current date
- `CURTIME()` - Current time
- `DATE()` - Extract date part
- `TIME()` - Extract time part
- `YEAR()` - Extract year
- `MONTH()` - Extract month
- `DAY()` - Extract day
- `HOUR()` - Extract hour
- `MINUTE()` - Extract minute
- `SECOND()` - Extract second
- `DATE_ADD()` - Add date interval
- `DATE_SUB()` - Subtract date interval
- `DATEDIFF()` - Date difference
- `DATE_FORMAT()` - Format date

## Control Flow

### Conditional Statements
- `IF()` - Conditional function
- `CASE` - Case statement
- `WHEN` - Case condition
- `THEN` - Case result
- `ELSE` - Case default
- `END` - End case statement
- `IFNULL()` - Handle null values
- `NULLIF()` - Return null if equal

### Loop Control (Procedures)
- `WHILE` - While loop
- `REPEAT` - Repeat loop
- `UNTIL` - Until condition
- `LOOP` - Basic loop
- `LEAVE` - Exit loop
- `ITERATE` - Continue loop

## Set Operations
- `UNION` - Combine results
- `UNION ALL` - Combine with duplicates
- `INTERSECT` - Common rows (MySQL 8.0+)
- `EXCEPT` - Difference (MySQL 8.0+)

## Window Functions (MySQL 8.0+)
- `OVER()` - Window function clause
- `PARTITION BY` - Partition window
- `ROW_NUMBER()` - Row number
- `RANK()` - Rank with gaps
- `DENSE_RANK()` - Rank without gaps
- `LAG()` - Previous row value
- `LEAD()` - Next row value
- `FIRST_VALUE()` - First value in window
- `LAST_VALUE()` - Last value in window

## Transaction Control
- `START TRANSACTION` - Begin transaction
- `BEGIN` - Begin transaction
- `COMMIT` - Commit transaction
- `ROLLBACK` - Rollback transaction
- `SAVEPOINT` - Create savepoint
- `RELEASE SAVEPOINT` - Release savepoint

## Data Control Language (DCL)
- `GRANT` - Grant privileges
- `REVOKE` - Revoke privileges
- `FLUSH` - Flush privileges

## Storage Engines
- `ENGINE` - Specify storage engine
- `INNODB` - InnoDB storage engine
- `MYISAM` - MyISAM storage engine
- `MEMORY` - Memory storage engine

## Other Important Keywords
- `USE` - Select database
- `SHOW` - Display information
- `DESCRIBE` / `DESC` - Show table structure
- `EXPLAIN` - Query execution plan
- `ANALYZE` - Analyze table
- `OPTIMIZE` - Optimize table
- `REPAIR` - Repair table
- `LOCK` - Lock tables
- `UNLOCK` - Unlock tables
- `LOAD DATA` - Load data from file
- `INTO` - Insert destination
- `VALUES` - Insert values
- `SET` - Set values
- `AS` - Alias
- `TEMPORARY` - Temporary table
- `IF EXISTS` - Conditional existence check
- `IF NOT EXISTS` - Conditional non-existence check