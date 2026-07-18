# Cryptosystem - TryHackMe

## 1. Thông tin phòng lab

* **Link:** [Cryptosystem (TryHackMe)](https://tryhackme.com/room/hfb1cryptosystem)
* **Category:** Cryptography
* **Difficulty:** Easy

| Thông tin | Giá trị |
| :--- | :--- |
| **Platform** | TryHackMe |
| **Tags** | `crypto`, `rsa`, `fermat-factorization` |

### Mô tả

> A simple RSA challenge

**Mục tiêu học:**

* Hiểu về thuật toán RSA và các thành phần của nó.
* Khai thác điểm yếu của hệ mật RSA khi chọn hai số nguyên tố p và q quá gần nhau.

**Công cụ sử dụng:** `Python`

---

## 2. Phân tích lỗ hổng RSA (Fermat's Factorization)

Thuật toán mã hóa RSA dựa trên độ khó của việc phân tích một số nguyên lớn $n$ thành tích của hai số nguyên tố bí mật ($p$ và $q$). Nếu kẻ tấn công có thể phân tích được $n = p \times q$, họ có thể dễ dàng tính toán được khóa bí mật $d$ và giải mã thông điệp.

Trong bài lab này, chúng ta được cung cấp Public Key bao gồm modulus $n$ và số mũ công khai $e$, cùng với bản mã $c$.

Thông thường, việc phân tích thừa số nguyên tố cho một số $n$ rất lớn là điều bất khả thi trong thời gian thực. Tuy nhiên, trong một số trường hợp cấu hình sai, nếu hai số nguyên tố $p$ và $q$ được sinh ra **quá gần nhau**, thì giá trị của chúng sẽ xấp xỉ bằng căn bậc hai của $n$:
$$p \approx \sqrt{n}$$
$$q \approx \sqrt{n}$$

Lợi dụng điểm yếu này, chúng ta có thể dễ dàng tìm ra $p$ và $q$ bằng cách tính căn bậc hai phần nguyên của $n$. Gọi giá trị căn bậc hai phần nguyên này là $r$. Số nguyên tố $p$ sẽ là số nguyên tố gần nhất nhỏ hơn hoặc bằng $r$, và $q$ sẽ là số nguyên tố gần nhất lớn hơn hoặc bằng $r$.

---

## 3. Khai thác (Exploitation)

Sử dụng thư viện `pycryptodome` trong Python, chúng ta có thể viết một đoạn script nhỏ để tự động tìm $p$, $q$ dựa trên ý tưởng trên. Khi đã có $p$ và $q$, việc tính toán $\phi(n)$ và sinh ra khóa bí mật $d$ để giải mã bản mã $c$ là hoàn toàn đơn giản.

Đoạn code giải mã (`task.py`):

```python
from Crypto.Util.number import *
import math

# Các giá trị được cung cấp từ bài lab
n_goal = 15956250162063169819282947443743274370048643274416742655348817823973383829364700573954709256391245826513107784713930378963551647706777479778285473302665664446406061485616884195924631582130633137574953293367927991283669562895956699807156958071540818023122362163066253240925121801013767660074748021238790391454429710804497432783852601549399523002968004989537717283440868312648042676103745061431799927120153523260328285953425136675794192604406865878795209326998767174918642599709728617452705492122243853548109914399185369813289827342294084203933615645390728890698153490318636544474714700796569746488209438597446475170891
c = 3591116664311986976882299385598135447435246460706500887241769555088416359682787844532414943573794993699976035504884662834956846849863199643104254423886040489307177240200877443325036469020737734735252009890203860703565467027494906178455257487560902599823364571072627673274663460167258994444999732164163413069705603918912918029341906731249618390560631294516460072060282096338188363218018310558256333502075481132593474784272529318141983016684762611853350058135420177436511646593703541994904632405891675848987355444490338162636360806437862679321612136147437578799696630631933277767263530526354532898655937702383789647510

def primo(n):
    n += 2 if n & 1 else 1 # n lẻ => n=n+2 , n chẵn => n=n+1
    while not isPrime(n):
        n += 2
    return n

def find_p_and_q(n_goal):
    r = math.isqrt(n_goal)
    # Tìm p (số nguyên tố gần nhất <= r)
    p = r
    p -= 2 if p & 1 else 1 
    while not isPrime(p):
        p -= 2
    
    # Tìm q (số nguyên tố gần nhất >= p)
    q = primo(p)
    return p, q

n = 0 
while n != n_goal:
    p, q = find_p_and_q(n_goal)
    n = p * q

e = 0x10001
# Tính toán khóa bí mật d
d = inverse(e, (p-1) * (q-1))

# Giải mã bản mã
m = long_to_bytes(pow(c, d, n)).decode()
print(f"Message: {m}")
```

Chạy script trên, chúng ta sẽ nhận được nội dung của flag.

**Kết quả:**

```bash
$ python task.py
Message: THM{J______________SA!}
```
