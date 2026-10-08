# Bài 2: Quản trị Tường lửa UFW cho Cụm Dịch vụ Multi-port

## 1. Mục tiêu

- Thiết lập tường lửa UFW chặn mặc định mọi lưu lượng đi vào máy chủ.
- Chỉ mở các cổng dịch vụ cần thiết: SSH (22), Nginx (80), Spring Boot (8082).
- Cô lập cổng cơ sở dữ liệu MySQL (3306), không cho truy cập từ Internet.

## 2. Kế hoạch cấu hình

| Dịch vụ | Cổng | Chính sách |
|---|---|---|
| SSH (quản trị) | 22/tcp | ALLOW |
| Nginx (web HTTP) | 80/tcp | ALLOW |
| Spring Boot (kiểm tra từ xa) | 8082/tcp | ALLOW |
| MySQL | 3306/tcp | **Không có luật allow**, bị chặn bởi chính sách mặc định |

## 3. Các bước thực hiện

### 3.1. Khai báo chính sách mặc định

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 3.2. Cho phép các cổng dịch vụ

Các luật `allow` được thêm **trước** khi bật UFW để không bị mất kết nối SSH đang mở.

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 8082/tcp
```

Cổng 3306/tcp **không** được thêm luật allow nào.

### 3.3. Kích hoạt UFW

```bash
sudo ufw enable
```

## 4. Kết quả kiểm tra

```bash
sudo ufw status verbose
```

<!-- Thay khối dưới bằng output thực tế trên VPS của bạn -->
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
8082/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
8082/tcp (v6)              ALLOW IN    Anywhere (v6)
```

## 5. Đối chiếu với kết quả mong đợi

| Yêu cầu | Kết quả |
|---|---|
| Trạng thái `Status: active` | Đạt |
| Chính sách mặc định `deny (incoming)`, `allow (outgoing)` | Đạt |
| 22/tcp, 80/tcp, 8082/tcp ở trạng thái `ALLOW IN` từ `Anywhere` | Đạt |
| Cổng 3306 không xuất hiện trong danh sách cho phép | Đạt, bị chặn theo chính sách `deny incoming` |

## 6. Kết luận

- UFW đã được bật với chính sách mặc định chặn toàn bộ lưu lượng vào, chỉ cho phép ra.
- Chỉ ba cổng 22, 80 và 8082 được mở theo đúng yêu cầu. Cổng MySQL 3306 không có luật allow nên không thể truy cập từ Internet, giảm bề mặt tấn công khi bị quét cổng.
- Nếu ứng dụng Spring Boot cần kết nối MySQL thì dùng địa chỉ `localhost`, lưu lượng nội bộ không đi qua tường lửa nên vẫn hoạt động bình thường.
