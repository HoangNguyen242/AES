# AES  
# Cách xây dựng thuật toán AES  
### 1. Xây dựng bảng X-box thuận  
 * Bảng S-box thuận được sinh ra bằng việc xác định nghịch đảo cho một giá trị nhất định trên GF(28) = GF(2)[x] / (x8+x4+x3+x+1) (trường hữu hạn Rijindael). Giá trị 0 không có nghịch đảo thì được ánh xạ với 0. Những nghịch đảo được chuyển đổi thông qua phép biến đổi affine.  
 * ![thuận](https://viblo.asia/uploads/ac735e46-c67f-4024-9989-45780195805e.png)    
### 2. Xây dựng bảng X-box đảo  
 * S-box nghịch đảo chỉ đơn giản là S-box chạy ngược. Nó được tính bằng phép biến đổi affine nghịch đảo các giá trị đầu vào  
 * ![đảo](https://viblo.asia/uploads/67e81061-f1ed-4a5a-b5c4-b2d5cc4cd79d.png)  
### 3. Giải thuật sinh khóa phụ  
 * Quá trình sinh khóa gồm 4 bước :  
   * Rotword: quay trái 8 byte  
   * SubBytes  
   * Rcon: tính giá trị Rcon(i) Trong đó :  
   * > Rcon(i) = x(i-1) mod (x8 + x4 + x3 + x + 1).  
   * ShiftRow   

# Quá trình mã hóa AES  
### 1. Hàm AddRoundKey  
 * Được áp dụng từ vòng lặp thứ 1 tới vòng lặp Nr  
 * Trong biến đổi Addroundkey(), một khóa vòng được cộng với state bằng một phép XOR theo từng bit đơn giản.  
 * Mỗi khóa vòng gồm có 4 từ (128 bit) được lấy từ lịch trình khóa. 4 từ đó được cộng vào mỗi cột của state, sao cho:  
 * > [S’0,c, S’1,c, S’2,c, S’3,c ] = [S0,c, S1,c, S2,c, S3,c ]  [W(4*i + c)] với 0 <= c < 4.  ```  
 ```  
  def __add_round_key(self, s, k):
    for i in range(4):
      for j in range(4):
        s[i][j] ^= k[i][j]  
 ```  
 
### 2. Hàm SubBytes  
 * Biến đổi SubBytes() thay thế mỗi byte riêng rẽ của state Sr,c bằng một giá trị mới S’ r,c sử dụng bảng thay thế (S - box) được xây dựng ở trên.  
 ```  
  def __sub_bytes(self, s):
    for i in range(4):
      for j in range(4):
        s[i][j] = Sbox[s[i][j]]


  def __inv_sub_bytes(self, s):
    for i in range(4):
      for j in range(4):
          s[i][j] = InvSbox[s[i][j]]

 ```  
 
### 3. Hàm ShiftRow  
 * Trong biến đổi ShiftRows(), các byte trong ba hàng cuối cùng của trạng thái được dịch vòng đi các số byte khác nhau (độ lệch). Cụ thể :  
 * > S’r,c = Sr,(c + shift ( r, Nb)) mod Nb (Nb = 4)  
 * Trong đó giá trị dịch shift (r, Nb) phụ thuộc vào số hàng r như sau:
 * > Shift(1,4) = 1, shift(2,4) = 2, shift(3,4) = 3.  
 * Hàng đầu tiên không bị dịch, ba hàng còn lại bị dịch tương ứng:  
   * Hàng thứ 1 giữ nguyên.  
   * Hàng thứ 2 dịch vòng trái 1 lần.  
   * Hàng thứ 3 dịch vòng trái 2 lần.  
   * Hàng thứ 4 dịch vòng trái 3 lần.   
```  


   
   
