# Bandit Level 0

Đây là thử thách đầu tiên trong chuỗi [Bandit](https://overthewire.org/wargames/bandit/) của OverTheWire. Mục tiêu là kết nối vào server qua SSH.

## Thông tin thử thách
- **Host**: `bandit.labs.overthewire.org`
- **Port**: `2220`
- **Username**: `bandit0`
- **Password**: `bandit0`

## Cách giải quyết

### Bước 1: Kết nối SSH
Sử dụng terminal và chạy lệnh sau để kết nối:

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

> [!NOTE]
> Khi hệ thống hỏi password, bạn hãy nhập `bandit0`. Chú ý là password sẽ không hiển thị trên màn hình khi bạn gõ.

### Bước 2: Tìm Flag
Sau khi đăng nhập thành công, chúng ta cần tìm file `readme` nằm trong thư mục home.

```bash
ls
cat readme
```

## Kết quả
Flag của level này là nội dung trong file `readme`. Sau khi có nội dung này, bạn có thể dùng nó làm password để đăng nhập vào `bandit1`.

---
*Chúc may mắn với các level tiếp theo!*
