Id: 22110028

Name: Nguyễn Mai Huy Hoàng

# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:

## 1. Set up docker
**1. Build docker** 

![image](https://github.com/user-attachments/assets/707598c4-61b6-46d2-8885-ad2e2fc0d01c)

**2. List of containers in the docker**

![image](https://github.com/user-attachments/assets/bbfab494-c7d0-4555-bf73-b09365fcfa34)


- docker Alice (alice-10.9.0.5) will be the sender
- docker Bob (bob-10.9.0.6) will be the receiver


## 2. On The sender container

**1. Access Alice container**

```
 docker exec -it alice-10.9.0.5 /bin/bash
```

![image](https://github.com/user-attachments/assets/ebda8963-59c8-4899-afb7-7a8848310890)


**2. Create a plain text in home directory**

```
echo "A plain text to transfer from Alice to Bob" > plain.txt
```

![image](https://github.com/user-attachments/assets/c7fce184-70f9-44b6-a2a0-82eae7385a3e)


**3. Randomly Create a secret key**
```
openssl rand -hex 32
```
![image](https://github.com/user-attachments/assets/ccaea41d-58f2-4aef-a322-a03ae20ddb85)

- `rand`: This subcommand generates random data.
- `-hex`: Specifies that the output should be in hexadecimal format.
- `32`: The number of bytes of random data to generate.

=> secret key: `121315f3d5dc5a016f764f1270ee10b8650a651a92a59d7ea5257a0f45942e4a`

**4.Encrypt the text file**

Using AES-256-ECB symmetric encryption to encrype the text file with the above secret key

```
openssl enc -aes-256-ecb -in plain.txt -out plain.enc -pass pass:121315f3d5dc5a016f764f1270ee10b8650a651a92a59d7ea5257a0f45942e4a
```

![image](https://github.com/user-attachments/assets/cbd1a95d-3cde-434e-bbc1-7965280f25db)


- `enc` is the encryption and decryption command of OpenSSL
- `-aes-256-ecb` specifies the encryption algorithm 
- `-in plain.txt` specifies the input file to be encrypted
- `-out plain.enc` specifies the output file to store the encrypted data
- `pass pass:...` Secret key used for encryption.

**5. Generate the hash file to check the Integrity of the file**

Use the SHA-256 to hash the encrypted file above (plain.enc) and save it in binary format:

```
openssl dgst -sha256 -binary plain.enc > plain.enc.sha256
```

![image](https://github.com/user-attachments/assets/fc37e7bd-62b2-492e-a800-74e602b952c0)


- `dgst` means "digest" used for calculating cryptographic hash functions.
- `-sha256` specifies the hash function
- `-binary` outputs the hash value in its raw binary form
- `plain.enc` input file
- `> plain.enc.sha256` output file

**6. Genererate the HMAC to check the Authenticity of the file**

Use the shared secret key to create an HMAC of the hash

```
openssl dgst -sha256 -hmac "121315f3d5dc5a016f764f1270ee10b8650a651a92a59d7ea5257a0f45942e4a" -binary plain.enc.sha256 > plain.enc.hmac
```

![image](https://github.com/user-attachments/assets/85d544d8-e8f8-40ba-9254-6001ecb33d68)


- `dgst` means "digest" used for calculating cryptographic hash functions.
- `-sha256` specifies the hash function
- `-hmac` specifies that an HMAC should be computed instead of a plain hash
- `"70c831a201c467d5b8c745...` the secret key used for HMAC generation
- `-binary` outputs the HMAC in raw binary form
- `plain.enc.sha256` input file
- `> plain.enc.hmac` output file

**7. Transfer files to Alice**

On bob machine, create an Bob user with password = `123`

![image](https://github.com/user-attachments/assets/69c0a745-df37-4a4f-ba56-0afe8fd32c14)

Change the permission access of bob user:

`sudo chmod ugo+w /home`

![image](https://github.com/user-attachments/assets/7cbecc61-57ea-41ae-a29f-f02ae33319d8)

- `chmod ugo+w`: Adds write permissions for the user, group, and others.

start ssh service in both containers:

![image](https://github.com/user-attachments/assets/9be009f4-6b52-46ac-b4cc-aabc0a252a54)

![image](https://github.com/user-attachments/assets/440c65f0-049d-4200-aff3-4c9d8805251c)

Copy the three files (`plain.enc`, `plain.enc.sha256`, `plain.enc.hmac`) to Alice using scp

```
scp plain.enc plain.enc.sha256 plain.enc.hmac alice@10.9.0.5:/home/
```

- `scp`: Secure Copy Protocol for transferring files securely over SSH.
- `plain.enc, plain.enc.sha256, plain.enc.hmac`: Files to be copied.
- `alice@10.9.0.5`: Specifies the remote user (alice) and the remote machine's IP address.
- `:/home/`: Target directory on the remote machine.

![image](https://github.com/user-attachments/assets/bd5ddd67-847d-4c4c-8ac0-ce3addcb39af)

## 2. On The Receiver container

**1. Verify in bob container:**

![image](https://github.com/user-attachments/assets/f324b945-3cdb-4a1e-ac34-14abfb62f7d4)

=> successfully receive the file

**2. Verify the HMAC:**

Computeing an HMAC (Hash-based Message Authentication Code) using the SHA-256 hash function and a provided secret key, then outputs the binary result to a file.

```
openssl dgst -sha256 -hmac "121315f3d5dc5a016f764f1270ee10b8650a651a92a59d7ea5257a0f45942e4a" -binary plain.enc.sha256 > plain.enc.hmac.verify
```

- `openssl dgst -sha256`: Computes a SHA-256 digest.
- `-hmac `: Specifies the secret key for generating the HMAC.
- `-binary`: Outputs the result in binary format (not hexadecimal).
- `plain.enc.sha256`: Input file to compute the HMAC.
- `> plain.enc.hmac.verify`: Redirects the binary HMAC output to plain.enc.hmac.verify.

After that, comparing the two files `plain.enc.hmac` and `plain.enc.hmac.verify` line by line and shows any differences.

```
diff plain.enc.hmac plain.enc.hmac.verify
```

- If the files are identical (e.g., the HMACs match), there will be no output.
- If the files differ, diff will display the differences.

![image](https://github.com/user-attachments/assets/7ec7b552-7458-429e-82a4-8fc5e96c3b6d)


=> There are no differnece shown. This confirms the authenticity of the text file.


**2. Verify the HASH:**

computes the SHA-256 hash of the file `plain.enc` and saves the binary hash output into the file `plain.enc.sha256.verify`.

```
openssl dgst -sha256 -binary plain.enc > plain.enc.sha256.verify
```
- `openssl dgst -sha256`: Computes a SHA-256 digest.
- `-binary`: Outputs the hash in binary format (instead of hexadecimal).
- `plain.enc`: Input file for which the SHA-256 hash is calculated.
- `> plain.enc.sha256.verify`: Redirects the binary hash output to the file `plain.enc.sha256.verify`.

After that, comparing the two files `plain.enc.sha256` and `plain.enc.sha256.verify`

```
diff plain.enc.sha256 plain.enc.sha256.verify
```

- No output: The files are identical, meaning the SHA-256 hashes match.
- Output: Indicates differences, meaning the hashes do not match.

![image](https://github.com/user-attachments/assets/99b06331-83f8-48ad-bb7d-fbc2e6408a0f)

=> There are no differnece shown. This confirms the Integrity of the text file.

**3. Decrypt the file:**

Decrypting a file encrypted using AES-256 in ECB mode.

```
openssl enc -aes-256-ecb -d -in plain.enc -out plain.txt -pass pass:121315f3d5dc5a016f764f1270ee10b8650a651a92a59d7ea5257a0f45942e4a
```
![image](https://github.com/user-attachments/assets/de732305-c5a2-4f6c-bb06-206ce3cb9e4b)

- `openssl enc`: Invokes OpenSSL's encryption/decryption functionality.
- `-aes-256-ecb`: Specifies the AES algorithm with a 256-bit key in ECB (Electronic Codebook) mode.
- `-d`: Indicates decryption mode.
- `-in plain.enc`: Specifies the input encrypted file (plain.enc).
- `-out plain.txt`: Specifies the output decrypted file (plain.txt).
- `-pass pass: `: Provides the decryption key directly in the command (121315f3...).

Read the file:

![image](https://github.com/user-attachments/assets/6528dced-630e-4e3a-a9d0-12d1814e7b5c)

=> successfully decrypt

# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:

## 1. Receiver creates a key pair 

the key pair contains public and private key, and after that share the key to sender
- Receiver's private key needs to keep secure
- Share the receiver's public key to sender

Firstly, create a private key: 

```
openssl genrsa -out privateKey.pem 2048
```

![image](https://github.com/user-attachments/assets/a8e43d88-7474-4279-b403-84a828193a0f)


- `genrsa`: generate a new RSA private key.
- `-out privateKey.pem` specifies the file to save generated private key.
- `2048`: specifies the size of the RSA key (bits).

Secondly, create a pubic key: 

```
openssl rsa -in privateKey.pem -pubout -out publicKey.pem
```

- `openssl rsa`: Handles RSA key operations.
- `-in privateKey.pem`: Specifies the input file containing the RSA private key.
- `-pubout`: Extracts the public key from the private key.
- `-out publicKey.pem`: Saves the extracted public key to the file publicKey.pem.

![image](https://github.com/user-attachments/assets/253ed5a7-dbe8-4e0d-ab7c-3a2643be051c)

## 2. Send to public key to sender

- Go to Alice container and update the password to `123`

```
useradd -m -s /bin/bash alice
passwd alice
```

![image](https://github.com/user-attachments/assets/2859203b-9dfb-49a2-b516-b39f017b27cb)

- Change the access permission to this user:

```
sudo chmod ugo+w /lab2
```
- Go to Bob container and transfer the public key to alice by using `scp`

```
scp publicKey.pem alice@10.9.0.5:/lab2
```

![image](https://github.com/user-attachments/assets/f3025a18-f97c-40b3-8610-0ff8f26543e8)

`scp`: Secure Copy Protocol used to transfer files securely over SSH.
`publicKey.pem`: The file to be copied.
`alice@10.9.0.5`: Specifies the remote user (alice) and the remote machine's IP address (10.9.0.5).
`:/lab2`: Target directory on the remote machine where the file will be copied (/lab2).

Go to Alice container and check the key

![image](https://github.com/user-attachments/assets/177ec3c7-f1ed-4197-b380-3e6469c1beb9)

## 3. Sender prepares the file to send 

**1. Create a plain text**

```
echo "Send some random message to Bob" > plain.txt
```

![image](https://github.com/user-attachments/assets/158ab8ac-6b98-4143-a3fc-db759bbf023b)

**2. Generate AES (Random Symmetric key)**

Generates a 256-bit (32-byte) random symmetric key (in hexadecimal form) and saves it to the file symmetricKey.txt

```
openssl rand -hex 32 > symmetricKey.txt
```

![image](https://github.com/user-attachments/assets/ac394bfc-6d98-4927-93f8-9901b0ae7cae)

- `openssl rand`: This generates random bytes using the OpenSSL library's secure random number generator.
- `-hex`: Specifies that the output should be in hexadecimal format.
- `32`: Requests 32 bytes of random data.
- `symmetricKey.txt`: The file where the generated key will be stored.`

**3. using AES-256-CBC to encrypt the file**

Encrypts the content of plain.txt using AES-256-CBC, with the symmetric key stored in symmetricKey.txt

```
openssl enc -aes-256-cbc -in plain.txt -out plain.enc -pass file:symmetricKey.txt
```

![image](https://github.com/user-attachments/assets/5115b12f-2429-4f88-b1da-e61f2e1f3c8c)

`openssl enc`: Invokes the OpenSSL encryption utility.
`-aes-256-cbc`: Specifies the encryption algorithm to use, in this case, AES (Advanced Encryption Standard) with a 256-bit key and CBC (Cipher Block Chaining) mode.
`-in plain.txt`: Specifies the input file to encrypt, plain.txt.
`-out plain.enc`: Specifies the output file where the encrypted data will be saved, plain.enc.
`-pass file:symmetricKey.txt`: Reads the encryption key from the file symmetricKey.txt.

**4. using the public key to encrypt the symmetric key**

Encrypts the symmetric key (stored in symmetricKey.txt) using the public RSA key (publicKey.pem) and saves the encrypted symmetric key to symmetricKey.enc

```
openssl rsautl -encrypt -inkey publicKey.pem -pubin -in symmetricKey.txt -out symmetricKey.enc
```

![image](https://github.com/user-attachments/assets/c5b90983-ff3f-443f-8fef-b0a3b8cd9688)

- `openssl rsautl`: Utilizes the RSA algorithm for encryption or decryption of data.
- `-encrypt`: Specifies that the operation is encryption.
- `-inkey publicKey.pem`: Uses the RSA public key stored in publicKey.pem for encryption.
- `-pubin`: Indicates that the key provided in -inkey is a public key.
- `-in symmetricKey.txt`: Specifies the input file to encrypt
- `-out symmetricKey.enc`: Specifies the output file 

## 4. Sender sends file to reciever

**1. Go to bob container and change the access permission**

`sudo chmod ugo+w /lab2`

![image](https://github.com/user-attachments/assets/80d2ebe3-f623-4025-a58e-4cb01b36b6b6)


**2. Go to Alice container and use `scp` to send to bob**

Transfers the files plain.enc and symmetricKey.enc to the /lab2 directory on the remote server with IP 10.9.0.6

```
scp plain.enc symmetricKey.enc bob@10.9.0.6:/lab2
```

![image](https://github.com/user-attachments/assets/848f9f34-4766-4724-8fd6-45d242e8ef0f)

- `plain.enc`: The first file to be copied (encrypted plaintext).
- `symmetricKey.enc`: The second file to be copied (encrypted symmetric key).
- `bob@10.9.0.6`: Specifies the username (bob) and the IP address of the remote server (10.9.0.6).
- `:/lab2`: Specifies the destination directory (/lab2) on the remote server.

## 5. Receiver decrypts the file

**1. Using the private key of Bob to decrypt the symmetric key**

Decrypts the encrypted symmetric key (symmetricKey.enc) using the private RSA key (privateKey.pem) and saves the decrypted key to decryptedSymmetricKey.txt

```
openssl rsautl -decrypt -inkey privateKey.pem -in symmetricKey.enc -out decryptedSymmetricKey.txt
```

![image](https://github.com/user-attachments/assets/8d3f4b19-acc1-4547-a663-6a317c9b3b12)

- `-aes-256-cbc`: the encryption/decryption algorithm to use, which is Advanced Encryption Standard (AES) with a 256-bit key size in Cipher Block Chaining (CBC) mode.
- `-d`: This flag indicates that we want to perform decryption.
- `-in message.enc`: the input file containing the encrypted data.
- `-out message_decrypted.txt`: the output file for the decrypted data.
- `-pass file:symmetric_key_decrypted.txt`: the file containing the symmetric key used for decryption.

**2. Using the decrypted symmetric key to decrypt the file**

Decrypts the encrypted file plain.enc using AES-256-CBC and the symmetric key stored in decryptedSymmetricKey.txt. The decrypted data is saved in decryptedPlain.txt.

```
openssl enc -aes-256-cbc -d -in plain.enc -out decryptedPlain.txt -pass file:decryptedSymmetricKey.txt
```

![image](https://github.com/user-attachments/assets/63846a96-b075-435a-9015-4719aa90548c)

=> completes the hybrid encryption.

**Conclusion for task 2:**

1. Bob generates a public-private key pair and shares his public key with Alice.
2. Alice encrypts a file with a symmetric key and then encrypts the symmetric key with Bob's public key.
3. Alice sends the encrypted file and the encrypted symmetric key to Bob.
4. Bob decrypts the symmetric key with his private key and uses it to decrypt the file.


# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:

## 1. install iptables in docker containers

install iptables for both alice and bob containers

```
apt-get update
apt-get install -y iptables
```

check iptable version for both containers: `iptables --version`

![image](https://github.com/user-attachments/assets/8f9a09d4-cb20-4eb9-ad55-8209770860e8)

![image](https://github.com/user-attachments/assets/8fb07fed-8da4-4046-8c4a-87a05a0adca5)

Also, add the `cap_add` option in docker-compose.yml in order to have permission to run iptables

![nothing](https://github.com/user-attachments/assets/4f9402fb-7524-4870-9d95-0bded85afaa4)


## 2. Configure alice as a web and SSH server

1. Install openssh-server and apache2 server on alice machine

```
apt-get install -y apache2 openssh-server
```

2. Start Apache (HTTP) and SSH services:

```
service apache2 start
service ssh start
```
![image](https://github.com/user-attachments/assets/c8cdce92-2928-4141-865e-e90de66ca009)

4. Verify the status of the servers

```
service apache2 status
service ssh status
```

![image](https://github.com/user-attachments/assets/9869eea8-fd71-4632-a9d4-da123300b5b5)

## 3. Block traffic using `iptables` on alice

**1. Block HTTP traffic (port 80) from bob**

```
iptables -A INPUT -p tcp --dport 80 -s 10.9.0.6 -j DROP
```

![image](https://github.com/user-attachments/assets/c80ed6ef-53d5-4f8f-ab03-b68f9b7f9fb9)

- `-A INPUT`: Append a rule to the INPUT chain to handle incoming traffic.
- `-p tcp`: Target TCP protocol.
- `-dport 80`: Match traffic destined for port 80 (HTTP).
- `-s 10.9.0.6`: Match packets from source IP 10.9.0.6 (Bob).
- `-j DROP`: Drop matching packets silently, without sending any response.

Check the iptables list: `iptables -L`

![image](https://github.com/user-attachments/assets/af1810ea-1699-47c6-b659-8da91f3248c4)


Try to access the alice web server from bob:

```
curl http://10.9.0.5
```

![image](https://github.com/user-attachments/assets/cb28240e-b5ad-468d-ab40-efc57ef3752a)

=> No html is sent back to bob. The connection fails.
=> successfully block HTTP traffic

**2. Block ICMP (ping) requests from bob**

blocking ICMP echo requests (ping requests) in both incoming and outgoing directions.

```
iptables -A INPUT -p icmp --icmp-type echo-request -s 10.9.0.6 -j DROP
iptables -A OUTPUT -p icmp --icmp-type echo-request -d 10.9.0.6 -j DROP
```
![image](https://github.com/user-attachments/assets/b11c9dd4-04f4-431f-b55e-adf286e44e76)


- `-A INPUT`/`-A OUTPUT`: Adds the rule to the INPUT/OUTPUT chain (incoming traffic).
- `-p icmp`: Specifies the rule applies to ICMP (Internet Control Message Protocol).
- `--icmp-type echo-request`: Matches ICMP echo request packets (ping requests).
- `-s 10.9.0.6`/`-d 10.9.0.6`: The rule applies to requests from IP address 10.9.0.6.
- `-j DROP`: Drops the matching packets, effectively blocking the ping requests from 10.9.0.6.

Check the iptables list: `iptables -L`

![image](https://github.com/user-attachments/assets/2dccc277-6a31-40aa-80b8-a31f70e4d245)

Try to ping alice web server from bob:
```
ping 10.9.0.5
```

![image](https://github.com/user-attachments/assets/b33e6fd4-2df6-4f70-afb0-517665093d5e)

=> the ping (ICMP) request fails because there are no reply 
=> successfully Block ping (ICMP) requests

**3. Block SSH access (port 22) from bob**

```
iptables -A INPUT -p tcp --dport 22 -s 10.9.0.6 -j DROP
```

![image](https://github.com/user-attachments/assets/cc82d855-dc76-4e68-b0be-45df1b139cec)

- `-A INPUT`: Adds the rule to the INPUT chain, which processes incoming traffic.
- `-p tcp`: Specifies that this rule applies to TCP traffic.
- `--dport 22`: The rule applies to packets destined for port 22, which is the default port for SSH (Secure Shell).
- `-s 10.9.0.6`: The rule applies to traffic coming from the IP address 10.9.0.6.
- `-j DROP`: The action is to drop the matching packets, meaning the packets will be silently discarded without any notification.

Check the iptables list: `iptables -L`

![image](https://github.com/user-attachments/assets/75eb2840-2f3e-443e-aced-3d1dfdd0613a)

Try connecting to alice via `SSH` from bob:

```
ssh 10.9.0.5
```

![image](https://github.com/user-attachments/assets/bbf64cb6-16f8-4a57-935f-eba7e891d347)

=> there are no response

We can also try to send a plain.txt from bob to alice:

```
scp plain.txt alice@10.9.0.5:/home
```

![image](https://github.com/user-attachments/assets/9427445b-e3b2-44bc-8b3e-660eb936b11f)

=> there are no response as well
=> sucessfully block SSH connection (port 22)

## 3. Unblock traffic using `iptables` on alice

**1. Unblock HTTP traffic (port 80) from bob**

```
iptables -D INPUT -p tcp --dport 80 -s 10.9.0.6 -j DROP
```

![image](https://github.com/user-attachments/assets/ef7856af-b111-4a3f-8a40-31fc7155e595)

- `-D INPUT`: This deletes a rule from the INPUT chain, which controls incoming traffic to the system.

- `-p tcp`: The rule applies to TCP packets, which is the most common protocol for reliable communication (used for web browsing, SSH, etc.).

- `--dport 80`: This specifies that the rule applies to packets with a destination port 80, which is typically used for HTTP (web) traffic. This means the rule is targeting web traffic.

- `-s 10.9.0.6`: This specifies the source IP address, restricting the rule to only packets coming from the IP address 10.9.0.6.

- `-j DROP`: This specifies the action to take when the rule matches: DROP means to silently discard the packet without any response.

Check the iptables list: `iptables -L`

![image](https://github.com/user-attachments/assets/e9cbd10d-af33-41e2-a04a-72fcfc95ff68)

=> we can't see that rule to block http request has been deleted

Try accessing the web server on alice from bob:

```
curl http://10.9.0.5
```

![image](https://github.com/user-attachments/assets/88f8067c-5e3e-44da-8d4f-b691fbf051ae)

=> Alice gets the html packet from bob
=> successfully unblock http request

**2. Unblock ICMP (ping) requests from bob**

Unblocking both incoming and outgoing ICMP echo requests

```
iptables -D INPUT -p icmp --icmp-type echo-request -s 10.9.0.6 -j DROP
iptables -D OUTPUT -p icmp --icmp-type echo-request -d 10.9.0.6 -j DROP
```

![image](https://github.com/user-attachments/assets/8c3d65cb-e668-46ff-9e11-7feae841ded7)

- `-D`: This flag is used to delete a rule from the iptables configuration.
- `INPUT`/`OUTPUT`: Specifies the INPUT/OUTPUT chain, which handles incoming network traffic (packets arriving at the server).
- `-p icmp`: Specifies the protocol to match, which in this case is ICMP (commonly used for "ping" requests).
- `--icmp-type echo-request`: This option specifies that the rule applies to echo-request packets (ping requests). When someone sends a ping to the server, this is the type of ICMP packet used.
- `-s 10.9.0.6`/`-d 10.9.0.6`: This matches the source IP address. Only ping requests coming from the IP address 10.9.0.6 will be affected by this rule.
- `-j DROP`: The -j flag specifies the target action. In this case, the target is DROP, meaning the matching packets (ping requests from 10.9.0.6) will be dropped and not allowed to reach the server.

Check the iptables list: `iptables -L`

![image](https://github.com/user-attachments/assets/80023c21-ceca-4ac0-8684-a380bfe43e86)

=> rules to block ping request have been deleted

try to ping alice from bob

```
ping 10.9.0.5
```

![image](https://github.com/user-attachments/assets/89926560-cdc0-4a94-94d6-be72a0587469)

Try to ping bob from alice

```
ping 10.9.0.6
```

![image](https://github.com/user-attachments/assets/1f3ec378-5710-4b34-b623-f9e78477bad7)

=> we can see ICMP replies from both directions
=> successfully unblock ping (ICMP) requests.

**3. Unblock SSH access (port 22) from bob**

```
iptables -D INPUT -p tcp --dport 22 -s 10.9.0.6 -j DROP
```

![image](https://github.com/user-attachments/assets/0c4512fe-9f7b-488e-87ed-ea568997d0a4)

- `-D`: This flag is used to delete a rule from the iptables configuration.
INPUT: Refers to the INPUT chain, which handles incoming traffic (packets arriving at the server).
- `-p tcp`: Specifies that the rule applies to the TCP protocol. TCP is a common protocol used for reliable communication between devices on a network.
- `--dport 22`: This option matches packets that are destined for port 22, which is the default port for SSH (Secure Shell) connections. So, this rule applies to SSH traffic.
- `-s 10.9.0.6`: Specifies the source IP address. In this case, the rule will apply to packets coming from the IP address 10.9.0.6.
- `-j DROP`: The -j flag specifies the target action. In this case, the target is DROP, meaning that any matching packet (SSH requests from 10.9.0.6) will be dropped (blocked) and not allowed to reach the server.

Check the iptables list: `iptables -L`

![image](https://github.com/user-attachments/assets/aff49a2a-1d83-455f-a45c-ee022663aaac)

=> rule to block SSH has been deleted

try to send a plain.txt from bob to alice:

```
scp plain.txt alice@10.9.0.5:/home
```

![image](https://github.com/user-attachments/assets/ee0f1d90-82a9-446f-abee-fe0525b7cdc6)

In alice:

![image](https://github.com/user-attachments/assets/48846ca1-a361-453b-a6b8-41c41b912d97)

=> successfully unblock ssh access
