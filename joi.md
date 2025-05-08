# Joi

[Joi](https://joi.dev/) là thư viện cho để đánh giá tính hợp lệ của dữ liệu (validation). So sánh với [Zod](https://zod.dev/), một thư viện ra đời sau với ưu điểm gọn nhẹ và hỗ trợ sẵn TypeScript, Joi cho thấy ưu điểm ở hướng thiết kế **hướng khai báo**, kể cả cho các yêu cầu phức tạp.

Joi (và Zod) mô tả dữ liệu dưới dạng schema, một object mô tả cấu trúc dữ liệu với kiểu dữ liệu và các ràng buộc. Ví dụ, một string có độ dài từ 5 đến 30 ký tự có schema là `Joi.string().min(5).max(30)` (utf-8 encoding)

Mỗi điều kiện áp dụng được gọi là **rules**, chúng được thiết kế để tạo **chainable calls**, *thêm rule mới cho schema sẽ tạo schema object mới* (**immutable**).

Có các kiểu dữ liệu sau trong Joi: `string`, `number`, `boolean`, `array`, `symbol`, `object`, `date`, `function`, `binary`, `alternatives` và `any`

## Các properties dùng chung cho các kiểu

- `type` đọc kiểu dữ liệu của schema
- `valid`, `invalid`, `disallow`, `not` lọc ra những values hợp lệ/không hợp lệ
- `allow` bổ sung vào danh sách matched values. Kết hợp sử dụng `only` thì tương đương với `valid`
- `default` dùng giá trị mặc định (literal, ref, function,) nếu giá trị đầu vào là `undefined`
- `forbidden` chỉ cho phép `undefined`
- `optional` cho phép cả `undefined`
- `required`, `exist` không cho phép `undefined`
- `cast` ép kiểu cho matched value
  - `map`: dùng cho object, chuyển về `Map`
  - `number`: dùng cho boolean, date
  - `set`: dùng cho array, chuyển về `Set`
  - `string`: dùng cho binary, boolean, date và number
- `concat` kết hợp rules giữa các schemas cùng kiểu
- `description` text mô tả cho schema

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

## Alternatives

- Dùng khi muốn tạo một schema dựa trên điều kiện.
- **Union Style**: sử dụng `try(...)` để thêm các schema cần kiểm tra (hoặc truyền mảng trực tiếp vào `alternatives()`)
  - Xác định mode kiểm tra qua `match(mode)`
    - `all`: yêu cầu khớp toàn bộ
    - `any`: có thể khớp bất kỳ schema nào
    - `one`: phải khớp đúng một schema
- **If...Else Style**: Có thể cấu hình nâng cao với `conditional(condition, options)`: cấu hình trong trường hợp nào thì dùng schema nào
  - **condition**: có thể là object key, reference hoặc schema
  - **options**: chỉ định khi `condition` thỏa mãn điều kiện thì sẽ dùng schema nào
    - `is` và `not`: Xác định điều kiện
    - `then` và `otherwise`: Khi điều kiện true/false thì sẽ dùng schema nào
    - `switch`: Mảng của `{is, then}` để tạo switch...case...

```ts
// When `b`'s value is a number then `a` must be a string
// Otherwise, `a` must be a number
const schema = {
    a: Joi.alternatives().conditional('b', { is: Joi.number(), then: Joi.string(), otherwise: Joi.number() }),
    b: Joi.any()
};

// When current value is a object with b is 5 (other keys can existed), then the value must be a object with key `a` is a string 
// Otherwise, the value must be a object with key `a` is a number
const schema = Joi.alternatives().conditional(Joi.object({ b: 5 }).unknown(), {
    then: Joi.object({
        a: Joi.string(),
        b: Joi.any()
    }),
    otherwise: Joi.object({
        a: Joi.number(),
        b: Joi.any()
    })
});
```

## Reference

Reference là một object đặc biệt trong Joi, dùng để tham chiếu tới một giá trị. Giá trị đó được phân giải tại thời điểm thực thi. 

Một ví dụ của dùng reference là trường `confirm_password` cần khớp với `password`, khi đó trong schema của `confirm_password`, ta sử dụng `ref` để trỏ tới `password`

Một reference được định nghĩa bởi
- `key` xác định target, có thể xác định như một 
  - relative path: trỏ tới một trường tổ tiên hoặc cùng cấp, ví dụ `c.d` trỏ tới trường `d` bên trong `c` và `c` cùng cấp với trường hiện tại, `.....c.d` để trỏ đến tổ tiên. Có thể xác định `options.ancestor` (number) để giảm bớt số `.` trong path hoặc sử dụng `/` để chỉ từ root.
  - context path: bắt đầu bằng `$`, đại diện cho vị trí được xác định trong context (context được chỉ định trong `options` của `validate`)
- `options`
  - `adjust` một hàm để biến đổi giá trị tham chiếu
  - `map` một mảng để ánh xạ, biển đổi giá trị tham chiếu
  - `in` tạo **In-Reference**
  - Xem thêm: [ref](https://joi.dev/api/?v=17.13.3#refkey-options)

**In-Reference** là một Reference được phân giải như một mảng. Ví dụ các hàm `allow`, `valid`, `invalid` cho phép nhận vào một mảng các giá trị, có thể sử dụng In-Reference để truyền giá trị cho chúng

## Validation

Sau khi tạo schema, ta có thể dùng nó để đánh giá tính hợp lệ của dữ liệu, việc đánh giá sẽ thực hiện theo thứ tự: ưu tiên các **inclusive rules** trước và sau đó là **exclusive rules**.

- `assert(value, schema, [message], [options])` **Throw lỗi** nếu không hợp lệ
- `attempt(value, schema, [message], [options])` Trả về dữ liệu hợp lệ và **Throw lỗi** nếu không hợp lệ
- `{value, error, warning, artifacts} = schema.validate(value, [options])` Trả về kết quả đánh giá

Những `options` cho validation

|Cấu hình|Ý nghĩa|
|--|--|
|`abortEarly`|Dừng validation và trả về ngay với lỗi đầu tiên|
|`convert`|True nếu muốn ép kiểu cho value|
|`dateFormat`|date string format dùng cho error message và ép kiểu. Các giá trị có thể là: `date`, `time`, `iso`, `utc`, `string` (JS date string)|
|`debug`|True nếu muốn hiển thị thêm thông tin trong lỗi|
|`errors.escapeHtml`|escape HTML trong error template|
|`errors.language`|Ngôn ngữ dùng để tìm error message trong khi cấu hình `messages()`|
|`messages`|Object để ghi đè message cho từng mã lỗi|
|`context`|Xác định context data để tạo [**Reference**](#reference)|

Xem thêm: [validate](https://joi.dev/api/?v=17.13.3#anyvalidatevalue-options)


## Cấu hình nâng cao

### Tạo custom rule

Có thể dùng method `custom` để tạo một validation rule bằng một hàm tự tạo. 

Hàm tự tạo này có thể thực hiện bất kỳ chuyển đổi nào. Nó cần trả về giá trị (đã biến đổi hoặc valid value) hoặc tạo error (invalid value)

Hàm tự tạo có thể nhận vào hai đối số
- `value` giá trị dùng để đánh giá
- `{schema, state, prefs, original, error, message, warn}`
  - `schema` biểu diễn schema hiện tại
  - `state` trạng thái kiểm tra hiện tại
  - `preferences`
  - `original` giá trị gốc của `value` trước khi thực hiện bất kỳ biến đổi nào
  - `error(code, [local], [localState])` hàm để sinh mã lỗi
  - `message(messages, [local])` hàm để sinh message
  - `warn(code, [local])` hàm để sinh cảnh báo

```ts
const method = (value, helpers) => {

    // Throw an error (will be replaced with 'any.custom' error)
    if (value === '1') {
        throw new Error('nope');
    }

    // Replace value with a new value
    if (value === '2') {
        return '3';
    }

    // Use error to return an existing error code
    if (value === '4') {
        return helpers.error('any.invalid');
    }

    // Override value with undefined to unset
    if (value === '5') {
        return undefined;
    }

    // Return the value unchanged
    return value;
};

const schema = Joi.string().custom(method, 'custom validation');
```

### Tạo custom message

**Khi chỉ muốn ghi đè key name dùng trong message**, sử dụng `label()`

**Khi muốn trả về custom error khi validation fail**, sử dụng `error()`
- Truyền vào một `Error` hoặc một hàm `(errors) => Error | errors` với `errors` là mảng của các validation reports
- Mọi validation fail đều dùng kết quả trả về của hàm này

### Kế thừa schema

Khi muốn dùng lại schema cho một trường trong object hoặc một schema khác, sử dụng `extract(path)`
- `path` có thể là dot string/mảng để chỉ keys path của object field (hoặc `id`s của schema)
- ID của schema được gán qua `id()`

```ts
const schema = Joi.object({ foo: Joi.object({ bar: Joi.number() }) });
const number = schema.extract('foo.bar');
const result = schema.extract(['foo', 'bar']); //same as number
```

**Khi muốn sao chép schema nhưng có một số chỉnh sửa trên các fields**, sử dụng `fork(paths, adjuster)`
- `adjuster` là một hàm nhận vào schema và trả về schema mới
- Với mỗi field `path` xác định trong `paths`, chỉnh sửa lại schema của nó bằng `adjuster`
