# Joi

[Joi](https://joi.dev/) là thư viện cho để đánh giá tính hợp lệ của dữ liệu (validation). So sánh với [Zod](https://zod.dev/), một thư viện ra đời sau với ưu điểm gọn nhẹ và hỗ trợ sẵn TypeScript, Joi cho thấy ưu điểm ở hướng thiết kế **hướng khai báo**, kể cả cho các yêu cầu phức tạp.

Joi (và Zod) mô tả dữ liệu dưới dạng schema, một object mô tả cấu trúc dữ liệu với kiểu dữ liệu và các ràng buộc. Ví dụ, một string có độ dài từ 5 đến 30 ký tự có schema là `Joi.string().min(5).max(30)` (utf-8 encoding)

Mỗi điều kiện áp dụng được gọi là **rules**, chúng được thiết kế để tạo **chainable calls**, *thêm rule mới cho schema sẽ tạo schema object mới* (**immutable**).

Sau khi tạo schema, ta có thể dùng nó để đánh giá tính hợp lệ của dữ liệu, việc đánh giá sẽ thực hiện theo thứ tự: ưu tiên các **inclusive rules** trước và sau đó là **exclusive rules**.

- `assert(value, schema, [message], [options])` **Throw lỗi** nếu không hợp lệ
- `attempt(value, schema, [message], [options])` Trả về dữ liệu hợp lệ và **Throw lỗi** nếu không hợp lệ
- `{value, error, warning, artifacts} = schema.validate(value, [options])` Trả về kết quả đánh giá
