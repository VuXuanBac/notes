# Prisma

Prisma là một thư viện ORM mã nguồn mở cho NodeJS và TypeScript, thiết kế theo hướng **schema-first** (giống như GraphQL), tức là chỉ cần định nghĩa ORM schema, thư viện có thể sinh cả database migrations, models và queries API.

Prisma schema (`schema.prisma`) sử dụng một ngôn ngữ hướng khai báo riêng, cho phép khai báo:
- Cấu hình để kết nối Database
- Cấu hình để sinh **Prisma Client**
- Cấu trúc Models

Có hai hướng sử dụng Prisma:
- Khai báo cấu trúc Models trong schema và tạo database migration sử dụng **Primsa Migrate**
- Tạo Models có sẵn trong database qua cơ chế **Introspection**

Sau khi có cấu trúc Models (trong schema) thì có thể tạo **Prisma Client** để tạo các truy vấn và thay đổi database với cú pháp của JS (hoặc TS).

## So sánh với các thư viện khác

Prisma là thư viện có mức abstraction cao, của một ORM. Trong khi [**knex.js**](https://knexjs.org/) lại hướng đến là một Query Builder (tránh phải viết SQL nhưng vẫn cần tư duy theo SQL)

Các ORMs truyền thống (như [**Sequelize**](https://sequelize.org/)) thiết kế theo hướng khai báo Models qua các Classes và ánh xạ chúng tới Database Tables. Việc ánh xạ này có thể dẫn tới [**object-relational impedance mismatch**](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch), mà một ví dụ của nó là **N+1 Query**.

Prisma, một ORM, nhưng được trang bị (và hướng tới cải thiện) khả năng tối ưu câu truy vấn và bộ công cụ sinh tự động, cho phép đạt được năng suất cao hơn so với các ORMs truyền thống.

Tất nhiên, hạn chế của nó là ít khả năng tùy chỉnh hóa và không đảm bảo câu truy vấn là tối ưu với mọi trường hợp.

## Schema



# Joi
