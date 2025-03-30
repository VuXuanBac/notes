# NodeJS

NodeJS là một Framework chạy trên Google Chrome V8 Javascript Engine, dùng để phát triển ứng dụng có thể chạy Javascript ở cả Client và Server. NodeJS là một **môi trường thực thi JS** mã nguồn mở và đa nền tảng. 
- **V8** *là một Javascript Engine được sử dụng cho **Google Chrome**, dùng để thực thi JS, và điểm đặc biệt là nó tách rời với Browser (Browser sẽ triển khai thêm DOM và Web Platform APIs). Engine này hiện nay được sử dụng phổ biến cho các ứng dụng NodeJS. Ngoài ra có một số JS Engines khác như: **SpiderMonkey** (Firefox), **JavaScriptCore/Nitro** (Safari),... Tất cả các JS Engine đều triển khai theo chuẩn ECMAScript.*
- *V8 được viết bằng C++, có thể triển khai và chạy trên đa nền tảng.*
- Javascript ban đầu là một ngôn ngữ thông dịch, nhưng có một số thay đổi biến nó thành một ngôn ngữ biên dịch, với mục tiêu nhằm tăng tốc thực thi. *Phiên bản compiler đầu tiên cho JS compiler được giới thiệu vào năm 2009 trong **SpiderMonkey** Engine.* V8 cũng thực hiện compile JS với JIT (just-in-time compilation)

Một ứng dụng NodeJS chạy trên một tiến trình riêng, cung cấp **một tập các lời gọi vào ra bất đồng bộ**, không cần tạo luồng mới cho từng requests và việc truy cập vào ra cũng không bị chặn dừng. NodeJS khắc phục được nhược điểm của AJAX khi mà số lượng request tới Server là quá lớn.

Cả Browser và NodeJS đều sử dụng Javascript. Tuy nhiên, xây dựng một ứng dụng NodeJS khác hoàn toàn với tạo một ứng dụng chạy trên Browser. (1) *Trên Browser, hầu hết các tác vụ của JS là liên quan đến **tương tác với DOM và các Web Platform APIs** (như Cookies); và các chức năng này không có trên NodeJS. Ngược lại, phía NodeJS cung cấp một số chức năng như **truy xuất hệ thống tệp**, điều mà JS trên Browser không hỗ trợ.* (2) *Có thể lựa chọn ngay phiên bản JS mới nhất cho ứng dụng NodeJS nhưng một ứng dụng trên Browser lại cần phải xem xét việc tương thích với các phiên bản của Browsers, đặc biệt là quá trình nâng cấp phiên bản ES.*

Download NodeJS: https://nodejs.org/en/

## Kiến trúc

NodeJS là sự kết hợp của **Chrome's V8 JS Engine**, **Event Loop** và **IO API**.

![nodejs architecture](img/node_express/nodejs-architecture.png)

Trong mô hình truyền thống (như Apache, Ruby Puma), các web servers dành một luồng riêng để xử lý mỗi request. Còn NodeJS được xây dựng theo kiến trúc **đơn luồng hướng sự kiện**. Cả ứng dụng web chỉ chạy trên một tiến trình, ở đó:
- Tất cả các requests được xử lý trên cùng một luồng (Single Thread)
- Tất cả các thao tác vào ra sẽ được xử lý bất đồng bộ.
Ứng dụng NodeJS chỉ sử dụng một số ít luồng trong hệ thống, bao gồm:
- Một luồng cho Event Loop - xử lý các yêu cầu **non-blocking I/O** cũng như các **callbacks** (khi hoàn thành). 
- Mỗi luồng cho mỗi Workers - thực thi các công việc phức tạp hơn (sử dụng C++) như **blocking I/O**.

### Event Loop

NodeJS sử dụng **Event Loop** để có thể xử lý các thao tác **non-blocking I/O**. 
- Khi có yêu cầu vào ra, nó sẽ được gửi xuống System Kernel và trả quyền kiểm soát về ứng dụng.
- Và khi hoàn thành, Kernel sẽ thông báo tới NodeJS để thêm một **callback** vào **Poll Queue** chờ thực thi.

Tiền đề của NodeJS là coi các thao tác vào ra là nút thắt cổ chai (bottleneck) chính và **Event Loop** hỗ trợ để thực thi chúng ở chế độ bất đồng bộ (non-blocking)
- Tuy nhiên, nếu một thao tác xử lý khác chiếm dụng CPU lớn thì nó cũng sẽ chặn dừng ứng dụng. Các callbacks cũng sẽ chỉ được thực thi khi ứng dụng xử lý xong phần việc cũ.
- Lúc này, có thể xem xét đưa thao tác đó vào **Worker** để xử lý đa luồng.
- Khi đó
