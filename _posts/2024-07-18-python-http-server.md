---
layout: post
title: "Python HTTP Server"
subtitle: "Building a HTTP server from scratch using Python 3.12.4"
description: "Building a HTTP server from scratch using Python 3.12.4"
author: "jxkxl"
header-style: text
tags:
  - Python
  - Project
---

I thought it would be a great idea for my first project to dive into the process of building an HTTP server from scratch using Python 3.12. I chose python as it is the language I am most familiar with, but am planning on doing this again in other languages, both to learn new languages, but to also see how they differ from one another. I'm excited to share this learning journey with you! If you want to see the project, you can visit the GitHub repository [here](https://github.com/jxkx1/py-http-server).

## Binding To A Port
To begin, I will set up a basic python program to create a basic tcp server, and bind it to a port. As the TCP/IP protocol is the underlying system in all HTTP servers, I though it would be a good place to start.
```python
import socket

def main():
    server_socket = socket.create_server(("localhost", 4221), reuse_port=True)
    server_socket.accept() # wait for client

if __name__ == "__main__":
    main()

```

## Sending A Response
Right now, out tcp server is pretty useless, if we send a HTTP request to out TCP server using curl we get the following output:
```bash
❯ curl -v http://localhost:4221
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 55372 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET / HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
* Recv failure: Connection reset by peer
* Closing connection
curl: (56) Recv failure: Connection reset by peer
```

We can see that the python server is online, but nothing is being served. To fix this, we are going to alter the code a little to enable the server to send the HTTP version, status code and a Carriage Return Line Feed (CRLF) to announce the end of the status lines along with another to announce the end of the headers. Lets update the code:
```python
server_socket.accept() # wait for client
```
_becomes_
```python
server_socket.accept()[0].sendall(b"HTTP/1.1 200 OK\r\n\r\n")
```
So lets now run `curl -v http://localhost:4221` and see what we receive now!

```bash
❯ curl -v http://localhost:4221
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 35454 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET / HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< 
* no chunk, no close, no size. Assume close to signal end
* Recv failure: Connection reset by peer
* Closing connection
curl: (56) Recv failure: Connection reset by peer
```
We can see that HTTP request starting to form nicely. Perfect!

## Extracting URL Paths
Here is where things started to change quickly. Before, our code simply sent a 200 OK response without any consideration for the actual request path. To combat this, I added a _**handle_requests**_ function, that reads and extracts the URL path and now only responds with a 200 status code for the '/' path, and a 404 status code for any other path. The original code also only handled a single connection before terminating. To fix this, we modified the main function to enter a loop that continuously accepts incoming connections and passes them to the handle_request function. After these patches, the following code is as follows:
```python
import socket

def handle_request(conn):
    request = conn.recv(1024).decode()

    lines = request.split('\r\n')
    request_line = lines[0]
    method, path, _ = request_line.split()

    if path == '/':
        response = "HTTP/1.1 200 OK\r\n\r\n"
    else:
        response = "HTTP/1.1 404 Not Found\r\n\r\n"

    conn.sendall(response.encode())

def main():
    server_socket = socket.create_server(("localhost", 4221), reuse_port=True)

    while True:
        conn, _ = server_socket.accept()
        handle_request(conn)
        conn.close()

if __name__ == "__main__":
    main()
```
Now if we run the following command: `curl -v http://localhost:4221/abcdefg` we get the following output:
```bash
❯ curl -v http://localhost:4221/abcdefg
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 41400 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET /abcdefg HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 404 Not Found
< 
* no chunk, no close, no size. Assume close to signal end
* Closing connection
```
Running the previous command `curl -v http://localhost:4221/` returns the following:
```bash
❯ curl -v http://localhost:4221/
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 34004 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET / HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< 
* no chunk, no close, no size. Assume close to signal end
* Closing connection
```
We can see that the two responses are different and now only returns a response code 200 if the "/" path is received.

## Response Bodies
The next major step in setting up this HTTP server is having a response body, otherwise, we wouldn't really see anything, and it would be pretty pointless. Response bodies handle content such as an entire web page, a file, a string, or anything else that can be represented with bytes. For this to work, I will be setting up an endpoint that responds with a string based off the request sent to the server. To do this, I added the following line of code the _**handle_request**_ function in my program:
```python
if path == '/':
        response = "HTTP/1.1 200 OK\r\n\r\n"
elif path.startswith('/echo/'):
        echo_str = path.split('/')[2]
        response = f"HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\nContent-Length: {len(echo_str)}\r\n\r\n{echo_str}"
else:
        response = "HTTP/1.1 404 Not Found\r\n\r\n"
    
```
The code now has an elif block to handle between different endpoints. Additionally if the path starts with /echo/, I extract the string after the /echo/ part and store it in the echo_str variable. I then construct the response by setting the status line to HTTP/1.1 200 OK, adding the required Content-Type and Content-Length headers, and setting the response body to the echo_str value, returning the provided string in the response body with the appropriate headers. We can test this with `curl -v http://localhost:4221/echo/we_have_some_text_working` and monitor the response as follows:
```bash
❯ curl -v http://localhost:4221/echo/we_have_some_text_working
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 37540 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET /echo/we_have_some_text_working HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Content-Length: 25
< 
* Connection #0 to host localhost left intact
we_have_some_text_working
```
We can see the response body is identical to what we requested. As basic as this is, this becomes the foundation of all HTTP requests. Cool! We are making some progress!

## User Agents & Reading Headers
HTTP headers are super important when we start dealing with real traffic, that is why I am going to implement an endpoint that can request the User Agent of a connection. This can be important when detecting legitamate traffic on a site vs. bot spamming and scrapers. Implementing this into our HTTP server is not a difficult task. As by now, you can probably guess, we are going to add a new check into the elif block inside of our _**handle_request**_ function. We can see the new code is as follows:
```python
elif path == '/user-agent':
        user_agent = next((line for line in lines if line.startswith('User-Agent:')), '').split(':', 1)[1].strip()
        response = f"HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\nContent-Length: {len(user_agent)}\r\n\r\n{user_agent}"
```
I use a list comprehension to find the line that starts with User-Agent:, split it on the colon to extract the value, and strip any leading/trailing whitespace. I then build the response by setting the status line to HTTP/1.1 200 OK as standard, adding the required Content-Type and Content-Length headers, and setting the response body to the extracted user_agent value. This will now return the value of the User-Agent header in the response body with the appropriate headers if the /user_agent endpoint is called on. Now to the testing, we will be using `curl -v  http://localhost:4221/user-agent` first to test if the endpoint is working correctly. Lets see:
```bash
❯ curl -v  http://localhost:4221/user-agent
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 59306 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET /user-agent HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Content-Length: 10
< 
* Connection #0 to host localhost left intact
curl/8.8.0
```
Perfect! We can read the User Agent directly with our endpoint. What if we use a custom User Agent using curl?
```bash
❯ curl -v http://localhost:4221/user-agent -H "User-Agent: special_agent/0.0.7"
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 43392 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET /user-agent HTTP/1.1
> Host: localhost:4221
> Accept: */*
> User-Agent: special_agent/0.0.7
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Content-Type: text/plain
< Content-Length: 19
< 
* Connection #0 to host localhost left intact
special_agent/0.0.7
```
Looks good, and even James Bond can access this HTTP server whenever he needs, although it wouldn't be very secure...

## Concurrent connections?
Up until now, the HTTP server could only support one connection at a time. This could be useful for something such as a secure file transfer service, but... we can do better. First, back up. What are concurrent connections? ["Concurrent connection" means the maximum number of TCP connections your server can handle at any one time. At any given time many TCP/IP requests are coming to your server.](https://webmasters.stackexchange.com/questions/121857/what-does-concurrent-connection-mean) As we can see, this seems pretty important if we want a smooth HTTP server to show off. Thankfully, we only need to change a _little_ piece of our _**main**_ function to facilitate this.
```python
    server_socket = socket.create_server(("localhost", 4221), reuse_port=True)

    while True:
        conn, _ = server_socket.accept()
        handle_request(conn)
        conn.close()
```
Changes to
```python
    server_socket = socket.create_server(("localhost", 4221), reuse_port=True)

    while True:
        conn, _ = server_socket.accept()
        threading.Thread(target=handle_request, args=(conn,)).start()
```
See? Pretty easy. the only extra think i had to add was the `import threading` module into the program. Now we can have as many active connections to our server as we want. (or can handle)

## Sending Files
Onto something a bit more exciting, I wanted to the server to be able to serve files to anyone. This, at the beginning, seemed pretty easy- and after getting it working, it was, but it came with many of its own problems. Firstly, I realized that my code up until now was not being prepared for this, which required me to rewrite a lot of core mechanics of the server. This led to a lot of small problems popping up along the way, including the server to completely stop serving the response body. I also started to see a decline in the responsiveness of the server when using the `curl` command. I will touch on both of these in this section. Before I do, Here is the updated code for program so far:
```python
import socket
import threading
import sys

def handle_request(client, addr):
    data = client.recv(1024).decode()
    req = data.split("\r\n")
    path = req[0].split(" ")[1]

    if path == "/":
        response = "HTTP/1.1 200 OK\r\n\r\n".encode()
    elif path.startswith("/echo"):
        response = f"HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\nContent-Length: {len(path[6:])}\r\n\r\n{path[6:]}".encode()
    elif path.startswith("/user-agent"):
        user_agent = req[2].split(": ")[1]
        response = f"HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\nContent-Length: {len(user_agent)}\r\n\r\n{user_agent}".encode()
    elif path.startswith("/files"):
        directory = sys.argv[2]
        filename = path[7:]
        print(directory, filename)
        try:
            with open(f"/{directory}/{filename}", "r") as f:
                body = f.read()
            response = f"HTTP/1.1 200 OK\r\nContent-Type: application/octet-stream\r\nContent-Length: {len(body)}\r\n\r\n{body}".encode()
        except Exception as e:
            response = f"HTTP/1.1 404 Not Found\r\n\r\n".encode()
    else:
        response = "HTTP/1.1 404 Not Found\r\n\r\n".encode()

    client.send(response)

def main():
    server_socket = socket.create_server(("localhost", 4221), reuse_port=True)

    while True:
        client, addr = server_socket.accept()
        threading.Thread(target=handle_request, args=(client, addr)).start()

if __name__ == "__main__":
    main()
```
As you see, The code has went though a metamorphosis, with the _**main**_ function's threading arguments changing from `conn` to `client, addr`. This change was made to facilitate the file transfer capabilities of the server. By including the client's address information (`addr`) in addition to the client socket object (`client`), the server can now differentiate between multiple concurrent client connections. This extra context is essential for effectively managing the file transfer use case, such as tracking the state of each client's file transfer session. On top of this, the change from:
```python
request = conn.recv(1024).decode()

lines = request.split('\r\n')
request_line = lines[0]
method, path, _ = request_line.split()
```
to:
```python
data = client.recv(1024).decode()
req = data.split("\r\n")
path = req[0].split(" ")[1]
```
was made to better align with the new `client, addr` approach. In the previous implementation, the code was relying on the conn object to read the incoming request and parse the request line. However, now that the server is working with the client object instead of the generic conn, the code needed to be updated to reflect this change. By using `client.recv(1024).decode()` and then splitting the resulting data string, the server can directly extract the necessary information, such as the requested path, without the intermediate steps of splitting the entire request string. Lets look into the new addition to the `elif` block:
```python
elif path.startswith("/files"):
        directory = sys.argv[2]
        filename = path[7:]
        print(directory, filename)
        try:
            with open(f"/{directory}/{filename}", "r") as f:
                body = f.read()
            response = f"HTTP/1.1 200 OK\r\nContent-Type: application/octet-stream\r\nContent-Length: {len(body)}\r\n\r\n{body}".encode()
        except Exception as e:
            response = f"HTTP/1.1 404 Not Found\r\n\r\n".encode()
```
Now whenever a client makes a request to the server, the server checks if the "`/files/`" endpoint. When the server identifies that it's a file request, it extracts two important pieces of information from the requested path: `directory` and `filename`, the server then attempts to open the requested file and if the file is successfully opened, reads the entire contents of the file and stores it in `response` variable, that includes the status line, relevant headers, and the file contents. Lets see it in action! First, lets put a file inside the "`files`" folder that is acting as our storage for our endpoint.
```bash
echo -n 'I am the chosen one!' > choose_me.txt
```
Now that the file is sitting there all cozy, lets see if we can request it!
```bash
❯ curl -v http://localhost:4221/files/choose_me.txt
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 37768 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET /files/choose_me.txt HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Content-Type: application/octet-stream
< Content-Length: 20
< 
* Connection #0 to host localhost left intact
I am the chosen one!
```
And just like that, we have a working file transfer HTTP server.

## Request Bodies
So we already got Response bodies working, cool. Now its time to get his older brother working. To do this, we are going to add support for the `POST` method to the "`/files`" endpoint, allowing a user to write files with the text from the request body. Once this is done, we will successfully have a two way file file transfer system set up, kind of like passing notes at the back of class, the only difference is, you will be able to read your notes, even after you send them. So lets get started. Firstly, we need tweak one of our variables a little.
```python
path = req[0].split(" ")[1]
```
becomes:
```python
method, path, _ = req[0].split(" ")
```
I did this to make a new variable, method, to keep the code clear on what it is doing. We could have left this alone but for future development, I felt it was best to do this. Now, onto the main change to the code. After setting up the previous endpoint to handle file transfering, i felt it was best to modify the existing code to allow for the opposite. Before I explain why I exactly did, here is the update endpoint code:
```python
elif path.startswith("/files"):
        directory = sys.argv[2]
        filename = path[7:]

        if method == "GET":
            try:
                with open(f"{directory}/{filename}", "r") as f:
                    body = f.read()
                response = f"HTTP/1.1 200 OK\r\nContent-Type: application/octet-stream\r\nContent-Length: {len(body)}\r\n\r\n{body}".encode()
            except FileNotFoundError:
                response = f"HTTP/1.1 404 Not Found\r\n\r\n".encode()
        elif method == "POST":
            content_length = 0
            for header in req[1:]:
                if header.startswith("Content-Length"):
                    content_length = int(header.split(": ")[1])

            if content_length > 0:
                body = req[-1]  # Assuming the request body is the last line of the request
                with open(f"{directory}/{filename}", "w") as f:
                    f.write(body)
                response = "HTTP/1.1 201 Created\r\n\r\n".encode()
            else:
                response = "HTTP/1.1 400 Bad Request\r\n\r\n".encode()
        else:
            response = "HTTP/1.1 405 Method Not Allowed\r\n\r\n".encode()
```
In the updated code, we have added support for the POST method by checking the value of the method variable. If the method is "`GET`", we continue with the original function of the endpoint and send the contents of an existing file, but If the method is "`POST`", we extract the content length from the request headers and retrieve the request body. We then create a new file with the specified filename specified in the request and write the request body into it. If the operation is successful, we send a "201 Created" response back to the user. So lets see it in action.
```bash
❯ curl -v --data "12345" -H "Content-Type: application/octet-stream" http://localhost:4221/files/test
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 57470 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> POST /files/test HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> Content-Type: application/octet-stream
> Content-Length: 5
> 
* upload completely sent off: 5 bytes
< HTTP/1.1 201 Created
< 
* no chunk, no close, no size. Assume close to signal end
```
We can see that the request responded with a "201 Created" response, but lets verify that it really exists.
```bash
❯ ls ./
choose_me.txt  test
❯ cat test
12345
```
Nice! Now, lets use our "`/files/`" endpoint with the "`GET`" method to see if we can obtain the contents of the file we just made with the same endpoint!
```bash
❯ curl -v http://localhost:4221/files/test
* Host localhost:4221 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
*   Trying [::1]:4221...
* connect to ::1 port 4221 from ::1 port 47956 failed: Connection refused
*   Trying 127.0.0.1:4221...
* Connected to localhost (127.0.0.1) port 4221
> GET /files/test HTTP/1.1
> Host: localhost:4221
> User-Agent: curl/8.8.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Content-Type: application/octet-stream
< Content-Length: 5
< 
* Connection #0 to host localhost left intact
12345       
```
Looks good to me. Now we have both a send/receive enabled file sharing server. The code is looking a lot beefier than it did back when it was just a simple socket connection.
## Finale 
Overall, in 56 lines of python code, we went from a basic socket connection into a usable HTTP transfer. We could continue this and implement compression headers, such as Gzip, but since this post is quite lengthy, I might make a post in the future covering this. Right now the server is responsive through the terminal, and is very basic, but this was just a demonstration on how it would be written if we had to do it ourselves. Thankfully, Python 3.12 has us covered and instead all we have to do is say:
```python
python3 -m http.server <port>
```
## Thank you for reading :)
 