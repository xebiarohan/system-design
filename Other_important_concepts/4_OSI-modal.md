## OSI modal (Open Systems Interconnection mode)

1. A conceptual framework used to understand how computers communicate over networks

2. Data travels from layer 7 to layer 1 when sending data and from layer 1 to layer 7 when receiving data

3. 7 layers

```bash

7 -> Application layer

6 -> Presentation layer

5 -> Session layer

4 -> Transport layer

3 -> Network layer

2 -> Data link layer

1 -> Physical

```

4. Layer 7 : Application layer

    - It provide network service directly to applications

    - Applications like Browser, email application, Messaging application

    - Example HTTP/HTTPS, FTP, SMTP, DNS, etc

    - Responsibilities: User interaction, requesting web pages, web service

5. Layer 6: Presentation layer

    - This layer ensures the data is presented in a readable format

    - Acts like a translator

    - Responsibilities: Encryption/decryption, compression, data formatting, etc

    - Example: SSL/TLS, MP3, UTF-8, etc

6. Layer 5: Session layer

    - Manages conversation between devices

    - Starting session, maintaining session and ending session

    - Responsibility: Authentication session, session recovery

    - Example when we logged into a website, this layer maintains the login session

5. Layer 4: Transport layer

    - Ensures reliable delivery of data

    - it breaks large data into smaller pieces called segments

    - Responsibilities: Reliable delivert, error checking, flow control

    - Main protocols : TCP and UDP

    - Transport layer uses ports like

        - HTTP: 80

        - HTTPS: 443

        - SMTP: 25

6. Layer 3: Network layer

    - Handles routing between networks

    - Determines the best path for data

    - Responsibilities: Routing, packet forwarding

    - Main protocol : IP (internet protocol)

    - Devices: Routers

    - Network layer uses IP addresses to find the destination devices

    - Example home router sends traffic to internet

7. Layer 2: Data link layer

    - Handles communication within the same local network

    - Responsibilities : MAC addressing, error detection

    - Devices: Switches, NICs (Network interface cards)

8. Layer 1: Physical layer

    - It transmits raw bits (0s and 1s)

    - Responsibilities: Electrical signals, radio signals, connectors

    - Example : Ethernet cables, fiber optics

9. Sender Side

```bash

Layer 7

Browser creates HTTP request.

↓

Layer 6

Data gets encrypted via TLS.

↓

Layer 5

Session established.

↓

Layer 4

TCP splits data into segments.

↓

Layer 3

IP adds source/destination IP addresses.

↓

Layer 2

Ethernet frame created with MAC addresses.

↓

Layer 1

Bits transmitted through cable/Wi-Fi.

```