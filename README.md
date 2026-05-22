# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421

Lớp: 58KTPM

**Bài tập 04:**  
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
# 
## deadline : 23h59 ngày 25 tháng 5 năm 2026.
## Link gửi bài: [Tại đây](https://docs.google.com/spreadsheets/d/1zftQMj748nRpS-_br4_jdHZocNVvo848zqxCGcTy4uU)

### SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:

1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file **docker-compose.yml** chứa: 
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)
- Phpmyadmin: sử dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1
- WordPress: sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường:  WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)
- Cloudflared: sử dụng **image: cloudflare/cloudflared:latest** , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose
- N8n : sử dụng **image: n8nio/n8n:latest**, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :
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
  + tạo tài khoản admin : nhớ điền đúng email
  + Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
  + Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.
  + Create workflow  (home page => overview => Create workflow)
  + Add trigger node: tìm node: Telegram => OnMessage  ; cấu hình Credential: Set up Credential => cần Nhập Access Token
    + Access Token thì lấy ở Telegram qua việc chát với @BotFather
    + Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp
    + Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)
  + Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
    + Lấy API KEY tại trang: https://aistudio.google.com  => https://aistudio.google.com/api-keys
    + cần tạo project mới, sẽ lấy được API KEY
    + Nhập API Key lên giao diện n8n
    + kéo thả **nội dung đã chát** với bot của telegram (phía bên trái) vào **nội dung phần PROMPT** kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)
    + Turn on Output Content as JSON : để kết quả trả về dạng json
    + Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?
  + Add (nối tiếp vào sau node Message a model) node: Code in JavaScript
    + Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
```
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```

  + Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post
    + Set up Credential: vào wp tại url: https://sub-domain1/wp-admin  => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential
    + Wordpress URL: điền giá trị https://sub-domain1/   (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
    + Ignore SSL Issues (Insecure): TURN ON
    + Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content
    + Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)
+ PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger
   
+ Kết quả cuối cùng cần đặt được:
  + từ điện thoại, chát với telegram bot
  + nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.
  + f5 wordpress để thấy bài viết mới đã lên sóng.

+ Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được
+ Nhận xét thành quả đạt được!!!

5 container chạy ok 
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/e52a37f5-5040-4d03-ae67-e292f1fc5fc6" />  

Cấu hình cloudflare tunnel cho 3 subdomain
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/8f303f6b-9966-41e6-9506-370982413a4d" />

Giao diện php admin
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/2a37aee3-c75d-4222-8424-3caefd9e50a2" />

Giao diện wordpress
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/baf68b5d-3750-44cc-908b-f05ab7884c02" />

Giao diện n8n
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/edd73456-4aff-47b2-b958-95f08291b6c4" />

Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/29f9f59e-a7bb-4135-8dc0-cab266aea7dc" />

Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/5326407a-6d95-4f35-aade-b6e6f2e77437" />

Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/637924cf-1033-4c07-8b54-05f301b5411e" />

cấu hình n8n
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d28337ec-d290-4f3d-b785-793a12db07ed" />

demo kết quả cuối cùng:
chát với bot:
<img width="1125" height="2436" alt="image" src="https://github.com/user-attachments/assets/5f04a5ce-40d4-48a8-94e8-b81e71bed8a9" />

thông qua n8n
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/b849317e-93e7-405c-8802-ccc7cf7c52e8" />

Tự động đăng bài lên wordpress
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/6c814cf3-95c9-4617-b2a5-1292c6947adb" />

## III. KẾT LUẬN VÀ NHẬN XÉT THÀNH QUẢ ĐẠT ĐƯỢC

Qua quá trình nghiên cứu, triển khai thực nghiệm và cấu hình tích hợp hệ thống, dự án tự động hóa biên tập nội dung đã hoàn thành xuất sắc các mục tiêu đề ra với những thành quả cụ thể sau:

### 1. Về mặt Hạ tầng và Đóng gói Ứng dụng (Infrastructure & Containerization)
* **Triển khai thành công kiến trúc Microservices:** Hệ thống đã được đóng gói và vận hành độc lập, ổn định trên môi trường **Docker** mã nguồn mở thông qua `docker-compose`. Việc tách biệt các dịch vụ (n8n, WordPress, MySQL) giúp tối ưu hóa tài nguyên máy chủ và nâng cao tính chịu lỗi của hệ thống.
* **Bảo mật và expose dịch vụ an toàn:** Tích hợp thành công giải pháp **Cloudflare Tunnel**, cho phép ánh xạ các dịch vụ nội bộ (local containers) ra Internet công khai thông qua tên miền định danh riêng (`luongnam2004.id.vn`) chuẩn mã hóa SSL (`https://`) mà không cần mở cổng modem (Port Forwarding), hạn chế tối đa nguy cơ bị tấn công mạng.

### 2. Về mặt Tự động hóa và Tích hợp luồng dữ liệu (Workflow Automation)
* **Xây dựng luồng xử lý thời gian thực hoàn chỉnh:** Thiết lập thành công chuỗi tự động hóa khép kín gồm 4 mắt xích cốt lõi: 
  $$\text{Telegram Trigger} \rightarrow \text{Google Gemini API} \rightarrow \text{Node Code JavaScript} \rightarrow \text{WordPress REST API}$$
* **Xử lý và tối ưu hóa dữ liệu thô phức tạp:** Phát triển thành công các bộ lọc chuỗi bằng **JavaScript (Regex)** trong n8n để giải quyết triệt để các bài toán kỹ thuật lỗi dòng như: bóc tách dữ liệu lồng nhau, dọn sạch ký tự escape (`\\n`, `\\"`), lọc bỏ hoàn toàn chuỗi mã hóa suy nghĩ ngầm (`thoughtSignature`) của mô hình Generative AI và tự động vá lỗi cú pháp JSON.

### 3. Về mặt Sản phẩm thực tế (Product & Content Delivery)
* **Giao tiếp ngôn ngữ tự nhiên thông minh:** Hệ thống cho phép người dùng chỉ cần ra lệnh bằng văn bản thuần qua ứng dụng Telegram trên điện thoại, AI sẽ tự động phân tích ngữ cảnh để biên tập và thiết lập bố cục bài viết theo đúng kịch bản mong muốn.
* **Hiển thị đa phương tiện (Rich Content):** Các bài đăng được xuất bản tự động lên WordPress đạt độ hoàn thiện cao, chuẩn SEO, hiển thị mượt mà các cấu trúc dữ liệu từ bảng biểu tổng hợp (thẻ `<table>`), hình ảnh trực quan (thẻ `<img>`), cho đến các khối đa phương tiện nhúng động như video YouTube (`<iframe>`) và trình phát âm thanh (`<audio>`).

---

### 📌 Đánh giá tổng quan
Dự án đã chứng minh được tính thực tiễn cao của việc kết hợp các công cụ mã nguồn mở và Trí tuệ nhân tạo. Hệ thống vận hành đúng theo thiết kế logic, thời gian phản hồi từ lúc nhận lệnh qua Telegram đến khi bài viết lên sóng WordPress chỉ mất từ **5 đến 10 giây**, đạt hiệu suất tối ưu và có khả năng mở rộng mạnh mẽ cho các bài toán tự động hóa doanh nghiệp trong tương lai.
