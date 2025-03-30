# Difference in Terms

## Web Server vs Application Server

| Thuật ngữ | Mô tả | Ví dụ |
|--|--|--|
|Web Server|Xử lý HTTP requests và responses, thường chỉ là dữ liệu tĩnh|Apache, Nginx, IIS, [Cherokee](https://cherokee-project.com/), [Caddy](https://caddyserver.com/)|
|Application Server|Host logic của ứng dụng, phục vụ thêm cả dữ liệu động (sinh HTML tùy vào dữ liệu DB). Thường được triển khai trên nền của Web Server và triển khai thêm tính năng khác: hỗ trợ thêm giao thức khác (như RMI/RPC, SOAP, RESTful, JDBC)|- Unicorn, Puma (Ruby)<br>- Gunicorn (Python)<br>- Tomcat, Jetty (Java)<br>- Kestrel (ASP Core)<br>- ColdFusion (CFML)|

Trong quá trình Development, Application Server có thể chạy mà không cần thêm Web Server.
Tuy nhiên, các Application Servers có thể không tốt bằng các Web Servers (như Apache hay Nginx) trong việc xử lý các HTTP Requests (I/O security, concurrency, connection timeouts,...). Do đó, trên Production, Web Server thường được dùng làm **Reverse Proxy** cho Application Server
- Web Server thường được coi như **Front-end Server**: nhận requests để gửi cho Application Server và trả về responses (HTML, CSS, JS,...) từ Application Server.
- Application Server thường được coi như **Back-end Server**, xử lý logic nghiệp vụ, tương tác với DB, các services và tài nguyên khác.

### Về NodeJS, Java và .NET

NodeJS, bên cạnh là một runtime environment, nó cũng là là một Application Server, bản thân NodeJS có chức năng xử lý HTTP Request được tích hợp sẵn.

Với Java, thành phần tương đương với Application Server có thể coi là **Servlet container** - cung cấp môi trường thực thi cho servlets và server pages (JSP). Một trong những ví dụ là Tomcat, WildFly, Jetty.

Với .NET (ASP, ASP Core,...), **IIS** (Internet Information Services) giống như một Web Server (thường làm reverse proxy và nó hỗ trợ hosting cả Python, PHP,...) còn **Kestrel** giống với Application Server (dành cho ASP Core)

### Web Server Interface

Web Application dùng một **Web Server Interface** để có thể giao tiếp với Web Server/Application Server. Khi có nhiều Web Frameworks (bộ công cụ giúp phát triển web app, web services) khác nhau để xây dựng Web Application thì Web Server Interface chính là cổng giao tiếp chung. Nhiệm vụ của Web Server Interface thường là đóng gói dữ liệu request đã được parsed bởi Web Server/Application Server trước ghi gửi tới Web Application. Một số ví dụ về Web Server Interface
- [Rack](https://github.com/rack/rack): Cổng giao tiếp giữa các Web Frameworks dựa trên Ruby (như Rails, Sinatra, Padrino,...) và các Web/App Servers hỗ trợ Ruby (như Puma, Unicorn, Falcon,...)
- Web Server Gateway Interface (WSGI): Python
- Plack (PSGI): Perl

Có thể hiểu Web Server Interfaces của Python/Ruby/Perl là: *"Cho phép các ứng dụng có thể dùng Python/Ruby/Perl để xử lý HTTP Requests"*

### Một chút về lịch sử

- Thời kỳ đầu, các Web Servers thường chỉ đọc các tệp tĩnh (HTML, images,...) và gửi về Browser.
- Mọi người bắt đầu muốn xử lý thêm gì đó với mỗi request gửi lên (**interactive/dynamic**), ví dụ gửi form.
- Ý tưởng ban đầu là ánh xạ mỗi URL với một script.
- **CGI** (**Common Gateway Interface**) là một chuẩn giao thức cho phép một ứng dụng bên ngoài (process chạy scripts) có thể giao tiếp với Web Server, giúp Web Server cung cấp dữ liệu mà nó đã parsed (cookies, query, form data,...) cho các scripts (và ngược lại).
  - Ví dụ: [Tạo "Hello World" Web Application với CGI](https://code-maven.com/set-up-cgi-with-apache)
- Tuy nhiên, CGI lại tạo một process mới với mỗi request (để chạy script). Do đó, **FastCGI** ra đời, giúp xử lý một chuỗi requests trên cùng một process, các processes này sẽ được quản lý bởi một FastCGI server (khác với Web Server, và chúng có thể giao tiếp với nhau qua socket, named pipe, TCP,...). **SCGI** (Simple CGI) cũng là một giao thức tương tự FastCGI. Hướng tiếp cận của **FastCGI/SCGI** có thể gọi là **Deamon Mode**.
- Một hướng tiếp cận khác để khắc phục hạn chế của CGI là nhúng module xử lý mã nguồn của ứng dụng (ví dụ Python/Ruby interpreter) vào trong tiến trình chạy của server. Hướng tiếp cận này có thể gọi là **Embedded Mode**.
  - Các Web Servers lâu đời như Apache hay Nginx được viết bằng C, do đó chúng **thường được trang bị thêm các modules để có thể nạp các interpreters vào trong tiến trình của mình**. Ví dụ Apache sử dụng `mod_ruby` để nhúng Ruby interpreter vào Apache process, từ đó có thể thực thi mã nguồn Ruby trực tiếp. Các modules khác mà Apache hỗ trợ là `mod_python` (Python), `mod_php` (PHP), `mod_rack`, `mod_rails` (Ruby)...
  - Cũng có nhiều Web Servers được **viết bằng chính ngôn ngữ của ứng dụng**. Ví dụ Puma được viết bằng Ruby nên bản thân nó có thể thực thi mã nguồn Ruby.
- Khi có nhiều Web Servers và Web Frameworks để lựa chọn, thì ứng dụng web cần sử dụng một cổng giao tiếp chung giữa Web Server và Web Framework. **Rack** (Ruby), **WSGI** (Python), **PSGI** (Perl)... là cổng giao tiếp chung đó. Chúng là các interfaces ở mức cao hơn so với CGI, FastCGI, SCGI hay `mod_ruby`, `mod_python`,....
  - Chúng có thể hoạt động ở cả Deamon Mode và Embedded Mode. 
  - Ví dụ: **Phusion Passenger** là một **Rack handler**, cho phép chạy ở cả **Standalone mode** và **Integration mode** (tích hợp sử dụng Apache/Nginx)

### Tham khảo

- [CGI to WSGI](https://docs.python.org/2/howto/webservers.html)
- [CGI to Rack](https://stackoverflow.com/a/39858533)
- [Phusion Passenger](https://www.phusionpassenger.com/docs/tutorials/fundamental_concepts/ruby/)

## Framework vs Library
