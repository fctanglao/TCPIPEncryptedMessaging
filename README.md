# TCP/IP Encrypted Messaging

## Project Motivation
I wrote this program for a final assignment in my Introduction to Cybersecurity class at Cal Poly Pomona. I wanted to apply what we learned about encryption and TCP/IP communication in a practical setting. Instead of studying RSA and socket communication separately, the goal was to integrate both into a working client–server system. By implementing encrypted messaging over raw TCP sockets, this project demonstrates how public-key cryptography secures network communication and highlights the difference between transport reliability (TCP) and confidentiality (encryption)

## Overview
- The application establishes a TCP connection on port `8080` and encrypts messages at the application layer using RSA with OAEP padding
- Each side generates its own RSA key pair and shares only its **public key** with the other party before communication begins
- Public keys are shared so that a sender can encrypt messages using the recipient’s public key, ensuring that only the recipient (who possesses the corresponding private key) can decrypt and read the message
### Message Flow
- Client -> Encrypt with server public key -> Send over TCP -> Server -> Decrypt with server private key  
- Server -> Encrypt with client public key -> Send over TCP -> Client -> Decrypt with client private key  
> Note: Typing **disconnect** closes the session

## Key Generation
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
## Public Key Sharing
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

## Build
### Compiling Server
- `gcc server.c -o server -lcrypto -lssl`
### Compiling Client
- `gcc client.c -o client -lcrypto -lssl`
