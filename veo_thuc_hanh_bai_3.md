# 🤖 BÀI 3: LẮP CÁNH TAY ROBOT HACK VÀO WEB (CONTENT SCRIPT)
> **Mục tiêu Bài 3:** Bộ Não (Bài 2) đã có lệnh, bây giờ ta cần 1 cái "Tay" để chọc thẳng trang `labs.google.com`. Thay vì bạn tự lấy chuột click "Generate", tự gõ phím nhập Prompt, thì con Robot này sẽ làm cực nhanh. Do trang web Google code bằng **ReactJS** rất xịn, mình không thể gõ chữ bình thường được mà phải dùng "Hack"!

---

## BƯỚC 1: Tạo Bộ Cốt Cho Cánh Tay (File flow-automator.js)
1. Lại nhìn vào cây thư mục bên trái của dự án (`VEO-Automation-Project`).
2. Nhấn chuột phải tạo một **Thư mục mới**, tên là: `content-scripts`.
3. Trong thư mục `content-scripts`, tạo 1 file tên là: `flow-automator.js`.
4. Mở file `flow-automator.js` ra. Đây là Cánh Tay Robot sẽ chạy ngầm ngay CÙNG LÚC trang Google khởi ĐỘNG.

---

## BƯỚC 2: Trang Bị "Mắt Nhìn Duyên Dáng" (Kỹ Thuật Chờ Đợi)
> Trang web hiện đại load vòng vòng (xoay loading) liên tục. Nếu Robot nhảy vào click ngay sẽ bị rớt mạng vì nút chưa hiện ra. Phải dạy nó ĐỢI.

1. **Copy và Paste khối Code Số 1 vào file `flow-automator.js`:**

```javascript
/* =======================================================
 * KHỐI 1: ÁNH MẮT CỦA CÚ MÈO (MutationObserver)
 * ======================================================= */

// Hàm này giúp Robot "Nhìn chằm chằm" vào cái web chờ 1 cái nút xuất hiện
// Thay vì dùng setTimeout (ngủ 5 giây) rất ngu xuẩn và dễ hỏng
function waitForElement(selector, timeout = 30000) {
    return new Promise((resolve, reject) => {
        // 1. Thấy nút cái là tóm luôn
        const element = document.querySelector(selector);
        if (element) return resolve(element);

        // 2. Lắp camera theo dõi (KỸ THUẬT SỐ 2: MutationObserver)
        const observer = new MutationObserver((mutations, me) => {
            const elm = document.querySelector(selector);
            if (elm) {
                me.disconnect(); // Tắt camera đi đỡ rác máy!
                resolve(elm);
            }
        });

        // Bật camera theo dõi sự thay đổi trên toàn bộ HTML của Body
        observer.observe(document.body, { childList: true, subtree: true });

        // 3. Nếu Đợi 30 giây (30000ms) mà éo thấy thì Báo Lỗi!
        setTimeout(() => {
            observer.disconnect();
            reject(new Error(`Đợi mỏi cổ ${timeout}ms không thấy cái ${selector}`));
        }, timeout);
    });
}
```
2. **Bấm Lưu (Ctrl + S)**.

📌 **GIÁO VIÊN GIẢI THÍCH (BÍ KÍP TỰ ĐỘNG HÓA):**
Ai cùi bắp hay dùng `setTimeout(click, 5000)` để đợi mạng load. Nhỡ mạng nhà mạng lởm load 10 giây? Lỗi mẹ code. Nên dân chuyên xài thuật toán `MutationObserver` (Người giám sát sinh học). Cứ lúc nào trang Web nó mọc thêm cái nút thì mổ vào bấm ngay lập tức trong 0.001 giây, không thừa không thiếu.

---

## BƯỚC 3: Dạy Kỹ Kỹ Thuật "Đục Két React" (The Native Setter)
> Thử thách lớn nhất đời Làm Bot: Bạn gõ chữ vào trang React bằng Code, chữ hiện lên NHƯNG bấm nút Gửi thì nó Trắng Tinh. Tại sao? Vì bộ não React bị mù, nó chỉ hiểu khi bàn phím người THẬT nảy lên. Ta phải HACK.

1. **Copy và Paste tiếp Khối Số 2 xuống bên dưới vào file `flow-automator.js`:**

```javascript
/* =======================================================
 * KHỐI 2: ĐỤC KÉT REACT BẰNG "NATIVE SETTER"
 * ======================================================= */

// KỸ THUẬT SỐ 3 (TRẤN PHÁI): Lừa React là bàn phím thật
function setReactInputValue(element, value) {
    // 1. Gõ lén chữ vào đít input (React éo biết)
    const nativeInputValueSetter = Object.getOwnPropertyDescriptor(
        window.HTMLTextAreaElement.prototype, 
        "value"
    ).set;
    nativeInputValueSetter.call(element, value);

    // 2. Giả vở "bắn pháo sáng" hét lên: Ê CÓ NGƯỜI VỪA GÕ PHÍM KÌA!!
    const event = new Event('input', { bubbles: true });
    element.dispatchEvent(event);
}

// Hàm giả vờ lấy Ngón Tay Bấm Chuột Cốc Cốc
function simulateClick(element) {
    // KỸ THUẬT RỘNG: Bấm vào cái nút nhưng không làm màn hình bị giật
    const event = new MouseEvent('click', { view: window, bubbles: true, cancelable: true });
    element.dispatchEvent(event);
}
```
2. **Bấm Lưu (Ctrl + S)**.

---

## BƯỚC 4: Ráp nối Thành Chuỗi Hành Động Tuyệt Mật (Start Robot)
> Robot đã có Mắt (Khối 1), có Tay Gõ Cực Xịn (Khối 2). Giờ ta nối dây với cái tai, bắt tin nhắn BỘ NÃO (Bài 2) gọi xuống!

1. **Dưới Khối 2, dán Khối Số 3 cuối cùng này vào:**

```javascript
/* =======================================================
 * KHỐI 3: HÀNH ĐỘNG THỰC TẾ TRÊN TRANG WEB GOOGLE FLOW
 * ======================================================= */

// Tai nghe: Khi Bộ Não Gào Lên "START_ROBOT"
chrome.runtime.onMessage.addListener(async (msg, sender, sendResponse) => {
    
    if (msg.action === 'START_ROBOT') {
        const job = msg.jobData;
        const promptText = job.prompt;

        console.log("ROBOT: Bắt đầu chạy Job " + job.id);

        try {
            // HÀNH ĐỘNG 1: Dùng Mắt chim ưng (Khối 1) tìm và Click nút Tạo Mới (New Project)
            // KỸ THUẬT SỐ 4: Selector chống vỡ (Tránh dùng class dài ngoằn bị đổi hàng ngày)
            const newBtn = await waitForElement("button[aria-label='Create content']");
            simulateClick(newBtn);
            
            // HÀNH ĐỘNG 2: Mở mắt tìm Ô nhập Textarea rỗng để điền Prompt
            const textarea = await waitForElement("textarea[placeholder*='prompt']");
            
            // HÀNH ĐỘNG 3: Dùng chiêu Vô Ảnh Giác (Khối 2) gõ chữ lừa React
            setReactInputValue(textarea, promptText);
            
            // HÀNH ĐỘNG 4: Tìm Ngôi sao đỏ (Nút Bấm Generate 🎬)
            const generateBtn = await waitForElement("button:has(span:contains('Generate'))");
            simulateClick(generateBtn);
            
            // THẦN CHÚ RIÊNG CỦA BẠN: (Khoan, video sinh ra mất 1 - 5 phút. 
            // Robot éo biết lúc nào xong. Làm sao để bốc cái link mp4 đó ném về cho Bộ Não?)
            // => Xin mời học tiếp BÀI 4 (MAIN World Injector) !!!

        } catch (error) {
            console.error("VEO Lỗi: ", error.message);
            // Má nó hỏng rồi, báo Bộ Não bỏ cuộc cái này đi
            chrome.runtime.sendMessage({ action: 'JOB_DONE', jobId: job.id, success: false });
        }
    }
});
```
2. **Bấm Lưu (Ctrl + S)**.

🎉 **CHÚC MỪNG!!** Gần xong rồi! Nếu bạn Refresh cài đặt, bây giờ Extension đã tự biết Gõ Text + Tự Động Bấm chữ Generate như ma nhập trên màn hình Chrome của bạn!

> Quá xịn! Nhưng... chờ 3 phút video đẻ ra, xong làm sao lấy link tải ở trong mã nguồn nhét về được Extension mà đem đi tải? Mời bạn làm nốt tuyệt kĩ đỉnh cao nhất ở **BÀI TẬP VỀ ĐÍCH CỐT LÕI (BÀI 4)**!!
