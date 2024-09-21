![image](https://github.com/user-attachments/assets/503ec49a-8ed1-4b68-9291-8482a4818176)

trong chương trình, ta thấy có lỗ hổng buffer overflow tại hàm fget khi biến buf có kích thước là 40 bytes trong khi fget có thể đọc tới 45 bytes, vì thế người dùng có thể nhập dữ liệu nhiều hơn 40 bytes để có thể ghi đè lên các bytes lân cận

![image](https://github.com/user-attachments/assets/e91ee0d9-4d7a-4372-9848-636d88c38fce)

đây là bản vẽ stack của chương trình, mục tiêu của ta là thay đổi biến check sao cho giá trị phải khác giá trị được gán ban đầu là 0x04030201 và bằng giá trị được đưa 0xdeadbeef


chạy chương trình 

`echo $(python -c "print('a'*40+'\xef\xbe\xad\xde')") | ./bof2.out`
- dùng echo ... | ./bof2.out : để chuyển kết quả đầu ra của eco làm đầu vào cho chương trình
- nhập 40 ký tự a và giá trị 0xdeadbeef bằng python

kết quả:

![image](https://github.com/user-attachments/assets/854ccaab-60a2-4983-b25e-0b94dfeb7c0c)

