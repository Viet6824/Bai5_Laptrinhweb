#  APP MONITOR + ALERT DATA REALTIME
- Môn học: Phát triển ứng dụng với mã nguồn mở - TEE0421
- Họ và tên: Nguyễn Đức Việt
- MSSV: K225480106075

---

# YÊU CẦU BÀI TẬP
## LÝ THUYẾT
```
+ docker là gì? 
+ các keyword được sử dụng trong docker-compose.yml
  để mô tả 1 service, network, volume,...
  liệt kê + ý nghĩa của từ khoá đó + ví dụ minh hoạ
+ ưu điểm khi triển app sử dụng docker là gì?
+ dùng docker: tạo app, test app OK trên laptop cá nhân
  giờ muốn triển khai app này trên máy chủ thật ko có internet
  thì các bước cần làm là?
```
## THỰC HÀNH ÁP DỤNG
```
sử dụng docker compose có nhiều serivce 
và các thành phần cần thiết để tạo thành ứng dụng:
 + nodered liên tục lấy dữ liệu từ nguồn nào đó (chứng khoán, thời tiết, giá vàng,...)
   nguồn thực tế, số liệu luôn động sau thời gian ngắn
 + nodered lưu trữ dữ liệu vào 2 database: mariadb để lưu giá trị tức thời
   lưu lịch sử vào influxdb
 + sử dụng grafana để trực quan hoá dữ liệu: vẽ biểu đồ
 + sử dụng nginx để làm webserver
   chạy 1 trang web html+js+css làm front-end
   js: lấy dữ liệu tức thời trong mariadb qua (ajax | socket) 
       gọi api (api tự build bằng Flask giống bt1)
       api trả về giá trị tức thời trong mariadb
       hiển thị lên web, auto hiển thị số mới khi thay đổi
   sử dụng iframe để gọi grafana
   hiển thị biểu đồ dữ liệu lịch sử của thông số đã lưu
 + QUAN SÁT DỮ LIỆU LỊCH SỬ => GIÁ TRỊ BẤT THƯỜNG
   (VD MIỀN A..B: OK, DƯỚI A: ALERT LOW, TRÊN B: ALERT HIGH)
 + nodered: kết hợp bot Telegram
   khi dữ liệu not OK, thì gửi tin nhắn từ bot => group trên telegram
   group đã add bot vào: (nhóm đã có 2 người), add thêm 1875746636 thành 3 người
   mỗi khi bot gửi dữ liệu vào nhóm: mọi member of group đều nhận đc
   nội dung alert: tường minh, có value gây alert

 xuất tất cả các container ra file nén.
 xoá mọi container đang chạy
 load lại các container  từ file nén để khôi phục các container đã xoá
```


# BÀI LÀM

# PHẦN 1: LÝ THUYẾT

## 1. Docker là gì?

Docker là một nền tảng mã nguồn mở được sử dụng để đóng gói, triển khai và vận hành ứng dụng trong các môi trường độc lập gọi là **Container**. Mỗi Container bao gồm mã nguồn, thư viện, dependencies và các cấu hình cần thiết để ứng dụng hoạt động.

Trong quá trình phát triển phần mềm thường xảy ra tình trạng ứng dụng chạy tốt trên máy lập trình viên nhưng khi triển khai lên máy chủ lại phát sinh lỗi do khác biệt về hệ điều hành, phiên bản thư viện hoặc cấu hình môi trường. Docker giúp giải quyết vấn đề này bằng cách đóng gói toàn bộ môi trường chạy vào Container, đảm bảo ứng dụng hoạt động nhất quán trên mọi hệ thống.

### Một số khái niệm quan trọng

| Khái niệm      | Ý nghĩa                                                |
| -------------- | ------------------------------------------------------ |
| Image          | Mẫu (template) dùng để tạo Container                   |
| Container      | Thực thể đang chạy được tạo từ Image                   |
| Dockerfile     | File mô tả các bước xây dựng Image                     |
| Docker Hub     | Kho lưu trữ Image trực tuyến                           |
| Volume         | Cơ chế lưu trữ dữ liệu độc lập với Container           |
| Network        | Mạng ảo cho phép các Container giao tiếp với nhau      |
| Docker Compose | Công cụ quản lý và triển khai nhiều Container cùng lúc |

---

## 2. Các keyword thường sử dụng trong file docker-compose.yml

File `docker-compose.yml` dùng để khai báo toàn bộ các dịch vụ (service), mạng (network) và vùng lưu trữ (volume) của hệ thống.

### 2.1. Cấu trúc cơ bản

```yaml
version: '3.8'

services:
  ...

networks:
  ...

volumes:
  ...
```

---

### 2.2. Các keyword mô tả Service

#### image

Dùng để chỉ định Image có sẵn để khởi tạo Container.

```yaml
services:
  nginx:
    image: nginx:latest
```

---

#### build

Dùng để build Image từ Dockerfile.

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
```

---

#### container_name

Đặt tên cụ thể cho Container.

```yaml
services:
  web:
    image: nginx
    container_name: web_server
```

---

#### ports

Ánh xạ cổng giữa Host và Container.

```yaml
services:
  web:
    image: nginx
    ports:
      - "80:80"
```

Ý nghĩa:

* Cổng 80 bên ngoài Host
* Kết nối tới cổng 80 bên trong Container

---

#### environment

Khai báo biến môi trường.

```yaml
services:
  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root123
```

---

#### env_file

Đọc biến môi trường từ file `.env`.

```yaml
services:
  mysql:
    env_file:
      - .env
```

---

#### volumes

Liên kết dữ liệu giữa Host và Container.

```yaml
services:
  mysql:
    volumes:
      - mysql_data:/var/lib/mysql
```

---

#### networks

Khai báo mạng mà Service sẽ tham gia.

```yaml
services:
  app:
    networks:
      - backend
```

---

#### depends_on

Xác định Service phụ thuộc.

```yaml
services:
  app:
    depends_on:
      - mysql
```

Trong ví dụ trên, MySQL sẽ được khởi động trước App.

---

#### restart

Chính sách tự khởi động lại Container.

```yaml
services:
  app:
    restart: always
```

Một số giá trị:

* always
* unless-stopped
* on-failure
* no

---

#### command

Ghi đè lệnh mặc định khi Container chạy.

```yaml
services:
  app:
    command: python app.py
```

---

#### expose

Mở cổng cho các Container khác trong cùng Network.

```yaml
services:
  api:
    expose:
      - "5000"
```

---

#### working_dir

Thiết lập thư mục làm việc mặc định.

```yaml
services:
  app:
    working_dir: /app
```

---

### 2.3. Các keyword mô tả Network

Ví dụ:

```yaml
networks:
  backend:
    driver: bridge
```

Các loại Network phổ biến:

| Driver | Ý nghĩa                          |
| ------ | -------------------------------- |
| bridge | Mạng mặc định giữa các Container |
| host   | Sử dụng trực tiếp mạng của Host  |
| none   | Không sử dụng mạng               |

---

### 2.4. Các keyword mô tả Volume

Ví dụ:

```yaml
volumes:
  mysql_data:
    driver: local
```

Các thuộc tính phổ biến:

| Keyword | Ý nghĩa                      |
| ------- | ---------------------------- |
| driver  | Loại Volume được sử dụng     |
| local   | Volume lưu trữ trên máy Host |

Volume giúp dữ liệu không bị mất khi Container bị xóa hoặc khởi động lại.

---

## 3. Ưu điểm khi triển khai ứng dụng bằng Docker

Docker mang lại nhiều lợi ích trong quá trình phát triển và vận hành phần mềm:

### Đồng nhất môi trường

Ứng dụng chạy giống nhau trên môi trường Development, Testing và Production.

### Cô lập dịch vụ

Mỗi dịch vụ hoạt động trong Container riêng biệt, tránh xung đột thư viện và cấu hình.

### Triển khai nhanh

Chỉ cần một lệnh:

```bash
docker compose up -d
```

là có thể khởi động toàn bộ hệ thống.

### Dễ dàng mở rộng

Có thể tăng số lượng Container khi cần xử lý nhiều yêu cầu hơn.

### Tiết kiệm tài nguyên

Container nhẹ hơn Virtual Machine do sử dụng chung Kernel của hệ điều hành Host.

### Dễ bảo trì

Việc cập nhật, rollback hoặc thay đổi phiên bản được thực hiện đơn giản thông qua Image.

### Hỗ trợ CI/CD

Docker tích hợp tốt với GitHub Actions, GitLab CI/CD và các hệ thống tự động hóa khác.

---

## 4. Triển khai ứng dụng Docker lên máy chủ không có Internet

Giả sử ứng dụng đã được phát triển và kiểm thử thành công trên máy cá nhân.

### Bước 1: Build hoặc Pull các Image

```bash
docker compose pull
docker compose build
```

---

### Bước 2: Export Image

Xuất Image ra file.

```bash
docker save my_app:latest -o my_app.tar
```

Hoặc nhiều Image:

```bash
docker save nginx mysql my_app | gzip > all_images.tar.gz
```

---

### Bước 3: Chuyển dữ liệu sang máy chủ

Có thể sử dụng:

* SCP
* USB
* Ổ cứng ngoài
* Mạng LAN

Ngoài các Image, cần copy thêm:

* docker-compose.yml
* Dockerfile
* File .env
* Source code
* Các file cấu hình

---

### Bước 4: Load Image trên máy chủ

```bash
docker load -i all_images.tar.gz
```

Kiểm tra:

```bash
docker images
```

---

### Bước 5: Khởi động hệ thống

```bash
docker compose up -d
```

---

### Bước 6: Kiểm tra trạng thái

```bash
docker compose ps
```

Xem log:

```bash
docker compose logs -f
```
# PHẦN 2: THỰC HÀNH

# 1. Tổng quan hệ thống

## 1.1. Giới thiệu

Dự án xây dựng một hệ thống giám sát và cảnh báo tự động theo thời gian thực (App Monitor + Alert Data Realtime) chạy trên nền tảng Docker Container. Đối tượng giám sát thực tế được lựa chọn là dữ liệu biến động giá vàng SJC.

Các thành phần chính trong hệ thống:

* **Node-RED:** Trung tâm điều phối dữ liệu (ETL Workflow), thực hiện lấy dữ liệu, phân tích theo ngưỡng Thấp/Cao, ghi dữ liệu đồng thời vào MariaDB và InfluxDB, đồng thời gửi cảnh báo Telegram khi vượt ngưỡng.

* **MariaDB:** Cơ sở dữ liệu quan hệ (RDBMS) lưu trạng thái giá vàng hiện tại nhằm phục vụ truy vấn nhanh từ Web.

* **InfluxDB 2.x:** Cơ sở dữ liệu chuỗi thời gian (Time-series Database) dùng lưu lịch sử biến động giá vàng.

* **Flask API:** Cung cấp API REST lấy dữ liệu mới nhất từ MariaDB.

* **Nginx:** Web Server phân phối giao diện Frontend và Reverse Proxy chuyển tiếp request đến Flask API.

* **Grafana:** Công cụ trực quan hóa dữ liệu, kết nối InfluxDB để hiển thị biểu đồ biến động giá vàng theo thời gian.

---

## 1.2. Cấu trúc thư mục dự án

```text
gold-monitor/
├── docker-compose.yml
├── flask_api/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
├── nginx/
│   ├── default.conf
│   └── html/
│       └── index.html
├── nodered_data/
├── mariadb_data/
├── influxdb_data/
└── grafana_data/
```

---

## 1.3. Danh sách Service

| Service Docker | Port Container | Port Host | Chức năng                 |
| -------------- | -------------: | --------: | ------------------------- |
| web_nginx      |             80 |      8085 | Frontend + Reverse Proxy  |
| gold_flask     |           5000 |      5000 | Flask API                 |
| gold_nodered   |           1880 |      1880 | Thu thập và xử lý dữ liệu |
| gold_mariadb   |           3306 |      3309 | CSDL quan hệ              |
| gold_influxdb  |           8086 |      8086 | CSDL Time-Series          |
| gold_grafana   |           3000 |      3000 | Dashboard trực quan       |

> Các container giao tiếp với nhau thông qua tên service Docker (ví dụ: `gold_mariadb`, `gold_influxdb`, `gold_flask`) thay vì sử dụng địa chỉ IP.

---

# 2. Cấu hình hệ thống

## 2.1. Cấu hình Docker Compose

File:
docker-compose.yml


```yaml
services:

  gold_mariadb:

    image: mariadb:11

    container_name: gold_mariadb

    restart: always

    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: gold_db

    ports:
      - "3309:3306"

    volumes:
      - ./mariadb_data:/var/lib/mysql


  gold_flask:

    build: ./flask_api

    container_name: gold_flask

    restart: always

    depends_on:
      - gold_mariadb

    ports:
      - "5000:5000"


  web_nginx:

    image: nginx:latest

    container_name: web_nginx

    restart: always

    ports:
      - "8085:80"

    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./nginx/html:/usr/share/nginx/html

    depends_on:
      - gold_flask


  nodered:

    image: nodered/node-red

    container_name: gold_nodered

    restart: always

    ports:
      - "1880:1880"

    environment:
      - TZ=Asia/Ho_Chi_Minh

    command: >
      npm start -- --userDir /data

    volumes:
      - ./nodered_data:/data

    depends_on:
      - gold_mariadb


  gold_influxdb:

    image: influxdb:2.7

    container_name: gold_influxdb

    restart: always

    ports:
      - "8086:8086"

    volumes:
      - ./influxdb_data:/var/lib/influxdb2


  gold_grafana:

    image: grafana/grafana

    container_name: gold_grafana

    restart: always

    ports:
      - "3000:3000"

    volumes:
      - ./grafana_data:/var/lib/grafana

    depends_on:
      - gold_influxdb
```

---

## 2.2. Flask API

### requirements.txt

```txt
Flask
Flask-Cors
mysql-connector-python
```

---

### app.py

```python
from flask import Flask, jsonify
from flask_cors import CORS
import mysql.connector

app = Flask(__name__)
CORS(app)

@app.route("/api/gold")
def get_gold():

    conn = mysql.connector.connect(
        host="gold_mariadb",
        user="root",
        password="root123",
        database="gold_db"
    )

    cursor = conn.cursor(dictionary=True)

    cursor.execute("""
        SELECT *
        FROM gold_live
        ORDER BY id DESC
        LIMIT 1
    """)

    row = cursor.fetchone()

    cursor.close()
    conn.close()

    return jsonify(row)

app.run(host="0.0.0.0", port=5000)
```

---

## 2.3. Reverse Proxy Nginx

File:

```text
nginx/default.conf
```

```nginx
server {

    listen 80;

    location / {

        root /usr/share/nginx/html;

        index index.html;
    }

    location /api/ {

        proxy_pass http://gold_flask:5000;

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

# 3. Node-RED

Node-RED đóng vai trò là trung tâm xử lý dữ liệu.

Luồng xử lý gồm:

```text
Inject
   ↓
Function
   ↓
 ┌───────────────┬─────────────┐
 ↓               ↓
MariaDB       InfluxDB
                   ↓
                Switch
             ┌────────┴────────┐
             ↓                 ↓
        Cảnh báo thấp      Cảnh báo cao
             ↓                 ↓
      Function format     Function format
             └────────┬────────┘
                      ↓
                 Telegram Bot
```

---

## 3.1. Function Node tạo dữ liệu

```javascript
let price = 11850000 + Math.floor(Math.random()*50000);

msg.topic =
"INSERT INTO gold_live(gold_price) VALUES(" + price + ")";

msg.payload = {
    price: price
};

return [msg,msg];
```

Output 1:

* Gửi sang MariaDB.

Output 2:

* Gửi sang InfluxDB.

---

## 3.2. MariaDB

Bảng:

```sql
gold_live
```

Cấu trúc:

```sql
DESCRIBE gold_live;
```

```text
+------------+-----------+------+-----+---------------------+----------------+
| Field      | Type      | Null | Key | Default             | Extra          |
+------------+-----------+------+-----+---------------------+----------------+
| id         | int(11)   | NO   | PRI | NULL                | auto_increment |
| buy_price  | double    | YES  |     | NULL                |                |
| sell_price | double    | YES  |     | NULL                |                |
| gold_price | double    | YES  |     | NULL                |                |
| created_at | timestamp | YES  |     | current_timestamp() |                |
+------------+-----------+------+-----+---------------------+----------------+
```

---

## 3.3. InfluxDB

Thông số cấu hình Node InfluxDB Out:

| Thuộc tính     | Giá trị                   |
| -------------- | ------------------------- |
| Version        | 2.0                       |
| URL            | http://gold_influxdb:8086 |
| Bucket         | 1                         |
| Measurement    | gold_price                |
| Time Precision | Milliseconds              |
| Organization   | f1ac141d8f8116f0          |

---

## 3.4. Bộ lọc cảnh báo

Node Switch chia thành hai nhánh:

### Nhánh 1: Cảnh báo thấp

Điều kiện:

```text
msg.payload < 132.32
```

---

### Nhánh 2: Cảnh báo cao

Điều kiện:

```text
msg.payload > 132.35
```

---

## 3.5. Function tạo cảnh báo Telegram

### Cảnh báo thấp

```javascript
let currentPrice = Number(msg.payload);

msg.payload = {
    chatId: -1001875746636,
    type: 'message',
    content:
    `⚠️ [CẢNH BÁO THẤP]

Giá vàng SJC đã giảm xuống ${currentPrice}`
};

return msg;
```

---

### Cảnh báo cao

```javascript
let currentPrice = Number(msg.payload);

msg.payload = {
    chatId: -1001875746636,
    type: 'message',
    content:
    `🚨 [CẢNH BÁO CAO]

Giá vàng SJC đã vượt ngưỡng ${currentPrice}`
};

return msg;
```

Sau đó hai nhánh được nối chung vào một node Telegram Sender để gửi thông báo về Group Telegram.
## 3.6: Chuẩn bị thông tin Telegram Bot

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/74aac72b-17db-41f9-a9cd-063162bfda85" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f7ae0a05-5f5b-408e-bd85-78adc1c7d651" />

---
# Kết quả 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c50ca23a-c3fc-47b5-b665-4181faacd5ff" />

# 4. Khởi động hệ thống

Build toàn bộ hệ thống:

```bash
docker compose up -d --build
```

Kiểm tra trạng thái container:

```bash
docker ps
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/087d11c0-01e5-4de9-85c5-75feea450e40" />

# 5. Dashboard Grafana

Grafana kết nối trực tiếp tới InfluxDB để hiển thị biểu đồ biến động giá vàng theo thời gian.

Các dữ liệu lịch sử được lưu dưới dạng Time-Series với:

* Measurement:

```text
gold_price
```

* Bucket:

```text
1
```

Dashboard Grafana được nhúng trực tiếp vào giao diện Web thông qua thẻ `iframe`, cho phép người dùng quan sát dữ liệu theo thời gian thực mà không cần đăng nhập Grafana.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0d4d3dcb-3237-48d0-8c52-8ed188eee90f" />

---

# 6. Kết quả đạt được

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/be44b318-3d1d-46b8-ad7e-fcbf3d41c6d2" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e83b7526-d4f3-4713-81fe-b81a4c19a7a2" />

Hệ thống cho phép:

* Thu thập dữ liệu giá vàng liên tục.
* Lưu dữ liệu đồng thời vào MariaDB và InfluxDB.
* Hiển thị dữ liệu thời gian thực trên Web.
* Vẽ biểu đồ lịch sử bằng Grafana.
* Gửi cảnh báo Telegram khi giá vàng vượt ngưỡng quy định.
* Triển khai toàn bộ trên Docker bằng nhiều container độc lập.
* Dễ dàng mở rộng và bảo trì.

```
```
# 7. Đóng gói và khôi phục hệ thống

### Bước 1: Sao lưu toàn bộ hệ thống

Thực hiện nén toàn bộ thư mục dự án để phục vụ cho việc sao lưu và di chuyển sang môi trường khác.

```bash
# Đứng tại thư mục Home
sudo tar -czvf gold_monitor_backup.tar.gz gold-monitor/
```

#### Giải nén tệp tin backup hệ thống giá vàng
sudo tar -xzvf gold_monitor_backup.tar.gz

### Bước 2: Dừng và xóa toàn bộ container

#### Truy cập vào thư mục vừa giải nén
cd gold-monito

##### Dừng toàn bộ các dịch vụ và xóa sạch container, network liên quan
docker compose down

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f97f904e-d970-4456-816b-379a1cc7926c" />

Sau khi thực hiện lệnh trên, toàn bộ các dịch vụ:

* MariaDB
* Flask API
* Node-RED
* InfluxDB
* Grafana
* Nginx

đều được dừng hoàn toàn.

Truy cập giao diện Web thi web sẽ không còn hoạt động, chứng minh hệ thống đã được dọn dẹp hoàn toàn.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5fa21218-2525-44d2-9d4f-86df6179258c" />

---

### Bước 3: Khôi phục lại từ file sao lưu

Giải nén lại thư mục dự án:

```bash
tar -xzvf gold_monitor_backup.tar.gz
```

Khởi động lại toàn bộ hệ thống:

```bash
cd gold-monitor

docker compose up -d
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6d882646-f1d6-4ff0-a2eb-c57e7114235f" />

Docker Compose sẽ tự động khởi tạo lại:

* gold_mariadb
* gold_flask
* gold_nodered
* gold_influxdb
* gold_grafana
* web_nginx

Do các thư mục dữ liệu đã được lưu trữ bên ngoài container nên:

* Dữ liệu MariaDB vẫn được giữ nguyên.
* Dữ liệu lịch sử InfluxDB không bị mất.
* Dashboard Grafana được bảo toàn.
* Flow Node-RED vẫn giữ nguyên trạng thái trước khi sao lưu.

Hệ thống được phục hồi hoàn chỉnh mà không cần cấu hình lại từ đầu.

---

### Bước 4: Kết quả sau khi khôi phục

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cea1e9e5-93b7-4d0d-b8de-b67bb3814696" />

Toàn bộ các chức năng tiếp tục hoạt động bình thường:

* Thu thập dữ liệu giá vàng SJC.
* Ghi dữ liệu vào MariaDB.
* Lưu dữ liệu lịch sử vào InfluxDB.
* Hiển thị Dashboard Web thời gian thực.
* Trực quan hóa dữ liệu bằng Grafana.
* Kích hoạt cảnh báo Telegram khi giá vàng vượt ngưỡng thiết lập.

---

# PHẦN 3: KẾT LUẬN
Hệ thống giám sát giá vàng SJC theo thời gian thực đã được xây dựng thành công trên nền tảng Docker Compose với các thành phần Node-RED, MariaDB, InfluxDB, Flask API, Nginx và Grafana.

Hệ thống cho phép thu thập dữ liệu tự động, lưu trữ đồng thời dữ liệu tức thời và dữ liệu lịch sử, trực quan hóa bằng Dashboard Web và Grafana, đồng thời hỗ trợ cơ chế cảnh báo tự động qua Telegram khi giá vàng vượt ngưỡng thiết lập.

Bên cạnh đó, việc đóng gói bằng Docker giúp hệ thống dễ dàng triển khai, sao lưu và khôi phục, đáp ứng tốt yêu cầu của một hệ thống giám sát dữ liệu thời gian thực.
