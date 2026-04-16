# Bandit Level 1

Thử thách này yêu cầu chúng ta tìm mật khẩu được lưu trữ trong một file có tên là `-`, nằm trong thư mục home.

## Thông tin thử thách
| Thông tin | Giá trị |
| :--- | :--- |
| **Host** | `bandit.labs.overthewire.org` |
| **Port** | `2220` |
| **Username** | `bandit1` |
| **Password** | *Mật khẩu thu được từ Level 0* |

---

## Phân tích & Cách giải quyết

### Vấn đề với file tên `-`
Trong môi trường Linux/Unix, ký tự `-` thường được các câu lệnh hiểu là **Standard Input (stdin)** hoặc **Standard Output (stdout)** thay vì là một tên file thông thường. 

Xem thêm về single dash `-` trong bash tại [đây](https://linuxvox.com/blog/bash-difference-between-and-options/)

Nếu bạn chạy lệnh `cat -` đơn thuần:
```bash
cat -
```
Hệ thống sẽ chờ bạn nhập dữ liệu từ bàn phím (stdin) và in ngược lại ra màn hình thay vì đọc nội dung của file có tên `-`.

### Giải pháp
Để xử lý các file có tên đặc biệt (bắt đầu bằng dấu gạch ngang hoặc chỉ bao gồm dấu gạch ngang), chúng ta cần chỉ định **đường dẫn (path)** rõ ràng để hệ thống không nhầm lẫn giữa tên file và tham số của lệnh.

#### Cách 1: Sử dụng đường dẫn tương đối (Khuyên dùng)
Sử dụng `./` để chỉ định file nằm ở thư mục hiện hành:
```bash
cat ./-
```
## Kết quả
![Cat Password](../../assets/images/ctf/overthewire/bandit1/first.png)


---
*Chúc may mắn với các level tiếp theo!*
