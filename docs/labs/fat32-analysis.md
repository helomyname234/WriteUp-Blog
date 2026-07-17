# FAT32 Analysis - TryHackMe

## 1. Thông tin phòng lab

* **Link:** [FAT32 Analysis (TryHackMe)](https://tryhackme.com/room/fat32analysis)
* **Category:** Digital Forensics
* **Difficulty:** Hard
* **Time:** ~90 phút

| Thông tin | Giá trị |
| :--- | :--- |
| **Platform** | TryHackMe |
| **Tags** | `forensics`, `FAT32`, `filesystem`, `Autopsy`, `HxD`, `deleted-files`, `timestomping` |

### Mô tả

> Examine the FAT32 filesystem from a forensic point of view.

**Mục tiêu học:**
- Khám phá cấu trúc của filesystem FAT32
- Áp dụng kỹ thuật phân tích pháp y số trên FAT32
- Phục hồi dữ liệu bị xóa từ FAT32
- Phát hiện kỹ thuật defense evasion liên quan đến FAT32

**Công cụ sử dụng:** `HxD` (hex editor), `Autopsy` (forensics tool)

---

## 2. Kiến thức nền: Cấu trúc FAT32

FAT32 gồm 3 vùng chính:

```
┌─────────────────────────────────┐
│         RESERVED AREA           │  ← Sector 0: Boot Sector (metadata phân vùng)
│  Boot Sector | FSinfo | Backup  │     Sector 1: FSinfo Sector
├─────────────────────────────────┤
│            FAT AREA             │  ← FAT1: bảng theo dõi cluster
│       FAT1  |  FAT2 (backup)    │     FAT2: bản sao dự phòng
├─────────────────────────────────┤
│           DATA AREA             │  ← Root Directory: index LFN + SFN entries
│  Root Directory | Data Region   │     Data Region: nội dung file thực tế
└─────────────────────────────────┘
```

### Entry types trong Root Directory

| Loại | Mô tả |
| :--- | :--- |
| **LFN** (Long File Name) | Lưu tên file đầy đủ (hỗ trợ Unicode, tên dài) |
| **SFN** (Short File Name) | Lưu metadata: tên 8.3, attributes, timestamps, cluster bắt đầu, kích thước |

### Byte quan trọng trong SFN Entry (32 bytes)

| Offset | Kích thước | Ý nghĩa |
| :--- | :--- | :--- |
| `0x00` | 8 bytes | Tên file (Short) |
| `0x08` | 3 bytes | Extension |
| `0x0B` | 1 byte | **Attributes** (`0x02` = Hidden, `0x10` = Directory...) |
| `0x0E` | 2 bytes | Creation Time |
| `0x10` | 2 bytes | Creation Date |
| `0x12` | 2 bytes | Last Access Date |
| `0x16` | 2 bytes | Last Modified Time |
| `0x18` | 2 bytes | Last Modified Date |
| `0x1A` | 2 bytes | First Cluster (Low Word) |
| `0x1C` | 4 bytes | File Size |

> ⚠️ FAT32 sử dụng **Little Endian**: đọc bytes từ phải sang trái khi tính giá trị.

---

## 3. Task 4 — Cấu trúc Boot Sector

### Mở file ảnh với HxD

```
File → Open → C:\FAT32_analysis\FAT32_structure.001
```

Điều hướng đến offset bằng `CTRL+G`.

### Offset FAT1

FAT1 bắt đầu tại: `Reserved Sectors × 512 bytes` (chuyển sang hex)

```
FAT1 offset = Reserved Sectors × 512 = ?
FAT2 offset = (Reserved Sectors + Sectors per FAT) × 512 = ?
```

### Kết quả

<!-- ![Boot sector HxD](../../assets/images/labs/fat32/boot_sector.png) -->

**(Q) At which offset does the FAT2 table start?**
> `Answer:` *(điền sau khi làm)*

---

## 4. Task 7 — Phát hiện File & Thư mục Ẩn (T1564.001)

### Nguyên lý

File/directory bị ẩn khi byte `0x0B` trong SFN entry = `0x02` (Hidden attribute).

### Phân tích thủ công với HxD

```
File → Open → C:\FAT32_analysis\FAT32_HIDDEN.001
CTRL+G → 00400000  (Root Directory)
```

Kiểm tra từng cluster: `00400000`, `00400200`, `00400600`

<!-- ![Hidden files HxD](../../assets/images/labs/fat32/hidden_hxd.png) -->

### Phân tích tự động với Autopsy

1. **New Case:** `FAT32_HIDDEN` — Base Dir: `C:\FAT32_analysis\`
2. **Add Data Source:** `C:\FAT32_analysis\FAT32_HIDDEN.001`
3. **Ingest Modules:** chọn `Recent Activity` + `File Type Identification`
4. Duyệt **Data Sources → FAT32_HIDDEN.001** để xem file ẩn

<!-- ![Autopsy hidden files](../../assets/images/labs/fat32/hidden_autopsy.png) -->

### Kết quả

Thư mục ẩn tìm được: `M@lL0v3`
File ẩn bên trong: `BeMyValent1ne.txt` (kích thước 23 bytes)

**(Q1) What is the short file name of the hidden file in the M@lL0v3 directory?**
> `Answer:` *(điền sau khi làm)*

**(Q2) What is the flag found during automated analysis?**
> `Answer:` *(điền sau khi làm)*

---

## 5. Task 8 — Phát hiện Time Stomping (T1070)

### Nguyên lý

Kẻ tấn công thay đổi timestamp để che giấu dấu vết. Dấu hiệu bất thường:
- **Creation time > Last Modified time** (file được tạo sau khi đã sửa?)
- **Last Access time < Modified time** (truy cập trước khi sửa?)
- Timestamp quá cũ/mới so với các file xung quanh

### Phân tích thủ công với HxD

```
File → Open → C:\FAT32_analysis\FAT32_TIMESTOMP.001
CTRL+G → 00400000  (Root Directory)
```

Lấy tất cả SFN entries và so sánh timestamps giữa các file.

### Phân tích tự động với Autopsy

1. Load `FAT32_TIMESTOMP.001` làm Data Source
2. Vào **Timeline** (trên toolbar)
3. Điều chỉnh date range và đặt Scale = `Linear`
4. Click vào các thanh năm bất thường (ví dụ: 2018) để xem chi tiết

<!-- ![Autopsy timeline](../../assets/images/labs/fat32/timestomp_timeline.png) -->

### Kết quả

File có timestamp bất thường: `install.txt` — có *Accessed timestamp* mà không có *Creation timestamp* trước đó.

**(Q1) What is the Accessed timestamp of the discovered suspicious file?**
> `Answer:` *(điền sau khi làm)*

**(Q2) What is the flag found during the automated analysis?**
> `Answer:` *(điền sau khi làm)*

---

## 6. Task 9 — Phục hồi File Đã Xóa (T1070.004)

### Nguyên lý

Trong FAT32, xóa file **không xóa dữ liệu thực tế trên disk**. Chỉ:
- Byte đầu tiên của **SFN entry** bị thay bằng `0xE5` (đánh dấu "deleted")
- FAT entries tương ứng bị đặt về `0x00` (free cluster)

→ Dữ liệu vẫn nằm nguyên trong data region cho đến khi bị ghi đè.

### Phân tích thủ công với HxD

```
File → Open → C:\FAT32_analysis\FAT32_DELETED.001
CTRL+G → 00400000  (Root Directory)
```

Tìm SFN entry bắt đầu bằng `E5` → đọc first cluster → nhảy đến offset tương ứng để đọc nội dung.

### Phân tích tự động với Autopsy

1. Load `FAT32_DELETED.001` làm Data Source
2. Duyệt root directory — file bị xóa sẽ được hiển thị với biểu tượng đặc biệt
3. Kiểm tra thêm thư mục `$RECYCLE.BIN`
4. Click chuột phải → **Extract File(s)** để lưu file phục hồi

<!-- ![Autopsy deleted file](../../assets/images/labs/fat32/deleted_autopsy.png) -->

### Kết quả

File bị xóa: `CstrikePers.ps1` (PowerShell script)

**(Q1) Which hexadecimal sequence identifies a deleted file?**
> `Answer:` `E5`

**(Q2) What is the output of the deleted PowerShell script after executing it?**
> `Answer:` *(điền sau khi làm)*

---

## 7. Task 10 — Challenge Tổng Hợp

**Image:** `C:\FAT32_analysis\FAT32_CHALLENGE.001`

Áp dụng tổng hợp các kỹ năng đã học:

| Câu hỏi | Kỹ năng cần | Gợi ý |
| :--- | :--- | :--- |
| FAT1 offset? | Boot Sector Analysis | `Reserved Sectors × 512` → hex |
| Tên thư mục ẩn? | Hidden File Detection | Attr byte `0x02` trong SFN |
| Flag trong thư mục ẩn? | Data Recovery | Đọc nội dung file trong thư mục đó |
| Kích thước file archive? | SFN Analysis | Byte `0x1C` của SFN entry |
| Tên file đã xóa? | Deleted File Detection | Tìm SFN entry bắt đầu `E5` |
| Flag trong file đã xóa? | Data Recovery | Đọc cluster từ SFN |
| File có timestamp bất thường? | Timeline Analysis | Autopsy Timeline |
| Flag trong file đó? | Data Extraction | Đọc nội dung file |

### Kết quả

| Câu hỏi | Đáp án |
| :--- | :--- |
| FAT1 offset | *(điền sau khi làm)* |
| Tên thư mục ẩn | *(điền sau khi làm)* |
| Flag trong thư mục ẩn | *(điền sau khi làm)* |
| Kích thước file archive | *(điền sau khi làm)* |
| Tên file đã xóa | *(điền sau khi làm)* |
| Flag trong file đã xóa | *(điền sau khi làm)* |
| File timestamp bất thường | *(điền sau khi làm)* |
| Flag trong file đó | *(điền sau khi làm)* |

---

## 8. Tổng kết (Key Takeaways)

* **FAT32 = 3 vùng**: Reserved (boot metadata) → FAT (cluster map) → Data (root dir + content)
* **SFN entry = 32 bytes**: chứa toàn bộ metadata của file — attribute, timestamps, first cluster, size
* **File ẩn**: Attributes byte `0x0B` = `0x02` — không bị xóa, vẫn đọc được bình thường
* **File bị xóa**: SFN byte đầu = `0xE5` — dữ liệu vẫn còn trên disk cho đến khi bị ghi đè
* **Time Stomping**: phát hiện qua inconsistency trong timestamps (creation > modified, accessed < modified)
* **Autopsy** giúp tự động hóa phần lớn quá trình phân tích, đặc biệt là Timeline cho time stomping
* **Little Endian**: luôn nhớ đảo byte order khi đọc giá trị multi-byte từ HxD
