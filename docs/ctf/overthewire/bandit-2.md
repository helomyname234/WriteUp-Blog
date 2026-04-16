# Bandit Level 2

Thử thách này yêu cầu chúng ta tìm mật khẩu được lưu trữ trong một file có chứa dấu cách (spaces) trong tên file, nằm ngay tại thư mục home.

## Thông tin thử thách
| Thông tin | Giá trị |
| :--- | :--- |
| **Host** | `bandit.labs.overthewire.org` |
| **Port** | `2220` |
| **Username** | `bandit2` |
| **Password** | *Mật khẩu thu được từ Level 1* |

---

## Phân tích & Cách giải quyết

### Vấn đề với dấu cách trong tên file
Trong Bash và hầu hết các shell khác, dấu cách được sử dụng để phân tách các tham số (arguments) của một lệnh. 

Để thấy rõ vấn đề, hãy xem ví dụ khi chúng ta chạy lệnh `cat` mà không xử lý dấu cách:

```bash
bandit2@bandit:~$ cat spaces in this filename
cat: spaces: No such file or directory
cat: in: No such file or directory
cat: this: No such file or directory
cat: filename: No such file or directory
```

Hoặc nếu bạn cố tình thêm các ký tự đặc biệt như `--`:
```bash
bandit2@bandit:~$ cat --spaces in this filename--
cat: unrecognized option '--spaces'
Try 'cat --help' for more information.
```

### Giải pháp
Để hệ thống không nhầm lẫn giữa tên file và các tham số của lệnh, chúng ta cần sử dụng **ký tự thoát (escape character)** và chỉ định đường dẫn rõ ràng.

Sử dụng dấu gạch chéo ngược `\` trước mỗi dấu cách để báo cho shell biết rằng đó là một phần của tên file:

```bash
bandit2@bandit:~$ cat ./\-\-spaces\ in\ this\ filename\-\-
```
## Kết quả
Sau khi thực hiện lệnh trên, bạn sẽ nhận được mật khẩu cho level tiếp theo.

![Result](../../assets/images/ctf/overthewire/bandit2/result.png)

---
*Chúc may mắn với các level tiếp theo!*
