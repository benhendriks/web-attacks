# 🌐 What is SSRF? (Server-Side Request Forgery)

## 🧠 Definition

**Server-Side Request Forgery (SSRF)** is a web security vulnerability that allows an attacker to make the server perform HTTP requests to **internal or external resources** on their behalf.

In SSRF, the attacker abuses functionality that fetches data from a URL, such as:

- Image URL previews
- PDF renderers
- URL validators
- Webhooks or callbacks

---

## ⚙️ How SSRF Works

A vulnerable application accepts a user-supplied URL and makes a server-side HTTP request, like:

```http
GET /fetch?url=http://example.com
``` 

An attacker can change the url parameter to:

```http
/fetch?url=http://localhost:8000/admin
```

#### 🔍 Why SSRF is Dangerous

SSRF lets attackers:

    - Access internal services (e.g., http://localhost, http://127.0.0.1, http://backend)

    - Enumerate cloud metadata services (e.g., AWS: http://169.254.169.254)

    - Bypass firewalls and reach networks inaccessible from the outside

    - Read internal configuration files or sensitive endpoints

    - Potentially trigger Remote Code Execution (RCE) or leak credentials

#### 🧪 SSRF Example

Vulnerable Request:
```http
GET /fetch?url=http://example.com
```

Exploited by Attacker:
```http
GET /fetch?url=http://localhost:8000/admin
```

Server Makes Internal Request:
```http
[Server] --> http://localhost:8000/admin
```

🛡️ How to Prevent SSRF

    - Whitelist allowed URLs or domains

    - Validate user input strictly (e.g., disallow IPs, internal hostnames)

    - Block requests to:

        localhost, 127.0.0.1, ::1

        Reserved IP ranges (10.0.0.0/8, 192.168.0.0/16, etc.)

    - Use network segmentation: separate internal APIs from public-facing web apps

    - Monitor server traffic for unusual outbound HTTP requests



---

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

---

#### When Double Encoding is Required:

You only need to double-encode non-alphanumeric and reserved characters, such as:

| Character | 1st Encode | 2nd Encode |
| --------- | ---------- | ---------- |
| `/`       | `%2F`      | `%252F`    |
| `:`       | `%3A`      | `%253A`    |
| `\n`      | `%0A`      | `%250A`    |
| space     | `%20`      | `%2520`    |
| `%`       | `%25`      | `%2525`    |




