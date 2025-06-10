

```shell
cd /var/www/html
```

```shell
sudo echo "Here is your 200 OK response" > target
```

```shell
sudo chmod 644 target
```

```shell
sudo systemctl start apache2
```

```shell
sudo systemctl restart apache2
```

```shell
sudo systemctl status apache2
```

```shell
call ip/target
```

# `sudo chmod 644` — Examples, Explanations & Use Cases

## What does `chmod 644` mean?

`chmod 644` sets the following file permissions:

- **Owner** → Read + Write (`rw-`)
- **Group** → Read only (`r--`)
- **Others (Everyone else)** → Read only (`r--`)

In numeric form:

| Entity | Permission | Binary | Octal |
|--------|------------|--------|-------|
| Owner  | rw-        | 110    | 6     |
| Group  | r--        | 100    | 4     |
| Others | r--        | 100    | 4     |

## When to use `chmod 644`?

- On **public files** that should be readable by everyone, but only editable by the owner.
- On **HTML files** and **static website content** (Apache/Nginx).
- On **configuration files** readable by system processes but only editable by admins.
- On **scripts** that are not intended to be executed directly.

---

## Common `chmod 644` Commands

### 1️⃣ Single File
```bash
sudo chmod 644 filename.txt



