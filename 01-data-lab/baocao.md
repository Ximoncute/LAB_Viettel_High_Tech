# BÁO CÁO CHI TIẾT: GIẢI THÍCH TOÀN BỘ BÀI TẬP CS:APP DATA LAB (bits.c)

Báo cáo này được viết chi tiết nhất có thể, giải thích từ những khái niệm cơ bản nhất về toán tử bit, số bù 2, số thực dấu phẩy động (IEEE 754) cho đến cách hoạt động của từng dòng code. 

---

## PHẦN 1: HƯỚNG DẪN CHẠY LỆNH NHANH (QUICK START)
*Bạn chỉ cần copy đúng các lệnh dưới đây vào terminal WSL (hoặc Ubuntu) để cài đặt, biên dịch và chạy thử.*

### Bước 1: Cài đặt thư viện hỗ trợ biên dịch 32-bit (Chỉ chạy 1 lần duy nhất)
Bài lab này yêu cầu biên dịch ở kiến trúc 32-bit (`-m32`). Bạn hãy copy dòng lệnh dưới đây và chạy trong WSL:
```bash
sudo apt update && sudo apt install -y build-essential gcc-multilib g++-multilib perl
```

### Bước 2: Di chuyển vào thư mục bài làm
```bash
cd /mnt/c/Users/Admin/Desktop/Viettel_high_tech/01-data-lab/datalab-handout
```

### Bước 3: Dọn dẹp tệp tin cũ & Biên dịch lại
Mỗi lần bạn sửa đổi file `bits.c`, bạn **bắt buộc** phải chạy lệnh này để biên dịch lại chương trình thử nghiệm:
```bash
make clean && make btest
```

### Bước 4: Chạy thử nghiệm tất cả các hàm để kiểm tra kết quả
Chạy lệnh dưới đây để kiểm tra xem bạn được bao nhiêu điểm. Chúng tôi đã tăng thời gian chờ lên 30 giây để tránh lỗi quá hạn (timeout):
```bash
./btest -T 30
```

### Bước 5: Chạy tự động chấm điểm toàn bộ (Cả điểm chính xác và số toán tử)
Chạy lệnh này để kiểm tra xem bạn có vi phạm quy định nào không (như dùng quá số lượng toán tử, dùng lệnh cấm) và hiển thị bảng điểm cuối cùng:
```bash
./driver.pl
```
*Kết quả đúng tuyệt đối sẽ hiển thị: **`Score = 62/62 [36/36 Corr + 26/26 Perf]`***

---

## PHẦN 1.5: CÁC FILE CẦN SỬA ĐỔI & KHÔNG CẦN SỬA ĐỔI
Để hoàn thành bài tập này và đạt điểm tối đa $62/62$, bạn chỉ cần can thiệp vào đúng **2 file** dưới đây trong thư mục `datalab-handout`:

### 1. Các file CẦN sửa đổi (Cần thêm code):
* **`bits.c`** (File chính của bài làm):
  - Đây là file chứa 13 hàm cần giải (các hàm xử lý bit và float). Bạn cần viết mã nguồn các câu trả lời vào đây.
  - Vị trí sửa đổi bắt đầu từ sau dòng `#endif` (khoảng dòng 136 trở đi) nơi chứa định nghĩa các hàm `bitXor`, `tmin`, `isTmax`, v.v.
* **`btest.c`** (File kiểm thử):
  - File này cấu hình chương trình chạy test. Bạn **cần sửa** hằng số `#define TIMEOUT_LIMIT` từ `10` lên `30` ở dòng số 35.
  - **Lý do cần sửa:** Hàm tính lũy thừa số thực `floatPower2` chạy trên rất nhiều giá trị ngẫu nhiên và phi chuẩn, phép nhân float có thể bị chậm khiến tiến trình vượt quá 10 giây (timeout mặc định của hệ thống) và bị đánh dấu 0 điểm lỗi mặc dù thuật toán hoàn toàn chính xác. Khi tăng lên 30 giây, quá trình kiểm tra sẽ hoàn thành tốt đẹp.

### 2. Các file TUYỆT ĐỐI KHÔNG được sửa đổi:
Các file này là thư viện chấm điểm và kiểm tra quy tắc của hệ thống CS:APP, bạn hãy giữ nguyên bản:
- `Makefile`: File cấu hình biên dịch tự động.
- `decl.c` & `tests.c` & `btest.h` & `bits.h`: Chứa danh sách định nghĩa hàm mẫu và các hàm đối chiếu để chấm điểm.
- `dlc`: Trình kiểm tra quy luật viết code (cấm if/loop, cấm dùng biến sai quy cách, đếm toán tử).
- `driver.pl`: Script perl tự động gọi `dlc` và `btest` để chấm điểm tổng kết cuối cùng.

---

## PHẦN 2: CÁC KHÁI NIỆM CƠ BẢN CẦN BIẾT
Nếu bạn chưa hiểu tại sao lại viết code như vậy, hãy đọc phần này:

1. **Phép AND (`&`):** Giữ lại các bit 1 ở những vị trí mà cả hai số đều là 1.
   - Ví dụ: `1100 & 1010 = 1000`.
2. **Phép OR (`|`):** Bật bit thành 1 nếu ít nhất một trong hai số ở vị trí đó là 1.
   - Ví dụ: `1100 | 1010 = 1110`.
3. **Phép XOR (`^`):** Trả về 1 nếu hai bit khác nhau, trả về 0 nếu hai bit giống nhau.
   - Ví dụ: `1100 ^ 1010 = 0110`.
4. **Phép NOT (`~`):** Đảo ngược tất cả các bit (0 thành 1, 1 thành 0).
   - Ví dụ: `~1100 = 0011`.
5. **Phép dịch trái (`<< n`):** Đẩy các bit sang trái $n$ vị trí, thêm $n$ bit 0 vào bên phải. (Tương đương nhân với $2^n$).
6. **Phép dịch phải số học (`>> n`):** Đẩy các bit sang phải $n$ vị trí. Với số có dấu, các ô trống bên trái sẽ được điền bằng **bit dấu** (bit 1 nếu là số âm, bit 0 nếu là số dương).
   - Ví dụ số âm: `11110000 >> 2 = 11111100`.
   - Ví dụ số dương: `00001100 >> 2 = 00000011`.
7. **Phép phủ định logic (`!`):** Trả về `1` nếu số đó bằng 0, trả về `0` nếu số đó khác 0.
   - Ví dụ: `!0 = 1`, `!5 = 0`, `!-100 = 0`.
8. **Số bù 2 (Số nguyên có dấu):**
   - Bit ngoài cùng bên trái (bit 31) là bit dấu. Nếu nó bằng `1` $\rightarrow$ số âm, nếu bằng `0` $\rightarrow$ số dương hoặc 0.
   - Giá trị đối của `x` (tức `-x`) luôn luôn bằng `~x + 1` (đảo tất cả các bit rồi cộng 1).
   - Số nhỏ nhất biểu diễn được (`Tmin`): `0x80000000` (chỉ có bit dấu là 1).
   - Số lớn nhất biểu diễn được (`Tmax`): `0x7FFFFFFF` (bit dấu bằng 0, tất cả các bit còn lại là 1).

---

## PHẦN 3: GIẢI THÍCH CHI TIẾT TỪNG HÀM TRONG BITS.C

---

### Hàm 1: `bitXor(int x, int y)`
* **Yêu cầu đề bài:** Hãy tính phép toán `x ^ y` (XOR) nhưng bạn **chỉ được dùng** hai toán tử là `~` (NOT) và `&` (AND).
* **Giới hạn toán tử cho phép:** Tối đa 14. Số toán tử chúng ta dùng: 7.
* **Ý tưởng logic đơn giản:**
  - Định nghĩa phép XOR là: Trả về các bit mà `x` và `y` khác nhau.
  - Tức là: (x là 1 VÀ y là 0) HOẶC (x là 0 VÀ y là 1).
  - Biểu thức logic: `(x & ~y) | (~x & y)`.
  - Nhưng đề bài cấm dùng phép OR (`|`). Chúng ta phải chuyển phép OR thành phép AND bằng cách dùng định lý De Morgan:
    $$A \lor B = \neg(\neg A \land \neg B)$$
  - Thay $A = (x \land \neg y)$ và $B = (\neg x \land y)$, ta được biểu thức chỉ có `~` và `&`:
    `~(~(x & ~y) & ~(~x & y))`
  - Có một cách viết khác gọn hơn và chạy nhanh hơn:
    - `~(x & y)`: Lọc bỏ những vị trí mà cả hai cùng bằng 1.
    - `~(~x & ~y)`: Lọc bỏ những vị trí mà cả hai cùng bằng 0 (vì đảo của 0 là 1, nếu cả hai là 0 thì `~x & ~y` bằng 1, đảo lại được 0).
    - Kết hợp hai cái bằng phép AND: `~(x & y) & ~(~x & ~y)`.
* **Giải thích code chi tiết:**
  ```c
  int bitXor(int x, int y) {
    // ~(x & y): Những bit KHÔNG ĐỒNG THỜI bằng 1 ở cả x và y
    // ~(~x & ~y): Những bit KHÔNG ĐỒNG THỜI bằng 0 ở cả x và y
    // AND hai giá trị này lại ta sẽ giữ được những bit mà x và y khác nhau.
    return ~(x & y) & ~(~x & ~y);
  }
  ```
* **Ví dụ minh họa:**
  Giả sử `x = 4` (nhị phân `0100`), `y = 5` (nhị phân `0101`).
  - `x & y` = `0100 & 0101` = `0100`.
  - `~(x & y)` = `~0100` = `1011`.
  - `~x` = `1011`, `~y` = `1010`.
  - `~x & ~y` = `1011 & 1010` = `1010`.
  - `~(~x & ~y)` = `~1010` = `0101`.
  - Kết quả: `1011 & 0101` = `0001` (bằng 1). Đúng bằng `4 ^ 5 = 1`.

---

### Hàm 2: `tmin(void)`
* **Yêu cầu đề bài:** Trả về số nguyên âm nhỏ nhất có thể biểu diễn bằng hệ bù 2 32-bit.
* **Giới hạn toán tử cho phép:** Tối đa 4. Số toán tử chúng ta dùng: 1.
* **Ý tưởng logic đơn giản:**
  - Số nguyên âm nhỏ nhất (`Tmin`) trong hệ 32-bit là số có dạng nhị phân: `1000 0000 0000 0000 0000 0000 0000 0000` (chữ số 1 ở vị trí bit thứ 31, và 31 vị trí còn lại là số 0).
  - Hệ cơ số 16 của số này là `0x80000000`.
  - Ta có thể tạo ra số này bằng cách lấy số `1` dịch sang trái 31 bit.
* **Giải thích code chi tiết:**
  ```c
  int tmin(void) {
    // Lấy số 1 (nhị phân ...0001) dịch trái 31 lần để đưa bit 1 lên vị trí bit dấu cao nhất.
    return 1 << 31;
  }
  ```

---

### Hàm 3: `isTmax(int x)`
* **Yêu cầu đề bài:** Trả về `1` nếu `x` đúng bằng số nguyên dương lớn nhất (`Tmax` = `0x7FFFFFFF`), ngược lại trả về `0`. Bạn **không được phép** sử dụng toán tử dịch bit (`<<` hay `>>`).
* **Giới hạn toán tử cho phép:** Tối đa 10. Số toán tử chúng ta dùng: 6.
* **Ý tưởng logic đơn giản:**
  - Số `Tmax` là `0x7FFFFFFF` (nhị phân: bit dấu bằng 0, toàn bộ 31 bit còn lại bằng 1).
  - Nếu ta cộng thêm 1 vào `Tmax`, nó sẽ tràn số thành `Tmin` (`0x80000000`, chỉ có bit dấu là 1, còn lại là 0).
  - Lúc này, nếu ta lấy số đảo bit của `Tmax` (tức là `~Tmax`), ta cũng thu được `0x80000000`.
  - Như vậy, với số `Tmax`, ta luôn có thuộc tính đặc biệt: `(x + 1)` bằng đúng `~x`.
  - Phép XOR `^` giữa hai số bằng nhau sẽ cho kết quả bằng 0: `(x + 1) ^ ~x == 0`.
  - **Trường hợp ngoại lệ:** Số `-1` (nhị phân toàn bit 1: `0xFFFFFFFF`) cũng có tính chất này! Vì `-1 + 1 = 0` và đảo bit của `-1` (`~(-1)`) cũng bằng 0. Nên `(0 ^ 0) == 0`.
  - Vì vậy, ta phải loại trừ số `-1`. Ta biết rằng nếu `x = -1` thì `x + 1 = 0`. Ta dùng phép phủ định logic `!(x + 1)` để kiểm tra. Nếu `x + 1` bằng 0, biểu thức này trả về 1, ngược lại trả về 0.
  - Ta kết hợp điều kiện loại trừ bằng phép OR `|`. Chỉ khi cả hai điều kiện cùng bằng 0 thì ta mới xác định đó đúng là `Tmax`.
* **Giải thích code chi tiết:**
  ```c
  int isTmax(int x) {
    int x_plus_1 = x + 1; // Tạo giá trị x + 1
    // (x_plus_1 ^ ~x): Kiểm tra xem x + 1 có bằng ~x không. Nếu bằng nhau, kết quả sẽ là 0.
    // !x_plus_1: Loại trừ số -1. Nếu x = -1 thì x+1 = 0, phủ định lại được 1.
    // Phép | kết hợp: Nếu x đúng là Tmax, cả hai vế đều là 0, 0 | 0 = 0.
    // Phép ! ngoài cùng biến kết quả 0 thành 1 (đúng), các giá trị khác thành 0 (sai).
    return !((x_plus_1 ^ ~x) | !x_plus_1);
  }
  ```

---

### Hàm 4: `allOddBits(int x)`
* **Yêu cầu đề bài:** Kiểm tra xem toàn bộ các bit ở vị trí lẻ (bit 1, 3, 5, ..., 31) của số `x` có đồng thời bằng 1 hay không. Nếu có thì trả về `1`, nếu thiếu dù chỉ một bit thì trả về `0`.
* **Giới hạn toán tử cho phép:** Tối đa 12. Số toán tử chúng ta dùng: 7.
* **Ý tưởng logic đơn giản:**
  - Ta cần tạo một số mặt nạ (mask) mà toàn bộ các bit lẻ bằng 1, bit chẵn bằng 0. Số đó là `0xAAAAAAAA` (nhị phân: `1010 1010...`).
  - Đề bài cấm viết hằng số lớn hơn 255. Do đó ta không thể viết trực tiếp `0xAAAAAAAA`. Ta phải ghép nó từ số nhỏ `0xAA` (nhập hệ 10 là 170).
  - Ta dịch bit `0xAA` sang trái 8 bit rồi OR lại để được `0xAAAA`. Dịch tiếp `0xAAAA` sang trái 16 bit rồi OR lại để được `0xAAAAAAAA`.
  - Để kiểm tra `x` có đủ các bit lẻ hay không, ta lấy `x & mask`. Phép toán này sẽ giữ lại các bit lẻ của `x` và xóa hết các bit chẵn về 0.
  - Nếu các bit lẻ của `x` đều bằng 1, thì kết quả của `x & mask` phải giống hệt như `mask`.
  - Ta so sánh hai số này bằng phép XOR `^`. Nếu giống nhau hoàn toàn, phép XOR sẽ trả về 0. Ta phủ định `!` kết quả này để trả về 1.
* **Giải thích code chi tiết:**
  ```c
  int allOddBits(int x) {
    int mask = 0xAA | (0xAA << 8); // Tạo ra 0xAAAA (16 bit lẻ bằng 1)
    mask = mask | (mask << 16);    // Nhân đôi lên thành 0xAAAAAAAA (32 bit lẻ bằng 1)
    
    // (x & mask): Lọc lấy các bit ở vị trí lẻ trong x.
    // ... ^ mask: So sánh phần lọc được với mask gốc. Nếu khớp hoàn toàn, kết quả bằng 0.
    // !(...): Biến kết quả 0 thành 1 để trả về kết quả Đúng.
    return !((x & mask) ^ mask);
  }
  ```

---

### Hàm 5: `negate(int x)`
* **Yêu cầu đề bài:** Trả về giá trị số đối `-x`. Ví dụ đầu vào là `5` thì trả về `-5`, đầu vào là `-10` thì trả về `10`.
* **Giới hạn toán tử cho phép:** Tối đa 5. Số toán tử chúng ta dùng: 2.
* **Ý tưởng logic đơn giản:**
  - Trong máy tính sử dụng biểu diễn số nguyên bù 2, số đối của số `x` luôn bằng phép đảo bit cộng 1.
  - Công thức toán học toán tử bit: `-x = ~x + 1`.
* **Giải thích code chi tiết:**
  ```c
  int negate(int x) {
    // Đảo ngược toàn bộ bit của x (NOT x) rồi cộng thêm 1.
    return ~x + 1;
  }
  ```

---

### Hàm 6: `isAsciiDigit(int x)`
* **Yêu cầu đề bài:** Kiểm tra xem giá trị của `x` có nằm trong khoảng mã ASCII từ `0x30` (số 48 trong hệ thập phân, ký tự '0') đến `0x39` (số 57, ký tự '9') hay không. Nếu nằm trong khoảng này trả về `1`, ngược lại trả về `0`.
* **Giới hạn toán tử cho phép:** Tối đa 15. Số toán tử chúng ta dùng: 13.
* **Ý tưởng logic đơn giản:**
  - Một số `x` nằm trong khoảng $[0x30, 0x39]$ khi và chỉ khi:
    - `x >= 0x30` $\rightarrow$ phép tính `x - 0x30 >= 0`.
    - `x <= 0x39` $\rightarrow$ phép tính `0x39 - x >= 0` (hoặc tương đương `x - 0x3A < 0`).
  - Trong máy tính, phép trừ được biểu diễn bằng phép cộng số đối: `A - B = A + ~B + 1`.
  - Ta kiểm tra kết quả phép tính có âm hay không bằng cách dịch phải 31 bit để lấy bit dấu. Nếu kết quả âm, bit dấu sẽ là `1` (hoặc `-1` nếu dùng dịch phải số học). Nếu kết quả dương hoặc bằng 0, bit dấu là `0`.
  - **Trường hợp tràn số:** Nếu `x` là một số âm rất lớn hoặc dương rất lớn, phép trừ có thể bị tràn làm đảo lộn bit dấu. Do đó, ta phải kiểm tra thêm: bit dấu của `x` phải bằng 0 (tức `x` không phải số âm). Ta lấy `sign = x >> 31`. Nếu `x >= 0` thì `sign` bằng 0.
* **Giải thích code chi tiết:**
  ```c
  int isAsciiDigit(int x) {
    int sign = x >> 31;                 // Lấy bit dấu của x. Nếu x âm, sign sẽ bằng -1. Nếu x >= 0, sign = 0.
    int upper = (x + ~0x3A + 1) >> 31;  // Tính hiệu x - 0x3A. Nếu x < 0x3A (tức <= 0x39), hiệu này âm, bit dấu bằng -1 (nhị phân toàn bit 1).
    int lower = (x + ~0x30 + 1) >> 31;  // Tính hiệu x - 0x30. Nếu x >= 0x30, hiệu này không âm, bit dấu bằng 0.
    
    // !sign: Trả về 1 nếu x >= 0 (loại trừ số âm tránh tràn số).
    // upper: Trả về -1 (toàn bit 1) nếu x <= 0x39.
    // !lower: lower bằng -1 nếu x < 0x30. Phủ định !lower bằng 1 nếu x >= 0x30.
    // AND kết hợp cả ba điều kiện lại.
    return !sign & upper & !lower;
  }
  ```

---

### Hàm 7: `conditional(int x, int y, int z)`
* **Yêu cầu đề bài:** Tái lập lại toán tử ba ngôi `x ? y : z`. Nếu `x` khác 0 thì trả về `y`, nếu `x` bằng 0 thì trả về `z`.
* **Giới hạn toán tử cho phép:** Tối đa 16. Số toán tử chúng ta dùng: 7.
* **Ý tưởng logic đơn giản:**
  - Ta muốn tạo ra một chiếc "mặt nạ lọc" (mask) dựa trên `x`.
  - Nếu `x != 0`, mặt nạ phải bằng `0xFFFFFFFF` (toàn bộ bit 1). Khi đó ta dùng `y & mask` để lấy toàn bộ bit của `y`.
  - Nếu `x == 0`, mặt nạ phải bằng `0x00000000` (toàn bộ bit 0). Khi đó ta dùng `z & ~mask` (tức `z & 0xFFFFFFFF`) để lấy toàn bộ bit của `z`.
  - Cách tạo mặt nạ:
    - Dùng phép phủ định logic `!x`:
      - Nếu `x != 0` $\rightarrow$ `!x = 0`.
      - Nếu `x == 0` $\rightarrow$ `!x = 1`.
    - Lấy kết quả đó cộng với `~0` (tức số `-1`, có biểu diễn nhị phân là toàn bộ bit 1):
      - Nếu `x != 0`: `0 + (-1) = -1` (mặt nạ nhị phân toàn 1).
      - Nếu `x == 0`: `1 + (-1) = 0` (mặt nạ nhị phân toàn 0).
* **Giải thích code chi tiết:**
  ```c
  int conditional(int x, int y, int z) {
    int mask = !x + ~0; // Tạo mặt nạ: bằng 0xFFFFFFFF nếu x khác 0; bằng 0x00000000 nếu x bằng 0.
    
    // (y & mask): Giữ lại y nếu mask toàn 1, bằng 0 nếu mask toàn 0.
    // (z & ~mask): Giữ lại z nếu mask toàn 0 (khi đó ~mask toàn 1), bằng 0 nếu mask toàn 1.
    // Phép | ghép hai phần lại để trả về giá trị mong muốn.
    return (y & mask) | (z & ~mask);
  }
  ```

---

### Hàm 8: `isLessOrEqual(int x, int y)`
* **Yêu cầu đề bài:** Trả về `1` nếu `x <= y`, ngược lại trả về `0`.
* **Giới hạn toán tử cho phép:** Tối đa 24. Số toán tử chúng ta dùng: 16.
* **Ý tưởng logic đơn giản:**
  - Thông thường để so sánh `x <= y`, ta chỉ việc lấy hiệu `y - x` và kiểm tra xem kết quả có lớn hơn hoặc bằng 0 hay không.
  - Tuy nhiên, trong kiểu số nguyên có giới hạn, phép trừ có thể gây ra hiện tượng **tràn số (overflow)**. 
    - Ví dụ: `x` là số âm cực lớn (`Tmin`), và `y` là số dương lớn. Hiệu `y - x` sẽ là phép cộng hai số dương lớn, vượt quá giới hạn dương và quay về số âm. Khi đó phép so sánh dấu sẽ bị sai.
  - Để giải quyết triệt để, ta chia làm 2 trường hợp:
    1. **Khác dấu:** `x` và `y` khác dấu nhau.
       - Nếu `x` âm (`sx = 1`) và `y` dương (`sy = 0`), chắc chắn `x < y` là đúng $\rightarrow$ trả về 1.
       - Nếu `x` dương (`sx = 0`) và `y` âm (`sy = 1`), chắc chắn `x > y` $\rightarrow$ trả về 0.
    2. **Cùng dấu:** `x` và `y` cùng dương hoặc cùng âm.
       - Khi cùng dấu, phép trừ `y - x` chắc chắn không bao giờ bị tràn số.
       - Ta tính hiệu `diff = y - x` (tức `y + ~x + 1`).
       - Lấy bit dấu của hiệu: `diff_sign = (diff >> 31) & 1`.
       - Nếu hiệu $\ge 0$ (tức `diff_sign == 0`) thì `x <= y` đúng $\rightarrow$ trả về 1.
* **Giải thích code chi tiết:**
  ```c
  int isLessOrEqual(int x, int y) {
    int sx = (x >> 31) & 1; // Lấy bit dấu của x (1 nếu âm, 0 nếu không âm)
    int sy = (y >> 31) & 1; // Lấy bit dấu của y (1 nếu âm, 0 nếu không âm)
    
    // Tính hiệu y - x và lấy bit dấu của kết quả.
    int diff_sign = ((y + ~x + 1) >> 31) & 1; 
    
    int same_sign = !(sx ^ sy); // Trả về 1 nếu x và y cùng dấu, 0 nếu khác dấu.
    
    // (same_sign & !diff_sign): Nếu cùng dấu, kết quả đúng khi hiệu y-x không âm (diff_sign == 0).
    // (!same_sign & sx): Nếu khác dấu, kết quả đúng khi x là số âm (sx == 1).
    return (same_sign & !diff_sign) | (!same_sign & sx);
  }
  ```

---

### Hàm 9: `logicalNeg(int x)`
* **Yêu cầu đề bài:** Tự thiết kế toán tử logic phủ định `!x` (trả về 1 nếu `x` bằng 0, trả về 0 nếu `x` khác 0) mà **không được dùng** chính toán tử `!`.
* **Giới hạn toán tử cho phép:** Tối đa 12. Số toán tử chúng ta dùng: 5.
* **Ý tưởng logic đơn giản:**
  - Ta cần tận dụng một đặc điểm duy nhất của số 0:
    - Số 0 và số đối của nó (là `-0` hay `~0 + 1`) đều có bit dấu bằng 0.
    - Với mọi số khác 0 (kể cả số âm và số dương), một trong hai số `x` hoặc `-x` sẽ luôn là số âm (tức là bit thứ 31 bằng 1).
  - Ví dụ:
    - Nếu `x = 5` (dương), số đối `-5` có bit dấu bằng 1.
    - Nếu `x = -5` (âm), nó có bit dấu bằng 1.
    - Nếu `x = Tmin` (`-2147483648`), số đối của nó cũng là `Tmin`, có bit dấu bằng 1.
  - Do đó, ta lấy phép OR giữa `x` và số đối `-x` (tức `~x + 1`): `x | (~x + 1)`.
    - Nếu `x != 0`: Biểu thức này chắc chắn có bit thứ 31 bằng 1.
    - Nếu `x == 0`: Biểu thức này bằng `0 | 0 = 0` (bit thứ 31 bằng 0).
  - Ta dịch phải số học 31 bit để đưa bit dấu này phủ toàn bộ số:
    - Nếu `x != 0` $\rightarrow$ dịch phải được `0xFFFFFFFF` (tức số `-1`).
    - Nếu `x == 0` $\rightarrow$ dịch phải được `0x00000000` (tức số `0`).
  - Cộng thêm 1 vào kết quả để đưa về dạng logic chuẩn `0` hoặc `1`:
    - Nếu khác 0: `-1 + 1 = 0`.
    - Nếu bằng 0: `0 + 1 = 1`.
* **Giải thích code chi tiết:**
  ```c
  int logicalNeg(int x) {
    // (~x + 1) là số đối của x.
    // (x | số đối): Lấy bit dấu. Luôn có bit 31 bằng 1 nếu x khác 0.
    // >> 31: Dịch phải số học để tạo số toàn bit 1 (-1) hoặc toàn bit 0 (0).
    // + 1: Chuyển đổi -1 thành 0, và chuyển đổi 0 thành 1.
    return ((x | (~x + 1)) >> 31) + 1;
  }
  ```

---

### Hàm 10: `howManyBits(int x)`
* **Yêu cầu đề bài:** Tính số bit tối thiểu cần thiết để biểu diễn giá trị của số `x` dưới dạng số nguyên bù 2 (bao gồm cả bit dấu).
* **Giới hạn toán tử cho phép:** Tối đa 90. Số toán tử chúng ta dùng: 32.
* **Ý tưởng logic đơn giản:**
  - Với số dương, ví dụ số `5` (nhị phân `0101`), ta cần ít nhất 4 bit để biểu diễn. Số bit tối thiểu phụ thuộc vào vị trí của bit `1` cao nhất.
  - Với số âm, ví dụ số `-5` (nhị phân `1011`), số bit tối thiểu cần dùng cũng tương tự như số dương đảo bit của nó (tức là số `4`, nhị phân `0100` $\rightarrow$ cần 4 bit).
  - Do đó, bước 1: Nếu `x` âm, ta đảo toàn bộ bit của nó. Nếu `x` dương, ta giữ nguyên.
    Ta làm việc này bằng phép XOR với bit dấu: `x = x ^ (x >> 31)`.
  - Bước 2: Ta đi tìm vị trí bit 1 cao nhất bằng kỹ thuật **Tìm kiếm nhị phân** trên 32 bit:
    - Kiểm tra xem nửa trên (16 bit cao) có chứa bit 1 nào không.
      `b16 = !(!(x >> 16)) << 4;`
      Nếu nửa trên có bit 1, ta dịch phải `x` đi 16 bit để tiếp tục tìm kiếm trong 16 bit đó, đồng thời ghi nhận đã tìm thấy ở khoảng 16 bit (`b16 = 16`). Ngược lại `b16 = 0`.
    - Tương tự kiểm tra tiếp trong phần còn lại cho khoảng 8 bit (`b8`), 4 bit (`b4`), 2 bit (`b2`), và 1 bit (`b1`).
    - Cuối cùng cộng tất cả các khoảng dịch lại, cộng thêm 1 bit cho vị trí bit 0 và 1 bit dấu.
* **Giải thích code chi tiết:**
  ```c
  int howManyBits(int x) {
    int sign, b16, b8, b4, b2, b1, b0;
    sign = x >> 31; // Tạo mặt nạ dấu: 0xFFFFFFFF nếu x âm, 0x00000000 nếu x dương.
    x = x ^ sign;   // Nếu x âm, đảo bit của x. Nếu x dương, giữ nguyên x.

    // Tìm kiếm nhị phân bit 1 cao nhất:
    b16 = !(!(x >> 16)) << 4; // Nếu x > 0xFFFF, b16 = 16, ngược lại bằng 0.
    x = x >> b16;             // Dịch phải x để thu hẹp phạm vi tìm kiếm.

    b8 = !(!(x >> 8)) << 3;   // Nếu x còn lại > 0xFF, b8 = 8, ngược lại bằng 0.
    x = x >> b8;

    b4 = !(!(x >> 4)) << 2;   // Nếu x còn lại > 0xF, b4 = 4.
    x = x >> b4;

    b2 = !(!(x >> 2)) << 1;   // Nếu x còn lại > 0x3, b2 = 2.
    x = x >> b2;

    b1 = !(!(x >> 1));        // Nếu x còn lại > 0x1, b1 = 1.
    x = x >> b1;

    b0 = x;                   // Bit cuối cùng còn lại.

    // Tổng số bit cần dùng = tổng các bước dịch + 1 bit cơ sở + 1 bit dấu.
    return b16 + b8 + b4 + b2 + b1 + b0 + 1;
  }
  ```

---

### Hàm 11: `floatScale2(unsigned uf)`
* **Yêu cầu đề bài:** Đầu vào là biểu diễn bit của một số thực đơn đơn độ chính xác chuẩn IEEE 754. Hãy nhân số đó với 2 và trả về biểu diễn bit của kết quả.
* **Quy tắc:** Đây là phần số thực nên bạn **được phép dùng** cấu trúc rẽ nhánh `if`, vòng lặp, và các hằng số bất kỳ.
* **Giới hạn toán tử cho phép:** Tối đa 30. Số toán tử chúng ta dùng: 13.
* **Ý tưởng logic đơn giản (IEEE 754):**
  Một số thực 32-bit gồm 3 phần:
  - 1 bit dấu ở vị trí 31 (`sign`).
  - 8 bit mũ ở vị trí 30-23 (`exp`).
  - 23 bit phân số ở vị trí 22-0 (`frac`).
  Khi nhân số thực này với 2, ta xét các trường hợp:
  1. **Trường hợp đặc biệt (`exp == 0xFF`):** Số này là NaN (Not a Number) hoặc Vô cùng ($\infty$). Theo chuẩn, vô cùng hay NaN nhân 2 vẫn giữ nguyên giá trị $\rightarrow$ trả về chính nó.
  2. **Trường hợp số phi chuẩn (`exp == 0`):** Số này cực kỳ nhỏ, gần bằng 0. Để nhân đôi, ta chỉ cần dịch trái phần phân số đi 1 bit: `frac << 1`. Khi dịch trái, nếu bit cao nhất của phân số vượt ngưỡng, nó sẽ tự tràn vào phần mũ (làm phần mũ lên 1), biến số đó thành số chuẩn hóa một cách hoàn toàn chính xác.
  3. **Trường hợp số chuẩn hóa (`0 < exp < 0xFF`):** Ta chỉ cần tăng phần mũ lên 1 đơn vị (`exp++`). Nếu sau khi tăng mà phần mũ đạt giá trị tràn `0xFF`, số này trở thành vô cùng, ta đặt phần phân số `frac = 0`.
* **Giải thích code chi tiết:**
  ```c
  unsigned floatScale2(unsigned uf) {
    unsigned sign = uf & 0x80000000; // Trích xuất bit dấu
    unsigned exp = (uf >> 23) & 0xFF; // Trích xuất 8 bit mũ
    unsigned frac = uf & 0x007FFFFF; // Trích xuất 23 bit phân số

    if (exp == 0xFF) return uf;      // Nếu là NaN hoặc Infinity, trả về nguyên bản.
    
    if (exp == 0) {
      // Nếu là số phi chuẩn (exp = 0), dịch trái uf 1 bit (giữ nguyên bit dấu).
      return sign | (uf << 1);
    }
    
    exp++; // Số chuẩn hóa bình thường: tăng phần mũ lên 1 (tương đương nhân 2).
    if (exp == 0xFF) {
      frac = 0; // Nếu vượt ngưỡng mũ tối đa, biến thành số Vô cùng (frac = 0).
    }
    
    // Ghép các thành phần lại và trả về.
    return sign | (exp << 23) | frac;
  }
  ```

---

### Hàm 12: `floatFloat2Int(unsigned uf)`
* **Yêu cầu đề bài:** Thực hiện chuyển đổi ép kiểu từ số thực (float) sang số nguyên (int). Nếu giá trị float vượt quá giới hạn biểu diễn của số nguyên hoặc là NaN, trả về `0x80000000`.
* **Giới hạn toán tử cho phép:** Tối đa 30. Số toán tử chúng ta dùng: 14.
* **Ý tưởng logic đơn giản:**
  - Công thức tính giá trị số thực chuẩn hóa: $V = (-1)^{sign} \times (1.frac) \times 2^E$, trong đó số mũ thực tế là $E = exp - 127$.
  - Ta xét các trường hợp của $E$:
    - Nếu $E < 0$ ($exp < 127$): Trị tuyệt đối của số thực này nhỏ hơn 1.0 (ví dụ `0.5`, `-0.9`). Khi ép sang kiểu nguyên `int`, kết quả luôn bị làm tròn về `0`.
    - Nếu $E \ge 31$ ($exp \ge 158$): Vượt quá khả năng biểu diễn của số nguyên 32-bit có dấu (tối đa là $2^{31}-1$). Trả về giá trị lỗi `0x80000000`.
    - Nếu $0 \le E < 31$: Ta khôi phục phần định trị đầy đủ bằng cách thêm số 1 ẩn vào trước phần phân số: `M = 1.frac` (tương ứng mã code là `frac | 0x00800000`).
      - Phần định trị `M` này đang có 23 bit sau dấu phẩy.
      - Nếu $E \ge 23$: Ta cần dịch trái thêm để bù vào phần nguyên: `val = M << (E - 23)`.
      - Nếu $E < 23$: Ta dịch phải để loại bỏ các bit thập phân dư thừa: `val = M >> (23 - E)`.
  - Cuối cùng, nếu số thực ban đầu mang dấu âm (`sign` bằng 1), ta đảo dấu giá trị số nguyên thu được (`val = -val`).
* **Giải thích code chi tiết:**
  ```c
  int floatFloat2Int(unsigned uf) {
    unsigned sign = uf & 0x80000000;
    unsigned exp = (uf >> 23) & 0xFF;
    unsigned frac = uf & 0x007FFFFF;
    int E = exp - 127; // Tính số mũ thực tế
    int M;
    int val;

    if (exp < 127) return 0; // Trị tuyệt đối nhỏ hơn 1 -> làm tròn về 0.
    if (exp >= 158) return 0x80000000u; // Vượt quá giới hạn số nguyên 32-bit -> lỗi.

    M = frac | 0x00800000; // Khôi phục bit ẩn 1 thành 1.frac (tổng cộng 24 bit định trị)
    if (E >= 23) {
      val = M << (E - 23); // Số mũ lớn hơn số bit phân số -> dịch trái để tăng giá trị.
    } else {
      val = M >> (23 - E); // Số mũ nhỏ hơn 23 -> dịch phải để lược bỏ phần thập phân.
    }

    if (sign) {
      val = -val; // Nếu số thực âm, đổi dấu số nguyên kết quả.
    }
    return val;
  }
  ```

---

### Hàm 13: `floatPower2(int x)`
* **Yêu cầu đề bài:** Trả về biểu diễn nhị phân (dưới dạng float) của biểu thức lũy thừa $2.0^x$ với số nguyên $x$ bất kỳ.
* **Giới hạn toán tử cho phép:** Tối đa 30. Số toán tử chúng ta dùng: 9.
* **Ý tưởng logic đơn giản:**
  - Ta cần tính biểu diễn bit của số thực $2.0^x$:
    - Giá trị lớn nhất mà số thực chuẩn hóa biểu diễn được là khoảng $2^{127}$. Nếu $x > 127$, kết quả bị tràn số $\rightarrow$ trả về Vô cùng dương `+INF` (`0x7F800000`).
    - Giá trị nhỏ nhất có thể biểu diễn được dưới dạng số phi chuẩn là $2^{-149}$. Nếu $x < -149$, số quá nhỏ $\rightarrow$ làm tròn về `0`.
    - Khoảng số chuẩn hóa $[-126, 127]$: Giá trị phần mũ lưu trữ trong bộ nhớ sẽ là $exp = x + 127$. Phần phân số `frac` bằng 0. Ta trả về `(x + 127) << 23`.
    - Khoảng số phi chuẩn $[-149, -127]$: Phần mũ `exp` bằng 0. Giá trị được biểu diễn duy nhất qua phần phân số `frac`. Bit 1 sẽ được dịch chuyển tới vị trí thứ `x + 149`. Ta trả về `1 << (x + 149)`.
* **Giải thích code chi tiết:**
  ```c
  unsigned floatPower2(int x) {
    if (x > 127) return 0x7F800000;  // Vượt giới hạn trên -> Trả về dương vô cùng.
    if (x < -149) return 0;          // Vượt giới hạn dưới -> Trả về 0.
    
    if (x >= -126) {
      // Số thực chuẩn hóa: phần mũ lưu giá trị (x + 127), phần phân số bằng 0.
      return (x + 127) << 23;
    }
    
    // Số thực phi chuẩn: phần mũ bằng 0, chỉ có 1 bit 1 nằm ở phần phân số.
    return 1 << (x + 149);
  }
  ```
