# Bandit Level 0

Đây là thử thách đầu tiên trong chuỗi [Bandit](https://overthewire.org/wargames/bandit/) của OverTheWire. Mục tiêu là kết nối vào server qua SSH để làm quen với hệ thống.

## Thông tin thử thách
| Thông tin | Giá trị |
| :--- | :--- |
| **Host** | `bandit.labs.overthewire.org` |
| **Port** | `2220` |
| **Username** | `bandit0` |
| **Password** | `bandit0` |

---

## Phân tích & Cách giải quyết

### Bước 1: Kết nối SSH
Sử dụng terminal và chạy lệnh sau để kết nối. Nếu bạn sử dụng Linux/macOS, hãy mở Terminal; nếu dùng Windows, bạn có thể dùng PowerShell, Command Prompt hoặc CMD.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

> [!NOTE]
> Khi hệ thống hỏi mật khẩu, hãy nhập `bandit0`. Lưu ý rằng ký tự mật khẩu sẽ không hiển thị trên màn hình khi bạn gõ (đây là tính năng bảo mật của Unix).

### Bước 2: Tìm Flag
Sau khi đăng nhập thành công, chúng ta cần tìm mật khẩu (flag) cho level tiếp theo. Thông thường, mật khẩu sẽ nằm trong file `readme` ở thư mục hiện tại.

```bash
ls
cat readme
```

## Kết quả
Flag của level này chính là nội dung bên trong file `readme`. Sau khi có nội dung này, bạn hãy lưu lại để dùng làm mật khẩu đăng nhập cho `bandit1`.

---
*Chúc may mắn với các level tiếp theo!*
