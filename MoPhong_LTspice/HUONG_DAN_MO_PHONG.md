# Hướng dẫn chạy mô phỏng PIR LHI778 và tạo tín hiệu chuyển động

Tài liệu này chỉ nêu **các bước thao tác** trong LTspice:

- chạy mô phỏng;
- xem kết quả;
- tạo và sửa tín hiệu đầu vào (tín hiệu mô phỏng chuyển động).

Toàn bộ mô phỏng nằm trong một file: `PIR_LHI778.asc`.

---

## 1. Chuẩn bị

- [ ] Đã cài LTspice (bản 24 trở lên).
- [ ] Thư mục `MoPhong_LTspice` có đủ các file dưới đây.

| File | Dùng để
|---|---|
| `PIR_LHI778.asc` | Sơ đồ mạch và các lệnh mô phỏng — **file cần mở** |
| `PIR_LHI778.plt` | Cấu hình đồ thị, LTspice tự nạp sau khi chạy. Để cùng thư mục với file `.asc` |
| `.raw`, `.log`, `.net`, `.db`, `.op.raw` | File kết quả, tự tạo lại mỗi lần chạy. Xóa đi cũng được |
| `ket_qua/*.svg` | Đồ thị đã xuất sẵn |

---

## 2. Chạy mô phỏng theo thời gian

1. Nhấp đúp vào `PIR_LHI778.asc` (hoặc trong LTspice chọn **File → Open**).
2. Chọn **Simulate → Run** (hoặc bấm nút hình người chạy).
3. Đợi khoảng 15–30 s.
4. Cửa sổ đồ thị tự mở, gồm 4 ô:

   | Ô | Tín hiệu |
   |---|---|
   | 1 | `V(pa)`, `V(pb)` — tín hiệu vào của phần tử A và B |
   | 2 | `V(s)` — chân S của LHI778 |
   | 3 | `V(a2)`, `V(a3)` — trước và sau mạch lọc |
   | 4 | `V(out)` — đầu ra |

5. Xem số đo tự động: **View → SPICE Error Log** (Ctrl+L).

Với thông số mặc định, log phải có các giá trị gần như sau:

| Số đo | Giá trị |
|---|---|
| `VS_DC` | 0.809 V |
| `VOUT_NGHI` | 1.499 V |
| `VOUT_MAX` / `VOUT_MIN` | 2.55 V / 0.32 V |
| `NHIEU_TRUOC_LOC` / `NHIEU_SAU_LOC` | 19 mVpp / 4.2 mVpp |
| `SK1_PP` … `SK5_PP` | 1.27 / 2.23 / 0.54 / 0.07 / 0.67 V |
| `VOUT_CUOI` | 1.505 V |

---

## 3. Xem thêm tín hiệu

| Muốn | Làm |
|---|---|
| Vẽ điện áp một nút | Mở cửa sổ đồ thị, rồi bấm vào dây đó trên schematic |
| Vẽ dòng qua linh kiện | Bấm vào thân linh kiện |
| Vẽ biểu thức | Chuột phải vào đồ thị → **Add Traces** → gõ, ví dụ `V(out)-1.5` |
| Phóng to một đoạn | Kéo chuột quét vùng cần xem trên đồ thị |
| Xóa một đường | Chuột phải vào tên đường ở đầu ô đồ thị → **Delete** |

Tên các nút có thể đo:

| Nút | Vị trí |
|---|---|
| `PA`, `PB` | Tín hiệu vào của phần tử A, B (1 V = 1 µW) |
| `TA`, `TB` | Nhiệt độ phần tử (1 V = 1 K) |
| `G`, `M`, `S` | Bên trong LHI778 và chân ra S |
| `SN` | Chân S sau khi cộng nhiễu |
| `A1`, `A2`, `A3` | Ra tầng khuếch đại 1, tầng 2, mạch lọc |
| `OUT` | Đầu ra cuối |
| `VDD`, `VREF` | Nguồn 3.3 V, điện áp tham chiếu 1.65 V |

> Chỉ các nút ở trên được lưu (lệnh `.save`). Muốn đo nút khác: xóa dòng `.save` trong khối **MO PHONG**, rồi chạy lại.

---

## 4. Chạy phân tích tần số (AC)

1. Chuột phải vào dòng `.tran 0 40 0 0.1m` → chọn **Comment** → OK.
2. Chuột phải vào dòng `.ac dec 100 0.01 1k` → chọn **SPICE directive** → OK.
3. **Simulate → Run** (chạy dưới 1 s).
4. **Add Traces**, gõ:
   - `V(s)*1e6` — độ nhạy cảm biến [V/W];
   - `V(out)/V(s)` — hệ số khuếch đại của mạch [dB].
5. Xem số đo `R_1HZ`, `K_1HZ`, `K_MAX`, `K_50HZ`, `K_1KHZ` trong SPICE Error Log.
6. Làm xong, đổi ngược lại: `.tran` → **SPICE directive**, `.ac` → **Comment**.

---

## 5. Tạo tín hiệu đầu vào (mô phỏng chuyển động)

### 5.1 Chỗ cần sửa

- Mọi tín hiệu vào nằm trong khối chữ **"KICH BAN CHUYEN DONG"** trên schematic.
- Không cần sửa dây hay linh kiện.
- Cách sửa: chuột phải vào khối chữ → sửa trong hộp soạn thảo → OK → chạy lại.

### 5.2 Các hàm tạo tín hiệu có sẵn

| Hàm | Tạo ra | Tham số |
|---|---|---|
| `gp(x,c,w)` | 1 xung | `c` = thời điểm đỉnh [s], `w` = độ rộng [s] |
| `walk(x,s,T,w,n)` | `n` xung cách đều (n tối đa 4) | `s` = xung đầu [s], `T` = khoảng cách giữa 2 xung [s], `w` = độ rộng [s] |
| `sstep(x,c,d)` | Bước nhảy từ 0 lên 1 | `c` = thời điểm [s], `d` = thời gian tăng [s] |

Tín hiệu vào phần tử A là hàm `IRA(x)`, của phần tử B là `IRB(x)`. Hai hàm này cộng các kịch bản lại với nhau.

### 5.3 Năm kịch bản có sẵn

| Kịch bản | Thời gian | Mô tả | Dòng tham số |
|---|---|---|---|
| SK1 | 3–6.6 s | Người đi bình thường (~3 m) | `.param P1=0.13 T1=0.8 W1=0.15 S1=3` |
| SK2 | 10–12 s | Người đi nhanh / ở gần (~1.5 m) | `.param P2=0.35 T2=0.5 W2=0.09 S2=10` |
| SK3 | 15–18.5 s | Người đi chậm / ở xa (~6 m) | `.param P3=0.04 T3=1.4 W3=0.25 S3=15` |
| SK4 | 22–26 s | Bức xạ nền tăng đều trên cả 2 phần tử | `.param P4=2 S4=22 END4=26 RT4=1` |
| SK5 | từ 31 s | Vật nóng xuất hiện rồi đứng yên | `.param P5=0.12 S5=31 RT5=0.3` |

Tham số dùng chung:

| Tham số | Nghĩa | Đơn vị |
|---|---|---|
| `P` | Biên độ tín hiệu | µW |
| `T` | Khoảng cách giữa 2 xung | s |
| `W` | Độ rộng xung | s |
| `S` | Thời điểm bắt đầu | s |

### 5.4 Sửa một kịch bản

| Muốn | Sửa |
|---|---|
| Tín hiệu mạnh hơn (người ở gần) | Tăng `P` |
| Tín hiệu yếu hơn (người ở xa) | Giảm `P` |
| Đi nhanh hơn | Giảm `T` và `W` (giữ `W ≈ 0.2 × T`) |
| Đi chậm hơn | Tăng `T` và `W` |
| Đổi thời điểm xảy ra | Đổi `S` |
| Tắt kịch bản | Đặt `P = 0` |

Ví dụ, cho SK1 mạnh và nhanh hơn, sửa:

```
.param P1=0.13 T1=0.8 W1=0.15 S1=3
```

thành:

```
.param P1=0.25 T1=0.6 W1=0.12 S1=3
```

> Giữ `P` ≤ khoảng 0.3 µW với các kịch bản đi bộ, để `V(out)` không chạm 0 V hoặc 3 V.

### 5.5 Thêm kịch bản mới — ví dụ: người đi ngược chiều

1. Thêm một dòng tham số, đặt sau dòng `P5`:

   ```
   .param P6=0.13 T6=0.8 W6=0.15 S6=41  ; SK6: nguoi di nguoc chieu
   ```

2. Trong dòng `.func IRA(x) {...}`, thêm vào **cuối**, ngay trước dấu `}`:

   ```
   +P6*walk(x,S6+T6/2,T6,W6,4)
   ```

3. Trong dòng `.func IRB(x) {...}`, thêm vào **đầu**, ngay sau dấu `{`:

   ```
   P6*walk(x,S6,T6,W6,4)+
   ```

4. Kéo dài thời gian mô phỏng: sửa `.tran 0 40 0 0.1m` thành `.tran 0 48 0 0.1m`.
5. (Tùy chọn) Thêm số đo vào khối **DO LUONG**:

   ```
   .meas TRAN SK6_PP PP V(OUT) FROM 40 TO 48
   ```

6. Chạy lại. Kết quả đúng: `SK6_PP` ≈ 1.28 V, và dạng sóng ở 41–44 s ngược dấu so với SK1.

Muốn thêm kịch bản **đi xuôi chiều** thì đảo lại: `+P6*walk(x,S6,T6,W6,4)` cho `IRA`, và `P6*walk(x,S6+T6/2,T6,W6,4)+` cho `IRB`.

### 5.6 Tín hiệu thử hình sin (tùy chọn)

1. Chuột phải vào khối **KICH BAN CHUYEN DONG**.
2. Thêm dấu `*` vào đầu hai dòng `.func IRA` và `.func IRB` gốc (biến chúng thành chú thích).
3. Thêm hai dòng:

   ```
   .func IRA(x) {0.05*(1+sin(2*pi*1*x))}
   .func IRB(x) {0.05*(1-sin(2*pi*1*x))}
   ```

4. Chạy lại. Kết quả đúng: `V(out)` dao động khoảng 1.06 Vpp quanh 1.5 V, tần số 1 Hz.
5. Đổi số `1` trong `2*pi*1*x` (tần số, Hz) thành `0.3`, `3`, `10`… để thử tần số khác.
6. Làm xong: xóa hai dòng mới và bỏ dấu `*` ở hai dòng gốc.

### 5.7 Quy tắc khi viết biểu thức

- Không dùng `**` (lũy thừa). Viết `(x-c)*(x-c)` thay cho `(x-c)**2`.
- Không đặt tên tham số là `tr`.
- Với kịch bản đi bộ: một hàm dùng `S`, hàm còn lại dùng `S+T/2` (xem mục 5.5).
- Các kịch bản cách nhau ít nhất 4–5 s.
- Đổi thời điểm kịch bản thì sửa luôn `FROM … TO …` của lệnh `.meas` tương ứng.

---

## 6. Các thông số khác có thể đổi

| Muốn | Sửa | Mặc định |
|---|---|---|
| Tắt nhiễu | `NOISE=0` (khối **NHIEU**) | `1` |
| Đổi biên độ nhiễu 50 Hz / 1 kHz / nhiễu trắng | `A50`, `AHF`, `AWN` | `50u`, `50u`, `10u` |
| Đổi độ lệch 2 phần tử | `MISM` (khối **THONG SO MO HINH LHI778**) | `0.01` |
| Đổi giá trị linh kiện | Chuột phải vào con số cạnh linh kiện → gõ giá trị mới | — |
| Chạy nhanh hơn | `NOISE=0` và sửa `.tran 0 40 0 0.1m` thành `.tran 0 40 0 1m` | — |

---

## 7. Khi gặp sự cố

| Hiện tượng | Cách xử lý |
|---|---|
| Báo lỗi `Expected ")" here` hoặc `Unknown parameter` | Kiểm tra dấu ngoặc và tên hàm / tham số ở dòng vừa sửa |
| Mô phỏng treo, báo `Time step too small` | Bỏ `**` trong biểu thức, viết dạng nhân |
| `V(s)` khác xa 0.81 V | Kiểm tra dòng `.options gmin=1e-15` còn trong khối **MO PHONG** |
| `V(out)` dính sát 0 V hoặc 3 V | Giảm `P` của kịch bản đó |
| Bấm vào dây không ra đồ thị | Xóa dòng `.save`, chạy lại |
| Không tự hiện 4 ô đồ thị | Để `PIR_LHI778.plt` cùng thư mục với `PIR_LHI778.asc` |
| Số đo `.meas` báo lỗi | Sửa `FROM … TO …` cho khớp thời gian kịch bản |
| Log có `Length shorter than recommended…` hoặc `Source stepping succeeded` | Bình thường, bỏ qua |
