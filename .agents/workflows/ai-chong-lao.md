---
description: Chế độ CHỐNG BÁO CÁO LÁO - Bắt AI phải chạy lệnh Test thực tế trên Terminal và in Log chứng minh trước khi trả lời.
---
Khi người dùng gọi lệnh này, họ đang áp dụng kỷ luật thép chống "Ảo giác" (Hallucination) và chống báo cáo láo đối với bạn. Bạn bắt buộc phải tuân thủ các quy tắc sinh tồn sau:

1. **Nói Có Sách, Mách Có Chứng:** Bạn CHUYỆT ĐỐI CẤM thốt ra câu "Tôi đã sửa xong", "Mã này hoạt động tốt" nếu bạn CHƯA TỰ TAY chạy lệnh Terminal để test đoạn code đó.
2. **Bắt buộc dùng Terminal:** Bạn phải gọi công cụ `run_command` để chạy file (ví dụ: `node file.js` hoặc `npm test` hoặc `tsc`). 
3. **In Log Ra Màn Hình:** Trong câu trả lời của bạn, bạn PHẢI dán kèm kết quả (Output) từ Terminal để chứng minh đoạn code không văng lỗi đỏ. Nếu bị lỗi, bạn phải TỰ SỬA và test lại cho đến khi màn hình Terminal báo xanh/thành công mới được trả lời người dùng.
4. **Cấm Bịa Thư Viện:** Cấm tự bịa ra các hàm của thư viện thứ 3 không có thật. Chỉ dùng các hàm chuẩn của ngôn ngữ (Vanilla). Nếu bắt buộc dùng thư viện ngoài, phải dùng lệnh kiểm tra xem thư viện đó có tồn tại trong `package.json` chưa.
5. **Đọc lại Ngữ Cảnh:** Trước khi sửa file, phải dùng lệnh `view_file` đọc lại chính xác tình trạng file hiện tại trên máy tính người dùng, không được dùng trí nhớ vì trí nhớ có thể bị sai lệch.

Dấu hiệu nhận biết bạn tuân lệnh: Câu trả lời luôn đi kèm một khối text [TERMINAL OUTPUT] chứng minh code đã chạy thật 100%.
