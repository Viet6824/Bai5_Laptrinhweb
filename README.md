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

Dự án xây dựng một hệ thống giám sát và cảnh báo giá vàng 24K theo thời gian thực (Gold Price Monitor & Alert Realtime) chạy trên nền tảng Docker Container.

Hệ thống có nhiệm vụ tự động thu thập giá vàng từ nguồn dữ liệu trực tuyến, lưu trữ dữ liệu tức thời và dữ liệu lịch sử, đồng thời gửi cảnh báo đến Telegram khi giá vàng tăng hoặc giảm vượt ngưỡng được thiết lập.

Các thành phần chính của hệ thống bao gồm:

* **Node-RED:** Đóng vai trò trung tâm điều phối dữ liệu (ETL Workflow). Hệ thống định kỳ lấy dữ liệu giá vàng từ API, xử lý dữ liệu, ghi vào cơ sở dữ liệu và kích hoạt cảnh báo Telegram khi phát hiện biến động bất thường.
* **MariaDB:** Cơ sở dữ liệu quan hệ dùng để lưu giá vàng mới nhất nhằm phục vụ các truy vấn thời gian thực từ giao diện Web.
* **InfluxDB:** Cơ sở dữ liệu chuỗi thời gian dùng để lưu trữ lịch sử biến động giá vàng phục vụ việc phân tích xu hướng.
* **Flask API:** Dịch vụ API được xây dựng bằng Python, kết nối tới MariaDB và cung cấp dữ liệu giá vàng mới nhất dưới dạng JSON.
* **Nginx:** Web Server phục vụ giao diện Frontend và Reverse Proxy chuyển tiếp request đến Flask API.
* **Grafana:** Công cụ trực quan hóa dữ liệu, kết nối tới InfluxDB để hiển thị biểu đồ lịch sử giá vàng theo thời gian.
* **Telegram Bot:** Hệ thống cảnh báo tự động khi giá vàng vượt ngưỡng được cấu hình.

---

## 1.2. Cấu trúc thư mục dự án

```text id="g1"
gold-monitor/
├── docker-compose.yml
│
├── nginx/
│   ├── default.conf
│   └── html/
│       └── index.html
│
├── flask_api/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
│
├── mariadb_data/
├── influxdb_data/
├── grafana_data/
└── nodered_data/
```

---

## 1.3. Danh sách Service và cổng sử dụng

| Tên Service   | Port Container | Port Host    | Chức năng                   |
| ------------- | -------------- | ------------ | --------------------------- |
| web_nginx     | 80             | 8085         | Giao diện người dùng        |
| gold_flask    | 5000           | Không public | API giá vàng                |
| gold_nodered  | 1880           | 1882         | Thu thập dữ liệu & Telegram |
| gold_mariadb  | 3306           | 3309         | Dữ liệu tức thời            |
| gold_influxdb | 8086           | 8089         | Dữ liệu lịch sử             |
| gold_grafana  | 3000           | 3002         | Dashboard biểu đồ           |

Tất cả các Service được kết nối thông qua mạng Docker nội bộ có tên:

```text id="g2"
gold_network
```

Nhờ đó các Container có thể giao tiếp với nhau thông qua tên Service thay vì địa chỉ IP.

Ví dụ:

```python id="g3"
host='gold_mariadb'
```

thay vì:

```python id="g4"
host='192.168.x.x'
```

---

# 2. Cấu hình hệ thống

## 2.1. Cấu hình docker-compose.yml

Toàn bộ hệ thống được triển khai thông qua Docker Compose.

### Khai báo Network

```yaml id="g5"
networks:
  gold_network:
    driver: bridge
```

Network này cho phép tất cả các Service giao tiếp với nhau trong môi trường Docker.

---

### Service MariaDB

MariaDB được sử dụng để lưu dữ liệu giá vàng mới nhất.

```yaml id="g6"
gold_mariadb:
  image: mariadb:10.11
  container_name: gold_mariadb

  environment:
    MYSQL_ROOT_PASSWORD: rootpassword
    MYSQL_DATABASE: gold_db
    MYSQL_USER: gold_user
    MYSQL_PASSWORD: gold_password

  ports:
    - "3309:3306"

  volumes:
    - ./mariadb_data:/var/lib/mysql

  networks:
    - gold_network

  restart: always
```

---

### Service InfluxDB

InfluxDB dùng để lưu trữ dữ liệu lịch sử giá vàng.

```yaml id="g7"
gold_influxdb:
  image: influxdb:1.8

  environment:
    - INFLUXDB_DB=gold_history
    - INFLUXDB_ADMIN_USER=admin
    - INFLUXDB_ADMIN_PASSWORD=adminpassword
```

---

### Service Node-RED

Node-RED thực hiện nhiệm vụ:

* Lấy dữ liệu giá vàng từ API.
* Chuẩn hóa dữ liệu.
* Lưu MariaDB.
* Lưu InfluxDB.
* Gửi Telegram Alert.

```yaml id="g8"
gold_nodered:
  image: nodered/node-red:latest

  ports:
    - "1882:1880"

  volumes:
    - ./nodered_data:/data
```

---

### Service Grafana

Grafana dùng để hiển thị biểu đồ giá vàng theo thời gian.

```yaml id="g9"
gold_grafana:
  image: grafana/grafana:latest

  ports:
    - "3002:3000"

  environment:
    - GF_SECURITY_ALLOW_EMBEDDING=true
    - GF_AUTH_ANONYMOUS_ENABLED=true
```

---

### Service Flask API

Flask API được build từ source code trong thư mục flask_api.

```yaml id="g10"
gold_flask:
  build: ./flask_api

  container_name: gold_flask

  networks:
    - gold_network

  depends_on:
    - gold_mariadb

  restart: always
```

---

### Service Nginx

Nginx phục vụ giao diện người dùng và Reverse Proxy.

```yaml id="g11"
web_nginx:
  image: nginx:latest

  ports:
    - "8085:80"

  volumes:
    - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    - ./nginx/html:/usr/share/nginx/html

  depends_on:
    - gold_flask
```

---

## 2.2. Luồng hoạt động của hệ thống

Quy trình hoạt động của hệ thống được mô tả như sau:

```text id="g12"
API Giá Vàng
      │
      ▼
  Node-RED
      │
 ┌────┴─────┐
 ▼          ▼
MariaDB   InfluxDB
 ▼          ▼
Flask     Grafana
 ▼          ▼
Nginx Dashboard
      │
      ▼
 Người dùng
```

Đồng thời Node-RED sẽ thực hiện kiểm tra các ngưỡng cảnh báo:

* Giá vàng vượt 120 triệu VNĐ/lượng.
* Giá vàng giảm xuống dưới 115 triệu VNĐ/lượng.

Khi điều kiện xảy ra:

```text id="g13"
Node-RED
    │
    ▼
Telegram Bot
    │
    ▼
Telegram Group
```

Tin nhắn cảnh báo sẽ được gửi ngay lập tức tới nhóm quản trị.

---

## Kết luận

Docker là công nghệ phổ biến giúp đóng gói và triển khai ứng dụng một cách hiệu quả. Việc sử dụng Docker giúp đảm bảo tính nhất quán giữa các môi trường, giảm thời gian triển khai, tiết kiệm tài nguyên và hỗ trợ tốt cho các quy trình DevOps hiện đại.
# 2.3. Xây dựng Flask API

## 2.3.1. Giới thiệu

Flask API được xây dựng nhằm cung cấp dữ liệu giá vàng mới nhất từ cơ sở dữ liệu MariaDB cho giao diện Web Dashboard.

Thay vì giao diện Web truy cập trực tiếp vào cơ sở dữ liệu, toàn bộ dữ liệu sẽ được lấy thông qua REST API. Cách tiếp cận này giúp tăng tính bảo mật, dễ bảo trì và dễ mở rộng hệ thống trong tương lai.

Trong dự án này Flask API thực hiện các chức năng:

* Kết nối MariaDB.
* Truy vấn giá vàng mới nhất.
* Chuyển đổi dữ liệu sang định dạng JSON.
* Cung cấp API cho Frontend.

---

## 2.3.2. Cấu trúc thư mục Flask API

```text
flask_api/
├── Dockerfile
├── requirements.txt
└── app.py
```

Trong đó:

| File             | Chức năng                 |
| ---------------- | ------------------------- |
| Dockerfile       | Build Flask Container     |
| requirements.txt | Danh sách thư viện Python |
| app.py           | Chương trình chính        |

---

## 2.3.3. Cài đặt thư viện

File requirements.txt:

```text
flask
flask-cors
pymysql
```

Ý nghĩa:

| Thư viện   | Chức năng                         |
| ---------- | --------------------------------- |
| Flask      | Xây dựng Web API                  |
| Flask-CORS | Cho phép truy cập API từ Frontend |
| PyMySQL    | Kết nối MariaDB                   |

---

## 2.3.4. Kết nối MariaDB

Thông tin kết nối:

```python
db_config = {
    "host": "gold_mariadb",
    "user": "gold_user",
    "password": "gold_password",
    "database": "gold_db"
}
```

Giải thích:

| Tham số  | Giá trị                          |
| -------- | -------------------------------- |
| host     | Tên Service MariaDB trong Docker |
| user     | Tài khoản truy cập               |
| password | Mật khẩu                         |
| database | CSDL giá vàng                    |

---

## 2.3.5. Xây dựng API lấy giá vàng mới nhất

Bảng dữ liệu:

```sql
gold_live
```

Cấu trúc:

```sql
CREATE TABLE gold_live(
    id INT AUTO_INCREMENT PRIMARY KEY,
    buy_price DOUBLE,
    sell_price DOUBLE,
    gold_price DOUBLE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

API:

```python
from flask import Flask, jsonify
from flask_cors import CORS
import pymysql

app = Flask(__name__)
CORS(app)

db_config = {
    "host": "gold_mariadb",
    "user": "gold_user",
    "password": "gold_password",
    "database": "gold_db"
}

@app.route('/api/gold')
def get_gold():

    conn = pymysql.connect(**db_config)

    cursor = conn.cursor()

    cursor.execute("""
        SELECT buy_price,
               sell_price,
               gold_price,
               created_at
        FROM gold_live
        ORDER BY id DESC
        LIMIT 1
    """)

    row = cursor.fetchone()

    conn.close()

    return jsonify({
        "buy_price": row[0],
        "sell_price": row[1],
        "gold_price": row[2],
        "timestamp": str(row[3])
    })

if __name__ == '__main__':
    app.run(
        host='0.0.0.0',
        port=5000
    )
```

---

## 2.3.6. Kết quả trả về

Khi truy cập:

```text
http://localhost/api/gold
```

API sẽ trả về:

```json
{
  "buy_price": 118000000,
  "sell_price": 119000000,
  "gold_price": 118500000,
  "timestamp": "2026-06-08 09:30:00"
}
```

Dữ liệu này sẽ được giao diện Dashboard sử dụng để hiển thị thông tin giá vàng theo thời gian thực.

---

## 2.3.7. Dockerfile cho Flask API

File Dockerfile:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python","app.py"]
```

Ý nghĩa các lệnh:

| Lệnh    | Chức năng        |
| ------- | ---------------- |
| FROM    | Chọn Image gốc   |
| WORKDIR | Thư mục làm việc |
| COPY    | Sao chép file    |
| RUN     | Cài thư viện     |
| EXPOSE  | Mở cổng          |
| CMD     | Chạy ứng dụng    |

Sau khi build thành công, Flask API sẽ chạy tại:

```text
http://gold_flask:5000
```

và sẵn sàng phục vụ dữ liệu cho Nginx và Frontend.

---

# 2.4. Cấu hình Nginx

## 2.4.1. Giới thiệu

Nginx được sử dụng làm Web Server cho hệ thống.

Nhiệm vụ của Nginx:

* Phục vụ giao diện Web Dashboard.
* Reverse Proxy tới Flask API.
* Quản lý kết nối HTTP.
* Tăng hiệu năng truy cập.

Mô hình hoạt động:

```text
User
 │
 ▼
Nginx
 │
 ├────────► Frontend
 │
 ▼
Flask API
 │
 ▼
MariaDB
```

---

## 2.4.2. Cấu trúc thư mục Nginx

```text
nginx/
├── default.conf
└── html/
    └── index.html
```

Trong đó:

| File         | Chức năng           |
| ------------ | ------------------- |
| default.conf | Cấu hình Nginx      |
| index.html   | Giao diện Dashboard |

---

## 2.4.3. Cấu hình Reverse Proxy

File:

```text
default.conf
```

Nội dung:

```nginx
server {

    listen 80;

    server_name localhost;

    root /usr/share/nginx/html;

    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {

        proxy_pass http://gold_flask:5000/;

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## 2.4.4. Giải thích cấu hình

### listen

```nginx
listen 80;
```

Nginx lắng nghe tại cổng 80.

---

### root

```nginx
root /usr/share/nginx/html;
```

Thư mục chứa giao diện Web.

---

### index

```nginx
index index.html;
```

Trang mặc định khi truy cập hệ thống.

---

### location /

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

Phục vụ giao diện Dashboard.

---

### location /api/

```nginx
location /api/
```

Tiếp nhận yêu cầu API.

Ví dụ:

```text
http://localhost/api/gold
```

---

### proxy_pass

```nginx
proxy_pass http://gold_flask:5000/;
```

Chuyển tiếp request tới Flask API Container.

Nginx sử dụng tên Service:

```text
gold_flask
```

để giao tiếp trong Docker Network.

---

## 2.4.5. Giao diện Dashboard

Trang Dashboard hiển thị:

* Giá vàng 24K hiện tại.
* Giá mua vào.
* Giá bán ra.
* Thời gian cập nhật cuối cùng.
* Trạng thái tăng hoặc giảm giá.

Frontend sẽ định kỳ gọi API:

```javascript
fetch('/api/gold')
```

sau mỗi 30 giây để cập nhật dữ liệu mới nhất từ hệ thống.

---

## 2.4.6. Kiểm tra hoạt động

Sau khi chạy:

```bash
docker compose up -d
```

Kiểm tra:

```bash
docker ps
```

Truy cập:

```text
http://localhost:8085
```

Kết quả:

* Nginx hiển thị Dashboard.
* Flask API cung cấp dữ liệu JSON.
* MariaDB lưu dữ liệu thời gian thực.
* Hệ thống sẵn sàng phục vụ người dùng.

```
```
# 2.5. Thiết kế giao diện Dashboard (HTML/CSS/JavaScript)

## 2.5.1. Giới thiệu

Giao diện Dashboard được xây dựng nhằm cung cấp cho người dùng khả năng theo dõi giá vàng 24K theo thời gian thực thông qua trình duyệt Web.

Dashboard có nhiệm vụ:

* Hiển thị giá vàng hiện tại.
* Hiển thị giá mua vào.
* Hiển thị giá bán ra.
* Hiển thị thời gian cập nhật gần nhất.
* Hiển thị trạng thái tăng hoặc giảm giá.
* Tự động cập nhật dữ liệu từ Flask API.

Frontend được xây dựng bằng:

* HTML5
* CSS3
* JavaScript (Fetch API)

Không sử dụng framework để giảm độ phức tạp và phù hợp với mục tiêu học tập.

---

## 2.5.2. Thiết kế giao diện

Dashboard gồm các thành phần chính:

### Header

Hiển thị tên hệ thống:

```text
GOLD PRICE MONITORING DASHBOARD
```

---

### Card Giá Vàng

Hiển thị:

* Giá vàng 24K hiện tại
* Giá mua vào
* Giá bán ra

---

### Card Trạng Thái

Hiển thị:

* Giá tăng
* Giá giảm
* Không thay đổi

---

### Card Thời Gian

Hiển thị thời gian cập nhật dữ liệu mới nhất.

---

## 2.5.3. Giao diện HTML

File:

```text
nginx/html/index.html
```

Nội dung:

```html
<!DOCTYPE html>
<html>

<head>

    <meta charset="UTF-8">

    <title>Gold Monitor Dashboard</title>

    <link rel="stylesheet" href="style.css">

</head>

<body>

    <div class="container">

        <h1>
            GOLD PRICE MONITORING DASHBOARD
        </h1>

        <div class="card">

            <h2>Giá vàng 24K</h2>

            <p id="gold_price">
                Loading...
            </p>

        </div>

        <div class="card">

            <h2>Giá mua vào</h2>

            <p id="buy_price">
                Loading...
            </p>

        </div>

        <div class="card">

            <h2>Giá bán ra</h2>

            <p id="sell_price">
                Loading...
            </p>

        </div>

        <div class="card">

            <h2>Trạng thái</h2>

            <p id="status">
                Waiting...
            </p>

        </div>

        <div class="card">

            <h2>Cập nhật lúc</h2>

            <p id="timestamp">
                Waiting...
            </p>

        </div>

    </div>

    <script src="script.js"></script>

</body>

</html>
```

---

## 2.5.4. Thiết kế CSS

File:

```text
style.css
```

Nội dung:

```css
body {

    font-family: Arial, sans-serif;

    background: #f4f6f9;

    margin: 0;

    padding: 0;
}

.container {

    width: 90%;

    margin: auto;

    text-align: center;
}

h1 {

    margin-top: 20px;

    color: #333;
}

.card {

    background: white;

    margin: 15px;

    padding: 20px;

    border-radius: 10px;

    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}

.card h2 {

    color: #444;
}

.card p {

    font-size: 28px;

    font-weight: bold;
}
```

---

## 2.5.5. Xây dựng JavaScript

File:

```text
script.js
```

JavaScript có nhiệm vụ:

* Gọi Flask API.
* Nhận dữ liệu JSON.
* Cập nhật Dashboard.
* Tự động refresh sau mỗi 30 giây.

---

### Biến lưu giá cũ

```javascript
let previousPrice = 0;
```

Biến này được sử dụng để xác định giá tăng hay giảm.

---

### Hàm lấy dữ liệu từ API

```javascript
async function loadGoldData() {

    const response =
        await fetch('/api/gold');

    const data =
        await response.json();

    document.getElementById(
        'gold_price'
    ).innerHTML =
        Number(data.gold_price)
        .toLocaleString('vi-VN');

    document.getElementById(
        'buy_price'
    ).innerHTML =
        Number(data.buy_price)
        .toLocaleString('vi-VN');

    document.getElementById(
        'sell_price'
    ).innerHTML =
        Number(data.sell_price)
        .toLocaleString('vi-VN');

    document.getElementById(
        'timestamp'
    ).innerHTML =
        data.timestamp;

    updateStatus(data.gold_price);
}
```

---

### Hàm xác định trạng thái

```javascript
function updateStatus(currentPrice) {

    let status =
        document.getElementById(
            'status'
        );

    if(previousPrice === 0){

        status.innerHTML =
            'Khởi tạo dữ liệu';
    }

    else if(currentPrice > previousPrice){

        status.innerHTML =
            'Giá tăng 📈';
    }

    else if(currentPrice < previousPrice){

        status.innerHTML =
            'Giá giảm 📉';
    }

    else{

        status.innerHTML =
            'Không thay đổi';
    }

    previousPrice =
        currentPrice;
}
```

---

### Tự động cập nhật dữ liệu

```javascript
loadGoldData();

setInterval(

    loadGoldData,

    30000

);
```

Ý nghĩa:

* Khi mở Dashboard sẽ gọi API ngay lập tức.
* Sau mỗi 30 giây hệ thống tự cập nhật dữ liệu mới.

---

## 2.5.6. Luồng hoạt động

Quá trình cập nhật dữ liệu được thực hiện như sau:

```text
Người dùng mở Dashboard
            │
            ▼
       JavaScript
            │
            ▼
     Fetch /api/gold
            │
            ▼
       Flask API
            │
            ▼
        MariaDB
            │
            ▼
      Trả JSON
            │
            ▼
     Cập nhật giao diện
```

---

## 2.5.7. Kết quả đạt được

Sau khi triển khai thành công:

* Người dùng truy cập địa chỉ:

```text
http://localhost:8085
```

* Dashboard hiển thị:

  * Giá vàng hiện tại.
  * Giá mua vào.
  * Giá bán ra.
  * Trạng thái tăng/giảm.
  * Thời gian cập nhật.

* Dữ liệu được cập nhật tự động mà không cần tải lại trang.

Giao diện đơn giản, dễ sử dụng và đáp ứng tốt yêu cầu theo dõi giá vàng 24K theo thời gian thực.
