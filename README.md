# TCP/IP Encrypted Messaging

## Project Motivation
I wrote this program for a final assignment in my Introduction to Cybersecurity class at Cal Poly Pomona. I wanted to apply what we learned about encryption and TCP/IP communication in a practical setting. Instead of studying RSA and socket communication separately, the goal was to integrate both into a working client–server system. By implementing encrypted messaging over raw TCP sockets, this project demonstrates how public-key cryptography secures network communication and highlights the difference between transport reliability (TCP) and confidentiality (encryption)

## How It Works
The application establishes a TCP connection on port `8080` and encrypts messages at the application layer
### Key Pairs
**Server:**
- `private.pem` (server private key)
- `public.pem` (server public key)
**Client:**
- `client_private.pem` (client private key)
- `client_public.pem` (client public key)
### Public Key Sharing
Before the encrypted chat can work, the public keys must be shared
The **server** gives the client: `public.pem` (server public key used by the client to encrypt messages to the server)
The **client** gives the server: `client_public.pem` (client public key used by the server to encrypt replies to the client)
> Note: This project assumes keys are shared out-of-band. No certificate authority (CA) or TLS validation is used
### Message Flow
- Encrypt with server public key -> Decrypt with server private key
- Decrypt with client private key <- Encrypt with client public key
- Typing `disconnect` closes the session

## Generate RSA Keys
### Server (Run On Server Machine)
- openssl genrsa -out private.pem 2048
- openssl rsa -in private.pem -pubout -out public.pem
### Client (Run On Client Machine)
- openssl genrsa -out client_private.pem 2048
- openssl rsa -in client_private.pem -pubout -out client_public.pem

## Share the Public Keys
### Server -> Client
- scp user@server_ip: /path/to/public.pem /path/to/client/code/directory/

### Client -> Server
- scp client_public.pem username@server_ip: /path/to/server/code/directory/



## Compiling and Running the Server
- ### gcc server.c -o server -lcrypto -lssl
- ### ./server

## Compiling and Running the Client
- ### gcc client.c -o client -lcrypto -lssl
- ### ./client
