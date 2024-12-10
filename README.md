# Web-deploy-series
A complete guide to deploying websites and AI models on VPS with Nginx and HTTPS.

# Phần 1: Thiết Lập Môi Trường Cơ Bản

## 1. Giới Thiệu

Xin chào, mình là Nam, hiện đang là sinh viên năm 3 chuyên ngành AI tại FPTU. Mình có niềm đam mê lớn với công nghệ và đã triển khai thành công dự án trên nền tảng Ubuntu VPS. Qua bài viết này, mình hy vọng sẽ giúp các bạn từng bước đưa website của mình đến gần hơn với người dùng một cách chuyên nghiệp và hiệu quả.

### 1.1 VPS Ubuntu là gì?

VPS (Virtual Private Server) là một máy chủ ảo độc lập hoạt động trong một máy chủ vật lý lớn hơn. Khác với shared hosting truyền thống, VPS cung cấp tài nguyên độc lập và được phân bổ riêng biệt.

<aside>
Ưu điểm nổi bật của VPS:

</aside>

- **Tài nguyên độc lập:** CPU, RAM, và bộ nhớ được phân bổ riêng, đảm bảo hiệu suất ổn định
- **Toàn quyền kiểm soát:** Quyền root cho phép tùy chỉnh server theo nhu cầu
- **Khả năng mở rộng:** Dễ dàng nâng cấp hoặc hạ cấp tài nguyên khi cần thiết
- **Bảo mật cao:** Môi trường độc lập giúp tăng cường bảo mật

## 2. Chuẩn Bị

### 2.1 Yêu Cầu Hệ Thống

- VPS với Ubuntu Server (phiên bản LTS mới nhất)
- Tối thiểu 1GB RAM
- Ít nhất 20GB bộ nhớ
- Kết nối internet ổn định

### 2.2 Cài Đặt Hệ Điều Hành Ubuntu

Để cài đặt hệ điều hành Ubuntu một cách chi tiết và chính xác, bạn có thể tham khảo hướng dẫn chính thức từ [trang chủ Ubuntu](https://ubuntu.com/tutorials/install-ubuntu-desktop#3-create-a-bootable-usb-stick) hoặc qua [Youtube Anh Việt Nguyễn AI](https://youtu.be/UoHK_gX_Q_o?si=kT3yDOdoMiHqvUJu). Tài liệu này cung cấp đầy đủ các bước từ tải file ISO, tạo USB bootable, đến quá trình cài đặt trực tiếp trên máy chủ hoặc máy tính của bạn.

Mình khuyến nghị sử dụng phiên bản [Ubuntu 20.04.6 LTS (Focal Fossa)](https://releases.ubuntu.com/focal/). Đây là phiên bản ổn định, được hỗ trợ lâu dài (LTS), và trong quá trình sử dụng mình nhận thấy nó tương thích tốt với các thư viện và ít gặp lỗi hơn so với các phiên bản khác.

### 2.3 Kiểm Tra Kết Nối

Trước khi tiến hành các bước tiếp theo, hãy đảm bảo rằng VPS của bạn có kết nối internet ổn định. Bạn có thể kiểm tra bằng cách sử dụng lệnh ping đến một trang web phổ biến, ví dụ như Google:

```bash
ping -c 4 google.com
```

```jsx
Output Example:

namhost@server-host-839:~$ ping -c 4 google.com
PING google.com(hkg07s48-in-x0e.1e100.net (2404:6800:4005:80a::200e)) 56 data bytes
64 bytes from hkg07s01-in-x0e.1e100.net (2404:6800:4005:80a::200e): icmp_seq=1 ttl=58 time=45.2 ms
64 bytes from hkg07s01-in-x0e.1e100.net (2404:6800:4005:80a::200e): icmp_seq=2 ttl=58 time=45.4 ms
64 bytes from hkg07s01-in-x0e.1e100.net (2404:6800:4005:80a::200e): icmp_seq=3 ttl=58 time=44.6 ms
64 bytes from hkg07s01-in-x0e.1e100.net (2404:6800:4005:80a::200e): icmp_seq=4 ttl=58 time=45.3 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 44.557/45.115/45.430/0.332 ms
```

4 packets transmitted, 4 received, 0% packet loss: Điều này cho thấy tất cả các gói tin đã được gửi và nhận thành công, đồng nghĩa với việc kết nối internet của bạn ổn định. Đến bước tiếp theo.

### 2.4 Cập Nhật Hệ Thống

Cập nhật toàn bộ hệ thống để đảm bảo an toàn và hiệu suất tối ưu:

```bash
sudo apt update && sudo apt upgrade -y
```

## 3. Thiết Lập Môi Trường

### 3.1 Cài Đặt Nginx

Nginx là web server hiệu năng cao, nhẹ nhàng và được sử dụng rộng rãi trong các ứng dụng web hiện đại.

<aside>
Các bước cài đặt Nginx:

</aside>

1. **Bước 1: Cài đặt Nginx**
    
    ```bash
    sudo apt update
    sudo apt install nginx -y
    ```
    
2. **Bước 2: Cấu hình tường lửa**
    
    Trước khi kiểm tra Nginx, bạn cần điều chỉnh phần mềm tường lửa để cho phép truy cập vào dịch vụ. Khi cài đặt, Nginx tự động đăng ký như một dịch vụ với ufw, giúp việc cấp quyền truy cập cho Nginx trở nên đơn giản.
    
    Bạn có thể liệt kê các cấu hình ứng dụng mà ufw hỗ trợ bằng cách nhập lệnh sau:
    
    ```bash
    sudo ufw app list
    ```
    
    ```bash
    Output:
    
    Available applications:
      Nginx Full
      Nginx HTTP
      Nginx HTTPS
      OpenSSH
    ```
    
    Nếu ufw chưa được cài đặt hãy chạy lệnh này và kiểm tra lại:
    
    ```bash
    sudo apt-get install ufw
    ```
    

Kết quả hiển thị sẽ bao gồm:

* Nginx Full: Cho phép cổng 80 (HTTP) và 443 (HTTPS)
* Nginx HTTP: Chỉ cho phép cổng 80
* Nginx HTTPS: Chỉ cho phép cổng 443
* OpenSSH: Cho phép kết nối SSH (cổng 22)

1. **Bước 3: Cấu hình quy tắc tường lửa**
    1. **Bạn nên kích hoạt profile tường lửa (firewall profile) hạn chế nhất** mà vẫn cho phép lưu lượng (traffic) mà bạn đã cấu hình trước đó. Điều này giúp bảo vệ hệ thống của bạn khỏi các mối đe dọa không mong muốn, chỉ mở những cổng cần thiết cho hoạt động của ứng dụng.
    
    ```bash
    sudo ufw allow 'Nginx HTTP'
    sudo ufw status
    ```
    
    ```bash
    Output:
    
    namhost@server-host-839:~$ sudo ufw status
    Status: active
    
    To                         Action      From
    --                         ------      ----                            
    Nginx HTTP                 ALLOW       Anywhere                                            
    Nginx HTTP (v6)            ALLOW       Anywhere (v6)
    ```
    
2. **Bước 4: Khởi động và kích hoạt Nginx**
    
    ```bash
    sudo systemctl start nginx
    sudo systemctl enable nginx #Automatic open Server when turn on Computer
    sudo systemctl status nginx
    ```
    
    ```bash
    namhost@server-host-839:~$ systemctl status nginx
    ● nginx.service - A high performance web server and a reverse proxy server
         Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
         Active: active (running) since Mon 2024-11-25 08:09:39 +07; 6h ago
           Docs: man:nginx(8)
       Main PID: 874 (nginx)
          Tasks: 9 (limit: 4470)
         Memory: 14.1M
         CGroup: /system.slice/nginx.service
                 ├─874 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
                 ├─875 nginx: worker process
    ```
    
    Sau khi hoàn tất cài đặt, bạn có thể truy cập website qua địa chỉ IP của VPS hoặc http://localhost or http://127.0.0.1/. Nếu thấy trang chào mừng của Nginx, chúc mừng bạn đã cài đặt thành công!
    
    ![](https://images.viblo.asia/d05cc05d-d8d2-4679-979d-c0bc545e0e37.png)

    
    Hoặc bạn có thể kiểm tra máy tính có thể giao tiếp với các thiết bị khác trong cùng mạng LAN (Local Area Network). 
    
    ```bash
    sudo apt update
    sudo apt install net-tools
    ```
    
    Để kiểm tra địa chỉ IP của máy tính, bạn có thể sử dụng lệnh.
    
    ```bash
    ifconfig
    ```
    
    ```bash
    Output:
    
    namhost@server-host-839:~$ ifconfig
    eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.100.221  netmask 255.255.255.0  broadcast 192.168.100.255
            inet6 fd00:db80::6  prefixlen 128  scopeid 0x0<global>
            inet6 2405:4802:80c2:cf00:b14:d108:2ab3:c023  prefixlen 64  scopeid 0x0<global>
            inet6 fd00:db80::c173:6ff3:2152:e89b  prefixlen 64  scopeid 0x0<global>
            inet6 2405:4802:80c2:cf00:bf20:6e4:4350:e2a  prefixlen 64  scopeid 0x0<global>
            inet6 fd00:db80::3d69:5558:97c4:e32c  prefixlen 64  scopeid 0x0<global>
            inet6 fe80::8e42:2ec0:47fb:6a2d  prefixlen 64  scopeid 0x20<link>
            inet6 2405:4802:80c2:cf00::6  prefixlen 128  scopeid 0x0<global>
            ether 10:e7:c6:7e:e6:0a  txqueuelen 1000  (Ethernet)
            RX packets 383751  bytes 114963673 (114.9 MB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 597949  bytes 249543541 (249.5 MB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 3299  bytes 314563 (314.5 KB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 3299  bytes 314563 (314.5 KB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    ```
    
    Địa chỉ IP của máy tính là 192.168.100.221. Bạn có thể kết nối đến máy tính này qua cổng 80 (HTTP) bằng cách truy cập vào địa chỉ: `http://192.168.100.221:80`
    ![](https://images.viblo.asia/1fd600d7-814f-491e-a0ec-6c2be3d33283.png)
 
    
### 3.2 Quản Lý Nginx

**Các lệnh quản lý cơ bản:**

- **Khởi động:** `sudo systemctl start nginx`
- **Dừng:** `sudo systemctl stop nginx`
- **Khởi động lại:** `sudo systemctl restart nginx`
- **Tải lại cấu hình:** `sudo systemctl reload nginx`
- **Kiểm tra trạng thái:** `sudo systemctl status nginx`

**Tài liệu tham khảo:**
* [Tài liệu về Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04#prerequisites)
* [Hướng dẫn chính thức Ubuntu](https://ubuntu.com/tutorials/install-ubuntu-desktop#1-overview)
* [Hướng dẫn chạy song song win and ubuntu](https://www.youtube.com/watch?v=UoHK_gX_Q_o&t=1167s)



## 4. Tiếp Nối Hành Trình: Đưa Website Ra Toàn Thế Giới

Sau khi thành công thiết lập môi trường mạng nội bộ và đảm bảo website hoạt động tốt trong mạng LAN, giờ đây, chúng ta đã sẵn sàng cho bước tiến lớn hơn. Ở **Phần 2**, bạn sẽ học cách mở port, trỏ tên miền về địa chỉ IP public, và biến website của mình thành một nền tảng có thể truy cập từ mọi nơi trên thế giới.

Hãy tưởng tượng, chỉ cần chia sẻ một domain ngắn gọn và chuyên nghiệp, bạn đã có thể mời mọi người ghé thăm website của mình, dù họ ở bất kỳ nơi đâu. **Hẹn các bạn ở phần 2 của mình nhé!🚀**

## 5. Lời cảm ơn

Cảm ơn bạn đã dành thời gian đọc bài viết này! Đây là bài viết đầu tiên của mình tìm hiểu qua nhiều nguồn và tổng hợp lại hy vọng những hướng dẫn trên sẽ giúp bạn triển khai website trên Ubuntu VPS một cách dễ dàng trong môi trường localhost. Nếu bạn có bất kỳ thắc mắc, góp ý hoặc cần hỗ trợ thêm, đừng ngần ngại để lại bình luận. Mọi ý kiến của bạn đều rất quý giá và sẽ giúp mình cải thiện nội dung tốt hơn trong tương lai!

**[Phần 2: Mở Rộng Kết Nối: Cấu Hình Port và Triển Khai Tên Miền Công Khai](https://viblo.asia/p/cac-buoc-de-trien-khai-trang-web-tren-ubuntu-vps-voi-ten-mien-thuc-te-E1XVOjGZLMz) **
