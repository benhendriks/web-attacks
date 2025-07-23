# 🔐 CeWL Notes

## 📌 Command

```bash
cewl --write exercise.txt --lowercase -m 5 http://enum-sandbox/
```

## 🧰 Tool Description

**CeWL** is a ruby-based custom wordlist generator that spiders a given URL to a specified depth, returning a list of words that can be used for password attacks.

---

## 🔍 Command Breakdown

| Option / Argument        | Description |
|--------------------------|-------------|
| `cewl`                   | Invokes the CeWL tool. |
| `--write exercise.txt`   | Writes the output wordlist to `exercise.txt`. |
| `--lowercase`            | Converts all words to lowercase. |
| `-m 5`                   | Minimum word length: 5 characters. |
| `http://enum-sandbox/`   | The target website to crawl. |

---

## 💡 Purpose

Generate a custom wordlist from the words found on the target website `http://enum-sandbox/`:

- Filters out short/irrelevant words (less than 5 characters)
- Normalizes all words to lowercase
- Saves to a reusable file

---

## 📝 Example Output (exercise.txt)

```text
sandbox
login
administrator
passwords
download
research
companyname
```

---

## ⚠️ Tips

- Add `--depth <n>` for deeper crawling (default is 2).
- Use `--useragent` if the site blocks default bot agents.
- Avoid crawling restricted areas (check `robots.txt`).

---

## 🧠 Usage Scenario

This command is ideal for:

- Ethical hacking / penetration testing
- Context-aware dictionary attacks
- Custom wordlist generation
