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

## Cài đặt môi trường linux(Docker desktop, Cài WSL + Ubuntu)

-Docker:https://www.docker.com/products/docker-desktop/

-powerShell chạy:wsl --install
 -- linux:
<img width="837" height="367" alt="image" src="https://github.com/user-attachments/assets/62232e39-3564-4808-b9f9-7a4681137965" />


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

## Tạo cây thư mục

<img width="386" height="239" alt="image" src="https://github.com/user-attachments/assets/9a1fb518-9fc0-4492-9115-e8660c7a8fb7" />

## Tạo file .yml để cài đặt container sau: 

   mariadb (3306), phpmyadmin (8080), nodered/node-red (1880), influxdb (8086), grafana/grafana (3000), nginx (80,443)
   
## Chèn user + products

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/95e226ac-ec69-4cb0-9cb9-cbe0f4ecc0ac" />

## Thiết lập Cơ sở dữ liệu MariaDB


-Tạo Bảng Dữ liệu:

-- Tạo bảng user:

<img width="1748" height="604" alt="image" src="https://github.com/user-attachments/assets/1913bfb2-6f7e-4276-a444-1b0be073d33c" />

--Tạo bảng nhóm sản phẩm:

<img width="1395" height="577" alt="image" src="https://github.com/user-attachments/assets/ff634151-602e-432b-828d-2ddb5cc4b8a4" />

--Tạo bẩng sản phẩm

<img width="1899" height="403" alt="image" src="https://github.com/user-attachments/assets/712f2f78-1c13-4045-a35b-f37c689b921f" />

--Bảng Đơn hàng (orders)  Chi tiết (order_details)

<img width="1919" height="462" alt="image" src="https://github.com/user-attachments/assets/146264b3-deea-4f77-b316-bda93a7eb149" />

--Chi tiết (order_details):

<img width="1194" height="391" alt="image" src="https://github.com/user-attachments/assets/048deeb9-0549-41ee-8899-23eac3c37e86" />

## Xây dựng Backend API (Node-RED)

-Cấu hình kết nối:

<img width="712" height="946" alt="image" src="https://github.com/user-attachments/assets/4a14e103-0561-48b1-b714-078cd18770c9" />

-Triển khai các API Endpoint:

--Tạo API Lấy danh sách Sản phẩm:

<img width="1035" height="149" alt="image" src="https://github.com/user-attachments/assets/6df6a2e4-3fa7-44f1-ac73-9c96b28b9b8f" />

-- Đã hiển thị danh sách sản phẩm:

<img width="878" height="375" alt="image" src="https://github.com/user-attachments/assets/36cfba0b-7b91-43bf-b149-1a79882f510c" />

--Xây dựng API Đăng nhập An toàn:

Cài đặt Node Mã hóa Mật khẩu

Tạo Mật khẩu Mã hóa cho Admin

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

--Xây dựng API đăng kí và đăng nhập:

<img width="1502" height="447" alt="image" src="https://github.com/user-attachments/assets/60a3d657-c97f-4a69-8222-1cb9627b5662" />

--Xây dựng API Lấy Danh mục/Nhóm Sản phẩm

-- Xây dựng API Giỏ hàng và Sản phẩm (Node-RED)

<img width="1016" height="147" alt="image" src="https://github.com/user-attachments/assets/ba10efaa-cc49-478a-9f56-afa303d1f8a5" />

--Xây dựng API Lấy Sản phẩm

<img width="1047" height="100" alt="image" src="https://github.com/user-attachments/assets/2a711559-0ba5-4f53-9f28-3eef3db127c0" />

--Xây dựng API Đặt hàng

<img width="1683" height="152" alt="image" src="https://github.com/user-attachments/assets/732c0387-5bda-477c-b7d1-bf5540671cb5" />

--Tổng thể API

<img width="1644" height="587" alt="image" src="https://github.com/user-attachments/assets/aea05222-dd8d-4b7c-878f-98f7a4953152" />


## Sửa file host để Cấu hình Tên miền Giả định

<img width="835" height="864" alt="image" src="https://github.com/user-attachments/assets/81c99ae5-1008-44ea-8663-6fd8fa1ac814" />

## Xây dựng Frontend

-Kết nối API:

## Kết quả giao diện

## Giao diện đăng ký và đăng nhập
<img width="1134" height="682" alt="image" src="https://github.com/user-attachments/assets/2675b0b6-9a16-479b-98b2-63c4dcb934af" 
 
-- Khi đăng kí db sẽ hiển thị ngay tài khoản vừa đăng kí

<img width="1096" height="37" alt="image" src="https://github.com/user-attachments/assets/26e97702-2815-4bef-9de4-b337a66db283" />
-- Đăng nhập thành công

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3fa23216-81d5-4e92-965a-bcdd49f35943" />

## Giao diện chính

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/79f34778-ee57-429a-be6d-027e2c12a3c9" />

## Chức năng lọc sản phẩm theo nhóm sản phẩm

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/8b2e9694-18ed-448f-98f0-747e23051cbb" />

## Chức năng tìm kiếm theo tên sản phẩm

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/18a6755b-94ec-4cd0-b23a-bf7b6d35f901" />

## Chức năng thêm vào giỏ hàng, khi ấn vào giỏ hàng sẽ hiện sản phẩm đã thêm và có thể thêm bớt số lượng sản phẩm

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/f8e9fed5-7bd1-4f07-9fd0-9fb4a38585c0" />

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d55dd957-0685-41c1-a51d-f76173dbf1f7" />

## Chức năng đặt hàng, điền thông tin nhận hàng và nhấn xác nhận đặt hàng

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/6379baef-ad78-4290-8036-fbea8f18c062" />

## Sau khi nhấn nút xác nhận đặt hàng hệ thống sẽ hiển thị thông báo đặt hàng thành công

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/823a754b-8b95-4a70-810b-ae92c89504db" />
