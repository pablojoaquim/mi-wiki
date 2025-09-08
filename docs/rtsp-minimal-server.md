# Minimal RTSP Server in Python

This is the first step in building an RTSP/RTP streaming system. An RTSP server accepts connections, parses a client request, and sends back a valid RTSP response.

* The server creates a TCP socket and listens on port 8554.
* Accepts one client connection.
* Reads one RTSP request (in plain text).
* Sends back a minimal 200 OK RTSP response.
* Closes the connection.

This is just the skeleton of an RTSP server. It does not yet support multiple requests, state transitions (SETUP, PLAY, TEARDOWN), or RTP media streaming.

------------------------------------------------------------------------

## 1. Creating the Server Socket

```python
import socket

def rtsp_server():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind(("0.0.0.0", 8554))
    server_socket.listen(1)
    print("RTSP server listening on port 8554...")

```

* `socket.AF_INET`: Use IPv4
* `socket.SOCK_STREAM`: Use TCP (RTSP runs normally on TCP).
* `bind(("0.0.0.0", 8554))`: Listen on port 8554 (default RTSP port is 554, but 8554 avoids admin privileges).
* `listen(1)`: Start listening for incoming connections.

At this point, the server is passively waiting for an RTSP client.

------------------------------------------------------------------------

## 2. Accepting a Client
```python
    while True:
        client_socket, addr = server_socket.accept()
        print(f"Client connected: {addr}")
```
* `accept()`: Waits until a client connects.
* `client_socket`: the dedicated socket for communication with this client.
* `addr`: the client’s (IP, port).

Now the server is ready to communicate with one client.

------------------------------------------------------------------------

## 3. Receiving an RTSP Request
```python
        request = client_socket.recv(1024).decode()
        print("RTSP request received:\n", request)
```
* `recv(1024)`: Read up to 1024 bytes from the socket.
* `.decode()`: RTSP is text-based (like HTTP), so we convert bytes to a string.

Typical client request example:
```
OPTIONS rtsp://localhost:8554/stream RTSP/1.0
CSeq: 1
```

------------------------------------------------------------------------

## 4. Sending an RTSP Response
```python
        response = "RTSP/1.0 200 OK\r\nCSeq: 1\r\nSession: 12345678\r\n\r\n"
        client_socket.send(response.encode())
```
* "RTSP/1.0 200 OK": Protocol version and status code (200 = success).
* "CSeq: 1": Sequence number (should match client’s request, here hardcoded).
* "Session: 12345678": Arbitrary session identifier.

In a real RTSP server we would parse the client’s CSeq and return the same number, but for now we keep it simple.

------------------------------------------------------------------------

## 5. Closing the Connection
```python
        client_socket.close()
```
* After responding, the server closes the connection.
* This is why the client will get a connection reset error if it tries to send more requests — the server is gone.
