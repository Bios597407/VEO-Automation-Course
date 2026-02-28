# 🕵️‍♂️ BÀI 4: CÀI "GIÁN ĐIỆP" BẮT LINK VIDEO VÀ LỤM THÀNH QUẢ (MAIN WORLD INJECTION)
> **Mục tiêu Bài 4:** Google Flow không hề có cái chữ "Link MP4" nào ngoài màn hình khi xuất xong Video để cánh tay Robot lấy. Nó ẩn tít sâu trong lớp API chạy ngầm. Ở bài cuối cùng này, bạn sẽ học Kỹ Thuật Đỉnh Của Đỉnh: **Chích thuốc (Inject)** một file nội gián vào thẳng bên trong Mạch máu của web Google!

---

## BƯỚC 1: Tạo Nội Gián Đi Cửa Trực Tiếp (File page-interceptor.js)
1. Quay lại thư mục `VEO-Automation-Project`.
2. Tạo 1 file mới thẳng ngay bên ngoài, ngang hàng với file `manifest.json`. Gọi tên nó là: `page-interceptor.js` (Interceptor nghĩa là kẻ đánh chặn).
3. Mở file đó ra.
4. **Copy và Paste đoạn code NỘI GIÁN sau vào file:**

```javascript
/* =======================================================
 * KHỐI 1 NỘI GIÁN: ĐÁNH CHẶN Ổ KHOÁ WINDOW.FETCH 
 * ======================================================= */

// Lưu lại hàm fetch GỐC của Google (kiểu như copy chìa khóa nhà)
const originalFetch = window.fetch;

// Mình ĐÁNH TRÁO chìa khóa giả vào cửa nhà Google (Lấy hàm fetch tự chế)
window.fetch = async function(...args) {
    const url = args[0] || '';

    // Cửa nhà vẫn mở bình thường cho khách, lấy dữ liệu gốc
    const response = await originalFetch.apply(this, args);
    
    // NHƯNG... Nếu khách vác ra gói hàng là "Video Mới Xuất Xong" (Cái Api Sinh Ra Video)
    if (url.includes("aisandbox-pa.googleapis.com") && response.ok) {
        
        // Cú lừa: Clone (nhân bản) 1 gói hàng y hệt để mở ra coi trộm URL 
        // Phải Dùng .clone() vì response dùng 1 lần là tự huỷ
        response.clone().json().then(data => {
            
            // Tìm tận ngóc ngách của Data, moi ra dải chữ có chữa .mp4
            const strData = JSON.stringify(data);
            const mp4Match = strData.match(/https:\/\/[^"]+\.mp4/);
            
            if (mp4Match) {
                const videoUrl = mp4Match[0]; // BINGO !! TÌM RA VIDEO TẢI VỀ RỒI
                console.log("Nội gián báo cáo: Tìm thấy Video VEO! ", videoUrl);
                
                // MẬT THƯ GỬI LẠI CONTENT SCRIPT: 
                // Éo thể gửi cho Service Worker được (Do Main world không có quyền)
                // Phải hét to bằng loa phát thanh CustomEvent
                document.dispatchEvent(new CustomEvent('__VEO_VIDEO_URL__', { 
                    detail: { url: videoUrl } 
                }));
            }
        }).catch(() => {});
    }

    // Cuối cùng: Nhét chìa khóa thật vào cửa cho Google chạy bình thường
    // (Làm giả mà như không có chuyện gì)
    return response;
};
```
5. **Bấm Lưu (Ctrl + S)**.

📌 **GIÁO VIÊN GIẢI THÍCH (BÍ KÍP CHÍCH THUỐC MAIN WORLD):**
- **Tại sao phải làm thế?** Bởi Cánh Tay Robot của mình (Content Script) mọc sừng sững nhưng BỊ CHẶN một lớp kính chống đạn. Nó không thể sửa `window.fetch` của trang web được (Do Chrome quy định cấm - Isolated World).
- Phải dùng mẹo: Dùng file `page-interceptor.js` này làm nội gián thâm nhập thẳng vào phần lõi MAIN WORLD (Thay đổi Code thực sự của Google Flow). Xong nội gián sẽ bắn loa phát thanh `CustomEvent(__VEO_VIDEO_URL__)`, ở ngoài Cánh tay Robot nghe thấy tiếng loa báo sẽ lấy cái Link đút lại về cho Bộ Não. Quá khôn lỏi!

---

## BƯỚC 2: Mở Cổng Thả Nội Gián (Inject File)
Vậy thả lén file này vào Web Google bằng cách nào? Ta phải nhờ lại Cánh tay Robot nhét vào trang `head` của Web!

1. **QUAY LẠI MỞ FILE ROBOT:** `content-scripts/flow-automator.js` (Nhớ là file của Bài 3 nhé!).
2. Kéo xuống dòng TẬN CÙNG CỦA FILE.
3. **Copy và Paste đoạn Code Nhét Cổng này vào**:

```javascript
/* =======================================================
 * KHỐI 4: CHÍCH THUỐC NỘI GIÁN VÀO CƠ THỂ GOOGLE (MAIN WORLD)
 * ======================================================= */

// Bơm file page-interceptor.js đâm ngập vào DOM của Header Web Google
const scriptEl = document.createElement('script');

// Móc chính xác đường dẫn của File đó bằng quyền Chrome Extension
scriptEl.src = chrome.runtime.getURL('page-interceptor.js');
scriptEl.onload = function() {
    this.remove(); // Bơm xong rút ống kim tiêm ra xóa dấu vết luôn
};
(document.head || document.documentElement).appendChild(scriptEl);

// BẬT LOA BẮT ĐÀI: Nằm im chờ nghe "Mật thư CustomEvent" thằng Nội gián bắn ra
document.addEventListener('__VEO_VIDEO_URL__', (e) => {
    const url = e.detail.url;
    
    // CÓ LINK MP4 RỒI ! Trả nó về tay cục quản lý Service Worker ở BÀI 2 !
    // Bộ Não ơi, Job này thành công, lấy Link mà Download đi!!
    // Lưu ý: Biến 'job' bạn lấy id từ cái Cú Gào START_ROBOT ở Bài 3.
    // Thực tế CODE HOÀN CHỈNH bạn sẽ gán job đang chạy vào biến toàn cục.
    chrome.runtime.sendMessage({ 
        action: 'JOB_DONE', 
        jobId: "Chỗ_này_điền_ID_đã_lưu_khi_START_ROBOT", 
        success: true, 
        videoUrl: url 
    });
});
```
4. **Bấm Lưu (Ctrl + S)**.

Nhớ là: Trong file `manifest.json`, ta KHÔNG CẦN khai báo `content_scripts` cho thư mục nội gián, ta chỉ cần khai báo "Tài sản công cộng (web_accessible_resources)" cho file này để Chrome cấp quyền tiêm thuốc.

*(Vì dụ trong bài này để nhắm mắt cũng làm được, bạn vào file `manifest.json` và thêm đoạn này xuống dưới dòng `side_panel`:)*
```jsonc
  "web_accessible_resources": [
    {
      "resources": ["page-interceptor.js"],
      "matches": ["https://labs.google.com/*"]
    }
  ]
```

---

## BƯỚC 3: KẾT THÚC CHUỖI & QUY TRÌNH RA MẮT
> Mọi nút gỡ đã xong. Bạn đã thực hiện xong cả 4 Tầng! 

**Hãy nhìn lại xem Extension của bạn tự động hoàn hảo thế nào:**
1. (Bài 1) Người dùng mở Web, nhập 50 con mèo dễ thương (Prompts), Bấm Bắt Đầu. Gửi thẳng sang (Bài 2).
2. (Bài 2) Tầng Service Worker ghi sổ lưu vào Ổ cứng `chrome.storage`. Sắp xếp chờ máy có mạng. Gọi (Bài 3).
3. (Bài 3) Tay Robot thò vào hack React gõ 2 nháy bằng đít chuột: Một cái tạo ra mèo vằn, nhảy lên Click Tối Tạo.
4. Bấm Tạo xong là việc của (Bài 4): Màn hình xuất vòng load quay quay 2 phút, tay chân nhàn rỗi. Tự dưng API rớt trúng hàm Fetch giả. Gián điệp vớ lấy bắn `Mật Thư`. Robot (Bài 3) truyền cho (Bài 2). (Bài 2) Tự tải video MP4 bằng `chrome.downloads`. Nó cập nhật qua (Bài 1) là "Tải xong 1 / 50 video ! Thích nhé". Và Lại Quắn Qué (Loop) tiếp con số 2.

**CÁCH XUẤT XƯỞNG EXTENSION "LÀM THÀNH THẦN" CHỈ TRONG 1 NODE NHẠC:**
1. Thư mục `VEO-Automation-Project` của bạn trên Desktop đang chứa trọn mọi mồ hôi nước mắt bạn Code nãy giờ. Click chuột phải, chọn Compress (Nén ra file Zip).
2. Quăng File Zip móp méo đó vào cho bọn bạn đúp click dùng chơi. Hoặc...
3. Nạp 5$ (125k VND) vô thẻ tín dụng, vào đường dẫn `chrome.google.com/webstore/devconsole` ném File Zip lên, Đặt tên kêu kêu, rồi ngủ qua đêm duyệt là Hùng Hồn hiện Extension trên Webstore Mỹ!!!

💪 **Kết thúc Khóa Học - Các bạn đã Cực Kỳ Pro RỒI đấy!!**
