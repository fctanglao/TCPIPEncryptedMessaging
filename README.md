# TCP/IP Encrypted Messaging

## Project Motivation
I wrote this program for a final assignment in my Introduction to Cybersecurity class at Cal Poly Pomona. I wanted to apply what we learned about encryption and TCP/IP communication in a practical setting. Instead of studying RSA and socket communication separately, the goal was to integrate both into a working client–server system. By implementing encrypted messaging over raw TCP sockets, this project demonstrates how public-key cryptography secures network communication and highlights the difference between transport reliability (TCP) and confidentiality (encryption)

## How It Works
The application establishes a TCP connection on port 8080

### Key Loading
**Server Loads:**
- `private.pem` (server private key)
- `client_public.pem` (client public key)

**Client Loads:**
- `client_private.pem` (client private key)
- `public.pem` (server public key)

### Message Flow
- Encrypt with server public key -> Decrypt with server private key
- Decrypt with client private key <- Encrypt with client public key

## Generating Server Public and Private Keys
- ### openssl genrsa -out private.pem 2048
- ### openssl rsa -in private.pem -pubout -out public.pem

## Generating Client Public and Private Keys
- ### openssl genrsa -out client_private.pem 2048
- ### openssl rsa -in client_private.pem -pubout -out client_public.pem

## Sharing Server Public Key
- ### scp user@server_ip: /path/to/public.pem /path/to/client/code/directory/

## Sharing Client Public Key
- ### scp client_public.pem user@server_ip: /path/to/server/code/directory/

## Compiling and Running the Server
- ### gcc server.c -o server -lcrypto -lssl
- ### ./server

## Compiling and Running the Client
- ### gcc client.c -o client -lcrypto -lssl
- ### ./client
