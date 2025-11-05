# BTVN3_WEB
# Bài tập 3   : môn Phát triển ứng dụng trên nền web
Giảng viên  : Đỗ Duy Cốp
Lớp học phần: 58KTPM
Ngày giao   : 2025-10-24 13:50
Hạn nộp     : 2025-11-05 00:00
--------------------------------------------------
Yêu cầu     : LẬP TRÌNH ỨNG DỤNG WEB trên nền linux
1. Cài đặt môi trường linux: SV chọn 1 trong các phương án
 - enable wsl: cài đặt docker desktop
 - enable wsl: cài đặt ubuntu
 - sử dụng Hyper-V: cài đặt ubuntu
 - sử dụng VMware : cài đặt ubuntu
 - sử dụng Virtual Box: cài đặt ubuntu
2. Cài đặt Docker (nếu dùng docker desktop trên windows thì nó có ngay)
3. Sử dụng 1 file docker-compose.yml để cài đặt các docker container sau: 
   mariadb (3306), phpmyadmin (8080), nodered/node-red (1880), influxdb (8086), grafana/grafana (3000), nginx (80,443)
4. Lập trình web frontend+backend:
 SV chọn 1 trong các web sau:
 4.1 Web thương mại điện tử
 - Tạo web dạng Single Page Application (SPA), chỉ gồm 1 file index.html, toàn bộ giao diện do javascript sinh động.
 - Có tính năng login, lưu phiên đăng nhập vào cookie và session
   Thông tin login lưu trong cơ sở dữ liệu của mariadb, được dev quản trị bằng phpmyadmin, yêu cầu sử dụng mã hoá khi gửi login.
   Chỉ cần login 1 lần, bao giờ logout thì mới phải login lại.
 - Có tính năng liệt kê các sản phẩm bán chạy ra trang chủ
 - Có tính năng liệt kê các nhóm sản phẩm
 - Có tính năng liệt kê sản phẩm theo nhóm
 - Có tính năng tìm kiếm sản phẩm
 - Có tính năng chọn sản phẩm (đưa sản phẩm vào giỏ hàng, thay đổi số lượng sản phẩm trong giỏ, cập nhật tổng tiền)
 - Có tính năng đặt hàng, nhập thông tin giao hàng => được 1 đơn hàng.
 - Có tính năng dành cho admin: Thống kê xem có bao nhiêu đơn hàng, call để xác nhận và cập nhật thông tin đơn hàng. chuyển cho bộ phận đóng gói, gửi bưu điện, cập nhật mã COD, tình trạng giao hàng, huỷ hàng,...
 - Có tính năng dành cho admin: biểu đồ thống kê số lượng mặt hàng bán được trong từng ngày. (sử dụng grafana)
 - backend: sử dụng nodered xử lý request gửi lên từ javascript, phản hồi về json.
 4.2 Web IOT: Giám sát dữ liệu IOT.
 - Tạo web dạng Single Page Application (SPA), chỉ gồm 1 file index.html, toàn bộ giao diện do javascript sinh động.
 - Có tính năng login, lưu phiên đăng nhập vào cookie và session
   Thông tin login lưu trong cơ sở dữ liệu của mariadb, được dev quản trị bằng phpmyadmin, yêu cầu sử dụng mã hoá khi gửi login.
   Chỉ cần login 1 lần, bao giờ logout thì mới phải login lại.
 - hiển thị giá trị mới nhất của các thông số đang giám sát, khi click vào thì hiển thị đồ thị lịch sử quá trình thay đổi (gọi grafana iframe để hiển thị)
 - backend: Sử dụng nodered để đọc dữ liệu từ các cảm biến (có thể dùng api online để lấy dữ liệu theo giời gian thực), 
   nodered sẽ lưu dữ liệu mới nhất (dạng update) vào cơ sở dữ liệu mariadb (sử dụng phpmyadmin để tạp table và quản trị lần đầu)
   nodered sẽ lưu dữ liệu (insert) vào influxdb để lưu giá trị lịch sử, để cho grafana dùng để hiển thị biểu đồ.
5. Nginx làm web-server
 - Cấu hình nginx để chạy được website qua url http://fullname.com  (thay fullname bằng chuỗi ko dấu viết liền tên của bạn)
 - Cấu hình nginx để http://fullname.com/nodered truy cập vào nodered qua cổng 80, (dù nodered đang chạy ở port 1880)
 - Cấu hình nginx để http://fullname.com/grafana truy cập vào grafana qua cổng 80, (dù grafana đang chạy ở port 3000)

Yêu cầu sinh viên lưu code trên github
có file readme.md có hình ảnh + text: ghi lại nhật ký quá trình làm bài.

# Quá trình thực hiện

## Cài đặt môi trường(Docker desktop, Cài WSL + Ubuntu)

-Docker:https://www.docker.com/products/docker-desktop/

-powerShell chạy:wsl --install

## Chuẩn bị MariaDB (schema, admin user, sample products)
## File SQL khởi tạo trong Ubuntu
## init.sql:
CREATE DATABASE IF NOT EXISTS ecommerce;
USE ecommerce;

CREATE TABLE IF NOT EXISTS users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(100) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  fullname VARCHAR(255),
  role ENUM('user','admin') DEFAULT 'user',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS products (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  price DECIMAL(10,2) NOT NULL DEFAULT 0,
  group_name VARCHAR(100),
  sold_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS orders (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  customer_name VARCHAR(255),
  address TEXT,
  phone VARCHAR(50),
  total DECIMAL(10,2),
  status VARCHAR(50) DEFAULT 'new',
  cod_code VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS order_items (
  id INT AUTO_INCREMENT PRIMARY KEY,
  order_id INT,
  product_id INT,
  qty INT,
  price DECIMAL(10,2)
);

CREATE TABLE IF NOT EXISTS sessions (
  id VARCHAR(128) PRIMARY KEY,
  user_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
## Tạo tài khoản admin

cần 1 bcrypt hash cho mật khẩu admin123

Cài Node & bcryptjs:
sudo apt update
sudo apt install -y nodejs npm

Tạo thư mục làm việc:
mkdir -p ~/bcrypt-test && cd ~/bcrypt-test

Khởi tạo npm và cài bcryptjs:
npm init -y
npm install bcryptjs

Tạo hash bằng 1 dòng lệnh Node:
node -e 'const b=require("bcryptjs"); b.hash("admin123",10,(e,h)=>{if(e)console.error(e); else console.log(h)});'

Hash cho mật khẩu admin123:$2b$10$xq0f1iinANkYqVyIDSx7N.4O/J.aC26OeM6z1g5VGA8Pqcw9YvmxG
## Chèn user + products
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/95e226ac-ec69-4cb0-9cb9-cbe0f4ecc0ac" />

# Node-RED: cài palette & tạo flow step-by-step
## Thiết lập MySQL config (global configuration for mysql node)
Mysql:<img width="734" height="770" alt="image" src="https://github.com/user-
       attachments/assets/ce02208a-c67e-4e23-a5f3-41d1a67d1973" />
## Endpoint: POST /api/login
Mục tiêu: client gửi {username,password} → Node-RED truy users table → compare bcrypt → tạo session id, lưu sessions table, trả Set-Cookie header session=...; HttpOnly.

Nodes to place (left→right):

http in (method POST, url /api/login)

function (name: buildQuery)

mysql (connected to MariaDB config)

bcrypt (compare) — node from node-red-contrib-bcryptjs

function (name: onSuccessOrFail)

http response

1) http in

Drag http in node.

Double click:

Method: POST

URL: /api/login

Name: POST /api/login

Done

## Thiết lập Cơ sở dữ liệu MariaDB

-
-Tạo Bảng Dữ liệu:

-- Tạo bảng user
--Tạo bảng nhóm sản phẩm
<img width="1123" height="564" alt="image" src="https://github.com/user-attachments/assets/209b440f-2442-4e1d-9419-3a3073f5239e" />
--Tạo bẩng sản phẩm
<img width="905" height="552" alt="image" src="https://github.com/user-attachments/assets/3ac66699-58f3-4c81-adc6-a3350e07128e" />
--Bảng Đơn hàng (orders) & Chi tiết (order_details)
<img width="1207" height="723" alt="image" src="https://github.com/user-attachments/assets/c75bf817-0a4f-4472-80e1-bfebd0768139" />
--Thêm dữ liệu

## Xây dựng Backend API (Node-RED)
-Cấu hình kết nối::
<img width="712" height="946" alt="image" src="https://github.com/user-attachments/assets/4a14e103-0561-48b1-b714-078cd18770c9" />
-Triển khai các API Endpoint:
--Tạo API Lấy danh sách Sản phẩm:
<img width="1035" height="149" alt="image" src="https://github.com/user-attachments/assets/6df6a2e4-3fa7-44f1-ac73-9c96b28b9b8f" />
-- Đã hiển thị danh sách sản phẩm:
<img width="878" height="375" alt="image" src="https://github.com/user-attachments/assets/36cfba0b-7b91-43bf-b149-1a79882f510c" />

--Xây dựng API Đăng nhập An toàn:
Cài đặt Node Mã hóa Mật khẩu
Tạo Mật khẩu Mã hóa cho Admin:
Tạo Flow Hash Mật khẩu->lấy hash->Cập nhật CSDL

<img width="1165" height="364" alt="image" src="https://github.com/user-attachments/assets/1cfb668a-82c9-4333-a802-0e7e0643e1f5" />
-- Mật khẩu mã hóa: Admin123($2a$10$WPFVHhJelqS/PElZOnzBlORUb.Uvzf/KO2waJNzEyfIN.tiij8Wku
)
<img width="675" height="513" alt="image" src="https://github.com/user-attachments/assets/b857dfca-5551-4d40-8e9a-fc6c7de82ab0" />

--Tạo Flow API Đăng nhập:
vào setting.js thêm lệnh:
//session: {
    secret: "mot_chuoi_bi_mat_rat_dai_va_kho_doan", // Thay bằng chuỗi bí mật của riêng bạn
    // resave: false,
    // saveUninitialized: true
//},
<img width="990" height="142" alt="image" src="https://github.com/user-attachments/assets/2544af8b-3c13-43a7-bb33-27f53faca223" />

đăng nhập
<img width="1714" height="509" alt="image" src="https://github.com/user-attachments/assets/a644c7e8-c525-4874-bd74-9bc1e376a869" />
-- Xây dựng API Giỏ hàng và Sản phẩm (Node-RED)




## Xây dựng Frontend

-Kết nối API:
