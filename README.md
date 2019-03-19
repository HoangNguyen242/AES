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
  def __shift_rows(self, s):
    s[0][1], s[1][1], s[2][1], s[3][1] = s[1][1], s[2][1], s[3][1], s[0][1]
    s[0][2], s[1][2], s[2][2], s[3][2] = s[2][2], s[3][2], s[0][2], s[1][2]
    s[0][3], s[1][3], s[2][3], s[3][3] = s[3][3], s[0][3], s[1][3], s[2][3]


  def __inv_shift_rows(self, s):
    s[0][1], s[1][1], s[2][1], s[3][1] = s[3][1], s[0][1], s[1][1], s[2][1]
    s[0][2], s[1][2], s[2][2], s[3][2] = s[2][2], s[3][2], s[0][2], s[1][2]
    s[0][3], s[1][3], s[2][3], s[3][3] = s[1][3], s[2][3], s[3][3], s[0][3]

```

### 4. Hàm MixColumns  
 * Biến đổi MixColumns() tính toán trên từng cột của state. Các cột được coi như là đa thức trong trường GF(28) và nhân với một đa thức a(x) với:  
 * > a(x) = (03)x^3 +(01)x^2 +(01)x + (02)  
 * Biến đổi này có thể được trình bày như phép nhân một ma trận, mà mỗi byte được hiểu như là một phần tử trong trường GF(28)  
 ```  
  def __mix_single_column(self, a):
  # please see Sec 4.1.2 in The Design of Rijndael
    t = a[0] ^ a[1] ^ a[2] ^ a[3]
    u = a[0]
    a[0] ^= t ^ xtime(a[0] ^ a[1])
    a[1] ^= t ^ xtime(a[1] ^ a[2])
    a[2] ^= t ^ xtime(a[2] ^ a[3])
    a[3] ^= t ^ xtime(a[3] ^ u)


  def __mix_columns(self, s):
    for i in range(4):
      self.__mix_single_column(s[i])


  def __inv_mix_columns(self, s):
  # see Sec 4.1.3 in The Design of Rijndael
    for i in range(4):
      u = xtime(xtime(s[i][0] ^ s[i][2]))
      v = xtime(xtime(s[i][1] ^ s[i][3]))
      s[i][0] ^= u
      s[i][1] ^= v
      s[i][2] ^= u
      s[i][3] ^= v

      self.__mix_columns(s)
 ```  
 
 

   
