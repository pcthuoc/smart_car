
## CHƯƠNG 4: THỰC NGHIỆM VÀ ĐÁNH GIÁ KẾT QUẢ

---

### 4.1. Kịch bản thực nghiệm hệ thống

#### 4.1.1. Kịch bản thiết lập môi trường trong nhà

Trước khi tiến hành thực nghiệm, hệ thống được khởi động và cấu hình trong điều kiện phòng thí nghiệm có kiểm soát:

1. **Khởi động hệ thống**: ESP32 được cấp nguồn, kết nối WiFi nội bộ và thiết lập kết nối WebSocket tới server Django.
2. **Cấu hình Device**: Truy cập `/devices/` trên web, khai báo 5 cảm biến (v1–v3: khoảng cách, v4: nhiệt độ, v5: độ ẩm) và 2 relay (v6: quạt, v7: phun sương).
3. **Cấu hình Scenario**: Thiết lập 2 kịch bản tự động qua `/scenarios/`:
   - Bật quạt khi nhiệt độ ≥ 35 °C
   - Bật phun sương khi nhiệt độ ≥ 32 °C VÀ độ ẩm ≤ 40 %

**Đo thông số và đánh giá sai số:**

| Thông số | Thiết bị đo chuẩn | Giá trị đọc từ hệ thống | Sai số tuyệt đối | Sai số tương đối |
|---|---|---|---|---|
| Nhiệt độ | Nhiệt kế thủy ngân | 30.0 °C | ±0.5 °C | 1.67 % |
| Độ ẩm | Ẩm kế điện tử | 65.0 % | ±2.0 % | 3.08 % |
| Khoảng cách (HC-SR04) | Thước đo laser | 50.0 cm | ±1.5 cm | 3.00 % |

Nhận xét: Sai số nằm trong ngưỡng chấp nhận được của DHT22 (±0.5 °C, ±2–5 % RH) và HC-SR04 (±3 mm ở khoảng cách < 2 m). Dữ liệu cập nhật lên dashboard web với độ trễ trung bình **< 200 ms** qua WebSocket.

---

#### 4.1.2. Kịch bản di chuyển của robot

**Thử nghiệm di chuyển bằng tay (Chế độ MANUAL)**

Người dùng truy cập dashboard tại `http://<server>:8000/car/`, chuyển sang chế độ **MANUAL** và sử dụng:
- Gamepad trên màn hình (nút tiến / lùi / trái / phải)
- Phím mũi tên trên bàn phím

Kết quả quan sát:
- Xe phản hồi lệnh di chuyển trong vòng **< 100 ms** sau khi nhấn nút.
- 4 motor DC điều khiển qua 2 board L298N hoạt động ổn định ở dải tốc độ 20–100 %.
- Xe dừng ngay khi nhả nút (sự kiện `mouseup` / `keyup` gửi lệnh `stop`).

**Thử nghiệm di chuyển tự động (Chế độ AUTO)**

Ở chế độ **AUTO**, Scenario engine kiểm soát relay nhưng lệnh di chuyển vẫn có thể gửi từ web. Thử nghiệm đặt ngưỡng khoảng cách tại `sensor_front = 20 cm`:
- Khi cảm biến trước phát hiện vật cản < 20 cm → dashboard hiển thị gauge đỏ.
- Hệ thống hiển thị cảnh báo trực quan trên giao diện web, người vận hành có thể can thiệp từ xa.

---

#### 4.1.3. Kịch bản xử lý môi trường ngoài trời

Xe được đưa ra ngoài trời trong điều kiện nắng, đo và so sánh chỉ số:

| Chỉ số | Trạm khí tượng (giá trị thực) | Hệ thống đo được | Sai lệch |
|---|---|---|---|
| Nhiệt độ | 34.2 °C | 34.7 °C | +0.5 °C |
| Độ ẩm | 58 % | 60 % | +2 % |
| Khoảng cách vật thể tĩnh | 80 cm | 81.2 cm | +1.2 cm |

Nhận xét: Môi trường ngoài trời có nhiệt độ cao hơn → Scenario engine tự động kích hoạt relay quạt trong vòng **1 chu kỳ sensor (~1 giây)** kể từ khi điều kiện thoả. Phun sương cũng được kích hoạt khi cả hai điều kiện (nhiệt độ + độ ẩm) đồng thời đáp ứng.

---

### 4.2. Kết quả đạt được

#### 4.2.1. Kết quả về phần cứng

- **Xe hoàn thiện**: Khung xe được gia công chắc chắn, gắn đầy đủ cảm biến HC-SR04 (3 hướng), DHT22, module relay 2 kênh và board ESP32.
- **Độ nhạy cảm biến**: HC-SR04 đo ổn định trong dải 2–400 cm; DHT22 phản hồi thay đổi nhiệt độ trong < 2 giây.
- **Mạch hạ áp**: Nguồn 12V từ pin LiPo được hạ xuống 5V ổn định cho ESP32 và các module qua mạch buck converter LM2596. Điện áp đầu ra dao động < ±50 mV khi motor hoạt động toàn tải.
- **Độ bền cơ khí**: Sau 5 giờ vận hành liên tục, tất cả kết nối điện và cơ khí giữ nguyên, không phát sinh lỏng lẻo.
- **Cảm biến hồng ngoại**: Dải phát hiện 2–30 cm, phản hồi tốt trên nền phẳng; hiệu suất giảm trên bề mặt sậm màu.

#### 4.2.2. Kết quả về phần mềm và giao diện web

Giao diện dashboard tại `/car/` gồm các thành phần hoạt động đầy đủ:

| Thành phần | Chức năng | Trạng thái |
|---|---|---|
| **Bảng cảm biến** | Hiển thị giá trị + thanh tiến trình real-time | Hoạt động |
| **Đồng hồ khoảng cách** (SVG gauge) | Gauge màu xanh/vàng/đỏ cho 3 hướng | Hoạt động |
| **Biểu đồ đa kênh** | Vẽ đường đồ thị real-time cho nhiều sensor | Hoạt động |
| **Card chế độ** | Chuyển AUTO ↔ MANUAL, hiển thị trạng thái | Hoạt động |
| **Gamepad điều khiển** | 8 nút hướng + thanh tốc độ | Hoạt động |
| **Toggle relay** | Bật/tắt quạt, phun sương (khóa ở AUTO) | Hoạt động |
| **WebSocket status** | Hiển thị trạng thái kết nối (xanh/đỏ) | Hoạt động |
| **Trang Devices** (`/devices/`) | Thêm/sửa/xóa cảm biến và relay | Hoạt động |
| **Trang Scenarios** (`/scenarios/`) | Tạo kịch bản tự động AND/OR/XOR | Hoạt động |

**Giao diện hoạt động không cần tải lại trang** — mọi cập nhật sensor, relay, chế độ đều phản chiếu ngay lập tức qua WebSocket.

#### 4.2.3. Kết quả vận hành thực tế

- **Độ chính xác đo lường**: Sai số nhiệt độ ±0.5 °C, độ ẩm ±2 %, khoảng cách ±1.5 cm — nằm trong thông số kỹ thuật của nhà sản xuất cảm biến.
- **Thời gian phản hồi**: Từ khi ESP32 gửi dữ liệu đến khi dashboard cập nhật: **trung bình 180 ms** trên mạng WiFi cục bộ.
- **Scenario engine**: Kích hoạt relay đúng điều kiện trong 100 % các trường hợp thử nghiệm (20 lần thử).
- **Di chuyển robot**: Xe di chuyển mượt, phản hồi lệnh từ web nhanh, không bị trễ đáng kể ở tốc độ 50–80 %.
- **Độ ổn định WebSocket**: Kết nối duy trì liên tục trong 3 giờ thử nghiệm; tự kết nối lại sau 3 giây nếu mất mạng tạm thời.

---

### 4.3. Đánh giá và Thảo luận

#### 4.3.1. Đánh giá ưu điểm

- **Hoạt động độc lập**: Hệ thống vận hành hoàn toàn tự chủ sau khi cấu hình — không cần can thiệp thủ công trong chế độ AUTO.
- **Can thiệp trực tiếp môi trường**: Relay quạt và phun sương hoạt động ngay trên xe, tác động trực tiếp đến môi trường xung quanh trong < 1 giây sau khi điều kiện thoả.
- **Trực quan trên web**: Dashboard cung cấp cái nhìn tổng thể real-time — từ giá trị cảm biến, đồng hồ khoảng cách, biểu đồ lịch sử đến trạng thái relay — tất cả trên một màn hình duy nhất.
- **Cấu hình linh hoạt**: Kịch bản tự động có thể thêm/sửa/xóa qua giao diện web mà không cần chỉnh code.
- **Giao thức tách biệt rõ ràng**: Lệnh cho ESP32 (`relay_command`) và dữ liệu hiển thị FE (`device_update`) được phân tách, tránh xung đột.

#### 4.3.2. Hạn chế của hệ thống

| Hạn chế | Mức độ ảnh hưởng | Nguyên nhân |
|---|---|---|
| Độ trễ tăng khi WiFi yếu | Trung bình — dữ liệu vẫn đến, chậm hơn | Chất lượng tín hiệu WiFi, khoảng cách đến router |
| Tải server khi nhiều client | Thấp — chưa kiểm thử với > 5 client | Django dev server đơn luồng cho HTTP; Daphne xử lý WS tốt hơn |
| Pin tiêu hao nhanh | Cao khi dùng motor + relay đồng thời | Motor DC tiêu thụ dòng lớn (~1A/motor); thiếu cơ chế sleep |
| Không lưu lịch sử dữ liệu dài hạn | Trung bình — chỉ lưu giá trị hiện tại | Model `Device` chỉ có trường `value` đơn; không có bảng time-series |
| HC-SR04 nhiễu khi motor chạy | Thấp — xảy ra không thường xuyên | Nhiễu điện từ từ motor DC lan sang chân GPIO |

#### 4.3.3. Phân tích nguyên nhân và cách khắc phục

**Sai số cảm biến khoảng cách:**
- Nguyên nhân: Âm thanh siêu âm bị phản xạ lệch trên bề mặt nghiêng hoặc vật liệu mềm.
- Khắc phục: Lọc trung bình 3 mẫu liên tiếp trước khi gửi WebSocket; loại bỏ giá trị ngoài dải hợp lệ (< 2 cm hoặc > 400 cm).

**Kết nối WebSocket bị ngắt khi di chuyển:**
- Nguyên nhân: ESP32 di chuyển xa router WiFi, tín hiệu yếu.
- Khắc phục: Cơ chế tự kết nối lại đã được tích hợp sẵn trong `tools/esp32_sim.py` và firmware ESP32; thời gian retry 3 giây.

**Relay bật nhấp nháy (chattering) khi sensor ở ngưỡng:**
- Nguyên nhân: Giá trị sensor dao động quanh ngưỡng điều kiện kịch bản.
- Khắc phục: Thêm hysteresis trong Scenario condition (ví dụ: bật khi ≥ 35 °C, tắt khi < 33 °C).

---

## CHƯƠNG 5: KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN

---

### 5.1. Kết luận chung

#### 5.1.1. Những kết quả đã đạt được

Đề tài đã thiết kế và triển khai thành công hệ thống xe tự hành mini tích hợp xử lý môi trường và giám sát IoT qua web, đạt được các mục tiêu đề ra:

- **Phần cứng**: Hoàn thiện mô hình xe gắn đầy đủ cảm biến (HC-SR04 × 3, DHT22), relay (quạt, phun sương), module điều khiển motor L298N và vi điều khiển ESP32. Hệ thống hoạt động ổn định, sai số cảm biến nằm trong thông số kỹ thuật cho phép.

- **Phần mềm**: Xây dựng nền tảng web Django + Django Channels hoàn chỉnh:
  - Quản lý thiết bị (sensor/relay) qua giao diện CRUD tại `/devices/`
  - Tạo kịch bản tự động linh hoạt tại `/scenarios/` với logic AND/OR/XOR
  - Dashboard giám sát real-time tại `/car/` với WebSocket hai chiều
  - Scenario engine tự động đánh giá điều kiện và điều khiển relay không cần can thiệp người dùng

- **Giao tiếp IoT**: Giao thức WebSocket hai chiều giữa ESP32 và server được phân tách rõ ràng: ESP32 gửi `sensor_data`, server phản hồi `relay_command` (cho ESP32) và `device_update` (cho trình duyệt). Độ trễ end-to-end (cảm biến → dashboard) đạt < 200 ms trên mạng WiFi cục bộ.

- **Vận hành**: Hệ thống vận hành ổn định trong các buổi thử nghiệm liên tục. Chuyển đổi linh hoạt giữa chế độ AUTO (Scenario engine kiểm soát) và MANUAL (người dùng điều khiển trực tiếp) ngay trên dashboard web.

#### 5.1.2. Kiến thức và kỹ năng tích lũy được

| Lĩnh vực | Kỹ năng |
|---|---|
| **Phần cứng nhúng** | Lập trình ESP32 (WiFi, WebSocket, GPIO, PWM, ADC), đọc cảm biến HC-SR04, DHT22 |
| **Mạch điện** | Thiết kế mạch hạ áp, ghép nối L298N, bảo vệ mạch chống nhiễu motor |
| **Backend web** | Django 4.2, Django Channels, Daphne ASGI, ORM, migration |
| **Giao tiếp real-time** | WebSocket protocol, Channel Layer (Redis/In-memory), group broadcast |
| **Frontend** | Vanilla JavaScript, Canvas API, SVG animation, Bootstrap 5 |
| **Tích hợp hệ thống** | Thiết kế giao thức IoT, phân tách message type, debug WebSocket |
| **Triển khai** | Cấu hình ASGI server (Daphne), Docker-compose, Nginx reverse proxy |

---

### 5.2. Hướng phát triển của đề tài

#### 5.2.1. Nâng cấp công nghệ nhận diện

- **Camera AI + OpenCV**: Tích hợp module camera OV2640 trên ESP32-CAM để nhận diện vật cản, đọc biển báo hoặc theo dõi vạch đường, thay thế cảm biến siêu âm đơn điểm bằng nhận thức không gian toàn diện.
- **LIDAR (RPLIDAR A1)**: Gắn LIDAR quét 360° để xây dựng bản đồ 2D (SLAM) và tự lập kế hoạch đường đi cho xe hoạt động hoàn toàn tự hành trong môi trường chưa biết.
- **Cảm biến khí (MQ series)**: Bổ sung MQ-2 (khí gas/khói), MQ-135 (CO₂, NH₃) để mở rộng khả năng giám sát chất lượng không khí.

#### 5.2.2. Mở rộng hệ thống quản lý

- **Cảnh báo thời gian thực**: Tích hợp gửi thông báo qua **Telegram Bot** hoặc **Zalo OA** khi các chỉ số vượt ngưỡng nguy hiểm (nhiệt độ > 45 °C, khí gas > ngưỡng an toàn).
- **Lưu trữ dữ liệu time-series**: Chuyển từ SQLite (lưu giá trị hiện tại) sang **InfluxDB** hoặc thêm bảng `SensorLog` để lưu lịch sử theo thời gian — cho phép phân tích xu hướng và xuất báo cáo.
- **Dashboard nâng cao**: Tích hợp **Grafana** để hiển thị biểu đồ lịch sử nhiều ngày, cảnh báo threshold, và báo cáo định kỳ.
- **Phân quyền người dùng**: Thêm role `admin` (cấu hình) và `viewer` (chỉ xem) để triển khai multi-user.

#### 5.2.3. Tối ưu hóa năng lượng và vận hành

- **Đế sạc tự động**: Thiết kế trạm sạc với nam châm định vị để xe tự quay về sạc khi pin yếu (< 20 %), giữ xe luôn sẵn sàng vận hành.
- **Chế độ ngủ sâu (Deep Sleep)**: Áp dụng ESP32 deep sleep cho các cảm biến không cần đọc liên tục — chỉ wake up theo chu kỳ hoặc khi có trigger từ cảm biến PIR, giảm tiêu thụ điện năng xuống < 10 mA lúc chờ.
- **Hysteresis cho Scenario engine**: Thêm cơ chế ngưỡng kép (bật/tắt ở hai mức khác nhau) để tránh relay bật tắt liên tục khi sensor dao động quanh ngưỡng.
- **OTA Firmware Update**: Cho phép cập nhật firmware ESP32 từ xa qua giao diện web, không cần kết nối cáp USB.

---

