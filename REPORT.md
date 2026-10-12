# BÁO CÁO KỸ THUẬT: PHÂN TÍCH KIẾN TRÚC CONTAINER VS VIRTUAL MACHINE

## 1. Mục tiêu

Báo cáo phân tích sự khác biệt giữa kiến trúc ảo hóa phần cứng sử dụng Virtual Machine (VM) và kiến trúc ảo hóa hệ điều hành sử dụng Container. Đồng thời, báo cáo trình bày cơ chế cô lập tiến trình, mạng, hệ thống tệp và quản lý tài nguyên trong Linux Container, cũng như các thông số cơ bản của Docker Engine trên hệ thống triển khai.

## 2. So sánh kiến trúc VM và Container

| Tiêu chí | Virtual Machine (VM) | Container |
|---|---|---|
| Dung lượng Image | Thường lớn do bao gồm hệ điều hành khách, thư viện và ứng dụng. | Thường nhỏ hơn vì chia sẻ kernel của hệ điều hành máy chủ và chỉ đóng gói các thành phần cần thiết. |
| Thời gian khởi động | Thường mất từ vài giây đến vài phút tùy hệ điều hành và cấu hình. | Thường khởi động trong vài giây hoặc nhanh hơn vì không cần khởi động một hệ điều hành khách hoàn chỉnh. |
| Hao phí OS | Mỗi VM thường cần kernel và các dịch vụ hệ điều hành riêng, tiêu tốn thêm RAM, CPU và dung lượng lưu trữ. | Nhiều Container có thể chia sẻ kernel của máy chủ, giảm tài nguyên dành cho hệ điều hành. |
| Mức độ cô lập | Cô lập mạnh hơn ở cấp máy ảo thông qua lớp Hypervisor và hệ điều hành khách riêng. | Cô lập ở cấp tiến trình bằng các cơ chế của kernel Linux; mức độ bảo vệ phụ thuộc cấu hình và các biện pháp bảo mật bổ sung. |
| Khả năng mở rộng | Có thể mở rộng nhưng thường cần nhiều tài nguyên và thời gian để khởi tạo VM mới. | Có thể tạo, thay thế và nhân bản nhiều Container nhanh chóng, thuận tiện cho triển khai ứng dụng theo nhu cầu. |

VM và Container đều hỗ trợ triển khai ứng dụng độc lập, nhưng sử dụng hai cơ chế kiến trúc khác nhau. VM ảo hóa phần cứng, trong khi Container tận dụng các tính năng cô lập và quản lý tài nguyên của kernel hệ điều hành.

## 3. Kiến trúc và cơ chế hoạt động

### 3.1. Virtual Machine và Hypervisor

Virtual Machine mô phỏng một máy tính độc lập trên hệ thống vật lý. Hypervisor chịu trách nhiệm phân phối tài nguyên phần cứng như CPU, RAM, ổ đĩa và thiết bị mạng cho các máy ảo.

Mỗi VM thường có hệ điều hành khách riêng, bao gồm kernel và các dịch vụ hệ thống. Nhờ vậy, nhiều hệ điều hành khác nhau có thể hoạt động trên cùng một máy chủ vật lý. Tuy nhiên, việc duy trì nhiều hệ điều hành làm tăng mức tiêu thụ tài nguyên và thời gian khởi động.

### 3.2. Container Engine và Docker

Docker sử dụng Container Engine để tạo và quản lý Container từ các Image. Container đóng gói ứng dụng cùng thư viện và các thành phần phụ thuộc cần thiết, nhưng thông thường không mang theo một kernel riêng.

Các Container Linux chạy trên cùng kernel của máy chủ Linux. Kiến trúc này giúp giảm chi phí tài nguyên, tăng tốc độ triển khai và hỗ trợ quy trình tích hợp, triển khai liên tục (CI/CD).

Docker không loại bỏ hoàn toàn nhu cầu sử dụng VM. Trong môi trường doanh nghiệp, Docker thường chạy bên trong các VM để kết hợp khả năng quản lý ứng dụng linh hoạt với lớp cô lập hạ tầng.

## 4. Cơ chế cô lập và quản lý tài nguyên trong Linux Container

### 4.1. Namespaces

Namespaces là cơ chế của kernel Linux giúp cô lập góc nhìn của tiến trình đối với các tài nguyên hệ thống. Các loại quan trọng gồm:

- **PID Namespace:** Cô lập không gian định danh tiến trình. Tiến trình trong Container có thể nhìn thấy các PID riêng trong namespace của mình thay vì toàn bộ tiến trình trên máy chủ.
- **Network Namespace:** Cung cấp không gian mạng riêng, bao gồm giao diện mạng, bảng định tuyến và các thiết lập mạng tương ứng.
- **Mount Namespace:** Cô lập góc nhìn về các điểm gắn kết hệ thống tệp, giúp Container sử dụng cấu trúc filesystem riêng.
- **UTS Namespace:** Cho phép cô lập hostname và domain name.
- **IPC Namespace:** Cô lập một số cơ chế giao tiếp liên tiến trình.

Namespaces giúp các ứng dụng trong những Container khác nhau hoạt động độc lập hơn. Tuy nhiên, Namespaces không tự mình cung cấp toàn bộ cơ chế bảo mật; quyền Linux, capabilities, seccomp và các biện pháp bổ sung vẫn rất quan trọng.

### 4.2. Control Groups (cgroups)

Control Groups, thường viết tắt là cgroups, là cơ chế của kernel Linux dùng để phân loại, theo dõi và kiểm soát mức tiêu thụ tài nguyên của các nhóm tiến trình.

Hai tài nguyên quan trọng là:

- **CPU:** Có thể đặt giới hạn hoặc phân bổ tỷ lệ sử dụng CPU để một Container không chiếm dụng quá nhiều năng lực xử lý của máy chủ.
- **RAM:** Có thể thiết lập giới hạn bộ nhớ để kiểm soát mức sử dụng RAM và hạn chế tác động của ứng dụng tiêu thụ quá nhiều bộ nhớ.

Nhờ cgroups, nhiều Container có thể dùng chung một máy chủ mà tài nguyên được kiểm soát tốt hơn. Khi kết hợp Namespaces với cgroups, Docker vừa cung cấp môi trường tiến trình được cô lập, vừa hỗ trợ quản lý tài nguyên hiệu quả.

## 5. Thông số Docker Engine trên hệ thống

Các thông số dưới đây được lấy từ kết quả thực tế của lệnh `docker info`.

| Thông số | Giá trị thực tế |
|---|---|
| Server Version | [Điền giá trị từ docker info] |
| Cgroup Driver | [Điền giá trị từ docker info] |
| Containers | [Điền tổng số Containers] |
| Images | [Điền tổng số Images] |

Lệnh sử dụng để kiểm tra:

```bash
docker version
docker info
docker info | grep -i cgroup
```

`Server Version` cho biết phiên bản Docker Engine phía máy chủ. `Cgroup Driver` thể hiện trình điều khiển cgroups Docker đang sử dụng. `Containers` thống kê tổng số Container hiện có, bao gồm các trạng thái được Docker báo cáo; `Images` cho biết số Docker Image hiện có trên hệ thống.

Các giá trị này phụ thuộc vào trạng thái thực tế của Docker Engine tại thời điểm kiểm tra, do đó cần được ghi nhận trực tiếp thay vì sử dụng số liệu minh họa.

## 6. Kết luận

Container và VM phục vụ những mục tiêu tương tự nhưng có đặc điểm kiến trúc khác nhau. VM cung cấp lớp ảo hóa phần cứng và hệ điều hành khách riêng, phù hợp khi cần ranh giới cô lập mạnh ở cấp máy ảo hoặc cần chạy nhiều hệ điều hành khác nhau. Container chia sẻ kernel máy chủ và tận dụng Namespaces cùng cgroups để cô lập tiến trình, mạng, filesystem và kiểm soát tài nguyên.

Doanh nghiệp chuyển từ mô hình chỉ sử dụng VPS sang Docker thường nhằm giảm chi phí tài nguyên trên mỗi ứng dụng, tăng tốc độ triển khai, chuẩn hóa môi trường chạy và hỗ trợ tự động hóa CI/CD. Tuy nhiên, Docker không thay thế hoàn toàn VPS hoặc VM; việc kết hợp cả hai thường mang lại sự cân bằng giữa hiệu suất, khả năng mở rộng và bảo mật.
