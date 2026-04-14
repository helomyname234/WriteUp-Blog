# Hướng dẫn sử dụng Blog CTF (HiW7n)

Đây là tài liệu hướng dẫn bạn cách quản lý nội dung và giao diện cho website blog của mình.

## 1. Cách đăng bài viết mới
1.  **Tạo file**: Tạo một file mới kết thúc bằng đuôi `.md` trong thư mục `docs/` (hoặc các thư mục con như `docs/ctf/picoctf/`).
2.  **Viết nội dung**: Sử dụng ngôn ngữ Markdown (hỗ trợ code block, bảng, ảnh...).
3.  **Cập nhật Menu**: Mở file `mkdocs.yml`, tìm đến phần `nav:` và thêm đường dẫn file mới vào. 
    *   *Ví dụ*: `- Tên Bài Viết: ctf/picoctf/ten-file.md`

## 2. Cách chỉnh sửa bài viết
1.  Mở file `.md` cần sửa trong trình soạn thảo.
2.  Lưu lại file. Nếu bạn đang chạy `mkdocs serve`, trình duyệt sẽ tự động cập nhật thay đổi ngay lập tức.

## 3. Cách xóa bài viết
1.  Xóa file `.md` trong thư mục `docs/`.
2.  Xóa dòng tương ứng trong phần `nav:` của file `mkdocs.yml`.

## 4. Cách thay đổi Giao diện (Theme) & Màu sắc
Mọi cấu hình nằm trong file `mkdocs.yml`:

*   **Đổi màu chủ đạo**: Thay đổi giá trị `primary` và `accent` trong phần `palette`. 
    *   Các màu hỗ trợ: `red`, `pink`, `purple`, `deep purple`, `indigo`, `blue`, `light blue`, `cyan`, `teal`, `green`, `light green`, `lime`, `yellow`, `amber`, `orange`, `deep orange`, `brown`, `grey`, `blue grey`.
*   **Đổi tên/Bio**: Sửa trong mục `extra` -> `personal`.
*   **Layout 3 cột**: Được cấu hình thông qua `overrides/partials/nav.html` và `docs/stylesheets/extra.css`. Nếu muốn quay về giao diện mặc định của MkDocs, hãy xóa dòng `custom_dir: overrides` trong `mkdocs.yml`.

## 5. Các lệnh terminal quan trọng
*   `python -m mkdocs serve`: Xem trước website tại máy (Local).
*   `python -m mkdocs build`: Đóng gói website thành các file HTML tĩnh (nằm trong thư mục `site/`).

---
**Chúc bạn có những bài viết CTF chất lượng!**
