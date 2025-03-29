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

**Khi chỉ định một mảng thuần nhất** với ba kiểu trên, có thể sử dụng `string[]`, `number[]` và `boolean[]`.

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

### Union Types

**Khi cần biểu diễn một kiểu dữ liệu là hỗn hợp của nhiều loại** (biến có thể nhận giá trị thuộc một trong nhiều kiểu cho trước), ta sử dụng toán tử `|` nối giữa các kiểu dữ liệu. => **union type**

Với **union type**, `tsc` sẽ báo lỗi nếu thực hiện một hành động mà một trong các kiểu dữ liệu không hỗ trợ. Ví dụ một `number | string` không thể gọi `toUpperCase()`
- Khi đó, ta có thể sử dụng `typeof` (hoặc các cách khác như `Array.isArray()`) để làm hẹp kiểu dữ liệu mà TS ngầm định cho biến. => **type guard**

**Type guard** có thể được triển khai dựa trên một số cách:
- Sử dụng `typeof` đối với `"string", "number", "bigint", "boolean", "symbol", "undefined", "object", "function"`
- Sử dụng Truthiness để xác định cụ thể hơn một giá trị có phải `null` hay `undefined` không.
- Sử dụng `switch..case..` và các so sánh bằng/khác (`===`, `!==`, `==`, `!=`)
- Sử dụng toán tử `in` (kiểm tra một thuộc tính có thuộc một biến hay prototype của nó hay không)
- Sử dụng toán tử `instanceof`
- Sử dụng [**type predicates**](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates) - định nghĩa một hàm predicate (boolean) trả về dạng `parameterName is Type`. Khi đó mỗi khi gọi hàm đó để kiểm tra một biến thì `tsc` biết khi nào biến đó sẽ có kiểu dữ liệu nào.

**Các giá trị hằng số thuộc `number`, `string` và `boolean` (`true`, `false`) có thể trở thành kiểu dữ liệu** => **literal types**. Điều đó, khi kết hợp với **union**, ta có thể tạo một kiểu enum đơn giản.

```tsx
function compareText(s1: string, s2: string, start: "left" | "right" | number): -1 | 0 | 1 {}
```

### Object

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

Xem thêm cú pháp tại phần [Interfaces](#interfaces)

### Type Aliases

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

### Interfaces

Tương tự aliases, **interfaces** cũng dùng để khai báo một kiểu custom, dành cho object.

```tsx
interface Point {
  x: number;
  y: number;
  z?: number
};
```

**Khi thuộc tính của object hỗ trợ indexing**, ta sử dụng cú pháp `[index: indexType]: valueType`, trong đó `indexType` chỉ có thể là `string`, `number`, `symbol`,template string và [**union type**](#union-types) của các kiểu trên.

Chú ý: Do `obj.property` giống với `obj["property"]` nên `tsc` sẽ báo lỗi nếu có thuộc tính với kiểu dữ liệu không phải là `valueType`.

```tsx
interface NumberDictionary {
  [index: string]: number;
 
  length: number; // ok
  name: string; // Property 'name' of type 'string' is not assignable to 'string' index type 'number'.
}

interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```

**Khi thuộc tính của object không được gán lại**, ta thêm từ khóa `readonly` trước tên thuộc tính trong khai báo.

**Ưu điểm của interfaces là nó cho phép kế thừa (hỗ trợ đa kế thừa)** với từ khóa `extends`

```tsx
interface Colorful {
  color: string;
}
 
interface Circle {
  radius: number;
}
 
interface ColorfulCircle extends Colorful, Circle {
  isFill: boolean
}
```

Một cách kế thừa tương tự là tạo giao giữa hai **interfaces** thông qua toán tử `&` => **intersection**

```tsx
type ColorfulCircle = Colorful & Circle;
```

Riêng với **intersection**, nếu có sự xung đột về kiểu với hai thuộc tính trùng tên ở hai **interfaces** thì kết quả giao sẽ là một thuộc tính với kiểu `never` (không thể có một thuộc tính mà đồng thời nhận hai kiểu dữ liệu).

#### Generic Types

Khi thuộc tính của một **interface** không phụ thuộc vào kiểu cụ thể, nếu như sử dụng `any` thì ta lại cần **type guards** mỗi khi thực hiện thao tác trên thuộc tính. Thay vào đó, có thể định nghĩa interface dưới dạng tổng quát với một kiểu giả định. Kiểu thực sự sẽ được tùy chỉnh khi cần annotate cho biến cụ thể.

```tsx
interface Box<Type> {
  contents: Type;
}
 
let boxA: Box<string> = { contents: "hello" }; // Type is now 'string'
let boxB: Box<number> = { contents: 123 }; // Type is now 'number'
```

Cú pháp này cũng có thể áp dụng cho **aliases**

```tsx
type Box<Type> = {
  contents: Type;
}
```

`Array` và phiên bản **readonly** `ReadonlyArray` cũng là các generic types

### Tuples

**Khi một mảng cố định về kích thước và kiểu dữ liệu của các phần tử**, ta có thể sử dụng cú pháp của **tuple**, với các tính năng sau:
- Optional với `?` (chỉ với những phần tử ở cuối)
- Readonly
- Rest elements

```tsx
const [x, y, z, color]: [number, number, number, string] = [1, 0, 1, 'red'];

// Optional z and color
const [x, y, z, color]: [number, number, number?, string?] = [1, 0, 1]; // [x, y, z, color] = [1, 0, 1, undefined]

// Rest elements
type restAreBooleans = [string, ...boolean[], number]; // expect any value like [string, number], [string, boolean, number], [string, boolean, boolean, number]...
```

### Special Data Types

Dưới đây là một số kiểu dữ liệu đặc biệt mà TS hỗ trợ:
- `void`: Dùng cho returnType để chỉ một hàm không trả về giá trị.
- `object`: Một kiểu khớp với mọi kiểu dữ liệu ngoài `string, number, bigint, boolean, symbol, null, undefined`. Chú ý: `object` khác với JS Plain object hay `Object`
- `unknown`: Tương tự `any` nhưng không hỗ trợ bất kỳ thao tác nào.
- `never`: Chỉ một trạng thái không thể xảy ra (ví dụ đã xét hết mọi trường hợp trong mệnh đề điều kiện hay một hàm luôn trả lỗi)
- `Function`: Đại diện cho một hàm (có thể gọi và có các thuộc tính như `bind`, `call`, `apply`,...)

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

### Function Annotating

**Khi muốn annotating cho cả một function** (ví dụ một callback), ta sử dụng cú pháp `(parameterName: Type,...) => returnType`. Khi bỏ trống `Type` cho các tham số thì mặc định chúng là `any`.

```tsx
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}
```

#### Generic Functions

Ta có thể định nghĩa **hàm dưới dạng tổng quát** (**generic functions**), ở đó chữ ký của hàm không phụ thuộc vào các kiểu cụ thể (thay vào đó hàm được định nghĩa để tương thích với nhiều kiểu khác nhau). Khi đó, `tsc` xác định chữ ký của hàm tùy thuộc vào lời gọi (khi truyền đối số cụ thể). Các kiểu dữ liệu mà hàm phụ thuộc sẽ được khai báo ngay sau tên hàm.

- Ví dụ, hàm `map` ở đây có chữ ký phụ thuộc vào hai kiểu tổng quát đại diện bởi `Input` và `Output`. `Input` và `Output` thực sự đại diện cho kiểu dữ liệu nào chỉ được xác định tùy thuộc vào đối số truyền cho `map`.
```tsx
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
 
// 'arr' is of type 'string[]' ==> Input is now 'string'
// 'parseInt' returns a 'number' ==> Output is now 'number' ==> 'parsed' is now 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));

// 'arr' is of type 'any' ==> Input is now 'any'
// 'Number.isInteger' returns a 'boolean' ==> Output is now 'boolean' ==> 'parsed' is now 'boolean[]'
const parsed = map([1, "abc", false], (n) => Number.isInteger(n));
```

Khi muốn **giới hạn phạm vi cho các `Type`s** trong **generic functions**, ta sử dụng từ khóa `extends` khi khai báo kiểu phụ thuộc cho hàm. Sau `extends` là một object khai báo thuộc tính và kiểu dữ liệu mà `Type` cần hỗ trợ.

```tsx
// allow using any data type that support `length` property (that returns a number).
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
```

Khi phân tích lời gọi hàm, TS sẽ tự động xác định kiểu dữ liệu cụ thể cho các kiểu phụ thuộc, tuy nhiên, việc "tự động" này có thể không đúng với ý muốn khi gọi hàm. Khi đó ta cần chỉ định rõ ràng khi gọi hàm.

```tsx
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {}

// TS infer that `Type` is now `number`
// ERROR: Type 'string' is not assignable to type 'number'.
const arr = combine([1, 2, 3], ["hello"]);

// Explicitly declare that `Type` is `string | number`
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

#### Special Parameters and Arguments

TS hỗ trợ **Rest Parameters** trong định nghĩa hàm, trong đó, kiểu dữ liệu của nó luôn phải ở dạng `Array<T>` hoặc `T[]`

```tsx
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

**Với Parameter Destructuring** (để unpack một object thành các biến cục bộ trong hàm), type annotation của nó là một object với key và kiểu dữ liệu (tương tự annotating cho các objects khác)

```tsx
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

# References

- [Documentation](https://www.typescriptlang.org/)
