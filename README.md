# b4_mobile
# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421
Lớp: 58KTPM
**Bài tập 04:**  
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
# 
## deadline : 23h59 ngày 25 tháng 5 năm 2026.

### SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:
1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file **docker-compose.yml** chứa: 
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)
- Phpmyadmin: sử dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1
- WordPress: sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường:  WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)
- Cloudflared: sử dụng **image: cloudflare/cloudflared:latest** , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose
- N8n : sử dụng **image: n8nio/n8n:latest**, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )
yesterday

Update btvn_04.md
2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :
yesterday

Create assignment for automating WordPress with N8N
- pull các images về và chạy chúng (up -d)
- Kiểm tra các service đã running ok (ko bị restart liên tục)
- Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)
- Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)
- Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!
- Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp
- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
- Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn **Phát triển ứng dụng với mã nguồn mở**
- Truy cập sub-domain3 để cấu hình n8n:
yesterday

Update btvn_04.md
  + tạo tài khoản admin : nhớ điền đúng email
  + Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
  + Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.
yesterday

Create assignment for automating WordPress with N8N
  + Create workflow  (home page => overview => Create workflow)
  + Add trigger node: tìm node: Telegram => OnMessage  ; cấu hình Credential: Set up Credential => cần Nhập Access Token
    + Access Token thì lấy ở Telegram qua việc chát với @BotFather
    + Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp
    + Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)
  + Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
    + Lấy API KEY tại trang: https://aistudio.google.com  => https://aistudio.google.com/api-keys
    + cần tạo project mới, sẽ lấy được API KEY
    + Nhập API Key lên giao diện n8n
yesterday

Revise AI Google Gemini integration steps
    + kéo thả **nội dung đã chát** với bot của telegram (phía bên trái) vào **nội dung phần PROMPT** kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)
yesterday

Create assignment for automating WordPress with N8N
    + Turn on Output Content as JSON : để kết quả trả về dạng json
    + Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?
  + Add (nối tiếp vào sau node Message a model) node: Code in JavaScript
yesterday

Revise AI Google Gemini integration steps
    + Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
yesterday

Create assignment for automating WordPress with N8N
```
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;
// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);
// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
yesterday

Update btvn_04.md
  title: cleanData.post_title,
  content: cleanData.post_content
yesterday

Create assignment for automating WordPress with N8N
};
```
  + Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post
    + Set up Credential: vào wp tại url: https://sub-domain1/wp-admin  => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential
    + Wordpress URL: điền giá trị https://sub-domain1/   (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
    + Ignore SSL Issues (Insecure): TURN ON
    + Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content
    + Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)
yesterday

Update btvn_04.md
+ PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger
yesterday

Create assignment for automating WordPress with N8N
   
+ Kết quả cuối cùng cần đặt được:
  + từ điện thoại, chát với telegram bot
  + nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.
  + f5 wordpress để thấy bài viết mới đã lên sóng.
+ Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được
+ Nhận xét thành quả đạt được!!!
# BÀI LÀM
## Bước 1: Docker ps
<img width="1478" height="442" alt="image" src="https://github.com/user-attachments/assets/beb4efd4-88c6-4ceb-98a6-8a21b78c49b0" />

<img width="1902" height="892" alt="Screenshot 2026-05-23 131843" src="https://github.com/user-attachments/assets/18d7eb6c-3f12-4bd5-8d82-705ce8278a2c" />
## Bước 2: File docker-compose.yml
<img width="1459" height="1003" alt="image" src="https://github.com/user-attachments/assets/dca88c2e-82ed-4f98-90c3-054324fb7c24" />

## Bước 3: Giao diện WordPress
<img width="1902" height="960" alt="image" src="https://github.com/user-attachments/assets/82dad49a-ac4b-4a44-89c9-9c9d4473475f" />
## Bước 4: Giao diện PhpMyAdmin
<img width="1908" height="914" alt="image" src="https://github.com/user-attachments/assets/5fcf64c3-24b4-4fa3-bd18-d52b25718c19" />

## Bước 5: Workflow n8n
https://n8n.nguyetlinh.id.vn/workflow/CxDJvSFE9l7z8YmV
<img width="1889" height="970" alt="image" src="https://github.com/user-attachments/assets/61d1b5e9-e210-44fc-984a-5f5ee914e02d" />

## Bước 6: Telegram bot
<img width="1080" height="2316" alt="image" src="https://github.com/user-attachments/assets/cbcf5e79-0065-4f2a-ae85-d17bf8029e40" />

## Bước 7: Bài đăng tự động
<img width="1900" height="908" alt="Screenshot 2026-05-25 201241" src="https://github.com/user-attachments/assets/e22b9e9c-14f4-4763-8a27-a6157e2a57bd" />  
## Bước 8: Log container hoạt động
<img width="1645" height="801" alt="image" src="https://github.com/user-attachments/assets/99de4ee1-e25e-4b8f-80a0-3b8ddbb6ac1a" />
