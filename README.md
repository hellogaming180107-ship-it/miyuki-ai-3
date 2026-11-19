# Miyuki AI Girlfriend v2

Ứng dụng chat người yêu ảo anime chạy trên trình duyệt, dùng Cloudflare Workers + Workers AI + D1.

## Tính năng
- Messenger UI responsive cho máy tính và điện thoại.
- Nhân vật Miyuki static 2D (minh họa SVG mặc định, có thể thay bằng ảnh của bạn).
- Chat tiếng Việt, nhập vai lãng mạn/gợi cảm không mô tả tình dục trực diện.
- POV và nút tương tác mô phỏng: nắm tay, ôm, vuốt tóc, chạm má.
- Tạo ảnh anime trưởng thành, có trang phục bằng Workers AI.
- Lịch sử chat và ký ức dài hạn tự động trích xuất.
- Quản lý/xóa ký ức, xóa lịch sử (API hỗ trợ).
- Mật khẩu truy cập qua Cloudflare Secret.

## Thành phần và lưu ý
- Chat: `@cf/meta/llama-3.1-8b-instruct`.
- Ảnh: `@cf/black-forest-labs/flux-1-schnell`.
- DB: Cloudflare D1.
- Auto-memory gọi thêm một lượt inference sau mỗi tin nhắn. Điều này tăng chi phí/hạn mức AI; nếu lượt trích xuất thất bại, chat vẫn hoạt động.
- Ký ức tự động chỉ cố gắng lưu thông tin dài hạn người dùng trực tiếp chia sẻ, và loại trừ một số dữ liệu nhạy cảm. Mô hình có thể trích xuất sai; người dùng có thể xem và xóa từng mục.
- Ảnh được tạo theo yêu cầu và giữ tạm trong trình duyệt; không lưu lâu dài trên máy chủ.
- Tương tác chạm là nhập vai bằng tin nhắn, không phải xúc giác thật.
- Các giới hạn miễn phí do Cloudflare quy định và có thể thay đổi; không có cam kết miễn phí vô hạn.

## Triển khai không cần cài phần mềm trên máy

### 1. GitHub
1. Vào https://github.com/new
2. Tạo repository `miyuki-ai`, nên chọn Private.
3. Mở repository > Add file > Upload files.
4. Upload toàn bộ nội dung trong ZIP, giữ nguyên `public/`, `src/`, `schema.sql`, `wrangler.jsonc`.
5. Commit changes.

### 2. Cloudflare D1
1. Đăng nhập https://dash.cloudflare.com/
2. Workers & Pages > D1 SQL Database > Create.
3. Database name: `miyuki-memory`.
4. Mở Console, chạy toàn bộ nội dung `schema.sql`.
5. Sao chép Database ID.

### 3. Cập nhật cấu hình
Mở `wrangler.jsonc` trên GitHub, thay `PASTE_YOUR_D1_DATABASE_ID_HERE` bằng ID database. Commit.

### 4. Tạo Worker từ Git
1. Cloudflare Dashboard > Workers & Pages > Create application.
2. Chọn Connect to Git và cấp quyền GitHub.
3. Chọn repository `miyuki-ai`.
4. Để root directory là `/`.
5. Chọn cấu hình Worker dùng `wrangler.jsonc` trong repository.
6. Deploy.
7. Kiểm tra Worker có binding `AI`, `DB` và assets `ASSETS`. Nếu bảng Bindings chưa nhận D1, thêm binding tên `DB` trỏ đúng database `miyuki-memory`; AI binding tên `AI`.

### 5. Mật khẩu
Trong Worker > Settings > Variables and Secrets:
- Add secret name: `APP_PASSWORD`
- Value: tự đặt mật khẩu dài, riêng tư.
- Save và deploy lại nếu được yêu cầu.

### 6. Mở website
Mở URL `*.workers.dev` được Cloudflare cấp. Nhập `APP_PASSWORD`.

## Thay ảnh chân dung Miyuki
Mặc định `public/index.html` dùng ảnh minh họa SVG nhúng trong code để trang chạy ngay. Nếu muốn ảnh anime riêng:
1. Upload ảnh bạn có quyền sử dụng vào `public/miyuki.png`.
2. Trong `index.html`, thay `$("portrait").src=avatarUrl;$("headAvatar").src=avatarUrl;` bằng:
   `$("portrait").src="/miyuki.png";$("headAvatar").src="/miyuki.png";`
3. Commit, Cloudflare tự deploy lại.

## Cách dùng
- Chat: nhập tin nhắn và Enter.
- Nhập vai chạm: nhấn Nắm tay / Ôm em / Vuốt tóc / Chạm má.
- POV: có thể nhắn "hãy nhập vai như em đang ngồi đối diện anh..." hoặc dùng các nút tương tác.
- Tạo ảnh: nhấn Tạo ảnh, mô tả nhân vật trưởng thành và trang phục.
- Bộ nhớ: hệ thống tự trích xuất sở thích/thông tin dài hạn từ lời người dùng; vào Ký ức để xem và xóa.
- Xóa lịch sử: endpoint đã có; nút giao diện hiện không hiển thị để tránh bấm nhầm. Có thể bổ sung sau.

## An toàn và riêng tư
- Chỉ dùng nhân vật trưởng thành hư cấu; hình ảnh tạo ra phải có trang phục, không khiêu dâm.
- Không đưa API token, mật khẩu cá nhân hoặc thông tin nhạy cảm vào chat.
- `APP_PASSWORD` là lớp khóa dùng chung cho bản cá nhân, không phải hệ thống tài khoản đa người dùng. Ai có mật khẩu có thể truy cập cùng lịch sử và ký ức.
- Nên bật Private repository. Không đưa secrets vào source code.
- Đây là mã nguồn mẫu có thể triển khai, không phải dịch vụ được kiểm thử trên tài khoản Cloudflare của bạn. Hãy kiểm tra logs nếu model/limit/binding thay đổi.

## Xử lý lỗi
- 401: kiểm tra secret `APP_PASSWORD`.
- `no such table`: chạy `schema.sql` trong đúng database D1.
- D1 binding error: tên binding phải là `DB`.
- AI binding error: tên binding phải là `AI`, model có thể không còn khả dụng ở một số khu vực/tài khoản.
- 429/limit: đã hết hạn mức, đợi reset hoặc chọn model/gói khác.
- Nếu Git integration không đọc cấu hình: kiểm tra `wrangler.jsonc` ở thư mục gốc và deploy log.
