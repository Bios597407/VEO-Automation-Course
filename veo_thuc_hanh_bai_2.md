# 🚀 BÀI 2: CHẾ TẠO "BỘ NÃO" QUẢN LÝ (SERVICE WORKER)
> **Mục tiêu Bài 2:** Nút bấm ở Bài 1 đã gõ cửa gửi lệnh `ADD_JOBS` nhưng chưa ai mở cửa. Ở bài này, ta sẽ tạo "Bộ Não" — một người trực tổng đài đứng ở Tầng 2 (Orchestration Layer). Nó sẽ nhận lệnh, xếp hàng đợi (Queue), và ra lệnh cho Tay Robot đi làm việc.

---

## BƯỚC 1: Tạo Nơi Chứa Bộ Não (File service-worker.js)
1. Quay lại cây thư mục ban đầu (`VEO-Automation-Project`).
2. Nhấn chuột phải tạo một **Thư mục mới**, đặt tên là: `background` (Nghĩa là "Chạy ngầm", vì thằng này không có mặt mũi, nó luôn chạy lén lút sau lưng Chrome).
3. Vaò trong thư mục `background`, tạo một file tên là: `service-worker.js`.
4. Mở file `service-worker.js` ra. Đây sẽ là một file code RẤT DÀI (Bởi nó chứa 50% sức mạnh của cái công cụ này). 

Đừng lo, hãy làm theo hướng dẫn cắt lát sau đây:

---

## BƯỚC 2: Khởi tạo "Cuốn Sổ Ghi Chép" (Queue & Trạng Thái)
> Bộ não cần ghi sổ xem khách hàng mướn làm bao nhiêu cái ảnh, đang làm tới đâu rồi.

1. **Copy và Paste khối Code Số 1 vào đầu file `service-worker.js`**:

```javascript
/* =======================================================
 * KHỐI 1: KHỞI TẠO BIẾN VÀ LẮNG NGHE LỆNH CỦA NGƯỜI DÙNG
 * ======================================================= */

// Sổ tay ghi chép
let jobsQueue = [];           // Danh sách chờ: Ai đến trước làm trước (FIFO)
let isRunning = false;        // Có đang chạy không?
let maxConcurrent = 2;        // KỸ THUẬT SỐ 7: Chạy Tối Đa 2 cái cùng lúc, tránh đơ máy
let activeJobs = 0;           // Hiện tại đang làm mấy cái?

// Khi Nút Bấm "Start" ở file panel.js gửi tin nhắn tới đây
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
    
    if (message.action === 'ADD_JOBS') {
        const prompts = message.prompts;
        
        // Cứ mỗi 1 prompt là 1 thẻ công việc (job)
        prompts.forEach((txt, index) => {
            jobsQueue.push({
                id: Date.now() + index,
                prompt: txt,
                status: 'pending' // pending nghĩa là: Đang xếp hàng chờ
            });
        });

        // Báo lại cho Cửa Sổ Giao Diện là: "Đã nhận lệnh rồi nhé!"
        sendResponse({ status: 'ok' });
        
        // Bắt đầu Hô To "Vào Viêcccc!"
        if (!isRunning) {
            isRunning = true;
            processQueue();
        }
    }
    return true; // Để Chrome biết mình sẽ trả lời nó
});
```
2. **Bấm Lưu (Ctrl + S)**.

📌 **GIÁO VIÊN GIẢI THÍCH (BÍ KÍP CONCURRENCY CONTROL):**
- Dòng `let maxConcurrent = 2;` cực kỳ lợi hại. Máy yếu mà chạy 5 cái tab Google Flow cùng lúc là treo máy màn hình xanh ngay. Số `2` này đóng vai trò như bảo vệ quán bar: *Tròn 2 khách vào rồi, đứa thứ 3 đứng ngoài đợi đi!*. 
- Đây chính là **Kỹ thuật Kiểm soát Luồng (Semaphore)** — một thủ thuật dân Pro hay xài.

---

## BƯỚC 3: Dạy Bộ Não Cách Điều Phối (Hàm processQueue)
> Có sổ rồi, giờ phải dạy nó lấy từng đơn hàng ra để nhét cho thằng làm thuê (Tầng 3) chạy. Dưới Khối 1 vừa dán, xuống dòng 2 lần và dán tiếp Khối 2:

1. **Copy và Paste khối Code Số 2 xuống bên dưới khối 1:**

```javascript
/* =======================================================
 * KHỐI 2: XỬ LÝ HÀNG ĐỢI (ĐIỀU PHỐI VIỆC LÀM)
 * ======================================================= */

async function processQueue() {
    // Nếu hết việc, hoặc đang làm đủ số lượng (2 người) rồi thì thôi, đứng đợi
    if (jobsQueue.length === 0 || activeJobs >= maxConcurrent) {
        if (jobsQueue.length === 0 && activeJobs === 0) {
            isRunning = false; // Xong hết việc, cho đi ngủ
        }
        return;
    }

    // Rút thẻ công việc MỚI NHẤT ra khỏi đầu hàng đợi (FIFO)
    const job = jobsQueue.shift();
    job.status = 'processing';
    activeJobs++; // Đánh dấu: "Đã thêm 1 thằng đang làm việc"

    // [KỸ THUẬT SỐ 6] GHI LƯU TRẠNG THÁI RA Ổ ĐĨA TRƯỚC KHI LÀM
    // Lỡ đang làm mà bị rớt mạng hay Chrome tạm dừng thì vẫn còn dữ liệu
    await chrome.storage.local.set({ state: { queue: jobsQueue } });
    
    // Gửi thông báo về Giao diện để cập nhật % (1/50, 2/50...)
    broadcastProgress();

    // 🔴 Gọi "Cánh Tay Robot" đi làm việc
    // Ở đoạn này, ta sẽ Inject (bơm) một file mã độc (content script)
    // vào trang labs.google.com/fx/tools/flow đẻ nó tự bấm chuột
    try {
        await executeAutomation(job); 
    } catch (err) {
        console.error("Lỗi khi làm việc: ", err);
    } finally {
        // Ghi lại: "Đứa này vừa làm xong, bớt 1 đứa đi"
        activeJobs--;
        
        // Gọi lại chính hàm này để lùa thằng tiếp theo trong hàng đợi vào làm
        processQueue(); 
    }
}
```
2. **Bấm Lưu (Ctrl + S)**.

📌 **GIÁO VIÊN GIẢI THÍCH:**
Phần này có dùng **Kỹ thuật số 6: State Persistence**. Bộ nhớ `chrome.storage.local` là ổ đĩa an toàn. Tầng 2 (Service worker) này bị Google cắm cờ là "thằng ngủ nướng" — tức là nếu 30 giây không ai xài gì là Chrome bắt nó **Sleep dọn RAM**. Xui rủi nó đi ngủ lúc chưa làm xong 50 video, là mất luôn cái list danh sách đó. Cách khắc phục: Lưu mọi thứ vào `storage` ra ổ cứng luôn!

---

## BƯỚC 4: Kết nối với cánh tay Robot và "Chôm" video (Hàm executeAutomation)
> Thùng rác cuối cùng của file `service-worker`. Dưới Khối 2, dán Khối 3.

1. **Copy và Paste khối Code Số 3 xuống cuối cùng:**

```javascript
/* =======================================================
 * KHỐI 3: RA LỆNH CHO ROBOT VÀ BẮT FILE TẢI VỀ
 * ======================================================= */

function executeAutomation(job) {
    return new Promise((resolve, reject) => {
        // Tìm cái Tab nào đang mở trang Google Flow
        chrome.tabs.query({ url: "https://labs.google.com/fx/tools/flow*" }, (tabs) => {
            if (tabs.length === 0) {
                return reject("Chưa mở trang Google Flow!");
            }
            
            const targetTab = tabs[0].id; // Lấy Tab đầu tiên tìm thấy

            // Đeo tai nghe vào để chờ Robot báo tin
            const listener = (msg, sender, sendResp) => {
                if (msg.action === 'JOB_DONE' && msg.jobId === job.id) {
                    
                    // Tháo tai nghe ra 
                    chrome.runtime.onMessage.removeListener(listener);
                    
                    // Robot trả về 1 cái Link phim. Ra lệnh tải nó về!
                    if (msg.videoUrl) {
                        chrome.downloads.download({
                            url: msg.videoUrl,
                            filename: `VEO_Output_${job.id}.mp4`,
                            saveAs: false // Đừng hỏi lưu ở đâu, tải thẳng luôn
                        });
                    }
                    resolve();
                }
            };
            chrome.runtime.onMessage.addListener(listener);

            // GIAO VIỆC CHO ROBOT: Cầm cái tờ giấy (job) này sang mà làm!
            chrome.tabs.sendMessage(targetTab, {
                action: 'START_ROBOT',
                jobData: job
            });
        });
    });
}

function broadcastProgress() {
    chrome.runtime.sendMessage({
        event: 'QUEUE_UPDATE',
        data: { stats: { done: 10, total: 50 /* Ví dụ nhử mồi tạm thời */ } }
    }).catch(() => {}); // Kệ nếu Giao diện đang tắt
}
```
2. **Bấm Lưu (Ctrl + S)**.

🎉 **CHÚC MỪNG!!** Vậy là "Trái tim - Bộ Não" của chúng ta đã hoàn chỉnh. Nếu bây giờ bấm Start ở giao diện, Giao diện đã gọi được Bộ não, Bộ não sẽ xếp 50 lệnh vào Queue... Nhưng khựng lại! Nó gào lên lệnh `START_ROBOT` nhưng con Robot chưa ra đời... Mời bạn qua Bài 3 để đúc cánh tay Robot!
