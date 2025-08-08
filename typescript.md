# Typescript

Typescript (TS) hỗ trợ cú pháp để giúp lập trình với JavaScript (JS) chặt chẽ về kiểu dữ liệu, từ đó phát hiện lỗi ngay khi lập trình. 

TS là phiên bản `static type` của JS. Thực tế mã nguồn ứng dụng TS sẽ được compiler **transpile** (`transform to another language that has a similar level of abstraction`) về JS.

Bộ compiler của TS là `tsc`. Quá trình transpile sẽ thực hiện loại bỏ các **Type Annotations**, do đó mà sẽ không ảnh hưởng đến logic xử lý của chương trình.

`tsc` cũng hỗ trợ **Downleveling** trong quá trình transpile, với option `--target`, ví dụ `tsc --target es2015`. Mặc định `target = es5`.

## Cấu hình và CLI

### Cấu hình

Để cấu hình cho `tsc` khi biên dịch mã nguồn, ta định nghĩa tệp `tsconfig.json` ở thư mục gốc.

Có thể sử dụng `npx tsc --init` để sinh tự động `tsconfig.json`.

Một số cấu hình

| Trường                                                                                                                 | Ý nghĩa                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `extends`                                                                                                              | Cho phép kế thừa (và ghi đè cục bộ) các cấu hình từ một `tsconfig.json` khác. VD: Dưới đây là danh sách các configs phổ biến cho từng loại projects, có thể `extends` chúng: https://github.com/tsconfig/bases                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `files`, `include`, `exclude`                                                                                          | Giới hạn các files cần biên dịch. `include` và `exclude` cho phép sử dụng glob patterns. `files` chỉ phù hợp khi muốn chỉ định cụ thể một số files. *`exclude` chỉ đối với các files đã `include`*                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `.outDir`                                                                                                              | Thư mục chứa kết quả biên dịch                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `.rootDir`                                                                                                             | Thư mục tham chiếu làm thư mục gốc khi biên dịch. Dễ hiểu thì đây là phần đường dẫn bị cắt bỏ của các files sau khi biên dịch (chỉ định qua `files`, `include` và `exclude`). VD: file `src/core/lib/extra/add.ts`, khi `rootDir`: `src` thì vị trí của file này trong `outDir` là `core/lib/extra/add.js`, khi `rootDir`: `src/core/lib` thì vị trí sau biên dịch của file là `extra/add.js`                                                                                                                                                                                                                                                                                 |
| `.sourceMap`                                                                                                           | Sinh map files ánh xạ từ JS (kết quả biên dịch) vào TS (file mã nguồn), rất phù hợp khi debugging                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `.declaration`                                                                                                         | True nếu muốn biên dịch ra cả declaration file `.d.ts` (các khai báo typescript) cùng với `.js` (nội dung runtime code)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `.target`                                                                                                              | Phiên bản JS cần biên dịch về. VD: Khi xác định `target: ES5` thì các cú pháp Arrow functions `() => {}` sẽ được chuyển về Function expression `function () {}`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `.lib`                                                                                                                 | Bổ sung các built-in APIs (VD: `Math`, `document`, `Map`, `Promise`...) mà môi trường thực thi hỗ trợ nhưng phiên bản `target` chưa hỗ trợ. **Xác định `target` cũng đã ngầm định các `lib`**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `.module`                                                                                                              | Xác định module loader để tìm kiếm modules và phân tích cú pháp mã nguồn, nó cũng quyết định syntax của compiled JS (như là dùng `import` hay `require`), cần chọn dựa trên môi trường sẽ thực thi JS. - `es___`, `esnext` hướng theo chuẩn của ECMA (thường chỉ hỗ trợ ESM)<br>- `node__`, `nodenext` hướng theo triển khai của NodeJS (hỗ trợ cả ESM và CJS)                                                                                                                                                                                                                                                                                                                |
| `.moduleResolution`, [`resolvePackageJsonImports`](https://www.typescriptlang.org/tsconfig/#resolvePackageJsonImports) | Chiến lược để tìm kiếm module file dựa trên đường dẫn trong lệnh `import/require`. VD: `import './models/user'` sẽ tìm kiếm `./models/user.ts`, `./models/user/index.ts`, `node_modules/...`. Giá trị: `node`/`node10` (chiến lược truyền thống của Node - phù hợp với CommonJS), `node16`/`nodenext` (chiến lược phù hợp với ESM, cần cung cấp file extension khi import - spec chặt hơn), `bundler` (phù hợp khi sử dụng một bundler để phân giải đường dẫn, không hỗ trợ một số tính năng phân giải của Node như `index`)                                                                                                                                                  |
| `.rootDirs`                                                                                                            | Danh sách thư mục tham chiếu làm thư mục gốc khi phân giải đường dẫn. VD: Nếu `rootDirs: ["./cores", "./helpers/utils"]` thì bên trong `cores/index.ts` có thể dùng `import './math'` (helpers/utils/math.ts) và `import './server.ts'` (cores/server.ts). Cấu hình này không có ý nghĩa sắp xếp kết quả biên dịch trong `outDir`                                                                                                                                                                                                                                                                                                                                             |
| [`.paths`](https://www.typescriptlang.org/docs/handbook/modules/reference.html#paths)                                  | Danh sách các ánh xạ để tạo relative import alias (giúp rút ngắn đường dẫn khi import). VD: `import "@vendor/math.js"` thay vì `import "./vendor/crypto/dist/math.js"`.<br/>**Chú ý: tsc sẽ giữ nguyên đường dẫn của lệnh import khi biên dịch, và sẽ cần một bundler (esbuild, vite, [webpack](https://webpack.js.org/configuration/resolve/)) hay resolver ([tsc-alias](https://www.npmjs.com/package/tsc-alias), [tsconfig-paths](https://www.npmjs.com/package/tsconfig-paths), [node v20 `imports` == `imports map` trên Browser](https://nodejs.org/api/packages.html#imports)) để phân giải thì mới chạy (runtime) được các file js trên môi trường Node hay Browser** |


```json
// tsconfig.json
{
  "compilerOptions": {
    "module": "ESNext", // project's module loader [ESM]
    "target": "ES2020", // target JS version [modern ECMAScripts]
    "moduleResolution": "nodenext", // module resolution strategy [NodeJS v12]
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true, // allow `import x from y` (instead of `import * as x from y`) even if the module `y` not use export default
    "skipLibCheck": true, // skip type checking the `.d.ts` files (if those files are trusted - not hand-written)
    "resolveJsonModule": true, // allow to import `.json`
    "forceConsistentCasingInFileNames": true, // strictly on casing
    "noEmit": false,
    "isolatedModules": true, // not consider import on compiling each file (assume that each isolated file is typed-defined well)
    "paths": {
      "app/*": ["./src/app/*"],
      "config/*": ["./src/app/_config/*"],
      "environment/*": ["./src/environments/*"],
      "shared/*": ["./src/app/_shared/*"],
    }, // allow import relatively same as absolute import
  }
}
```

Dưới đây là một số Lint rules:

```json
{
  "compilerOptions": {
    /** report errors on unused local variables. */
    "noUnusedLocals": true,
    /** report errors on unused parameters in functions. */
    "noUnusedParameters": true,
    /** non-empty switch cases must end with `break`, `return` or `throw`. */
    "noFallthroughCasesInSwitch": true,
    /** methods that override superclass methods must have the `override` modifier. */
    "noImplicitOverride": true,
    /** "implicit return" is only allowed if the return type is void. */
    "noImplicitReturns": true,
    /** add `undefined` to any un-declared fields in the type (which support indexing like object) */
    "noUncheckedIndexedAccess": true,
    /** consider absence differs from setting values to `undefined` on optional properties `?` */
    "exactOptionalPropertyTypes": true,
    /** only allow using `.` accessing for defined property (e.g. not defined as indexing)  */
    "noPropertyAccessFromIndexSignature": true,
    /** require `type` on import/export types TS (differentiate with run time import, e.g a Class can be a type and a runtime object) */
    "verbatimModuleSyntax": true
  }
}

```

Xem thêm:
- [Ý nghĩa của các cấu hình](https://2ality.com/2025/01/tsconfig-json.html)
- [Hướng dẫn lựa chọn cấu hình](https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html)
  - Khi sử dụng bundler (FE), nên chọn: `module="esnext", moduleResolution="bundler", noEmit=true, esModuleInterop=true`
  - Với ứng dụng NodeJS (BE), nên chọn `module="nodenext" (moduleResolution="nodenext", esModuleInterop=true)`
- [Chi tiết về module và module resolution](https://www.typescriptlang.org/docs/handbook/modules/reference.html)

### CLI

Về `tsc` command, ta có một số options sau:

```bash
# Run a compile based on a backwards look through the fs for a tsconfig.json
tsc
# Emit JS for any .ts files in the folder src, with the default settings
tsc src/*.ts
# Set used replaced path to `tsconfig.json`
tsc --project tsconfig.production.json
# Emit d.ts files for a js file with showing compiler options which are booleans
tsc index.js --declaration --emitDeclarationOnly
# Emit a single .js file from two files via compiler options which take string arguments
tsc app.ts util.ts --target esnext --outfile index.js


# Check the files which should be processed
tsc --listFilesOnly

# Delete outputs
tsc --clean
```

### Strict Modes

Mã nguồn ứng dụng có thể chỉ thực hiện type annotating cho một số phần và để những phần khác sử dụng loose typed của JS. `tsc` vẫn có thể dịch nó về JS. Ta có thể thực hiện cấu hình **strictness** để chỉ thị `tsc` mức độ nghiêm ngặt của việc áp dụng type annotating trong mã nguồn. `strict` flag chứa một họ các strict mode, tương ứng với sự kiểm tra nghiêm ngặt trong một số trường hợp cụ thể:
- `noImplicitAny: true`: Báo lỗi cho các biến được suy đoán ngầm định (implicitly infer) hoặc được gán là kiểu `any` (kiểu mặc định nếu `tsc` không thể suy đoán một kiểu cụ thể)
- `strictNullChecks: true`: Báo lỗi nếu một biến có thể nhận giá trị `null` hoặc `undefined` trong khi thực hiện một hành động mà cần một giá trị cụ thể (ví dụ truy cập thuộc tính trên biến đó)
- Danh sách các **[strict modes](https://www.typescriptlang.org/tsconfig/#strictNullChecks)**
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
- Sử dụng **[type predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)** - định nghĩa một hàm predicate (boolean) trả về dạng `parameterName is Type`. Khi đó mỗi khi gọi hàm đó để kiểm tra một biến thì `tsc` biết khi nào biến đó sẽ có kiểu dữ liệu nào.

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

Ta có thể định nghĩa **hàm dưới dạng tổng quát** (**generic functions**), ở đó chữ ký của hàm không phụ thuộc vào các kiểu cụ thể (thay vào đó hàm được định nghĩa để tương thích với nhiều kiểu khác nhau).
- Lúc này, `tsc` xác định chữ ký của hàm tùy thuộc vào lời gọi (khi truyền đối số cụ thể). Các kiểu dữ liệu mà hàm phụ thuộc sẽ được khai báo ngay sau tên hàm.
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

## Declaration

**Declaration files** (`*.d.ts`) được sử dụng với mục đích khai báo các types cho ứng dụng, chúng mặc định sẽ không được `tsc` biên dịch (trừ khi option `skipLibCheck: false`). Với các packages được viết trên JS thuần thì declaration file giúp bổ sung phần Type cho chúng.

Hầu hết các thư viện phổ biến đều có phiên bản TS tương ứng, nổi tiếng nhất là sản phẩm từ cộng đồng **[DefinitelyTyped](https://github.com/DefinitelyTyped)**

Ví dụ với Express được viết trên JS thuần, có thể tải package

```bash
npm install --save-dev @types/express
```

để bổ sung phần Typescript.

Mặc định, `tsc` sẽ nạp các declaration files từ thư mục `node_modules/@types`, ta có thể bổ sung các thư mục tìm kiếm khác thông qua cấu hình `typeRoots`. Ví dụ

```json
{
  "compilerOptions": {
    "typeRoots": ["node_modules/@types", "./types"]
  }
}
```

*Chú ý, không cần thiết phải chỉ định `typeRoots` vì mặc định TSC tự động nạp toàn bộ các tệp `.ts` và `.d.ts` bên trong thư mục hiện tại*

### Declaration Merging

Declaration Merging đề cập đến việc `tsc` tự động gộp nhiều khai báo có cùng tên thành một declaration chứa những đặc trưng của tất cả các khai báo ban đầu.

Các khai báo trong TS có thể tạo ra ít nhất một trong ba đối tượng:
- **namespace**
- **type**
- **value**, các đối tượng có thể xuất hiện trong tệp output (JS)

Ví dụ, khi khai báo 2 interfaces cùng tên thì chúng được gộp thành một interface:
- Khi có thuộc tính cùng tên là hàm thì sẽ được coi là overloading
- Khi có thuộc tính cùng tên không phải là hàm và khác kiểu thì sẽ báo lỗi

```ts
interface Box {
  height: number;
  width: number;
  clone(inside: Animal): Animal;
}
interface Box {
  scale: number;
  clone(inside: Item): Item;
}

// Should be merged as
interface Box {
  scale: number;
  height: number;
  width: number;

  clone(inside: Item): Item;
  clone(inside: Animal): Animal;
}
```

Về namespace, chúng giúp bao đóng các khai báo và tránh xung đột tên với các khai báo bên ngoài.
- Namespace cho phép truy cập tới các thành viên bên trong (interface, type, class, function, variables,...) nếu như chúng được `export`.
- Có thể truy cập tới thành viên bên trong qua tên namespace.

Khi gộp các namespaces, các thành viên bên trong cũng được gộp với nhau (nếu cùng tên), tuy nhiên, các thành viên nội bộ (không được `export`) cũng không được gộp.

```ts
namespace Animals {
  export class Zebra {}
  export interface Legged {
    numberOfHooves: number;
  }
}
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Dog {}
}

// Should be merged as
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
    numberOfHooves: number;
  }
  export class Dog {}
  export class Zebra {}
}
```

Đặc biệt, namespace có thể gộp với class, function, enum

```ts
function buildLabel(name: string): string {
  return buildLabel.prefix + name + buildLabel.suffix;
}
namespace buildLabel {
  export let suffix = "";
  export let prefix = "Hello, ";
}

console.log(buildLabel("Sam Smith")); // Hello Sam Smith
```

### Ví dụ thực tế

> Thư viện `@types/express` định nghĩa kiểu dữ liệu cho `Request`, tuy nhiên, nếu muốn thêm dữ liệu vào `Request` (ví dụ như `cookie-parser` thêm `cookies` vào `Request`) thì cần làm như thế nào để báo cho `tsc` biết kiểu dữ liệu của đối tượng đó?

Câu trả lời là sử dụng Declaration Merging.

Thực tế, `Request` trong `@types/express` cũng được xây dựng trên `Request` trong `@types/express-serve-static-core`.

Các thư viện middlewares khác cũng thực hiện mở rộng `Request` type:
- `cookie-parser` thêm vào `cookies`
- `urlencoded` và `json` thêm vào `body`
- `session` thêm vào `session`
- `passport` thêm vào `user`

Để mở rộng `Request`, ta tạo **declaration file**

```ts
declare global {
  namespace Express {
    interface Request {
      locale: 'en' | 'vi'
    }
  }
}
```

# References

- [Documentation](https://www.typescriptlang.org/)
