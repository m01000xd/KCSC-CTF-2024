Load file vào IDA:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/54807b99-8ae9-4051-8765-4910d7c4dc91)

Nhìn sơ qua thì đây là một bài mã hóa RC4 bình thường, cho một mảng Cipher, rồi qua hàm RC4, trả về decrypt, sau đó so sánh Input với decrypt bằng hàm lstrcmpA. Thử đặt breakpoint ở hàm lstrcmpA:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/54e4574e-09ca-488a-8650-b0bd3298e889)

Nhưng sau khi debug, decrypt lại trả về 1 fake flag:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/5a22acc9-1470-42c4-9866-4c4bf2dee7a0)

Nháy vào hàm lstrcmpA, ta thấy nó trỏ đến địa chỉ của 1 hàm __sub_7FF7B7911230__
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/71b727d2-e732-42d3-871f-c9e1388a4bb8)

## sub_7FF7B7911230
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/27efef37-22a0-4500-9e9d-3133fbcc5af3)

Hàm này thậm chí còn không quan tâm đến Input từ người dùng, nó sẽ xor từng kí tự lấy từ chuỗi decrypt lúc nãy với một chuỗi cố định Str cho sẵn rồi gán giá trị mới vào mảng v5. Từ đây có thể suy ra v5 chính là mảng chứa flag. v5 được tạo bằng cách xor decrypt với Str, mà Str thì cố đinh, còn decrypt thì lại được tạo từ mã hóa RC4 qua Key và Cipher.

Quay ngược lại, xref cả Key và Cipher:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/c6065dd2-89c0-4c0f-b76b-9c0550ee6383)

Cipher thì không thấy được gọi bởi hàm gì khác ngoài main. Giờ còn Key:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/32cc5d3e-1f66-4c0d-8491-447bbb327699)

Key thì được gọi khá nhiều lần bởi 1 hàm ngoài main. Nháy vào hàm trên.

![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/062553aa-ff6c-4f4e-8813-01ca0ba7c2f6)

Key ở đây được mã hóa bằng cách | 4 kí tự cuối với 1.
```python3
key = 'F@**!'
key = [hex(ord(key[0]))] + [hex(ord(key[i]) | 1) for i in range(1,5)]
# ['0x46', '0x41', '0x2b', '0x2b', '0x21']
```
Giờ chúng ta sẽ patch lại key
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/5eac91f8-d204-4d9e-8b3c-ba6b2f494215)
Nháy vào dòng Str, sau đó lên Edit, Patch program, Change byte
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/42eb3441-2bf0-479d-b7fe-31d31ebada96)
Sửa thành 46 41 2B 2B 21
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/0030dd9e-5b62-424e-8adc-b73e005cb029)
Sau đó đặt breakpoint lại
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/da237857-d24b-4f3e-af67-064930ad98ad)
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/e4223e59-2da5-46f5-bf22-1d1e48c6c90f)
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/5ab223fc-b0be-4c76-9bfe-9429c13c38b8)

## Flag:
  **KCSC{1t_co5ld_be_right7_fla9_here_^.^@@}**
