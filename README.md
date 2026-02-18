# TCP/IP Encrypted Messaging

## Project Motivation
I wrote this program for a final assignment in my Introduction to Cybersecurity class at Cal Poly Pomona. I wanted to apply what we learned about encryption and TCP/IP communication in a practical setting. Instead of studying RSA and socket communication separately, the goal was to integrate both into a working client–server system. By implementing encrypted messaging over raw TCP sockets, this project demonstrates how public-key cryptography secures network communication and highlights the difference between transport reliability (TCP) and confidentiality (encryption)

## Overview
- The application establishes a TCP connection on port `8080` and encrypts messages at the application layer using RSA with OAEP padding
- Each side generates its own RSA key pair and shares only its **public key** with the other party before communication begins
- Public keys are shared so that a sender can encrypt messages using the recipient’s public key, ensuring that only the recipient (who possesses the corresponding **private key**) can decrypt and read the message
### Message Flow
- Client -> Encrypt with server public key -> Send over TCP -> Server -> Decrypt with server private key  
- Server -> Encrypt with client public key -> Send over TCP -> Client -> Decrypt with client private key  
> Note: Typing **disconnect** closes the session

## Generate Key Pairs
- Each side generates its own 2048-bit RSA key pair
### Generating Server Key Pair
```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```
> Note: `private.pem` (server private key) & `public.pem` (server public key)
### Generating Client Key Pair
```bash
openssl genrsa -out client_private.pem 2048
openssl rsa -in client_private.pem -pubout -out client_public.pem
```
> Note: `client_private.pem` (client private key) & `client_public.pem` (client public key)
## Share Public Keys
- Before the encrypted chat can work, the public keys must be shared
- The public keys were distributed separately from the messaging application via SCP prior to communication
- > Note: Private keys are never shared. Only public keys are exchanged
### Sharing Server Public Key
```bash
scp user@server-ip:/path/to/public.pem /path/to/client/code/directory/
```
### Sharing Client Public Key
```bash
scp client_public.pem username@server_ip:/path/to/destination/directory/
```

## Requirements
- Linux or macOS
- GCC
- OpenSSL Development Libraries
### Ubuntu/Debian
```bash
sudo apt update
sudo apt install libssl-dev
```

## Build the Executables
### Building the Server
```bash
gcc server.c -o server -lcrypto -lssl
```
### Building the Client
```bash
gcc client.c -o client -lcrypto -lssl
```

## Run the Program
### Start the Server
```bash
./server
```
### Configure Client IP
- In `client.c`, update the current IP address with your server's LAN IP
```bash
inet_pton(AF_INET, "192.168.119.128", &serv_addr.sin_addr)
```
### Run the Client
```bash
./client
```

## Packet Analysis Using Wireshark
- Wireshark was used to verify that transmitted messages are encrypted
### Filter Used
```ini
tcp.port == 8080
```
### Wireshark Capture & Observations
![Wireshark Capture](https://github.com/fctanglao/TCPIPEncryptedMessaging/blob/main/Wireshark%20Capture.png)
- Packet 1: A SYN (synchronize) packet sent by the client (192.168.119.129) to initiate the connection
- Packet 2: The server (192.168.119.128) responds with a SYN-ACK (synchronize-acknowledge) packet to accept the connection
- Packet 3: The client completes the handshake by sending an ACK (acknowledge)
- Packet 8:
  - Source: 192.168.119.129 (client)
  - Destination: 192.168.119.128 (server)
  - Length: 256 bytes of payload sent from the client
  - Info: A PSH, ACK flag is present, indicating that the client is pushing data to the server, and the data must be immediately processed.
  > Note: The hexadecimal content in the **bottom pane** shows the encrypted data being transmitted.
- Packets 9–22: These are ACK packets confirming the receipt of data between the client and server. This is part of TCP's reliable delivery mechanism
- Packet 21: The client sends a FIN (finish) packet to signal the end of the connection
- Packet 22: The server responds with an ACK, acknowledging the client's termination request
- Packet 23: The server sends its own FIN to terminate its side of the connection
- Packet 24: The client responds with an ACK, completing the connection termination process
 
