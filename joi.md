# Joi

[Joi](https://joi.dev/) là thư viện cho để đánh giá tính hợp lệ của dữ liệu (validation). So sánh với [Zod](https://zod.dev/), một thư viện ra đời sau với ưu điểm gọn nhẹ và hỗ trợ sẵn TypeScript, Joi cho thấy ưu điểm ở hướng thiết kế **hướng khai báo**, kể cả cho các yêu cầu phức tạp.

Joi (và Zod) mô tả dữ liệu dưới dạng schema, một object mô tả cấu trúc dữ liệu với kiểu dữ liệu và các ràng buộc. Ví dụ, một string có độ dài từ 5 đến 30 ký tự có schema là `Joi.string().min(5).max(30)` (utf-8 encoding)

Mỗi điều kiện áp dụng được gọi là **rules**, chúng được thiết kế để tạo **chainable calls**, *thêm rule mới cho schema sẽ tạo schema object mới* (**immutable**).

Sau khi tạo schema, ta có thể dùng nó để đánh giá tính hợp lệ của dữ liệu, việc đánh giá sẽ thực hiện theo thứ tự: ưu tiên các **inclusive rules** trước và sau đó là **exclusive rules**.

- `assert(value, schema, [message], [options])` **Throw lỗi** nếu không hợp lệ
- `attempt(value, schema, [message], [options])` Trả về dữ liệu hợp lệ và **Throw lỗi** nếu không hợp lệ
- `{value, error, warning, artifacts} = schema.validate(value, [options])` Trả về kết quả đánh giá

Có các kiểu dữ liệu sau trong Joi: `string`, `number`, `boolean`, `array`, `symbol`, `object`, `date`, `function`, `binary`, `alternatives` và `any`

## String

- Mặc định, empty string không pass, trừ khi xác định `allow('')`
- Một số dạng validations: `alphanum` (`[a-zA-Z0-9]`), `token` (`\w`), `base64`, `case(upper|lower)`, `dataUri`, `email`, `guid`, `hex`, `insensitive`, `isoDate`, `isoDuration`, `length`, `max`, `min`, `normalize(NFC|NFD|NFKC|NFKD)`, `pattern`, `truncate`,...
- Các methods có thể hoạt động với **`convert: true`**: `lowercase`, `uppercase`, `trim` và `replace`

## Number

- Khớp với cả số nguyên, số thực và string ở dạng số, ngoại trừ `Infinity` và `-Infinity`
- Mặc định, chỉ pass đối với các số và chuỗi trong khoảng `[Number.MIN_SAFE_INTEGER, Number.MAX_SAFE_INTEGER]`. Có thể sử dụng `unsafe` để cho phép cả các số nằm ngoài dải
- Một số dạng validations: `greater`, `less`, `max`, `min`, `multiple` (đánh giá sử dụng %, nên có thể sai với số thực), `integer`, `negative`, `positive`, `sign(positive|negative)`
- Các methods có thể hoạt động với **`convert: true`**: `precision`, `number` (ép kiểu string về số)

## Boolean/Bool

- Các methods có thể hoạt động với **`convert: true`**: `boolean/bool` (ép kiểu từ string `"true"|"false"`)
  - Có thể xác định danh sách các giá trị là truthy hay falsy

```ts
const boolean = Joi.boolean().falsy('N', 'no', 'disabled').sensitive();
```

## Date

- Khớp với JS Date, Date string và Timestamp (số ms từ Unix Epoch), sử dụng `Date.parse` để kiểm tra
- Một số dạng validations: `greater`, `less`, `max`, `min`, `iso` (ISO 8601), `timestamp(javascript|unix)` (javascript biểu diễn theo ms, unix biểu diễn theo s)

## Array

- Mặc định có item là `undefined` sẽ không pass, trừ khi sử dụng `sparse`
- Yêu cầu ít nhất 1 item có kiểu xác định: `has(schema)`
- Yêu cầu các items phải thuộc một trong các kiểu xác định: `items(...)`
- Yêu cầu các items theo thứ tự phải có kiểu xác định: `ordered(...)`
- Cho phép một giá trị **không phải mảng** cũng pass: `single`
- Yêu cầu (hoặc **`convert: true`**) mảng phải được sắp xếp theo thứ tự: `sort({order: ascending|descending, by: <name>})`
- Giới hạn số lượng items: `max`, `min`
- Yêu cầu các items không lặp lại: `unique(comparator, options)`, comparator có thể là một hàm hoặc một "dot string" xác định level dùng để so sánh items

```ts
const schema = Joi.array().unique((a, b) => a.customer.id === b.customer.id);

const schema = Joi.array().unique('customer.id');
```

## Object

- Yêu cầu object là instance của class: `instance(constructor)`, `regex` (`RegExp`)
- Đánh giá mối quan hệ giữa các fields:
  - `and`: Cùng xuất hiện hoặc không xuất hiện
  - `nand`: Ít nhất một không xuất hiện
  - `or`: Ít nhất một xuất hiện
  - `xor`: Đúng một xuất hiện
  - `oxor`: Tối đa một xuất hiện
  - `with`: Khi một trường `key` xác định, yêu cầu tất cả các trường `peers` cũng xác định
  - `without`: Khi một trường `key` xác định, yêu cầu tất cả các trường `peers` không được xác định

```ts
const schema = Joi.object({
    a: Joi.any(),
    b: Joi.any()
}).and('a', 'b');

```

- Merge các object schema: `append`, `keys`
- Giới hạn số lượng keys: `length`, `max`, `min`
- Cho phép có các trường không xác định: `unknown`
- Đánh giá giá trị cho các trường không xác định: `pattern(pattern, schema)`, trong đó pattern để khớp với key và schema là kiểu dữ liệu gán cho matched key

```ts
const schema = Joi.object({
    a: Joi.string()
}).pattern(Joi.string().min(2).max(5), Joi.boolean());
```

- Đổi tên cho các trường: `rename(from, to)`
