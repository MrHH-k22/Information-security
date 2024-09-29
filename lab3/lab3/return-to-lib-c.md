![image](https://github.com/user-attachments/assets/56167c7c-b461-43b3-8b7c-8705ff462cfd)# Lab #3: exploit return-to-lib-c vulnerability, delete file "dummy" in directory /tmp/dummy using environment variable.

Chương trình vuln.c:

![image](https://github.com/user-attachments/assets/bbbda87b-19ca-436b-a825-79336ed28113)

Đầu tiên, Tạo 1 dummyfile bằng lệnh `touch dummyfile`

![image](https://github.com/user-attachments/assets/5c21e1cb-25e0-4f41-b6c4-a5f7aa4e7d76)

Bắt đầu biên dịch chương trình

![image](https://github.com/user-attachments/assets/64448c37-6eed-4393-96ac-c79c1d2571d1)

Tắt chế độ cấp phát địa chỉ stack ngẫu nhiên

`sudo sysctl -w kernel.randomize_va_space=0`

![image](https://github.com/user-attachments/assets/37cc1b16-9177-4c97-88ce-51e552de75cc)

Bắt đầu biên dịch file_del.asm

![image](https://github.com/user-attachments/assets/eff4c2c1-2389-418f-92b3-94dbe04c7a8c)

Sau đó tạo 1 biến môi trường có tên là `my_path` với đường dẫn đến file 'file_del'

![image](https://github.com/user-attachments/assets/d917b8b7-ef11-4ad3-94a3-9df999ea2fe2)

Vé stackframe ta thấy được rằng














