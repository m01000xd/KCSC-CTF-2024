Như tên bài, chal được viết bằng Rust. Load file vào IDA:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/e2ebc73b-03be-446a-8414-91fb4a7fb82e)

Có thể thấy, pseudocode của hàm main khá khó đọc, cú pháp của các hàm trong Rust làm chương trình trở nên khá rối rắm. Nhưng đọc kĩ, chương trình sẽ mã hóa flag qua 4 phase:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/3b008dc1-e3a3-4144-bc8e-87b84f38351f)


### Phase 1: 
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/9ccfd73f-57f5-4f16-b85e-878ba758fe14)

Nhìn sơ qua thì thấy chương trình tạo nhiều biến để làm gì đó, nhưng  thực chất chỉ là đang cố đảo ngược các kí tự trong ban đầu:
```python3
def phase1(flag):
  for i in range(0,len(flag)):
    v7 = flag[i]
    flag[i] = flag[len(flag)-1-i]
    flag[len(flag)-1-i] = v7
    return flag
```

### Phase 2:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/5e138475-70be-4908-8b32-17c5a3db36f1)

Nhìn qua thì cũng khá dễ hiểu cách mã hóa: 

```python3
def phase2(flag):
  for i in range(0,len(flag)-2,2):
      v4 = flag[i] & 0xF | flag[i+1] & 0xF0;
      flag[i] = flag[i+1] & 0xF | flag[i] & 0xF0;
      flag[i+1] = v4;
```

### Phase 3:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/ff7f0e1c-3a38-4838-87bd-0f7ca82705da)
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/ffad0915-1352-4a3f-9a22-c90afc0916f8)

Hơi nhiều biến hơn phase2, nhưng vẫn khá dễ hiểu:

```python3
def phase3(flag):
  for i in range(0,len(str)-2):
      v10 = str[i]
      str[i] = v10 - str[i+2]
      v8 = str[i+2]
      str[i+2] = v8 - str[i]
```
### Phase 4:
![image](https://github.com/m01000xd/KCSC-CTF-2024/assets/122852491/672b6cb7-e7ec-4c44-a175-8f19c2320110)

Phase 4 sẽ xor flag với v5, v5 là 1 số ngẫu nhiên từ 0-255:
```python3
def phase4(flag):
	for i in range(len(flag)):
		flag[i] ^= (v5 & 0xFF) ^ ((v5 >> 8) & 0xFF) ^ ((v5 >> 16) & 0xFF) 
```
Phase 4 này có thể brute force vì flag chỉ xor với 1 trong 0-255 kí tự
Sau 4 phase, thì sẽ ra được flag_enc trong file flag_enc.

Giờ thì sẽ viết script lấy flag:

### Script

```python3
def phase4(flag, letter):
    for i in range(len(flag)):
        flag[i] ^= letter
    return flag
def phase3(flag):
    for i in range(len(flag)-2-1,-1,-1):
        v8 = flag[i+2]
        flag[i+2] = v8 + flag[i]
        v10 = flag[i]
        flag[i] = v10 + flag[i+2]
    return flag
def phase2(flag):
    for i in range(0, len(flag) - 1, 2):
        v4 = (flag[i] & 0xf) | (flag[i + 1] & 0xf0)
        flag[i] = (flag[i + 1] & 0xf) | (flag[i] & 0xf0)
        flag[i + 1] = v4
    return flag
def phase1(flag):
    return flag[::-1]



for i in range(256):
    with open('flag.enc', 'rb') as file:
        flag_enc = file.read()
    
    enc = list(flag_enc)
    x = phase4(enc,i)
    y = phase3(x)
    z = phase2(y)
    t = phase1(z)

    a = t
    b = ''.join([chr(i) for i in a])
    if 'KCSC' in b:
        if '}' in b:
            print(b)


```
### Flag
**KCSC{r3v3rs3_rust_1s_funny_4nd_34sy_227da29931351}**

