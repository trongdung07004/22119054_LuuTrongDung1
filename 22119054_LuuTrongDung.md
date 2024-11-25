# Lab #2,22119054, Luu Trong Dung, INSE331280E_01FIE
# Setup

Dockerfile
```
FROM ubuntu
WORKDIR /app
```

Bulid image
```
docker build ubuntu1 .
```



Start containers u1, u2, u3
![](./images/Screenshot%202024-11-25%20081718.png)
![](./images/Screenshot%202024-11-25%20091911.png)
Inside u1 use the command:
```
docker attach u1
```
```
docker apt-get update 
```

```
docker apt-get install openssl
```


Inside u2 use the command:
```
docker attach u2
```
```
docker apt-get update 
```

```
docker apt-get install openssl
```
# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:

Create private key
```
openssl genrsa -out private_k.pem 2048
```

Create public key from private key
```
openssl rsa -in private_k.pem -pubout -out public_k.pem
```

Create plaintext.text and hash it
```
echo "Hello" > plaintext.txt 
openssl dgst -sha256 -binary -out file.hash plaintext.txt
```

Create signature with private key and file.hash
```
 openssl pkeyutl -sign -inkey private_k.pem -in file.hash -out file.signature
```

Open HTTP for U2 to retrieve the files in U1
```
python3 -m http.server 8000
```

In U2 get encrypted_file.bin, file.hash, file.signature, aes_k.txt from u1
```
wget http://172.17.0.2:8000/encrypted_file.bin
wget http://172.17.0.2:8000/file.hash
wget http://172.17.0.2:8000/file.signature
wget http://172.17.0.2:8000/aes_k.txt

```

In U2
```
openssl dgst -sha256 -out file.hash encrypted_file.bin

```

```
openssl pkeyutl -verify -in file.hash -sigfile file.signature -pubin -inkey public_k.pem
```

![](./images/Screenshot%202024-11-25%20104337.png)

# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:
Generate RSA private key
```
openssl genpkey -algorithm RSA -out private.pem -aes256
```
Extract the public key from the private key
```
openssl rsa -pubout -in private.pem -out public.pem
```
Generate a random 256-bit symmetric key
```
openssl rand -out symmetric.key 32
```
Encrypt the file using AES-256 with the symmetric key
```
openssl enc -aes-256-cbc -in example.txt -out example.txt.enc -pass file:symmetric.key
```
Encrypt the symmetric key with Computer B's RSA public key
```
openssl rsautl -encrypt -inkey /path/to/computer_b/public.pem -pubin -in symmetric.key -out symmetric.key.enc
```

# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:
## ICMP
Initially, when ICMP is not blocked, U2 can ping to U3 (172.17.0.4)
```
ping 172.17.0.4
```
![](./images/Screenshot%202024-11-25%20092848.png)



In U3 block ICMP from source coming from U2 (172.17.0.3) 
```
iptables -A INPUT -p icmp --source 172.17.0.3 -j REJECT
```

Alter blocked u2 cant ping to U3

```
ping 172.17.0.4
```
![](./images/Screenshot%202024-11-25%20092731.png)



## HTTP
Listen on port 80 of U3 by nc
```
nc -l -p 80

```

Try connecting to U3 using HTTP
```
curl 172.17.0.4:80

```

U3 agrees to connect from U2 to HTTP
![](./images/Screenshot%202024-11-25%20095230.png)


U3 blocks port 80 with power coming from U2
```
iptables -A INPUT -p tcp -s 172.17.0.3 --dport 80 -j REJECT
```

Try connecting to U3 using HTTP, Unable to connect to U3 using HTTP because it has been blocked
```
curl 172.17.0.4:80
```
![](./images/Screenshot%202024-11-25%20095319.png)

After deletion, U3 received an HTTP connection from U2
```
iptables -D INPUT -p tcp -s 172.17.0.3 --dport 80 -j REJECT
```
![](./images/Screenshot%202024-11-25%20101249.png)

#SSH
Listen on port 22 of U3 by nc
```
nc -l -p 22

```

Try connecting to U3 using SSH
```
ssh root@172.17.0.4
```
U3 agrees to connect from U2 to SSH

![](./images/Screenshot%202024-11-25%20102810.png)


U3 blocks port 22 with power coming from U2
```
 iptables -A INPUT -p tcp -s 172.17.0.3 --dport 22 -j REJECT
```
