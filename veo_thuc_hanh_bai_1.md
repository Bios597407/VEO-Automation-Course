# 💻 THỰC HÀNH CẦM TAY CHỈ VIỆC: Chế Tạo "VEO Automation" Từ Số 0
> **Phiên bản Đặc Biệt:** Dành cho học sinh làm từng bước. Làm đến đâu, hiểu đến đó, **KHÔNG CẮT BỚT** bất cứ kiến thức chuyên sâu nào. Cứ làm đúng các bước, tự nhiên bạn sẽ làm được!

---

## 🎯 GIỚI THIỆU LỘ TRÌNH THỰC HÀNH
Chúng ta sẽ cùng nhau "code" lại công cụ VEO Automation (có khả năng tự động hóa Google Flow, tạo hàng loạt video/ảnh). 

**Chúng ta sẽ chia làm 4 buổi (4 Bài):**
- **Bài 1:** Đặt móng & Làm Giao Diện (Manifest + Side Panel)
- **Bài 2:** Chế tạo "Bộ Não" quản lý hàng đợi (Service Worker)
- **Bài 3:** Lắp "Tay Robot" hack trang web (Content Script & Kỹ thuật React)
- **Bài 4:** Cài "Gián điệp" bắt link video & Chạy thử nghiệm (MAIN World)

---

# 🚀 BÀI 1: KHỞI ĐỘNG, ĐẶT MÓNG VÀ LÀM GIAO DIỆN

## BƯỚC 1: Chuẩn bị "Đồ nghề" và Tạo Thư mục gốc
1. Bạn hãy mở máy tính lên, ở ngoài màn hình chính (Desktop).
2. Tạo một Thư mục (Folder) mới, đặt tên là: `VEO-Automation-Project`.
3. Mở phần mềm tự gõ code (Khuyên dùng **Visual Studio Code** hoặc nếu không có thì dùng **Notepad**).
4. Mở thư mục `VEO-Automation-Project` vừa tạo vào trong phần mềm gõ code đó. Mọi file chúng ta tạo từ giờ sẽ nằm trong thư mục này.

---

## BƯỚC 2: Viết "Giấy Khai Sinh" cho Extension (File manifest.json)
> Bất cứ Extension nào cũng phải có file này để trình duyệt Chrome biết nó là cái gì.

1. Bấm nút **Tạo file mới** (New File).
2. Đặt tên CHÍNH XÁC là: `manifest.json` (Viết thường, không viết hoa).
3. **Copy và Paste đoạn code sau** vào file vừa tạo:

```jsonc
{
  "manifest_version": 3,
  "name": "Auto VEO siêu cấp",
  "version": "1.0.0",
  "description": "Tự động gửi prompt và tải video trên Google Flow",
  
  "permissions": [
    "sidePanel",
    "storage",
    "downloads",
    "scripting",
    "tabs"
  ],
  
  "host_permissions": [
    "https://labs.google.com/*",
    "https://aisandbox-pa.googleapis.com/*"
  ],
  
  "background": {
    "service_worker": "background/service-worker.js",
    "type": "module"
  },
  
  "content_scripts": [
    {
      "matches": ["https://labs.google.com/fx/tools/flow/*"],
      "js": ["content-scripts/flow-automator.js"],
      "run_at": "document_idle"
    }
  ],
  
  "side_panel": {
    "default_path": "side-panel/panel.html"
  }
}
```
4. **Bấm Lưu (Ctrl + S)**. 

📌 **GIÁO VIÊN GIẢI THÍCH (BẠN CHỈ CẦN ĐỌC ĐỂ HIỂU):**
- `manifest_version: 3`: Quy định bắt buộc của Google năm nay, dùng phiên bản số 3.
- `permissions`: Đây là lúc ta **"xin quyền"**. Mình xin quyền `sidePanel` để có thanh công cụ bên cạnh, quyền `downloads` để tí nữa tự tải video về, và `storage` để lưu dữ liệu.
- `host_permissions`: Nghĩa là công cụ này **CHỈ ĐƯỢC PHÉP CHẠY** trên trang Google chứ không chạy trên Facebook hay Youtube.
- `background` và `content_scripts`: Là các file code ta sẽ tạo ở các bước sau!

---

## BƯỚC 3: Xây Giao Diện Người Dùng (Side Panel)
> Giao diện là nơi để người dùng nhập chữ (prompt), bấm nút Start, xem 100%...

1. Quay lại cây thư mục. Chột phải tạo một **Thư mục (Folder) mới**, đặt tên là: `side-panel`.
2. Bên trong thư mục `side-panel`, tạo file mới tên là: `panel.html`.
3. Mở file `panel.html` lên, **Copy và Paste đoạn HTML này vào**:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    /* Chỉnh cho nó đẹp tí nào */
    body { font-family: Arial; padding: 10px; background: #222; color: white;}
    textarea { width: 100%; height: 100px; margin-top: 10px; background: #333; color: white;}
    button { width: 100%; padding: 10px; background: #e94560; color: white; border: none; font-weight: bold; cursor: pointer; margin-top: 10px;}
    #status { margin-top: 20px; color: #a6e3a1; }
  </style>
</head>
<body>
  <h2>🚀 VEO AUTOMATION</h2>
  <label>Dán danh sách Prompt vào đây (Mỗi dòng 1 prompt):</label>
  <textarea id="promptInput" placeholder="Ví dụ: A cute cat running..."></textarea>
  
  <!-- Nút bấm bắt đầu -->
  <button id="startBtn">BẮT ĐẦU CHẠY TỰ ĐỘNG</button>
  
  <!-- Khu vực hiển thị phần trăm tiến trình -->
  <div id="status">Sẵn sàng chờ lệnh...</div>

  <!-- Nối với file JS để có tác dụng khi bấm nút -->
  <script src="panel.js"></script>
</body>
</html>
```
4. **Bấm Lưu (Ctrl + S)**.

---

## BƯỚC 4: Lắp "Dây Điện" Cho Nút Bấm (File panel.js)
> HTML chỉ là cái xác (cái nút). Ta cần làm cho cái nút bấm vào có tác dụng (gửi lệnh đi). Đây gọi là Tầng Giao Diện (UI Layer).

1. Vẫn ở trong thư mục `side-panel`, tạo thêm 1 file có tên là: `panel.js`.
2. **Copy và Paste đoạn code sau vào:**

```javascript
// DÒNG NÀY ĐỂ BẮT ĐƯỢC CÁI NÚT VÀ CÁI Ô NHẬP TEXT
const startBtn = document.getElementById('startBtn');
const promptInput = document.getElementById('promptInput');
const statusDiv = document.getElementById('status');

// KHI NGƯỜI DÙNG BẤM CHUỘT VÀO NÚT
startBtn.addEventListener('click', () => {
    
    // 1. Lấy toàn bộ chữ người dùng gõ
    const text = promptInput.value;
    if (!text) return alert("Bạn chưa nhập mã Prompt!");

    // 2. Tách từng dòng ra thành 1 mảng (vì mình muốn làm hàng loạt)
    // Tức là 10 dòng sẽ thành 10 cái lệnh (job)
    const promptsArray = text.split('\n').filter(p => p.trim() !== '');

    // 3. Đổi chữ trên màn hình cho ngầu
    statusDiv.innerText = "⏳ Đang gửi " + promptsArray.length + " lệnh đi...";

    // 4. GỬI LỆNH CHO "BỘ NÃO" (Service Worker)
    chrome.runtime.sendMessage({
        action: 'ADD_JOBS', 
        prompts: promptsArray
    }, (response) => {
        // Nhận tin nhắn phản hồi từ Bộ Não
        if(response.status === 'ok') {
            statusDiv.innerText = "✅ Đã lên lịch xong! Mở thẻ Google Flow để chờ máy tự chạy.";
        }
    });
});

// LẮNG NGHE TIN NHẮN TỪ BỘ NÃO (Để cập nhật phần trăm %)
chrome.runtime.onMessage.addListener((msg) => {
    if (msg.event === 'QUEUE_UPDATE') {
        const done = msg.data.stats.done;
        const total = msg.data.stats.total;
        statusDiv.innerText = `Đã làm xong: ${done} / ${total} video.`;
    }
});
```
3. **Bấm Lưu (Ctrl + S)**.

📌 **GIÁO VIÊN GIẢI THÍCH (BÍ KÍP KIẾN TRÚC):**
Bạn nhớ cái sơ đồ 4 tầng không? Đây chính là **Tầng 1 (UI Layer)**. Bạn để ý hàm `chrome.runtime.sendMessage()` nhé. File này không tự đi làm video, nó chỉ biến thành cái "Bộ đàm" gọi điện cho Tầng 2 (Service Worker - Bộ Não) bảo: *"Này Bộ Não ơi, tao có danh sách 10 cái prompt này, ADD_JOBS cho tao"*.

---

## BƯỚC 5: Chạy Thử Lần Đầu Tiên (Load Extension)
Bây giờ, chúng ta chưa code xong, nhưng ta có thể tận mắt nhìn thấy giao diện Extension của chính mình trên Chrome!

1. Hãy mở trình duyệt Chrome lên.
2. Gõ vào thanh địa chỉ dòng này và Enter: `chrome://extensions/`
3. Nhìn phía trên cùng góc BÊN PHẢI, có chữ **"Developer mode" (Chế độ dành cho nhà phát triển)**. Hãy gạt công tắc đó sang **ON** (Màu xanh).
4. Phía trên cùng góc TRÁI, sẽ hiện ra nút **"Load unpacked" (Tải tiện ích đã giải nén)**. Bấm vào nút đó.
5. Sẽ có 1 cửa sổ chọn file hiện ra. Bạn DẪN tới cái thư mục `VEO-Automation-Project` của chúng ta, bấm một cái vào tên thư mục đó rồi chọn **Select Folder**.
6. BÙM 💥! Tiện ích của bạn đã xuất hiện!
7. **Cách mở lên xem:** Hãy mở một cái Tab mới (bất kỳ trang nào), nhìn góc trên bên phải Chrome bấm vào biểu tượng "Mảnh ghép xếp hình", sẽ thấy extension của ban. Bấm ghim nó.
8. Bấm vào biểu tượng extension, nhìn sang lề phải màn hình — **Bảng điều khiển Side Panel của bạn đã xuất hiện!**

*(Lúc này nếu bạn nhập prompt và bấm Thử, nó sẽ báo lỗi đỏ trong console vì "Bộ não" chưa được tạo. Hãy sang ngay BÀI 2!)*
