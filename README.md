# Báo cáo Bài 5: Thiết kế và Cấu hình dịch vụ Systemd Service cho ứng dụng Java

## 1. Mục tiêu
- Biết cách thiết kế cấu hình dịch vụ Systemd để quản lý vòng đời ứng dụng tự động.
- Cấu hình chạy dưới user giới hạn (non-root) để nâng cao bảo mật hệ thống.
- Cài đặt chính sách tự động khởi động lại dịch vụ khi sập đột ngột.

## 2. Các bước cấu hình

Đầu tiên, em tạo một tài khoản hệ thống chuyên biệt tên là `java-runner`. Điểm quan trọng là chặn không cho tài khoản này đăng nhập trực tiếp (nologin) để bảo mật tối đa:
```bash
sudo useradd -r -s /usr/sbin/nologin java-runner
```

Tiếp theo, tạo file cấu hình service tại `/etc/systemd/system/java-app.service` (nội dung chi tiết đã được em đính kèm trong repo ở tệp `java-app.service`). Các điểm sáng cấu hình bao gồm:
- **User=java-runner:** Giới hạn quyền chạy tiến trình Java.
- **ExecStart=...:** Chạy file JAR giả lập (hoặc vòng lặp sleep như một tiến trình mô phỏng liên tục).
- **Restart=on-failure / RestartSec=10:** Đảm bảo hệ điều hành sẽ bắt lỗi khi tiến trình bị sập và tự động khởi động lại sau 10 giây.

Sau khi lưu file service, em bắt buộc phải báo cho systemd biết về sự thay đổi cấu hình mới này bằng lệnh:
```bash
sudo systemctl daemon-reload
```

Kích hoạt và khởi chạy dịch vụ:
```bash
sudo systemctl start java-app.service
sudo systemctl enable java-app.service
```

## 3. Kết quả kiểm tra
Em kiểm tra trạng thái hoạt động của tiến trình bằng lệnh:
```bash
sudo systemctl status java-app.service
```

**Đầu ra kết quả trả về của lệnh:**

```text
● java-app.service - Java Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/java-app.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2026-10-07 10:10:05 UTC; 15s ago
   Main PID: 7788 (sleep)
      Tasks: 1 (limit: 4627)
     Memory: 216.0K
     CGroup: /system.slice/java-app.service
             └─7788 /usr/bin/sleep 1000

Oct 07 10:10:05 ubuntu-server systemd[1]: Started Java Spring Boot Application Service.
```

*(Ghi chú: Kết quả hiển thị rõ dịch vụ đang chạy dưới trạng thái `active (running)`. Main PID đang hoạt động ổn định và log báo Started thành công).*
