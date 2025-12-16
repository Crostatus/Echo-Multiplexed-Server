# Echo Multiplexed Server

This repository provides a complete example of a **multiplexed TCP echo server** implemented in Java using the **Java NIO (New I/O) API**, together with a simple client used for testing and interaction.  
The project demonstrates how to build a scalable, non-blocking server using a **single-threaded event loop** based on `Selector`.

---

## Technical Description

The server is implemented using **non-blocking I/O primitives** provided by `java.nio` and follows an **event-driven architecture**.  
Rather than dedicating a thread per connection, the server relies on a single `Selector` to monitor multiple channels and react to I/O readiness events.

The core design goals are:

- Efficient handling of multiple simultaneous TCP clients
- Non-blocking accept, read, and write operations
- Clear separation between connection management and I/O logic
- Minimal abstraction overhead, focusing on raw NIO usage

---

## Architecture Overview

### Server Side

The server uses the following components:

- **ServerSocketChannel**
  - Configured in **non-blocking mode**
  - Listens for incoming TCP connections

- **Selector**
  - Central event multiplexer
  - Monitors registered channels for:
    - `OP_ACCEPT`
    - `OP_READ`
    - `OP_WRITE`

- **SocketChannel**
  - Represents individual client connections
  - Registered dynamically with the selector

- **ByteBuffer**
  - Used explicitly for all read/write operations
  - Manually flipped, cleared, and reused

#### Event Loop

The server runs an infinite loop structured as follows:

1. Call `selector.select()` to block until at least one channel is ready.
2. Iterate over the selected `SelectionKey` set.
3. Handle each key depending on its readiness state:
   - **Acceptable**: accept a new connection and register it for reading.
   - **Readable**: read data from the client into a buffer.
   - **Writable**: write response data back to the client.
4. Remove processed keys from the selected set.

This pattern ensures scalability without spawning additional threads.

---

## Client Side

The client is implemented using `SocketChannel` and blocking I/O for simplicity.  
Its responsibilities are:

- Establish a TCP connection to the server
- Read user input from standard input
- Send raw byte data to the server
- Receive and display the echoed response

The client is intended primarily as a test harness for the server and does not attempt to implement advanced error handling or reconnection logic.

---

## Data Flow

1. Client sends a message over the TCP connection.
2. Server receives the message through a `SocketChannel` in non-blocking mode.
3. The message is stored in a `ByteBuffer`.
4. The server processes the buffer and prepares an echo response.
5. The response is written back to the client using the same channel.
6. The client reads and prints the response.

---

## Source Code Structure

- **EchoServer.java**  
  Initializes the selector and server socket channel. Contains the main event loop and handles accept, read, and write events.

- **EchoClient.java**  
  Connects to the server, sends user input, and reads server responses.

- **EchoByteBuffer.java**  
  Utility/helper class for managing `ByteBuffer` operations and simplifying buffer manipulation.

- **MainClass.java**  
  Application entry point used to start the server or client.

---

## Compilation and Execution

Example using the command line:

```bash
javac *.java
java MainClass
```

---

## Key Concepts Demonstrated

- Java NIO selectors and channel multiplexing
- Non-blocking TCP server design
- Event-driven I/O handling
- Manual buffer management with `ByteBuffer`
- Single-threaded scalability model

---

## Intended Audience

This project is intended for:

- Students studying operating systems or computer networks
- Developers learning Java NIO
- Anyone interested in low-level, scalable server design without frameworks
