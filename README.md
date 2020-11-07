# Microservices with SpringBoot and Java
## Part 1: Getting started with Microservices Architecture
**Nội dung bạn sẽ được học:**
- Monolith là gì?
- Microservices là gì?
- Những thách thức với microservices là gì?
- Làm thế nào để SpringBoot và Spring Cloud phát triển Microservices một cách dễ dàng?
- Làm thế nào để thực hiện client side load balancing với Ribbon?
- Làm thế nào đê thự thi một Naming Server (Eureka Naming Server)?
- Làm thế nào để kết nối các microservices với Ribbon và Naming Server?

### Tổng quát Microservices - Một bức tranh lớn (A Big Picture)
Trong phần này chúng ta sẽ tạo ra 2 microservices:
- Forex Services - Viết tắt là FS (Abbreviated as FS) (Service ngoại hối)
- Currency Conversion Service - Viết tắt là CCS (Abbreviated as CCS)

#### 1. Forex Service
Forex Service (FS) là Service Provider (Nhà cung cấp dịch vụ). Nó cung cấp các giá trị trao đổi tiền tệ cho các loại tiền tệ khác nhau.
Giả sử rằng nó kết nối với một sàn giao dịch ngoại hối (Forex Exchange) và cung cấp giá trị trao đổi hiện tại giữa các loại tiền tệ.
Một ví dụ về request và response được biểu diễn như bên dưới:
{
	"id": 10002,
	"from": "EUR",
	"to": "INR",
	"conversionMutiple": 75,
	"port": 8000
}

Request trên là giá trị quy đổi tiền tệ từ EUR sang INR. Trong respone,conversionMutiple là 75.

#### 2. Currency Conversion Service (CCS)
Dịch vụ chuyển đổi tiền tệ (Currency conversion Service) có thể chuyển đổi một nhóm tiền tệ sang một loại tiền tệ khác.
Nó sử dụng FS (Forex Service) để nhận các giá trị quy đổi hiện tại. CCS là Service Consumer (Service tiêu dùng).
Một ví dụ về request và response như sau:
{
	"id": 10002
	"from": "EUR",
	"to": "INR",
	"conversionMutiple": 75,
	"quantity": 10000,
	"totalCalculatedAmount": 750000,
	"port": 8000
}  

Request trên tìm giá trị 10000 EUR tính bằng INR. TotalCalculatedAmount là 750000 INR.
![](https://www.springboottutorial.com/images/Spring-Boot-Microservice-1-CCS-FS.png)

#### 3. Eureka Naming Server and Ribbon.
Dựa trên tải trọng (based on the load) chúng ta có nhiều thể hiện của Currency Conversion Service và Forex Service đang chạy.
![](https://www.springboottutorial.com/images/Spring-Boot-Microservice-2-CCS.png)
![](https://www.springboottutorial.com/images/Spring-Boot-Microservice-3-FSInstances.png)

Và số lượng thể hiện của mỗi Service có thể thay đổi theo thời gian. Dưới đây là một trường hợp cụ thể trong đó có 5 thể 
hiện của Forex Service:
![](https://www.springboottutorial.com/images/Spring-Boot-Microservice-4-5FSInstances.png)

Điều gì cần xảy ra để trong tình huống trên việc tải nên được phẩn bổ đồng đều giữa 5 thể hiện (instances) này.
![](https://www.springboottutorial.com/images/Spring-Boot-Microservice-5-CCSToFS5instances.png)
Trong loạt bài này, chúng ta sẽ sử dụng Ribbon cho việc cân bằng tải (Load Balancing) và Eureka Naming Server cho việc đăng ký
tất cả các microservices.
![](https://www.springboottutorial.com/images/Spring-Boot-Microservice-6-EurekaNamingServer-Deployment.png)

### Một Monolith Application là gì?
Bạn đã bao giờ làm việc trong một dự án:
> Được release vài tháng một lần
> Có một loạt các tính năng mới
> Một dự án có hơn 50 người làm việc
> Vấn đề gỡ lỗi là một thách thức lớn
> Việc đưa công nghệ mới và quy trình mới vào hầu như là không thể
==> Đây là những đặc điểm chính của một ứng dụng Monolith

**Monolith được đặc trưng bởi:**
> Kích thước của application lớn
> Chu kỳ release (phát hành) dài
> Một team lớn (Có nhiều người)

**Những thách thức điển hình bao gồm:**
> Thách thức về khả năng mở rộng (scalability challenges)
> Áp dụng công nghệ mới
> Quy trình mới - sự linh hoạt (Agile)
> Khó Automation Test
> Khó thích ứng với thực tiễn phát triển hiện đại
> Khó thích ứng với các loại thiết bị mới
### Microservices
Microservices phát triển (evolve: phát triển, tiến hoá) như một giải pháp cho những thách thức về khả năng mở rộng và đổi
mới kiến trúc của Monolith.
Có một số định nghĩa được đề xuất cho Microservices:
> Các dịch vụ tự trị nhỏ hoạt động cùng nhau
> Phát triển các ứng dụng đơn lẻ như một bộ các service (dịch vụ) nhỏ, mỗi service chạy trong một quy trình riêng của nó và
giao tiếp với cơ chế nhẹ, thường ra một HTTP resource API. Các services này được xây dựng dựa trên khả năng bussiness và
triển khai độc lập bằng máy móc triển khai hoàn toàn tự động. Quản lý tập trung các service này là tối thiểu, và chúng có 
thể được viết bằng các ngôn ngữ khác nhau và sử dụng các công nghệ lưu trữ dữ liệu khác nhau.


Không có một khái niệm riêng nào dành cho microservice, nhưng có một số đặc điểm quan trọng của Microservices như sau:
- REST: Được xây dựng xung quanh RESTful Resources. Giao tiếp có thể là HTTP hoặc dựa trên sự kiện (based on events)
- Việc triển khai có thể thực hiện trên các đơn vị nhỏ, riêng lẻ - giới hạn bối cảnh.
- Khả năng Cloud - khả năng mở rộng động. (Cloud Enable - Dynamic Scaling)

### Kiến Trúc Của Microservices Như Thế Nào?
Đây là một ứng dụng Monolith. Một ứng dụng cho mọi thứ
![](https://www.springboottutorial.com/images/MonolithApplication.png)

Còn đây là ứng dụng tương tự khi được triển khai bằng Microservices:
![](https://www.springboottutorial.com/images/MicroservicesArchitectureSplit.png)
Kiến trúc Microservice liên quan (involve) đến một số thành phần nhỏ, được thiết kế tốt, các thành phần tương tác với messages (components interacting with messages)
![](https://www.springboottutorial.com/images/Microservices-Chain-Example.png)

### Các lợi thế của Microservices
- Công nghệ và quy trình mới được áp dụng một cách dễ dàng hơn. Bạn có thể thử công nghệ mới cùng với microservice mới hơn mà chúng ta tạo
- Chu kỳ phát hành (release) nhanh hơn
- Mở rộng quy mô với đám mây (Cloud)

### Các thử thách với kiến trúc Microservices
Trong khi phát triển các thành phần nhỏ hơn có thể dễ dàng hơn, có một số phức tạp liên quan đến kiến trúc microservices:
- Cần thiết lập nhanh: Bạn không thế dành một tháng để thiết lập các Microservice. Bạn sẽ có thể tạo microservices một cách nhanh chóng.
- Tự động hoá (Automation): Bởi vì có một số thành phần nhỏ hơn thay vì nguyên khối(monolith), bạn cần phải tự động hoá mọi thứ (Build, Deploy, Monitoring (giám sát))
- Khả năng hiển thị: Bây giờ bạn có một số thành phần nhỏ hơn để triển khai và duy trì. Có thể là 100 hoặc 1000 thành phần. Bạn có thể theo dõi và xác định vấn đề
một cách tự động. Bạn cần khả năng hiển thị tốt xung quanh tất cả các thành phần.
- Bối cảnh giới hạn: Quyết định ranh giới của một microservice không phải là một nhiệm vụ dễ dàng. Bối cảnh giới hạn từ Domain Driven Design là một điểm tốt
để bắt đầu. Sự hiểu biết của bạn về miền (domain) sẽ phát triển trong một khoảng thời gian. Bạn cần đảm bảo rằng các ranh giới microservice phát triển.
- Quản lý cấu hình: Bạn cần duy trì cấu hình cho hàng trăm thành phần trên các môi trường. Bạn phải cần một giải pháp quản lý cấu hình.
- Tăng và giảm quy mô: Lợi thế của Microservice sẽ chỉ được nhận ra nếu ứng dụng của bạn có thể tăng và giảm quy mô dễ dàng trên cloud.
- Gói thẻ: Nếu một microservice ở cuối chuỗi gọi không thành công, nó có thể ảnh hưởng tới tất cả các microservice khác. Microservices phải được thiết kế chịu lỗi.
- Gỡ lỗi: Khi có vấn đề xảy ra, bạn cần phải xem xét trên nhiều service khác nhau. Ghi Log tập trung và Trang tổng quan (Dashboards) là cần thiết để giúp bạn dễ
dàng gỡ lỗi các sự cố.
- Tính nhất quán: Bạn không thể có nhiều công cụ để giải quyết cùng một vấn đề. Mặc dù điều quan trọng là thúc đẩy sự đổi mới, nhưng điều quan trọng là phải có
một số quản trị tập trung xung quanh ngôn ngữ, nền tảng, công nghệ và công cụ được sử dụng để thực hiện/ triển khai/ giám sát các microservices.

































































