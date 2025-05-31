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

| Cấu hình            | Ý nghĩa                                                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `abortEarly`        | Dừng validation và trả về ngay với lỗi đầu tiên                                                                                      |
| `convert`           | True nếu muốn ép kiểu cho value                                                                                                      |
| `dateFormat`        | date string format dùng cho error message và ép kiểu. Các giá trị có thể là: `date`, `time`, `iso`, `utc`, `string` (JS date string) |
| `debug`             | True nếu muốn hiển thị thêm thông tin trong lỗi                                                                                      |
| `errors.escapeHtml` | escape HTML trong error template                                                                                                     |
| `errors.language`   | Ngôn ngữ dùng để tìm error message trong khi cấu hình `messages()`                                                                   |
| `messages`          | Object để ghi đè message cho từng mã lỗi                                                                                             |
| `context`           | Xác định context data để tạo [**Reference**](#reference)                                                                             |

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

# Zod

Zod là thư viện validation, tương tự [Joi](joi), tuy nhiên có nhiều điểm khác biệt:
- Viết từ Typescript và hỗ trợ infer kiểu dữ liệu từ Schema
- Không phụ thuộc vào thư viện khác (Joi sử dụng thư viện `hoek`)
- Thiết kế nhỏ gọn, đồng thời cũng yêu cầu customize nhiều hơn với những yêu cầu đặc biệt

## Datatype Infer

Về kiểu dữ liệu, Zod hỗ trợ sẵn `infer` để tạo một TS type từ schema (với tính năng này, Joi sẽ cần sử dụng thư viện thứ ba như `joi-extract-type` hoặc `joi-to-typescript`)

```ts
import z from 'zod';

const Player = z.object({ 
  username: z.string(),
  xp: z.number()
});
 
// extract the inferred type
type Player = z.infer<typeof Player>;


// When used with `transform` (input type and output type can be different)
const mySchema = z.string().transform((val) => val.length);
 
type MySchemaIn = z.input<typeof mySchema>;
// => string
 
type MySchemaOut = z.output<typeof mySchema>; // equivalent to z.infer<typeof mySchema>
// number
```

## Validation

Về validation, Zod cung cấp các methods sau:

- `parse`: Throw `ZodError` nếu validation fail; Trả về dữ liệu nếu validation pass
- `safeParse`: Trả về object chứa `success`, `data` và `error`
- `parseAsync` và `safeParseAsync`: Sử dụng khi các hàm validate hoặc [refine](#tùy-chỉnh) là các hàm `async`

Các kiểu dữ liệu cơ bản mà Zod hỗ trợ sẵn tính năng validation là `string`, `number`, `boolean`, `bigint`, `symbol`, `undefined` (`void`) và `null`

### Literal và Enum

```ts
const colors = z.literal(["red", "green", "blue"]);
 
colors.parse("green"); // ✅
colors.parse("yellow"); // ❌
colors.values; // => Set<"red" | "green" | "blue">

const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);
 
FishEnum.parse("Salmon"); // => "Salmon"
FishEnum.parse("Swordfish"); // => ❌
const enum_values = FishEnum.enum // { Salmon: "Salmon", Tuna: "Tuna", Trout: "Trout" }

// Exclude
const TunaOnly = FishEnum.exclude(["Salmon", "Trout"]);
const SalmonAndTroutOnly = FishEnum.extract(["Salmon", "Trout"]);
```

### String

```ts
z.string().max(5);
z.string().min(5);
z.string().length(5);
z.string().regex(/^[a-z]+$/);
z.string().startsWith("aaa");
z.string().endsWith("zzz");
z.string().includes("---");
z.string().uppercase();
z.string().lowercase();

// Transform
z.string().trim(); // trim whitespace
z.string().toLowerCase(); // toLowerCase
z.string().toUpperCase(); // toUpperCase

// Format (v4)
z.email({pattern: z.regexes.html5Email});    // default: use rules much like Gmail
z.uuid({ version: "v4" });  // z.guid(), z.uuidv4(),... [1-8]
z.url({ hostname: /^example\.com$/, protocol: /^https$/ });
z.emoji();         // validates a single emoji character
z.base64();
z.base64url();
z.nanoid();
z.cuid();
z.cuid2();
z.ulid();
z.ipv4();
z.ipv6();
z.cidrv4();        // z.cidrv6(): CIDR
z.iso.date();      // YYYY-MM-DD
z.iso.time();      // HH:MM:SS[.s+]
z.iso.datetime({ offset: true }); // `offset` true to allow timezone, `precision` for ms
z.iso.duration();
```

### Number

```ts
z.number();  // in v4, `number = int` (not allow float)
z.int();     // restricts to safe integer range
z.int32();    // [-2147483648, 2147483647]
z.uint32();
z.bigint();
z.float32();  // [-3.4028234663852886e38, 3.4028234663852886e38]
z.float64();  // [-1.7976931348623157e308, 1.7976931348623157e308]

// Comparation (similar onto `bigint`)
z.number().gt(5);
z.number().gte(5);                     // alias .min(5)
z.number().lt(5);
z.number().lte(5);                     // alias .max(5)
z.number().positive();       
z.number().nonnegative();    
z.number().negative(); 
z.number().nonpositive(); 
z.number().multipleOf(5);              // alias .step(5)

// in v3, NaN and Infinity is passed with `number` but not passed in v4
z.nan();
```

### Date

```ts
// Parse Date instance only
z.date().safeParse(new Date()); // ✅
z.date().safeParse("2022-01-12T00:00:00.000Z"); // ❌

z.date().min(new Date("1900-01-01"));
z.date().max(new Date());
```

### Nullish and Special

```ts
// allow undefined
z.optional(z.literal("yoda")); // or z.literal("yoda").optional()

// allow null
z.nullable(z.literal("yoda")); // or z.literal("yoda").nullable()

// allow undefined or null
z.nullish(z.literal("yoda"));

// infer original type
optional.unwrap(); // ZodLiteral<"yoda">
nullable.unwrap(); // ZodLiteral<"yoda">

// ------------------ Typescript Compatible --------------------
// any value will passed
z.any(); // inferred type: `any`
z.unknown(); // inferred type: `unknown`

// no value will passed
z.never(); // inferred type: `never`
```

### Object

```ts
z.object();           // strip unknown keys
z.strictObject();     // not allowed unknown keys
z.looseObject();      // allow and keep unknown keys
z.object().catchall(); // use schema to validate all unknown keys

// Extract internal schema
schema.shape.<key>    // extract schema for provided key
schema.keyof()        // returns enum schema that contains all keys

// Inherit
schema.extend()       // add additional fields
schema.pick({<key>: true}) // just has these keys
schema.omit({<key>: true}) // not has these keys

// Required and Optional
schema.partial()     // all keys are optional
schema.partial({<key>: true})     // these keys are optional
schema.required()     // all keys are required
schema.required({<key>: true})     // these keys are required
```

### Record and Collection

```ts
// Record<string | number | symbol, value_schema>
z.record(z.string(), z.string()); // Record<string, string>
z.record(z.enum(["id", "name", "email"]), z.string()); // { id: string; name: string; email: string }
z.partialRecord(z.enum(["id", "name", "email"]), z.string()); // { id?: string; name?: string; email?: string }

// Map<key_schema, value_schema>
z.map(z.string(), z.number());

// Set<schema>
z.set(z.number());
z.set(z.string()).min(5); // must contain 5 or more items
z.set(z.string()).max(5); // must contain 5 or fewer items
z.set(z.string()).size(5); // must contain 5 exactly
```

### Array và Tuple

```ts
z.array(schema);
z.array(schema).min(5); // must contain 5 or more items
z.array(schema).max(5); // must contain 5 or fewer items
z.array(schema).length(5); // must contain 5 items exactly

z.tuple([schemas]);
z.tuple([schemas], rest_schema); // fixed schemas and rest schemas
```

### Logical

```ts
// union
z.union([schemas]); // first schema matched means matched
z.union([schemas]).options; // array of internal option schemas

// discriminated union: search match by object key (faster search than using `union`)
z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("failed"), error: z.string() }),
]);

// intersection
z.intersection([schemas]); // merge keys on objects but differ from `z.object().extend()`
```

### Một số validations đặc biệt

```ts
// Validate File instance
z.file();
z.file().min(); // minimum File.size in bytes
z.file().max(); // maximum File.size in bytes
z.file().mime([]); // one of provided MIME type

// Validate instanceOf
z.instanceof()
```

## Lỗi

Quá trình validation fail sẽ được coi là lỗi, lỗi đó là instance của `$ZodError` (v4 triển khai subclass `ZodError`).

Lỗi validation mang các thông tin sau:
- `issues`: Các thông tin chi tiết về từng lỗi xảy ra:
  - `code`: Tên lỗi mô tả kiểu lỗi
  - `path`: Mảng các thuộc tính xác định vị trí xảy ra lỗi (như là thuộc tính của object)
  - `message`: Thông báo chi tiết về lỗi
  - `input`: Giá trị đầu vào ở bước validation hiện tại
  - Ngoài ra tùy vào từng lỗi sẽ có thêm các trường đặc biệt, mô tả chi tiết hơn.

### Ghi đè thông báo lỗi

Thông báo chi tiết về lỗi `message` có thể ghi đè ở một số cấp sau, và mức độ ưu tiên là giảm dần

**Schema level message**

Có thể xác định thông báo lỗi ngay khi định nghĩa schema

```ts
// passing directly
z.string("Bad!");
z.string().min(5, "Too short!");
z.uuid("Bad UUID!");
z.iso.date("Bad date!");
z.array(z.string(), "Not an array!");
z.array(z.string()).min(5, "Too few items!");
z.set(z.string(), "Bad set!");

// passing as options
z.string({ error: "Bad!" });
z.string().min(5, { error: "Too short!" });
z.uuid({ error: "Bad UUID!" });
z.iso.date({ error: "Bad date!" });
z.array(z.string(), { error: "Bad array!" });
z.array(z.string()).min(5, { error: "Too few items!" });
z.set(z.string(), { error: "Bad set!" });

// passing as callback options
// return value (except undefined) will be used as issue's message
z.string({ error: (issue)=>`[${Date.now()}]: Validation failure.` });
z.int64({
  error: (issue) => {
    // override too_big error message
    if (issue.code === "too_big") {
      return { message: `Value must be <${issue.maximum}` };
    }
 
    //  defer to default
    return undefined;
  },
});
```

**Parse-level message**

Có thể xác định thông báo lỗi ngay trong lời gọi các hàm validate (`parse`,...).

```ts
schema.parse(12, {
  error: iss => "per-parse custom error"
});
```

**Global-level message**

Sử dụng `z.config()` và ghi đè thuộc tính `customError` để tạo message ở mức toàn cục cho các lỗi

```ts
z.config({
  customError: (iss) => {
    return "globally modified error";
  },
});
```

## Tùy chỉnh

Các Zod schemas đều cho phép định nghĩa một mảng các **refinements**, cho phép tùy chỉnh cách thức validation cho schema đó.

### refine

Để thêm một refinement cho schema hiện tại, sử dụng method `refine` với đối số là một hàm nhận vào giá trị cần validate và trả ra true/false xác định giá trị đầu vào có hợp lệ không.

Bên cạnh hàm callback, có một số options sau có thể cung cấp cho `refine`:
- `abort`: true nếu muốn kết thúc validation tại bước hiện tại nếu validation fail (*non-continuable*). Mặc định Zod sẽ đi lần lượt qua các bước validation trên schema và thu gom toàn bộ các lỗi xảy ra (`abort: false`)
- `message`, `path`,...: cung cấp data cho ZodIssue nếu validation fail

*ZodIssue tạo bởi `refine` luôn có `code: 'custom'`*

```ts
// validate password and confirm must matched
const passwordForm = z
  .object({
    password: z.string(),
    confirm: z.string(),
  })
  .refine((data) => data.password === data.confirm, {
    message: "Passwords don't match",
    path: ["confirm"], // path of error
  });
```

### check

Ngoài ra, có thể dùng `check`, một phiên bản cho phép tùy chỉnh cao hơn so với `refine`. 

Nó cũng nhận vào một hàm callback, nhưng đối số lúc này là `context` đại diện cho trạng thái validate hiện tại:
- Có thể gọi `context.value` để lấy giá trị cần validate
- Có thể gọi `context.issues.push()` để thêm một ZodIssue cho quá trình validation


## Biến đổi giá trị đầu vào

Zod cho phép tạo schema đặc biệt, không chỉ dùng để validate dữ liệu đầu vào, mà còn thực hiện biến đổi nó về một giá trị khác

### coerce

`coerce` cho phép tạo một schema để ép kiểu giá trị đầu vào. Zod sẽ dùng JS constructor của kiểu tương ứng để ép kiểu.

```ts
const schema = z.coerce.string(); // Convert using `new String(value)`
 
schema.parse("tuna");    // => "tuna"
schema.parse(42);        // => "42"
schema.parse(true);      // => "true"
schema.parse(null);      // => "null"
```

### transform

`transform` là phiên bản cho phép tùy chỉnh cao hơn so với `coerce`, nó không chỉ dùng để ép kiểu, mà còn có thể thực hiện thao tác biến đổi bất kỳ.

Đối số của nó là một hàm callback có thể nhận vào một hoặc hai đối số theo thứ tự:
- Giá trị đầu vào của dữ liệu
- Trạng thái hiện tại của quá trình validation `context`, từ đây có thể thêm ZodIssue cho quá trình biến đổi

Giá trị trả về từ callback này sẽ được dùng làm giá trị trả về (`result.data`) khi validate.

```ts
const coercedInt = z.transform((val, ctx) => {
  try {
    const parsed = Number.parseInt(String(val));
    return parsed;
  } catch (e) {
    ctx.issues.push({
      code: "custom",
      message: "Not a number",
      input: val,
    });
 
    // this is a special constant with type `never`
    // returning it lets you exit the transform without impacting the inferred return type
    return z.NEVER; 
  }
});
```

### pipe

Khi muốn thực hiện bước validation trước, rồi mới thực hiện biến đổi giá trị, ta có thể gọi `schema.transform(callback)` trực tiếp trên schema, hoặc gọi `schema.pipe(z.transform(callback))`

```ts
const stringToLength = z.string().transform(val => val.length); 

// Async transform
const idToUser = z
  .string()
  .transform(async (id) => {
    // fetch user from database
    return db.getUserById(id); 
  });
 
const user = await idToUser.parseAsync("abc123");
```

### preprocess

Khi muốn thực hiện biến đổi giá trị trước, rồi mới thực hiện validation, ta có thể gọi `z.preprocess(callback, schema)` (callback của `z.transform`)

```ts
const coercedInt = z.preprocess((val) => {
  if (typeof val === "string") {
    return Number.parseInt(val);
  }
  return val;
}, z.int());
```

### default

Khi muốn trả về một giá trị khi giá trị đầu vào là `undefined`, ta sử dụng `default(value_or_callback)`

```ts
// with value
const defaultTuna = z.string().default("tuna");
 
defaultTuna.parse(undefined); // => "tuna"


// with callback
const randomDefault = z.number().default(Math.random);
 
randomDefault.parse(undefined);    // => 0.4413456736055323
randomDefault.parse(undefined);    // => 0.1871840107401901
randomDefault.parse(undefined);    // => 0.7223408162401552
```

Trong trường hợp muốn thực hiện biến đổi trên giá trị mặc định, sử dụng `prefault(value_or_callback)`

```ts
const a = z.string().trim().toUpperCase().prefault("  tuna  ");
a.parse(undefined); // => "TUNA"
 
const b = z.string().trim().toUpperCase().default("  tuna  ");
b.parse(undefined); // => "  tuna  "
```

### catch

Khi muốn trả về một giá trị ngay cả khi validation fail, ta sử dụng `catch(value_or_callback)` (callback ở đây có đối số là `context`)

```ts
// with value
const numberWithCatch = z.number().catch(42);
 
numberWithCatch.parse(5); // => 5
numberWithCatch.parse("tuna"); // => 42

// with callback
const numberWithRandomCatch = z.number().catch((ctx) => {
  ctx.error; // the caught ZodError
  return Math.random();
});
 
numberWithRandomCatch.parse("sup"); // => 0.4413456736055323
numberWithRandomCatch.parse("sup"); // => 0.1871840107401901
numberWithRandomCatch.parse("sup"); // => 0.7223408162401552
```

### readonly

Khi muốn giá trị sau validation ở dạng readonly, sử dụng `schema.readonly`, giá trị sau `parse` sẽ được truyền vào `Object.freeze()` để trả về

```ts
const ReadonlyUser = z.object({ name: z.string() }).readonly();
type ReadonlyUser = z.infer<typeof ReadonlyUser>;
// Readonly<{ name: string }>

const result = ReadonlyUser.parse({ name: "fido" });
result.name = "simba"; // throws TypeError
```
