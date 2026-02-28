# 🚨 BÀI 5 (KỸ NĂNG SINH TỒN): VẠCH LÁ TÌM BUG & TRỊ BỆNH AI "MẤT TRÍ NHỚ"
> **Sự thật phũ phàng:** Chẳng có cái Tool nào chạy phát ăn ngay cả! Bạn sẽ bị kẹt, bị lỗi đỏ lòm màn hình, và đến lúc nhờ vả AI (như ChatGPT, Claude) thì con AI lại "quên mất" bạn đang code cái gì và đưa ra đoạn code tào lao phá nát cả Tool.

Bài này là tấm khiên bảo vệ bạn khỏi sự tuyệt vọng. Hãy học cách tự cứu mình bằng các quy tắc sau:

---

## PHẦN 1: BÁC SĨ BẮT MẠCH — TÌM LỖI (BUG) ẨN GIẤU Ở ĐÂU?
Đa số người mới làm Chrome Extension đều khóc ròng vì "Code hỏng mà chả thấy máy báo lỗi gì cả!". Đó là do bạn **chưa biết tìm ổ bệnh ở đâu.** Extension có 4 tầng, mỗi tầng lại giấu lỗi ở một chỗ khác nhau:

1. **Lỗi ở Tầng Giao Diện (Side Panel) & Robot (Content Script):**
   - **Triệu chứng:** Nút Start bấm không ăn, Robot không chịu điền chữ trên trang GOOGLE.
   - **Khám bệnh:** Bạn mở **F12** ngay tại trang Google Flow -> Qua tab **Console**. Chữ đỏ (Error) sẽ hiện chình ình ở đó.

2. **Lỗi ở Bộ Não (Service Worker) — ĐÂY LÀ CHỖ DỄ CHẾT KHUẤT NHẤT:**
   - **Triệu chứng:** Giao diện vẫn gõ được, Robot không báo lỗi, nhưng chả có cái tệp nào tải về cả!
   - **Khám bệnh:** Bấm F12 ở trang web KHÔNG BAO GIỜ thấy lỗi của thằng này. Bạn phải vào lại dọnng `chrome://extensions`, tìm cái tên "Auto VEO siêu cấp" của bạn. Bạn sẽ thấy dòng chữ xanh biển **"Trình chạy dịch vụ (Service Worker)"** hoặc **"Lỗi (Errors)"** đỏ rực mọc ra. BẤM TRỰC TIẾP VÀO ĐÓ, bảng Console bí mật của não bộ mới hiện ra!

---

## PHẦN 2: 3 CĂN BỆNH KINH ĐIỂN NHẤT KHI LÀM TOOL
Khi con Cua (Chrome) nó cắn, thường là do 3 nguyên nhân bất hủ này:

**🔥 Bệnh #1: "Treo máy" vì mạng chậm (Lỗi Timing)**
- Cố tình dùng Lệnh Ngủ `setTimeout(click_nút, 2000)`. Máy nghẽn mạng cần 3 giây để load xong. Ở giây thứ 2 con Robot đã vội nhắm mắt nhắm mũi Click vào hư không. Mọi thứ gãy đổ!
- **Cách trị:** TUYỆT ĐỐI KHÔNG xài `setTimeout` để chờ nút. Phải xài cái "Ống nhòm" `MutationObserver` (Hàm `waitForElement` ta đã học ở Bài 3) để luôn rình rập, web cứ nảy nút ra là Robot đấm nút ngay tức khắc!

**🔥 Bệnh #2: ReactJS Bị Điếc (Chữ điền rồi nhưng bấm Gửi bị mất)**
- Bạn thấy Robot điền được chữ "Con báo hồng" vào ô Prompt rồi. Mừng quá. Nhưng chớp mắt 1 cái lúc Robot nhấn nút Hút Cỏ (Generate) thì form báo Trống Rỗng, Không cho gửi.
- **Cách trị:** Do trang web Google làm bằng React. Nó không ghi nhận chữ bị "Nhét đít" vào (như hàm `textarea.value = 'hello'`). Phải xài bí kíp **hack Native Setter** cùng lệnh `dispatchEvent(new Event('input'))` y hệt BÀI 3 để lừa cái form là có người thật đang gõ bàn phím xạch xạch.

**🔥 Bệnh #3: Quỷ Lùn Mất Kết Nối (Extension Context Invalidated)**
- Khi bạn sửa code `.js`, bạn ra ngoài `chrome://extensions` bấm Refresh vòng lặp (Cập nhật). RỒI BẠN QUA TRANG GOOGLE BẤM CHẠY LUÔN. Nó văng ra lỗi đỏ chót: *"Extension context invalidated"*.
- **Vì sao:** Vì trang Google đó đã nạp (nhét) file Robot cũ từ cái đời tám hoảnh rồi. Bạn làm mới Extension nhưng cái Trang Web nó không biết.
- **Cách trị cực kỳ đơn giản (Nhưng ai cũng gà):** Mỗi lần sờ tay vào sửa code và bấn Cập nhật Extension ở ngoài, BẠN PHẢI QUAY VỀ TRANG WEB VÀ BẤM F5 (TẢI LẠI TRANG) để nó nạp lại Robot mới tiêm vào!!

---

## PHẦN 3: BÍ KÍP "HUẤN LUYỆN THÚ" — CHỐNG AI (CHATGPT/CLAUDE) BỊ ẢO GIÁC
> Khi bí quá, học sinh thường vứt code bảo AI: "Sửa cho tao". AI nó sửa xong, bạn dán vào... Tool chết ngắc và rối nát bét. Bọn AI có căn bệnh nan y là **NHANH QUÊN NGỮ CẢNH (Context Loss)**. Hãy quản lý nó như quản lý nhân viên dưới quyền sếp!

**1. Bệnh AI "Quên mất mày là ai"**
- **Đừng nói trống không:** *"Lỗi rồi, nút ko chạy, cho code khác đi"*. Lúc này con AI sẽ ngơ ngác đẻ ra một đoạn code đè nát kiến trúc 4 tầng ở Bài 1-4 bạn vừa dày công xây.
- **Lệnh chuẩn chỉ (Prompt vàng):** *"Mày hãy đóng vai Chuyên gia Viết Chrome Extension. Đây là file `content-script.js` HIỆN TẠI của tao. Tao muốn nhấp cái nút có CSS Selector là `.btn-blue`. Cấu trúc code phải GIỮ NGUYÊN, chỉ bổ sung lệnh click đúng vào chỗ [KHỐI 3] cho tao"*.

**2. Bệnh AI "Ảo tưởng sức mạnh" (Sinh ra UI thừa thãi)**
- AI rất hay ngứa tay tự đè thêm các thư viện lạ (như xài JQuery) hoặc bắt bạn phải tạo hàm lạ.
- **Lệnh chuẩn chỉ:** *"Cấm mày sử dụng bất kỳ Framework nào, chỉ viết cho tao bằng JavaScript thuần (Vanilla JS). Cấm mày xóa dòng mã cũ của tao"*.

**3. Nhồi lại Ngữ cảnh (Ép AI nhớ lại từ đầu)**
- Rất nhiều khi chat dài tầm 20 dòng, con AI nó sẽ NGÁO, nó quên cả việc nó đang code cái Chrome Extension.
- **Tuyệt chiêu:** Lấy toàn bộ File Manifest nhét lại cho nó đọc: *"Tao đang làm Extension Manifest V3. Đây là Manifest của tao (Dán vô). Tao đang gửi tin nhắn từ SidePanel sang Service Worker báo lỗi này: (Chụp lỗi console của Google Chrome quăng vô). Sửa đúng chỗ cho tao"*. AI lập tức BỪNG TỈNH và đưa code cực kỳ chính xác.

**4. Quy luật Cắt Nhỏ "Miếng Bánh"**
- ĐỪNG YÊU CẦU AI LÀM CẢ CÁI TOOL: *"Viết cho tao thằng Auto Google FLow đi"*. Nó sẽ cho một cái code Rác Rưởi vứt đi.
- **Hãy bảo AI đi từng bước hệt như cuốn giáo trình này:** *"Viết cho tao Cửa Sổ HTML trước" -> (Xong) -> "Viết hàm bắt Input từ Form" -> (Xong) -> "Giờ viết hàm gửi tin nhắn sang File X"*. Cứ làm chậm và nhỏ giọt, AI biến thành thiên tài!

> **Lời kết:** Nắm chắc File này, Lập trình Automation chết kiểu gì bạn cũng túm được tóc nó để dựng lại. Hãy dán 3 căn bệnh và 4 tư duy sai khiến AI này lên tường, bạn sẽ không sợ bất cứ trang Cứng Đầu nào mặt đất nữa! Chúc bạn Code vững vàng!
