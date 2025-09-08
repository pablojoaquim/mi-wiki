# RTSP, RTP, and RTCP -- An Introduction

When building a video streaming system, three protocols often come
together: **RTSP**, **RTP**, and **RTCP**.
Each plays a different role in the control and delivery of multimedia
streams.

------------------------------------------------------------------------

## 1. RTSP -- Real Time Streaming Protocol

-   **Purpose**: Think of RTSP as the *remote control* for a media
    session.
-   **Transport**: Runs over TCP (default port 554).
-   **What it does**:
    -   Establishes and controls media sessions.
    -   Defines methods like:
        -   `OPTIONS`: ask the server what methods are supported
        -   `DESCRIBE`: request media description (usually via SDP)
        -   `SETUP`: negotiate transport parameters (UDP ports,
            multicast, etc.)
        -   `PLAY`: start streaming
        -   `PAUSE`: pause playback
        -   `TEARDOWN`: end the session

RTSP itself does **not carry media data**. It only controls how media
should flow.

------------------------------------------------------------------------

## 2. RTP -- Real-time Transport Protocol

-   **Purpose**: Delivers the actual media (audio, video) over the
    network.
-   **Transport**: Usually over UDP (but can be tunneled over TCP).
-   **Packet structure**:
    -   12-byte header (minimum), including:
        -   Sequence number → detect packet loss or reordering
        -   Timestamp → synchronize playback
        -   SSRC → identify the stream
    -   Payload → media data (e.g., encoded video frame, audio samples)

RTP is designed for **low-latency delivery**, not guaranteed
reliability. If a packet is lost, it's usually skipped rather than
retransmitted.

------------------------------------------------------------------------

## 3. RTCP -- RTP Control Protocol

-   **Purpose**: Works alongside RTP to provide feedback and
    synchronization.
-   **Transport**: Sent periodically, usually on the UDP port right
    after RTP.
-   **Functions**:
    -   Reports on packet loss, jitter, delay.
    -   Conveys sender/receiver statistics.
    -   Synchronizes multiple streams (e.g., audio + video).

RTCP ensures that the streaming session can adapt and remain
synchronized, even under network issues.

------------------------------------------------------------------------

## 4. Putting It All Together

A typical RTSP/RTP session looks like this:

``` mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: OPTIONS (RTSP over TCP)
    Server-->>Client: 200 OK

    Client->>Server: DESCRIBE (RTSP over TCP)
    Server-->>Client: SDP (media description)

    Client->>Server: SETUP (RTSP over TCP)
    Server-->>Client: 200 OK (transport negotiated)

    Client->>Server: PLAY (RTSP over TCP)
    Server-->>Client: 200 OK

    Note over Server,Client: Media delivery phase

    Server-->>Client: RTP Packets (UDP)
    Server-->>Client: RTCP Reports (UDP)
    Client-->>Server: RTCP Reports (UDP)

    Client->>Server: TEARDOWN (RTSP over TCP)
    Server-->>Client: 200 OK
```


