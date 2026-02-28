# 🔬 BÀI 3.5 (ĐẶC BIỆT): TUYỆT KĨ F12 - CÁCH TỰ TRUY TÌM NÚT BẤM KHI GOOGLE ĐỔI GIAO DIỆN
> **Vấn đề sống còn:** Hôm nay code của bạn chạy ầm ầm. Ngày mai Google cập nhật giao diện, code lăn ra chết, Robot bị mù không nhấp được nút. Làm sao để Robot luôn tìm đúng nút? Đây là bài học **cốt lõi nhất** của nghề viết tool Automation!

Chúng ta sẽ học cách dùng **"Kính Lúp DevTools" (F12)** có sẵn trong mọi trình duyệt Chrome để soi mã nguồn (DOM).

---

## BƯỚC 1: Gọi Kính Lúp DevTools Ra Bằng F12
Bất cứ một trang web mẹo nào (kể cả Google Flow), đằng sau cái giao diện đẹp đẽ đều là cái khung xương HTML. Để soi khung xương:

1. Bạn hãy mở mở trang web Google Flow lên.
2. Tìm cái nút bạn muốn Robot nhấp chuột (Ví dụ: cái nút **"Create content"** màu xanh).
3. **Bôi chuột thẳng vào cái nút đó**, nhấn CHUỘT PHẢI, kéo xuống dưới cùng và chọn **Kiểm tra (Inspect)** (Hoặc phím tắt siêu nhân: **Bấm F12**).
4. Lúc này, màn hình Chrome sẽ bổ đôi ra. Một nửa bên phải (hoặc bên dưới) hiện ra một đống Code màu chữ loằng ngoằng. Đừng hoảng sợ! Nó đang tô màu xanh dương thẳng vào đúng dòng Code tạo ra cái nút đó.

---

## BƯỚC 2: Học Cách Phân Biệt "Hàng Fake" & "Hàng Xịn"
Khi bạn nhìn vào dòng code tô xanh của cái nút, nó sẽ trông đại loại như thế này:
`<button class="btn-xs d-flex align-items-center random-x9a2b" aria-label="Create content" role="button">...</button>`

Làm sao để ra lệnh cho Robot nhận diện thằng này? (Hành động gọi là Viết **Selector**)

❌ **SAI LẦM CHÍ MẠNG (Treo tool): Chọn Hàng Fake (`class`)**
- Rất nhiều người dạy Automation bảo lấy Class. Tức là bảo Robot: *"Hãy bấm vào thằng có class tên là `random-x9a2b`"*.
- **Hậu quả:** Bọn React/Google cực kỳ lưu manh. Bọn nó xài tool tự trộn tên class. Ngày mai class nó đổi thành `random-z782c`. Thế là Tool của bạn bị Mù, Robot báo không tìm thấy nút.

✅ **BÍ KÍP CAO NHÂN: Chọn Hàng Xịn (Thuộc tính Tự nhiên - Semantic)**
- Ngó vào dòng code xem nó có những chữ như `aria-label=...`, `placeholder=...`, hoặc `role="..."` không? Bọn Google có cớ đổi giao diện liên tục nhưng rất lười đổi mấy cái Tên Mù màu cho người khuyết tật (`aria-label`).
- Bạn thấy chữ `aria-label="Create content"`.
- Tuyệt vời! Bạn hãy lập ra lệnh cho Robot (Viết Selector): **`"button[aria-label='Create content']"`**. 
- Lệnh này nghĩa là: *"Ê Robot, đi tìm cho tao cái thẻ button nào có đeo cái biển tên aria-label bằng chữ Create content"*. Bất chấp ngày mai Google đổi màu nút, đổi class, đổi giao diện, Robot cứ bám lấy cái biển tên đó mà phang!

---

## BƯỚC 3: Dùng Công Cụ "Ống Ngắm" Để Bắt Chọn 
Nếu bạn bấm F12 lên mà dòng code xanh không nhảy trúng cái nút bạn muốn? Bạn làm như sau:

1. Bấm F12 để mở bảng Code lên.
2. Nhìn lên góc CAO NHẤT bên TRÁI của cái bảng Code (DevTools), có một **biểu tượng hình Vòng tròn có Mũi Tên thụt vào** (nằm cạnh biểu tượng Điện thoại). Nhấp 1 nhấp vào nó cho nó sáng lên.
3. Rời chuột ra ngoài giao diện Web. Bây giờ con chuột của bạn đã biến thành Ống Ngắm. Bạn lia chuột vào ô nhập Text (Ví dụ ô gõ chữ Prompt).
4. Click Thẳng Cánh 1 cái.
5. Lập tức bảng Code sẽ Focus (Cuộn thẳng) đến dòng Code của ô Text đó: Phìa nó sẽ ghi: `<textarea id="..." placeholder="Describe your prompt here..." class="xyz...">`
6. Bạn lại dùng Bí kíp ở Bước 2. Bỏ qua id dài loằng ngoằng, bỏ qua class. Múc ngay cái ô `placeholder`.
7. Selector của bạn sẽ là: **`"textarea[placeholder*='prompt']"`** (Dấu `*=` nghĩa là "Có chứa cái chữ này là được", không cần bê nguyên cả câu dài).

---

## BƯỚC 4: Test Thử Thẳng Tay (Dùng Console) TRƯỚC Khi Lắp Cho Robot
> Trước khi ném cái Đoạn lệnh Selector đó cho Robot, bạn phải test thử xem cái Lệnh đó có Móc Mắt mò đúng Nút không. Nếu ra đúng thì mới bỏ vào file code.

1. Ngay tại bảng F12 (DevTools), nhìn thanh menu phía trên thay vì thả ở tab `Elements` (Khung xương), bạn bấm sang thẻ **`Console`** (Hệ điều hành ngầm).
2. Dưới cùng màn hình Console có dấu nháy nháy `>_` cho bạn gõ phím.
3. Bạn nhập đúng Cú Pháp Thần Thánh này và bấm Enter:
   **`document.querySelector("button[aria-label='Create content']")`**
4. Kết quả:
   - Nếu nó Trả về dòng chữ `null` (mẹ ơi, không thấy): Bạn viết sai Selector rồi. Lệnh hỏng. Viết lại đi.
   - Nếu nó văng ra nguyên cái Đoạn **`<button ...>....</button>`** xanh xanh đỏ đỏ: **BINGO!!** Xó đúng vào cái nút rồi! Chuẩn Men! Bạn copy thẳng Selector quẳng vào hàm của Robot thôi.

5. Để chắc kèo hơn, ấn thử cho nó ra lệnh bằng cái Code ngầm này luôn, gõ rồi Enter:
   **`document.querySelector("button[aria-label='Create content']").click()`**
   Lập tức trên màn hình cái Nút Đỏ nhảy tưng lên và Giao diện hiện ra bảng Tạo mới.

---

**TỔNG KẾT BÀI 3.5:** Hành trang quan trọng nhất của Lập trình viên Automation không phải là giỏi Code, mà là giỏi **F12 + Mũi tên Ngắm + Console Test + Né vụ bắt Class.** Cắm cái này vào não, mọi web Google, Facebook hay Tiktok đều có thể biến thành tool của riêng bạn! Và nhớ sửa lại ngay ở Bài 3 (chỗ Robot bấm nút) bằng cái Selector xịn xò mình vừa mò ra nhé!
