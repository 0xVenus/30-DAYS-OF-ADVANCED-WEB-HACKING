
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
- **CL.TE** : This is when the frontend uses the *content-length* and the backend uses the *Transfer-encoding* header
```
POST / HTTP/1.1
Host: website.com
Content-Length: 13
Transfer-Encoding: chunked
 
0
MALICIOUS-REQUEST
```

*The front-end server looks at the Content-Length header and decides the body is 13 bytes long, so it reads up to the end of the data labeled "MALICIOUS-REQUEST" and forwards that whole thing to the back-end. The back-end, however, sees Transfer-Encoding: chunked and parses the body using chunked rules. It reads the first chunk, which is marked as zero length, and that marks the end of the request for the back-end. Anything that comes after that zero-length chunk, in this case the bytes spelling "MALICIOUS-REQUEST", is left over. The back-end treats those leftover bytes as the start of the next request.*

**Performing CL.TE HTTP request smuggling attack**

- Firstly you have to downgrade the http version if the version running is greater than 1.x from your Burpsuite Inspector so you can be able to perform the attack

As you can see here we have an http v2
![image](https://github.com/user-attachments/assets/2e6a3046-2d0b-44e4-b9d8-ba8b49769840)

Downgrading it
![image](https://github.com/user-attachments/assets/8c9712a9-ec31-409c-a5e9-646646c7e6c7)

- Then change the request method to `POST` since you'll be sending data in the request
![img](https://github.com/user-attachments/assets/516b3cd6-e5ed-43f3-936d-14f95c6cae86) 

As you can see i removed every other headers from the request

TIP: Toggle off the "update content length" and the "non printable character" from the Repeater request settings

![i](https://github.com/user-attachments/assets/9d2c3bd0-5e88-4ed6-bb7d-05b88cff2fae)

- Now proceed to detecting if the server is CL.TE or NOT
  you can use the below image to determine that (sending random data)

![i](https://github.com/user-attachments/assets/31d625b1-328e-4aae-86cd-1b87eb9154d3)

Now we can confirm the vulnerability

![i](https://github.com/user-attachments/assets/e91e4fc8-0735-4e05-b4f2-28173cc2f096)

*For CLARITY*

![w](https://github.com/user-attachments/assets/10bfc441-6841-4e02-b4dc-ad4e451a31ea)

`Exploiting`

Now you can poison the server by sending a `terminating chunk (0)` which the server stops the execution of that request and treats anything after it as a separate request
![i](https://github.com/user-attachments/assets/63b85ce2-0a67-4a03-b33e-1637eab3a996)

NB: send the request Twice
And you've successfully exploited the CL.TE http request smuggling.

**Performing TE.CL http request smuggling attack**

 Here, the front-end server uses the Transfer-Encoding header and the back-end server uses the Content-Length header. We can perform a simple HTTP request smuggling attack as follows:
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

 *The front-end server processes the Transfer-Encoding header, and so treats the message body as using chunked encoding. It processes the first chunk, which is stated to be 8 bytes long, up to the start of the line following SMUGGLED. It processes the second chunk, which is stated to be zero length, and so is treated as terminating the request. This request is forwarded on to the back-end server.*
*The back-end server processes the Content-Length header and determines that the request body is 3 bytes long, up to the start of the line following 8. The following bytes, starting with SMUGGLED, are left unprocessed, and the back-end server will treat these as being the start of the next request in the sequence.*

**NB:** To send this request using Burp Repeater, you will first need to go to the Repeater menu and ensure that the "Update Content-Length" option is unchecked like i showed you earlier.

You need to include the trailing sequence `\r\n\r\n` following the final `0`. 














