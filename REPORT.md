# BÁO CÁO KỸ THUẬT: PHÂN TÍCH KIẾN TRÚC CONTAINER VÀ VIRTUAL MACHINE

## 1. Mục tiêu

Báo cáo phân tích sự khác biệt giữa ảo hóa phần cứng bằng Virtual Machine (VM) và ảo hóa hệ điều hành bằng Container. Đồng thời, báo cáo giải thích vai trò của Linux Namespaces, Control Groups (cgroups) và trích xuất các thông số cơ bản của Docker Engine.

## 2. So sánh VM và Container

Virtual Machine sử dụng Hypervisor để tạo các máy ảo. Mỗi VM thường chạy một hệ điều hành khách với kernel riêng. Container sử dụng kernel của hệ điều hành máy chủ, đồng thời dựa vào các cơ chế cô lập và quản lý tài nguyên để chạy ứng dụng trong những môi trường riêng biệt.

| Tiêu chí | Virtual Machine (VM) | Container |
|---|---|---|
| Dung lượng Image | Thường lớn vì bao gồm hệ điều hành khách và các thành phần hệ thống. | Thường nhỏ hơn, chủ yếu gồm ứng dụng, thư viện và các thành phần phụ thuộc. |
| Thời gian khởi động | Thường lâu hơn do phải khởi động hệ điều hành khách. | Thường nhanh hơn do không phải khởi động một kernel riêng. |
| Hao phí OS | Mỗi VM thường cần tài nguyên cho một hệ điều hành khách riêng. | Nhiều container có thể dùng chung kernel của máy chủ, giảm chi phí tài nguyên hệ điều hành. |
| Mức độ cô lập | Có ranh giới ảo hóa phần cứng và kernel riêng, thường cung cấp mức cô lập mạnh hơn. | Cô lập ở cấp hệ điều hành; độ an toàn phụ thuộc cấu hình và các biện pháp bảo vệ bổ sung. |
| Khả năng mở rộng | Có thể mở rộng nhưng thường cần nhiều thời gian và tài nguyên hơn khi tạo máy ảo. | Có thể tạo, thay thế và nhân bản nhanh, phù hợp với triển khai ứng dụng linh hoạt. |

VM phù hợp khi cần chạy các hệ điều hành khác nhau hoặc yêu cầu ranh giới cô lập mạnh. Container phù hợp khi cần triển khai ứng dụng nhanh, đồng nhất môi trường và sử dụng tài nguyên hiệu quả. Hai công nghệ có thể kết hợp với nhau thay vì hoàn toàn thay thế nhau.

## 3. Namespaces và Control Groups (cgroups)

### 3.1. Linux Namespaces

Namespaces là cơ chế của Linux giúp cô lập góc nhìn về tài nguyên hệ thống giữa các nhóm tiến trình. Nhờ đó, ứng dụng trong container có thể hoạt động trong một môi trường riêng dù dùng chung kernel với máy chủ.

- **PID Namespace:** Cô lập không gian định danh tiến trình. Tiến trình trong container chỉ nhìn thấy các tiến trình thuộc phạm vi PID namespace của nó, thay vì mặc nhiên nhìn thấy toàn bộ tiến trình của máy chủ.
- **Network Namespace:** Cung cấp môi trường mạng riêng, bao gồm giao diện mạng, bảng định tuyến và các cấu hình mạng liên quan.
- **Mount Namespace:** Cô lập góc nhìn về các điểm gắn kết hệ thống tệp, giúp container có cây thư mục và cách gắn kết riêng.

Namespaces giúp phân tách môi trường hoạt động của ứng dụng nhưng không tạo ra một kernel độc lập cho mỗi container và không tự bảo đảm an toàn tuyệt đối.

### 3.2. Control Groups (cgroups)

Control Groups là cơ chế Linux dùng để tổ chức, theo dõi và kiểm soát việc sử dụng tài nguyên của các nhóm tiến trình. Docker có thể sử dụng cgroups để áp dụng các giới hạn tài nguyên cho container.

- **CPU:** Có thể giới hạn lượng CPU được sử dụng hoặc điều chỉnh mức ưu tiên tương đối giữa các nhóm tiến trình, giúp hạn chế tình trạng một container chiếm quá nhiều năng lực xử lý.
- **RAM:** Có thể giới hạn lượng bộ nhớ được sử dụng. Khi vượt giới hạn, hệ thống có thể từ chối cấp phát thêm hoặc kích hoạt cơ chế xử lý thiếu bộ nhớ tùy cấu hình.
- **Theo dõi tài nguyên:** cgroups hỗ trợ thu thập thông tin sử dụng tài nguyên để quản trị viên giám sát và kiểm soát tải ứng dụng.

Namespaces tập trung vào cô lập góc nhìn tài nguyên, còn cgroups tập trung vào quản lý và giới hạn mức sử dụng tài nguyên. Hai cơ chế kết hợp giúp container vận hành độc lập hơn về môi trường và giảm nguy cơ một ứng dụng sử dụng quá nhiều tài nguyên máy chủ. Tuy nhiên, vẫn cần các biện pháp bảo mật bổ sung.

## 4. Thông số Docker Engine

Thực hiện các lệnh sau bằng user thường, không sử dụng sudo:

```bash
docker version
docker info
```

Các thông số được lấy trực tiếp từ kết quả thực tế của lệnh `docker info`.

| Thông số | Giá trị thực tế |
|---|---|
| Server Version | 29.9.0 |
| Cgroup Driver | systemd |
| Containers | 6 |
| Images | 5 |

Theo kết quả thực thi `docker info`, Docker Engine đang quản lý tổng cộng 6 container, trong đó có 2 container đang chạy và 4 container đã dừng. Hệ thống hiện có 5 Docker images. Cgroup Driver được sử dụng là `systemd`, với Cgroup Version 2. Docker Server phiên bản 29.9.0 đang hoạt động trên Ubuntu 26.04.1 LTS trong môi trường WSL2.

## 5. Lý do doanh nghiệp chuyển từ VPS sang Docker

Docker giúp đóng gói ứng dụng cùng các thư viện phụ thuộc vào image, nhờ đó giảm sự khác biệt giữa môi trường phát triển, kiểm thử và triển khai. Container thường khởi động nhanh, sử dụng tài nguyên hiệu quả và thuận tiện cho việc nhân bản dịch vụ khi nhu cầu tăng.

Tuy nhiên, Docker không nhất thiết thay thế VPS hoặc máy ảo. Doanh nghiệp có thể chạy Docker bên trong VPS hoặc VM để kết hợp khả năng quản lý hạ tầng với tính linh hoạt của container. Việc lựa chọn còn phụ thuộc yêu cầu bảo mật, khối lượng công việc, khả năng vận hành và chi phí.