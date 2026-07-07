---
title: "Long Polling"
date: 2026-03-07
draft: false
tags:
  - system-design
  - tech
---


Explanations of Long Polling start with ther server *"holds the connection open."*. That means nothing, not to me at least.

To understand Long Polling, it helps to go one level lower than HTTP.

Suppose a client wants to fetch messages.

```text
GET /messages
```

The HTTP request is converted into bytes, passed to TCP, encapsulated into IP packets and sent over the network.

Before any HTTP request is sent, TCP establishes a connection.

```text
Client                     Server

SYN ---------------------->

      <------------------- SYN-ACK

ACK ---------------------->
```

Once this handshake completes, the client and server have an established TCP connection. This connection is simply a bidirectional byte stream between two machines.

The client is not only sending the server's address. Every packet also contains the client's IP address and port.

For example,

```text
Client
10.1.1.5:444

↓

Server
54.23.11.7:123
```

The server therefore knows exactly where the client is. It doesn't have to "remember" who called it like a function call stack. The TCP connection itself maintains the state.

After the connection is established, the client sends the HTTP request.

```text
GET /messages
```

The server receives it and checks whether there are any new messages.

If messages exist, the server immediately writes the HTTP response onto the same TCP connection.

If no messages exist - The server does absolutely nothing.

It does not send "No Content."

It does not create another connection.

It simply waits.

The TCP connection already exists, so there is no reason to establish another one. The server simply delays writing the HTTP response.

Meanwhile, the client is blocked waiting for bytes to arrive on the socket.

Conceptually, it is doing something similar to

```java
socket.read();
```

No CPU is continuously checking.

No packets are continuously flowing.

The operating system simply waits.

Later, when a new message arrives, the server finally writes

```text
HTTP 200 OK

{
    "message": "Hello"
}
```

Those bytes travel back over the exact same TCP connection.

There is no second SYN and no new connection.

The server is not calling the client back.

It is only sending bytes over an already established connection.

This is  *"holding the connection open"* .

---

What happens if the client disappears while the server is waiting.

Suppose the client crashes.

The server does not immediately know.

The server still believes the TCP connection exists.

Only when it eventually tries to send data does TCP discover the problem.

TCP retransmits the packets multiple times.

If acknowledgements never arrive, TCP eventually times out and reports that the connection has been lost.

Only then does the application know the client is gone.

Now suppose the client restarts.

The old TCP connection is already dead.

The client establishes an entirely new connection.

Earlier the client may have been connected as

```text
10.1.1.5:444

↓

54.23.11.7:123
```

After restarting it may become

```text
10.1.1.5:555

↓

54.23.11.7:123
```

This is not an update to the old connection.

It is a completely new TCP connection.

The server creates a new socket for it while the old socket eventually times out and disappears.

This also explains why applications never identify users using TCP connections.

TCP connections come and go.

Ports change.

IP addresses may even change.

Applications instead use cookies, session IDs or JWTs so that multiple TCP connections can still represent the same logged-in user.

---

Acknowledgements.

TCP acknowledgements only confirm that bytes reached the operating system on the other side.

They do not confirm that the application processed them.

The client operating system may acknowledge the packet while the browser crashes before reading it.

From TCP's point of view, delivery succeeded.

From the application's point of view, the message was never processed.

Applications therefore build their own acknowledgement protocols.

Messaging applications such as WhatsApp are a good example.

The transport layer confirms that bytes reached the device.

The application later confirms that the message has been received and processed.

Another application-level acknowledgement confirms that the user has actually opened the chat.

These are completely different layers of acknowledgement.

---

Scalability.

How can one machine keep hundreds of thousands of TCP connections open?

It is tempting to think there are only 65,535 possible connections because TCP ports are 16-bit.

That limit applies to port numbers, not file descriptors.

The server usually listens on a single port.

For example,

```text
Server Port 443
```

Every incoming TCP connection creates a separate socket.

Conceptually,

```text
Listening Socket
FD = 3

Client A
FD = 4

Client B
FD = 5

Client C
FD = 6
```

The server port remains 443.

Each client connection gets its own socket and its own file descriptor.

The challenge is not creating the sockets.

The challenge is efficiently monitoring them.

Creating one operating system thread per socket clearly does not scale.

Most connections are idle almost all the time.

The solution is event multiplexing.

Linux provides `epoll` for this purpose.

The application registers every socket with epoll and then simply waits.

```text
epoll_wait()
```

The operating system now monitors every registered socket.

Suppose there are five hundred thousand connections.

If only one client sends data, the kernel processes the incoming packet, places the bytes into that socket's receive buffer and marks only that socket as readable.

`epoll` wakes the application and returns something conceptually similar to

```text
FD 58291
Readable
```

The application reads only that socket, processes the request, possibly writes a response and then goes back to waiting.

It never scans all five hundred thousand sockets.

The operating system performs the multiplexing.

Only sockets with actual work are returned to the application.

This is why servers like Nginx, Redis, Netty and Node.js can efficiently support enormous numbers of mostly idle connections.

---

WebSockets

Long Polling keeps an HTTP request waiting before eventually completing it.

WebSockets establish a TCP connection, perform an HTTP Upgrade handshake and then simply continue using the same bidirectional TCP connection indefinitely.

Both client and server may write bytes whenever they wish.

The underlying transport is still TCP.

Only the protocol on top changes.

---

The operating system maintains sockets, buffers, sequence numbers, acknowledgement numbers and connection state.

Applications simply read and write byte streams.

Long Polling, WebSockets and epoll all become different pieces of the same picture.
