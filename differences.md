# Difference in Terms

## Web Server vs Application Server

| Thuật ngữ | Mô tả | Ví dụ |
|--|--|--|
|Web Server|Xử lý HTTP requests và responses, thường chỉ là dữ liệu tĩnh|Apache và Nginx|
|App Server|Host logic của ứng dụng, phục vụ thêm cả dữ liệu động (sinh HTML tùy vào dữ liệu DB). Thường được triển khai trên nền của Web Server và triển khai thêm tính năng khác: hỗ trợ thêm giao thức khác (như RMI/RPC, SOAP, RESTful, JDBC)|- Unicorn, Puma (Ruby)<br>- Gunicorn (Python)<br>- Tomcat (Java)<br>IIS (.NET)<br>Jetty|

Trong quá trình Development, Application Server có thể chạy mà không cần thêm Web Server
Trên Production, Web Server thường được dùng làm **Reverse Proxy** cho Application Server (nó có thể cache lại một số tài nguyên như JS, CSS)
- Web Server thường được coi như **Front-end Server**: nhận requests và gửi cho Application Server và trả về responses (HTML, CSS, JS,...) từ Application Server.
- Application Server thường được coi như **Back-end Server**, xử lý logic nghiệp vụ, tương tác với DB, các services và tài nguyên khác.

Web Application dùng một **Web Server Interface** để có thể giao tiếp với Web Server/Application Server. Khi có nhiều Web Frameworks (bộ công cụ giúp phát triển web app, web services) khác nhau để xây dựng Web Application thì Web Server Interface chính là cổng giao tiếp chung. Nhiệm vụ của Web Server Interface thường là đóng gói dữ liệu request đã được parsed bởi Web Server/Application Server trước ghi gửi tới Web Application. Một số ví dụ về Web Server Interface
- [Rack](https://github.com/rack/rack): Cổng giao tiếp giữa các Web Frameworks dựa trên Ruby (như Rails, Sinatra, Padrino,...) và các Web/App Servers hỗ trợ Ruby (như Puma, Unicorn, Falcon,...)
- Web Server Gateway Interface (WSGI): Python
- Plack (PSGI): Perl
- JSGI: JS

Có thể hiểu Web Server Interfaces của Python/Ruby/JS là: *"Cho phép các ứng dụng có thể dùng Python/Ruby/JS để xử lý HTTP Requests"*

Một chút về lịch sử:
- Thời kỳ đầu, các Web Servers thường chỉ đọc các tệp tĩnh (HTML, images,...) và gửi về Browser.
- Mọi người bắt đầu muốn xử lý thêm gì đó với mỗi request gửi lên (**interactive**), ví dụ gửi form. Ý tưởng ban đầu là ánh xạ mỗi URL với một script.
- **CGI** (**Common Gateway Interface**) là một chuẩn giao thức cho phép một ứng dụng bên ngoài (process chạy scripts) có thể giao tiếp với Web Server, giúp Web Server cung cấp dữ liệu mà nó đã parsed (cookies, query, form data,...) cho các scripts (và ngược lại).
  - Ví dụ: [Tạo "Hello World" Web Application với CGI](https://code-maven.com/set-up-cgi-with-apache)
- Tuy nhiên, CGI lại tạo một process mới với mỗi request (để chạy script). Do đó, **FastCGI** ra đời, giúp xử lý một chuỗi requests trên cùng một process, các processes này sẽ được quản lý bởi một FastCGI server (khác với Web Server, và chúng có thể giao tiếp với nhau qua socket, named pipe, TCP,...). **SCGI** (Simple CGI) cũng là một giao thức tương tự FastCGI

https://docs.python.org/2/howto/webservers.html
## Framework vs Library
