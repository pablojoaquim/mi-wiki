# Minimal RTSP Client in Python

The client will connect to the server via TCP, send an RTSP request, and
listen for RTP packets via UDP.

* The client connects to the server using **RTSP over TCP** and sends
    an `OPTIONS` request.
* It then opens a **UDP socket** on port 5004 to receive RTP packets.
* Each RTP packet contains:
    *   a 12-byte header
    *   a JPEG-encoded frame as payload\
*   The client decodes JPEG frames using OpenCV and displays them in
    real time.

This minimal client is a **prototype**: it only handles one request
(`OPTIONS`) and assumes that the server immediately starts streaming to
port 5004. In a real RTSP client, we would implement the full sequence
(`DESCRIBE`, `SETUP`, `PLAY`) and negotiate the transport.

------------------------------------------------------------------------

## 1. RTSP Control Connection (TCP)

``` python
import socket
import cv2
import numpy as np

def rtsp_client():
    # RTSP TCP control
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(("127.0.0.1", 8554))
    sock.send(b"OPTIONS rtsp://localhost:8554/stream RTSP/1.0\r\nCSeq: 1\r\n\r\n")
    print(sock.recv(1024).decode())
```

* `socket.AF_INET, socket.SOCK_STREAM`: IPv4 + TCP
* `sock.connect(("127.0.0.1", 8554))`: Connect to the RTSP server on
    localhost, port 8554.
* `sock.send(...)`: Send an RTSP `OPTIONS` request (as raw bytes).
* `sock.recv(1024)`: Receive and print the server's RTSP response.

This is the **RTSP control plane**: a simple request/response exchange.

------------------------------------------------------------------------

## 2. Setting up the RTP Data Channel (UDP)

``` python
    # RTP UDP data
    udp_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    udp_sock.bind(("0.0.0.0", 5004))
```

-   `socket.SOCK_DGRAM`: Use UDP for RTP.
-   `bind(("0.0.0.0", 5004))`: The client listens locally on port 5004
    for incoming RTP packets.
-   This port must match what the server sends to. In our minimal
    server, we hardcoded `("127.0.0.1", 5004)`.

------------------------------------------------------------------------

## 3. Parsing RTP Packets

``` python
def parse_rtp_packet(packet):
    header = packet[:12]   # RTP header is 12 bytes (minimal)
    payload = packet[12:]  # The rest is video data (JPEG)
    return payload
```

* **RTP Header**: 12 bytes (we ignore details for now: sequence,
    timestamp, SSRC).\
* **Payload**: Encoded video frame (in this example, JPEG).

------------------------------------------------------------------------

## 4. Decoding and Displaying Frames

``` python
    while True:
        packet, _ = udp_sock.recvfrom(65536)
        payload = parse_rtp_packet(packet)
        frame = cv2.imdecode(np.frombuffer(payload, np.uint8), cv2.IMREAD_COLOR)
        if frame is not None:
            cv2.imshow("RTP Client", frame)
            if cv2.waitKey(1) == ord("q"):
                break
```

* `recvfrom(65536)`: Receive UDP packet. RTP packets can be large, so
    we allow up to 65 KB.\
* `parse_rtp_packet`: Strip the header and get the JPEG payload.\
* `cv2.imdecode(...)`: Decode JPEG bytes into an OpenCV frame.\
* `cv2.imshow`: Display the frame in a window.\
*  Press **q** to quit.

------------------------------------------------------------------------

## 🎯 Summary

-   The client connects to the server using **RTSP over TCP** and sends
    an `OPTIONS` request.\
-   It then opens a **UDP socket** on port 5004 to receive RTP packets.\
-   Each RTP packet contains:
    -   a 12-byte header\
    -   a JPEG-encoded frame as payload\
-   The client decodes JPEG frames using OpenCV and displays them in
    real time.

This minimal client is a **prototype**: it only handles one request
(`OPTIONS`) and assumes that the server immediately starts streaming to
port 5004. In a real RTSP client, we would implement the full sequence
(`DESCRIBE`, `SETUP`, `PLAY`) and negotiate the transport.

------------------------------------------------------------------------

👉 Next step: we'll refine both server and client to handle multiple
RTSP methods and synchronize the RTP stream startup properly.
