
## HTTP REQUEST SMUGGLING

```
 HTTP request smuggling is a technique for interfering with the way a web site processes sequences of HTTP requests that are received from one or more users. Request smuggling vulnerabilities are often critical in nature, allowing an attacker to bypass security controls, gain unauthorized access to sensitive data, and directly compromise other application users.

Request smuggling is primarily associated with HTTP/1 requests. However, websites that support HTTP/2 may be vulnerable, depending on their back-end architecture. 
```


### How do HTTP request smuggling vulnerabilities arise?

Most HTTP request smuggling vulnerabilities arise because the HTTP/1 specification provides two different ways to specify where a request ends: the **Content-Length** header and the **Transfer-Encoding** header.

The Content-Length header is straightforward: it specifies the length of the message body in bytes. For example:
```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

The Transfer-Encoding header can be used to specify that the message body uses chunked encoding. This means that the message body contains one or more chunks of data. Each chunk consists of the chunk size in bytes (expressed in hexadecimal), followed by a newline, followed by the chunk contents. The message is terminated with a chunk of size zero. For example:
```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

b
q=smuggling
0
```
**NB:**

```
In HTTP/1, there are two different ways to tell how long a message is. Sometimes both methods can be used at the same time, and they might give conflicting information. To avoid confusion, the specification says that if both Content-Length and Transfer-Encoding are present, the Content-Length should be ignored.

This rule usually works fine when only one server is involved. But when multiple servers are chained together, things can go wrong for two main reasons:

Some servers don’t support the Transfer-Encoding header in requests.

Even when they do support it, the header can sometimes be written in a tricky or unusual way so that one server ignores it.

If the front-end and back-end servers treat the Transfer-Encoding header differently, they can end up disagreeing on where one request ends and the next begins. This mismatch can open the door to request smuggling attacks.
```
### Performing an HTTP request smuggling attack
HTTP request smuggling involves combining `Content-Length header and the Transfer-Encoding` header into a single HTTP/1 request and manipulating these so that the front-end and back-end servers process the request differently.

**Types of Request smuggling:**
- CL.TE : This is when the frontend uses the *content-length* and the backend uses the *Transfer-encoding* header
```
POST / HTTP/1.1
Host: website.com
Content-Length: 13
Transfer-Encoding: chunked
 
0
MALICIOUS-REQUEST
```

*The front-end server looks at the Content-Length header and decides the body is 13 bytes long, so it reads up to the end of the data labeled "MALICIOUS-REQUEST" and forwards that whole thing to the back-end. The back-end, however, sees Transfer-Encoding: chunked and parses the body using chunked rules. It reads the first chunk, which is marked as zero length, and that marks the end of the request for the back-end. Anything that comes after that zero-length chunk, in this case the bytes spelling "MALICIOUS-REQUEST", is left over. The back-end treats those leftover bytes as the start of the next request.*




