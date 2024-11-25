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

```
openssl dgst -sha256 -out file.hash encrypted_file.bin

```

```
openssl pkeyutl -verify -in file.hash -sigfile file.signature -pubin -inkey public_k.pem
```
Valid Signature Notice   
![](./images/Screenshot%202024-11-25%20104337.png)

# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:
Generate RSA private key and public key
```
openssl genpkey -algorithm RSA -out private_key.pem
openssl rsa -pubout -in private_key.pem -out public_key.pem
```

Generate a random 256-bit symmetric key
```
openssl rand -out secret_key.bin 32
```
encrypt secret_key.bin
```
openssl pkeyutl -encrypt -in secret_key.bin -pubin -inkey public_key.pem -out encrypted_key.bin
```

Create input_file.txt
```
echo "This is a test file." > input_file.txt
```
encrypt input_file.txt
```
openssl enc -aes-256-cbc -salt -in input_file.txt -out encrypted_file.bin -pass file:./secret_key.bin
```
Create Http for U2 get file
```
python3 -m http.server 8000
```
In U2 get file from U1
```
wget http://172.17.0.2:8000/encrypted_file.bin -O encrypted_file.bin
wget http://172.17.0.2:8000/encrypted_key.bin -O encrypted_key.bin
wget http://172.17.0.2:8000/public_key.pem -O public_key.pem
wget http://172.17.0.2:8000/private_key.pem -O private_key.pem
```
The message from U3 is that U2 has obtained the file
![](./images/Screenshot%202024-11-25%20141309.png)

Decrypt encrypted_key.bin
```
openssl pkeyutl -decrypt -inkey private_key.pem -in encrypted_key.bin -out secret_key.bin
```
Decrypt encrypted_file.bin
```
 openssl enc -d -aes-256-cbc -in encrypted_file.bin -out decrypted_file.txt -pass file:./secret_key.bin
```
Same as file in U1 (input_file.txt)
```
cat decrypted_file.txt 
```
![](./images/Screenshot%202024-11-25%20141607.png)

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
