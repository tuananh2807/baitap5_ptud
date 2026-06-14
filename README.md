# I. Lý Thuyết
# 1. DOCKER LÀ GÌ?
- Docker là một nền tảng mã nguồn mở cho phép các lập trình viên và chuyên gia hệ thống tự động hóa việc triển khai, đóng gói và chạy các ứng dụng bên trong các môi trường ảo hóa gọn nhẹ được gọi là Container.

- Để hiểu Docker, chúng ta cần phân biệt rõ giữa hai khái niệm: Virtual Machine (Máy ảo truyền thống) và Docker Container:

- Máy ảo (VM - như VMware, VirtualBox): Mỗi máy ảo bao gồm cả ứng dụng, các thư viện cần thiết và toàn bộ một hệ điều hành khách (Guest OS) chạy phía trên phần mềm phân luồng (Hypervisor). Điều này khiến VM rất nặng (vài GB), khởi động lâu và tốn tài nguyên.

- Docker Container: Các container dùng chung nhân hệ điều hành (Kernel) của máy Host. Nó chỉ cô lập tầng ứng dụng và các thư viện phụ thuộc bằng công nghệ của Linux (Namespaces và Cgroups). Vì vậy, Container cực kỳ nhẹ (chỉ vài chục MB), khởi động trong vài giây và tiêu tốn rất ít tài nguyên phần cứng.

# 2. CÁC KEYWORD TRONG docker-compose.yml
File docker-compose.yml cấu trúc theo định dạng YAML dùng để định nghĩa và quản lý đa container (Multi-container). Dưới đây là các từ khóa cốt lõi phân theo từng thành phần:

A. Từ khóa cấp cao nhất (Root level)
services:Định nghĩa danh sách các container (dịch vụ) sẽ được khởi tạo trong hệ thống.
networks:Định nghĩa các mạng nội bộ ảo để các container có thể kết nối và truyền thông tin bảo mật với nhau.

volumes:Định nghĩa các vùng lưu trữ dữ liệu độc lập, giúp dữ liệu không bị mất đi khi các container bị xóa hoặc khởi động lại.

B. Từ khóa mô tả một Dịch vụ (Service Level)
Dưới đây là các từ khóa nằm bên dưới phân mục của từng Service cụ thể:

image: Chỉ định tên và phiên bản (tag) của Docker Image sẵn có trên Docker Hub dùng để dựng container.
Ví dụ: image: mariadb:10.6 (Sử dụng ảnh hệ trị cơ sở dữ liệu MariaDB bản 10.6).

container_name: Đặt tên cố định cho container khi khởi chạy thay vì để Docker tự sinh tên ngẫu nhiên. Giúp dễ quản lý và gõ lệnh log/restart.
Ví dụ: container_name: flask_service: Quy định chính sách tự động khởi động lại container nếu nó bị sập đột ngột hoặc khi máy chủ reboot.
restart" 
environment: Khai báo các biến môi trường (Environment Variables) dưới dạng cặp Key=Value truyền vào bên trong container để cấu hình ứng dụng (như mật khẩu DB, cấu hình port, v.v.).
Ví dụ:

```YAML
environment:
  - MYSQL_ROOT_PASSWORD=root_password
  - MYSQL_DATABASE=monitor_db
ports
```
volumes (Dạng Mount)" : Gắn kết một thư mục/file hoặc một Volume được định nghĩa sẵn vào một đường dẫn bên trong container để đồng bộ hoặc lưu trữ dữ liệu vĩnh viễn.
Ví dụ:

```YAML
volumes:
  - ./frontend:/usr/share/nginx/html  # Ánh xạ thư mục code ngoài máy thật vào Nginx
  - mariadb_data:/var/lib/mysql        # Ánh xạ vùng lưu dữ liệu của DB
networks
```
Ý nghĩa: Chỉ định container này tham gia vào mạng nội bộ ảo nào để nói chuyện được với các container khác cùng mạng.

Ví dụ:

```YAML
networks:
  - gold_monitor_net
depends_on
```
Ý nghĩa: Thiết lập thứ tự khởi chạy giữa các dịch vụ. Container này chỉ được bật lên sau khi các container được chỉ định trong danh sách đã khởi chạy trước.

Ví dụ: Thằng Backend API chỉ được chạy sau khi Database đã sẵn sàng:

```YAML
depends_on:
  - mariadb
build
```
Ý nghĩa: Sử dụng khi không lấy image có sẵn từ Docker Hub mà muốn Docker tự biên dịch (build) một image mới từ file Dockerfile cục bộ.

Ví dụ: build: ./backend (Tìm file Dockerfile trong thư mục backend để build).

C. Ví dụ minh họa cấu trúc tổng thể một file docker-compose.yml
```YAML
version: '3.8'

services:
  # Định nghĩa dịch vụ database
  mariadb:
    image: mariadb:10.6
    container_name: mariadb_service
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: monitor_db
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - gold_monitor_net

  # Định nghĩa dịch vụ hiển thị biểu đồ
  grafana:
    image: grafana/grafana:latest
    container_name: grafana_service
    restart: always
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ALLOW_EMBEDDING=true
    networks:
      - gold_monitor_net
    depends_on:
      - mariadb

# Định nghĩa các tài nguyên độc lập cấp Root
volumes:
  mariadb_data: # Khai báo volume lưu trữ DB vĩnh viễn

networks:
  gold_monitor_net: # Khai báo mạng dùng chung
    driver: bridge
```
# 3. ƯU ĐIỂM KHI TRIỂN KHAI APP SỬ DỤNG DOCKER
Tính nhất quán môi trường ("Build Once, Run Anywhere"): Giải quyết dứt điểm câu nói kinh điển của lập trình viên: "Code chạy ngon lành trên máy tôi nhưng lỗi trên máy chủ". Docker đóng gói toàn bộ code, thư viện, biến môi trường vào 1 Image nên chạy ở máy cá nhân thế nào thì mang lên máy chủ thật sẽ chạy y hệt như thế.

Tiết kiệm tài nguyên phần cứng tối đa: Do cơ chế chia sẻ chung nhân hệ điều hành (Kernel) nên một máy chủ vật lý có thể chạy hàng trăm Container cùng một lúc, tối ưu hơn rất nhiều so với việc chỉ chạy được vài máy ảo VM nặng nề.

Triển khai, nâng cấp cực nhanh: Việc khởi tạo, tắt, bật hay nâng cấp phiên bản ứng dụng chỉ diễn ra trong vài giây thông qua các lệnh đơn giản (docker compose up / down / restart), tăng tốc quy trình CI/CD và bàn giao sản phẩm.

Cô lập và bảo mật tốt: Mỗi container hoạt động độc lập trong môi trường Sandbox của riêng mình. Nếu một container ứng dụng (ví dụ Flask API) bị hacker tấn công chiếm quyền, mã độc cũng bị khóa chặt trong container đó và không thể tự do xâm nhập sang dữ liệu của Database hoặc hệ điều hành của máy chủ thật.

Dễ dàng mở rộng (Scalability): Rất dễ dàng nhân bản ứng dụng lên thành nhiều thực thể (scale up) để chia tải (Load Balancing) khi lượng người dùng tăng đột biến.

# 4. QUY TRÌNH TRIỂN KHAI ĐIỀU KIỆN OFFLINE (MÁY CHỦ KHÔNG CÓ INTERNET)<br>
Đây là bài toán thực tế rất phổ biến trong các doanh nghiệp lớn, ngân hàng hoặc cơ quan nhà nước nhằm bảo mật thông tin tối đa. Khi máy chủ thật hoàn toàn "mất mạng", bạn không thể dùng lệnh docker pull hay pip install trực tiếp. Quy trình xử lý sẽ gồm 4 bước như sau: <br>

Bước 1: Đóng gói toàn bộ Docker Images tại máy Laptop (Có Internet)<br>
Trên máy laptop cá nhân, nơi bạn đã kiểm thử ứng dụng chạy thành công (OK), bạn cần quét và xuất tất cả các Image cấu thành nên ứng dụng ra thành các file nén vật lý (.tar).<br>

Lệnh nén từng Image:<br>

```Bash<br>
docker save -o mariadb_image.tar mariadb:10.6<br>
docker save -o grafana_image.tar grafana/grafana:latest<br>
docker save -o flask_api_image.tar python:3.9-slim<br>
```
(Nếu dự án của bạn tự build image riêng từ Dockerfile, hãy chạy lệnh docker build trước trên laptop rồi tiến hành docker save tương tự).<br>

Bước 2: Chuẩn bị bộ cài Docker Engine Offline cho Máy chủ thật<br>
Vì máy chủ thật chưa cài Docker và không có mạng để chạy apt-get install docker.io, bạn cần tải sẵn các file cài đặt Offline thích hợp với hệ điều hành của máy chủ (Ví dụ file .deb cho Ubuntu/Debian hoặc .rpm cho CentOS/RHEL) từ trang chủ Docker và lưu vào USB/Ổ cứng di động.<br>

Bước 3: Sao chép dữ liệu sang máy chủ vật lý<br>
Sử dụng các thiết bị ngoại vi an toàn (USB, ổ cứng di động) hoặc kết nối dây mạng LAN nội bộ trực tiếp giữa laptop và máy chủ để copy các file sau sang máy chủ:<br>

Các file nén hình ảnh: *.tar (Đã tạo ở Bước 1).<br>

Các file cài đặt Docker Engine Offline (Đã chuẩn bị ở Bước 2).<br>

Toàn bộ thư mục mã nguồn chứa cấu hình: gồm file docker-compose.yml, thư mục frontend/, backend/,...<br>

Bước 4: Cài đặt và kích hoạt hệ thống trên máy chủ thật<br>
Đăng nhập vào Terminal của máy chủ thật và thực hiện chuỗi lệnh sau:<br>

Cài đặt Docker Engine (Offline):<br>

Bash<br>
# Ví dụ trên Ubuntu dùng lệnh cài đặt các gói đính kèm trong USB<br>
```<br>
sudo dpkg -i path_to_folder_usb/*.deb<br>
```<br>
Nạp (Load) các file ảnh từ file nén vào Docker:<br>

```Bash<br>
docker load -i mariadb_image.tar<br>
docker load -i grafana_image.tar<br>
docker load -i flask_api_image.tar<br>
```<br>
Kiểm tra lại bằng lệnh docker images, bạn sẽ thấy các image xuất hiện sẵn sàng trong máy chủ mà không tốn 1 byte băng thông internet nào.<br>

Khởi chạy hệ thống bằng Docker Compose:<br>
Di chuyển vào thư mục chứa dự án trên máy chủ (nơi chứa file docker-compose.yml) và kích hoạt ứng dụng:<br>

```Bash<br>
cd ~/path_to_project/<br>
docker compose up -d<br>
```
# 2. Thực Hành<br>
- Tạo thư mục chứa dự án:<br>
  <img width="717" height="66" alt="image" src="https://github.com/user-attachments/assets/683de2d2-8ce0-41a2-8add-9849554b6ef8" />
Bước 1: Xây dựng Backend Flask API:<br>
1. backend/requirements.txt<br>
```<br>
flask<br>
flask-cors<br>
mysql-connector-python<br>
```<br>
<img width="698" height="159" alt="image" src="https://github.com/user-attachments/assets/c5d02bb3-ee21-4013-a64c-26753ec6c234" />
2. backend/Dockerfile<br>
```<br>
FROM python:3.9-slim<br>
WORKDIR /app<br>
COPY requirements.txt .<br>
RUN pip install --no-cache-dir -r requirements.txt<br>
COPY . .<br>
CMD ["python", "app.py"]<br>
```<br>
<img width="699" height="227" alt="image" src="https://github.com/user-attachments/assets/9f65399c-6d9e-45a3-9240-1a74e45046b9" />
3. backend/app.py"<br>
<img width="1090" height="626" alt="image" src="https://github.com/user-attachments/assets/7fc7c579-4f08-44e9-8bfe-ff10221a8bf6" />
Bước 2: Xây dựng Frontend Giao diện:<br>
- frontend/index.html:<br>
  <img width="1103" height="622" alt="image" src="https://github.com/user-attachments/assets/88f18aeb-9966-43cc-9e0c-6a067fc64de3" />
Bước 3: Thiết lập cấu hình hệ thống docker-compose.yml<br>
<img width="1115" height="627" alt="image" src="https://github.com/user-attachments/assets/a18ae097-9e46-4888-99c7-69708af9cbc3" />
Bước 4: Khởi chạy cụm Container:<br>
<img width="1095" height="535" alt="image" src="https://github.com/user-attachments/assets/1b8ad7a0-de50-4b1c-a06d-ed7c3740b9df" />
  Bước 5: Cấu hình nodered để lấy API và gửi thông báo, lưu dữ liệu tại các db:<br>
  <img width="1154" height="326" alt="image" src="https://github.com/user-attachments/assets/d1201ce9-f6dd-4d91-8524-781891598900" />

- Sau khi nodered hoạt động, bot sẽ tự động gửi tin nhắn vào nhóm:<br>
  <img width="1537" height="920" alt="image" src="https://github.com/user-attachments/assets/81ce326c-007d-42d1-bf8b-5c3293b73383" />
   Bước 6: Cấu hình Grafana để kết nối tới Influxdb:<br>
  <img width="1218" height="888" alt="image" src="https://github.com/user-attachments/assets/3875d37b-5537-4393-a42d-949d61f15704" />
<img width="1550" height="710" alt="image" src="https://github.com/user-attachments/assets/0aa93582-78c7-4adf-ab65-b1d3d0197679" />
- Tạo biểu đồ bằng các lệnh truy vấn tới flux:<br>
  <img width="1626" height="947" alt="image" src="https://github.com/user-attachments/assets/f8dccf0d-3d12-4e24-915c-d29080ba2939" />
 Bước 7 : Cấu hình index.html để thay iframe của grafana và hiển thị biểu đồ trên web:<br>
<img width="1109" height="627" alt="image" src="https://github.com/user-attachments/assets/4df025c0-79ff-4ec8-8652-8a87e4f97c9a" />
<img width="1652" height="977" alt="image" src="https://github.com/user-attachments/assets/a2e91608-0759-4481-991f-02696520bb40" /> 
  Bước 8: Backup dữ liệu:<br>
  - xuất tất cả các container ra file nén:<br>
```<br>
# 1. Tạo thư mục chứa các file nén backup<br>
mkdir -p ~/docker_backup && cd ~/docker_backup<br>

# 2. Chạy vòng lặp commit trạng thái thực tế và nén lại thành file .tar.gz<br>
for container in mariadb_service influxdb_service nodered_service grafana_service flask_service nginx_service; do<br>
    echo "📦 Đang đóng băng và nén container: $container..."<br>
    # Commit container hiện tại thành một Image tạm thời<br>
    docker commit $container img_backup_$container<br>
    # Xuất Image đó ra thành file nén vật lý<br>
    docker save img_backup_$container | gzip > backup_$container.tar.gz<br>
done<br>

echo "✅ Hoàn thành! Tất cả các container đã được xuất ra file nén tại ~/docker_backup"<br>
```<br>
  - xoá mọi container đang chạy:<br>
```Bash<br>
# 1. Cưỡng chế dừng và xóa sạch đích danh 6 container của bài tập<br>
docker rm -f mariadb_service influxdb_service nodered_service grafana_service flask_service nginx_service<br>

# 2. Kiểm tra lại danh sách hệ thống<br>
docker ps -a<br>
```<br>
  - load lại các container  từ file nén để khôi phục các container đã xoá:<br>
```Bash<br>
# 1. Di chuyển vào thư mục chứa file nén dữ liệu dự phòng<br>
cd ~/docker_backup<br>

# 2. Vòng lặp giải nén và nạp ngược các file ảnh (.tar.gz) vào Docker Engine<br>
for file in *.tar.gz; do<br>
    echo "🔄 Đang giải nén và nạp lại: $file..."<br>
    gzip -dc $file | docker load<br>
done<br>

# 3. ĐỔI LẠI TAG (MẸO THẦN CHÚ): Đổi tên các image backup về đúng tên gốc trong file docker-compose.yml <br>
# Việc này giúp giữ nguyên 100% dữ liệu code/flow/database bạn đã cấu hình bên trong container trước đó<br>
docker tag img_backup_mariadb_service mariadb:10.6<br>
docker tag img_backup_influxdb_service influxdb:1.8-alpine<br>
docker tag img_backup_nodered_service nodered/node-red:latest<br>
docker tag img_backup_grafana_service grafana/grafana:latest<br>
docker tag img_backup_nginx_service nginx:alpine<br>
# Đối với Flask API tự build, ta gắn tag đè lại cho tên ảnh local build của dự án<br>
docker tag img_backup_flask_service bt5_complete-flask_api:latest <br>

# 4. Quay trở về thư mục gốc và ra lệnh hồi sinh toàn bộ hệ thống<br>
cd ~/bt5_complete<br>
docker compose up -d<br>
```<br>
