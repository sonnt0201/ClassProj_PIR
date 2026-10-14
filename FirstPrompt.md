# Project cảm biến và xử lý tín hiệu đo

## Mô phỏng

Tạo  1 project LTSpice mô phỏng mạch cảm biến cho LHI778 (PIR) bao gồm:

### Mô phỏng đầu thu cảm biến LHI778

Thiết kế mạch mô phỏng nguyên lý cho LHI778 bao gồm đầu thu và các chân đầu ra

Đồng thời mô phỏng tín hiệu đầu vào (tín hiệu chuyển động trong thực tế tác động lên đầu vào cảm biến), đảm bảo đầu ra tương ứng đúng với nguyên lý của cảm biến. 

### Mạch cảm biến cho LHI778

Sau khi đã mô phỏng được đầu thu kèm tín hiệu đầu vào, đảm bảo đầu ra đúng nguyên lý của LHI778, bạn tiến hành thiết kế mạch khuếch đại là mạch lọc cho đầu thu cảm biến này.

- Mạch khuếch đại: khuếch đại đầu ra của LHI778 sử dụng mạch khuếch đại Op Amp.

- Mạch lọc: mạch lọc thông thấp (lọc nhiễu là tần số cao biên độ nhỏ, nên sử dụng op amp).

- Nguồn cấp: 3.3V

Đầu ra của mạch khuếch đại là tín hiệu tương tự trong khoảng từ 0 -> 3V. Ở điều kiện ổn định không có chuyển động (tín hiệu vào), điện áp đầu ra giữ ở mức 1.5V.

Khi có chuyển động, tùy thuộc vào tín hiệu chuyển động, điện áp đầu ra biến đổi tương ứng trong khoảng 0 -> 3V.

### Lưu ý

Toàn bộ phần mô phỏng đầu cảm biến, tín hiệu đầu vào và mạch lọc khuếch đại nằm trong 1 project LTSpice duy nhất.

## Báo cáo

Mô tả sau, làm mô phỏng trước trong session này