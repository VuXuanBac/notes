# Typescript

Typescript (TS) hỗ trợ cú pháp để giúp lập trình với JavaScript (JS) chặt chẽ về kiểu dữ liệu, từ đó phát hiện lỗi ngay khi lập trình. 

TS là phiên bản `static type` của JS. Thực tế mã nguồn ứng dụng TS sẽ được compiler **transpile** (`transform to another language that has a similar level of abstraction`) về JS.

Bộ compiler của TS là `tsc`. Quá trình transpile sẽ thực hiện loại bỏ các **Type Annotations**, do đó mà sẽ không ảnh hưởng đến logic xử lý của chương trình.

TS compiler cũng hỗ trợ **Downleveling** trong quá trình transpile, với option `--target`, ví dụ `tsc --target es2015`. Mặc định `target = es5`.

## Strict Modes

Mã nguồn ứng dụng có thể chỉ thực hiện type annotating cho một số phần và để những phần khác sử dụng loose typed của JS. TS Compiler vẫn có thể dịch nó về JS. Ta có thể thực hiện cấu hình **strictness** để chỉ thị `tsc` mức độ nghiêm ngặt của việc áp dụng type annotating trong mã nguồn. `strict` flag chứa một họ các strict mode, tương ứng với sự kiểm tra nghiêm ngặt trong một số trường hợp cụ thể:
- `noImplicitAny: true`: Báo lỗi cho các biến được suy đoán ngầm định (implicitly infer) hoặc được gán là kiểu `any` (kiểu mặc định nếu `tsc` không thể suy đoán một kiểu cụ thể)
- `strictNullChecks: true`: Báo lỗi nếu một biến có thể nhận giá trị `null` hoặc `undefined` trong khi thực hiện một hành động mà cần một giá trị cụ thể (ví dụ truy cập thuộc tính trên biến đó)
- Danh sách các [**strict modes**](https://www.typescriptlang.org/tsconfig/#strictNullChecks)
- `strict: true`: Kích hoạt tất cả các **strict modes**

## Data Types

**Ba kiểu dữ liệu cơ bản** trong JS được dùng lại trong TS là: `string`, `number` (bao gồm cả `int` và `float`, TS không tách biệt) và `boolean`.

Khi chỉ định một mảng thuần nhất với ba kiểu trên, có thể sử dụng `string[]`, `number[]` và `boolean[]`.

**Khi không muốn xác định kiểu**, có thể sử dụng `any`, từ đó có thể thực hiện các thao tác mà `tsc` sẽ không kiểm tra.

**Về khai báo hàm**, có thể xác định kiểu dữ liệu cho cả tham số và kết quả trả về. Kiểu trả về thông thường sẽ được ngầm định qua kiểu dữ liệu của các lệnh `return`. Ta có thể sử dụng kiểu `void` nếu hàm không trả về giá trị.

`tsc` sẽ thực hiện kiểm tra:
- Số lượng các đối số truyền vào
- Kiểu dữ liệu và thứ tự của các đối số (nếu xác định kiểu dữ liệu)
- Kiểu dữ liệu của giá trị trả về (nếu chỉ định)

```tsx
function add(n1: number, n2: number): number {
  // return type can be infered based on `return` statements
  return n1 + n2;
}
```

**Với anonymous function và arrow function**, TS có thể ngầm định kiểu dựa trên bối cảnh thực thi của chúng (ví dụ một callback cho hàm `forEach` trên mảng các `string`) => **contextual typing**

**Với objects**, type annotation cho chúng là một object của các thuộc tính và kiểu dữ liệu tương ứng của chúng, phân tách bởi dấu `,` hoặc `;`.

**Khi thuộc tính của object có thể không được xác định**, có thể sử dụng `?` phía sau tên thuộc tính.

```tsx
function printCoord(pt: { x: number; y: number, z?: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
  console.log("The coordinate's z value is " + pt.z); // undefined if missing
}
printCoord({ x: 3, y: 7 });
```

### Union Types

**Khi cần biểu diễn một kiểu dữ liệu là hỗn hợp của nhiều loại** (biến có thể nhận giá trị thuộc một trong nhiều kiểu cho trước), ta sử dụng toán tử `|` nối giữa các kiểu dữ liệu. => **union type**

Với **union type**, `tsc` sẽ báo lỗi nếu thực hiện một hành động mà một trong các kiểu dữ liệu không hỗ trợ. Ví dụ một `number | string` không thể gọi `toUpperCase()`
- Khi đó, ta có thể sử dụng `typeof` (hoặc các cách khác như `Array.isArray()`) để làm hẹp kiểu dữ liệu mà TS ngầm định cho biến. => **type guard**

**Type guard** có thể được triển khai dựa trên một số cách

**Các giá trị hằng số thuộc `number`, `string` và `boolean` (`true`, `false`) có thể trở thành kiểu dữ liệu** => **literal types**. Điều đó, khi kết hợp với **union**, ta có thể tạo một kiểu enum đơn giản.

```tsx
function compareText(s1: string, s2: string, start: "left" | "right" | number): -1 | 0 | 1 {}
```

### Type Aliases and Interfaces

**Khi một kiểu là phức tạp**, ta có thể sử dụng **type aliases** để gán tên cho kiểu đó, từ đó có thể dùng lại ở nhiều vị trí.

```tsx
type Point = {
  x: number;
  y: number;
  z?: number
};

type ID = number | string;
 
function updatePoint(id: ID, pt: Point) {}
```

Một cú pháp khác có thể sử dụng là **interfaces**, với cú pháp hoàn toàn tương tự.

```tsx
interface Point {
  x: number;
  y: number;
  z?: number
};
```

### Type Assertions

**Ta có thể thực hiện ép kiểu** với từ khoá `as`, việc này giúp giới hạn kiểu ứng với context cụ thể hiện tại. Tuy nhiên, điều này chỉ có ý nghĩa đối với `tsc` trong việc phân tích mã nguồn (*compile time*), ép kiểu sẽ không thực sự được chuyển đổi thành `null` hay `undefined` khi thực thi (*run time*) nếu như kiểu dữ liệu không đúng.
- Việc ép kiểu chỉ được thực hiện theo hướng cụ thể hóa hoặc tổng quát hóa
- Khi muốn ép kiểu giữa hai kiểu không có điểm chung (ví dụ `number` và `string`), ta có thể chuyển về `any` trước. Ví dụ: `const a = expression as any as T`

**Đối với literal types**, việc gán giá trị là một kiểu rộng hơn sẽ gây lỗi. Ví dụ:

```tsx
function printText(s: string, alignment: "left" | "right" | "center") {}

const data = {message: "hello world", alignment: "left"};
// ERROR: Argument of type 'string' is not assignable to parameter of type '"left" | "right" | "center"'.
printText(data.message, data.alignment);
```

Lý do ví dụ trên có lỗi là `data` đang được ngầm định với kiểu `data: {message: string; alignment: string;};`

Ta có thể sử dụng `as const` để ép kiểu cho `data` về dạng object của các **literal types**.

```tsx
function printText(s: string, alignment: "left" | "right" | "center") {}

const data = {message: "hello world", alignment: "left"} as const;
// Now the type of `data` is
// data: {
//     readonly message: "hello world";
//     readonly alignment: "left";
// };
```

**Đối với `null` và `undefined`**, ta có thể đảm bảo một biểu thức không thể nhận giá trị `null` và `undefined` bằng việc thêm `!` vào sau biểu thức đó. Khi đó, ngay cả khi strict mode `strictNullChecks` được kích hoạt, thì `tsc` cũng không báo lỗi.

```tsx
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

# References

- [Documentation](https://www.typescriptlang.org/)
