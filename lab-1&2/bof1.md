![image](https://github.com/user-attachments/assets/b2479ce0-366d-4b41-be08-9482478ccca8)

Trong chương trình này, ta thấy được lỗ hổng buffer overflow nằm trong hàm vuln() là hàm get().
hàm get() có điểm yếu là không kiểm tra độ dài chuỗi đầu vào, nên ta có thể nhập vào vượt quá số lượng và đè lên các bytes khác, từ đó dẫn tới buffer overflow

![image](https://github.com/user-attachments/assets/47059d6b-f4d5-409b-b8a9-59dc5e907e53)

Đây là bản vẽ stack của chương trình, ta thấy được nếu nhập 200 ký tự + thêm 4 bytes của ebp để đến return address của hàm vuln, ta có thể thay thế return address của hàm vuln() thành địa chỉ của hàm secretFunc()

dùng gdb để tìm giá trị hàm secretFunc()

![image](https://github.com/user-attachments/assets/4bbd9a1d-8ae4-45c0-8fe7-700addb93dae)


=> 0x0804846b

chạy chương trình

`echo $(python -c "print('a'*204 + '\x6b\x84\x04\x08')") | ./bof1.out`
- dùng echo ... | ./bof1.out : để chuyển kết quả đầu ra của eco làm đầu vào cho chương trình
- nhập 204 ký tự a và giá trị trả về của hàm secretFunc bằng python

Kết quả:

![image](https://github.com/user-attachments/assets/0039f07b-17c4-45a1-9ad6-bec288d6551f)









