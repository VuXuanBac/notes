# Vitest

[Vitest](https://vitest.dev/) là Testing Framework phát triển từ [Jest](https://jestjs.io/) để tận dụng các điểm mạnh có sẵn trong Vite. Jest và Vite vốn dĩ đã có nhiều hỗ trợ giống nhau, việc kết hợp sử dụng là một sự lặp lại không cần thiết, nên Vitest ra đời, kế thừa đặc trưng của Jest và tận dụng sự hỗ trợ từ Vite.

```bash
npm install -D vitest

npm install -D @vitest/ui # for view the coverage report at http://localhost:51204/__vitest__/

# run the test (with watch mode except in non-interactive environment - like CI)
vitest

# run specific test case
vitest path/to/test/file:10

# list all test cases in matched test file
vitest list --json=./test-cases.json --filesOnly

# run with generate coverage report
vitest run --coverage
```

Mặc định, Vitest sẽ tìm kiếm các test files dựa trên tên file: chứa `.test.` hoặc `.spec.`

Một số tính năng:
- Có thể chạy tests đồng thời với `.concurrent`
- **Snapshot**: Lưu lại bản sao giá trị để so sánh sự thay đổi, tương tự Jest
- **Matchers**: Kế thừa `expect`, `should`, `assert` từ [Chai](https://www.chaijs.com/) để tương thích với Jest
- **[Mocking](#mocking)**: Xây dựng [tinyspy](https://github.com/tinylibs/tinyspy) để tương thích với Jest
- **[In-source testing](https://vitest.dev/guide/in-source.html)**: Test trực tiếp vào source code thông qua scope `import.meta.vitest`, từ đó có thể test những tính năng không được export.
- **[Coverage](https://vitest.dev/guide/coverage.html)**: Hỗ trợ đo coverage dựa trên [v8](https://v8.dev/blog/javascript-code-coverage) hoặc [istanbul](https://istanbul.js.org/)

## Cấu hình

Tham khảo: https://vitest.dev/config/

Vitest có thể dùng chung cấu hình với Vite `vite.config.ts`, hoặc một tệp cấu hình riêng `vitest.config.ts` (nếu ứng dụng không sử dụng Vite như một build tool).

Cấu hình cho Vitest được để trong option: `test`

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // match the test files
    include: ['**/*.{test,spec}.?(c|m)[jt]s?(x)'],
    // use NodeJS as running environment (other options: `jsdom`, `happy-dom`,... - for Browser environment)
    environment: 'node',
    // directory to lookup test files
    dir: 'tests',
    // file to execute before runing each test file
    setupFiles: ['tests/config/setup.server.ts'],
    // coverage config
    coverage: {
      provider: 'v8', // or 'istanbul'
      // how to render coverage result
      reporter: ['text', 'json', 'html'], // or custom with Reporter instance
      reportsDirectory: 'tests/unit/coverage'
    },
    // config the running order
    sequence: {
      // run setup files in parallel
      setupFiles: 'parallel', // 'list'
      // first defined `after` hooks will run last
      hooks: 'stack', // 'list', 'parallel',
      // test can be run parallel
      concurrent: true
    }
  },
})
```

> Dù thế nào thì Vitest cũng sử dụng bộ công cụ từ Vite, nên trong tệp cấu hình vẫn nên có các cấu hình cho Vite

## API

- Sử dụng `describe(name, suiteFn)` để mô tả một suite (bao đóng các tests và benchmarks có liên quan)
- Sử dụng `test(name, testFn)` hoặc `it(name, testFn)` để tạo một test

Có thể gọi `describe.skip` hoặc `test.skip` để không thực hiện các tests trong đó

Khi muốn các tests chạy song song, có thể gọi `describe.concurrent` hoặc `test.concurrent`
- Tương tự có các option: `sequential`, `shuffle`

### TestContext

TestContext là tham số tự động được truyền vào `testFn`.
- `task`: metadata về test hiện tại
- `expect`: tương tự global expect nhưng mang đặc điểm và dữ liệu (như Snapshot) của test hiện tại

Có thể mở rộng TestContext để tạo các fixtures: Sử dụng `test.extend` để tạo một `test` API mới, sau đó định nghĩa thêm TestContext ở đây

```ts
interface MyFixtures {
  todos: number[]
  archive: number[]
}

// provide custom context and create new `test` API
const test = baseTest.extend<MyFixtures>({
  todos: [],
  archive: []
})

// use extended `test` API, so can access the custom context
test('types are defined correctly', ({ todos, archive }) => {
  expectTypeOf(todos).toEqualTypeOf<number[]>()
  expectTypeOf(archive).toEqualTypeOf<number[]>()
})
```

Ngoài ra, Vitest cho phép cung cấp giá trị đầu vào cho TestContext thông qua hai hàm `each` (hoặc `for`) trên suite hoặc test. *Bằng việc cung cấp giá trị đầu vào khác nhau cho TestContext, ta có thể thực hiện chạy test với nhiều trường hợp khác nhau*.

```ts
test.each([
  { a: 1, b: 1, expected: 2 },
  { a: 1, b: 2, expected: 3 },
  { a: 2, b: 1, expected: 3 },
])('add($a, $b) -> $expected', ({ a, b, expected }) => {
  expect(a + b).toBe(expected)
})

// this will return
// ✓ add(1, 1) -> 2
// ✓ add(1, 2) -> 3
// ✓ add(2, 1) -> 3

test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('add($0, $1) -> $2', (a, b, expected) => {
  expect(a + b).toBe(expected)
})

// this will return
// ✓ add(1, 1) -> 2
// ✓ add(1, 2) -> 3
// ✓ add(2, 1) -> 3
```

### Matchers

Có thể sử dụng theo
- Jest `expect`: https://jestjs.io/docs/expect
- Chai `expect`, `should`: https://www.chaijs.com/guide/styles/

Có thể thêm các hàm đánh giá tùy chỉnh với `expect.extend`

```ts
expect.extend({
  toBeFoo(received, expected): ExpectationResult {
    const { isNot } = this
    return {
      // do not alter your "pass" based on isNot. Vitest does it for you
      pass: received === 'foo',
      message: () => `${received} is${isNot ? ' not' : ''} foo`
    }
  }
})

// ---------- config extend type for typescript ----------
// vitest.d.ts
import 'vitest'

interface CustomMatchers<R = unknown> {
  toBeFoo: () => R
}

declare module 'vitest' {
  interface Assertion<T = any> extends CustomMatchers<T> {}
  interface AsymmetricMatchersContaining extends CustomMatchers {}
}
```

## Hook

Hook cho phép thực thi các đoạn mã callback trước và sau khi thực hiện test hoặc suite:
- `beforeEach`: thực thi callback trước khi thực thi mỗi test
- `afterEach`: thực thi callback sau khi thực thi mỗi test
- `beforeAll`: thực thi callback trước khi thực thi mọi test
- `afterAll`: thực thi callback sau khi thực thi mọi test

**Trước khi chạy mỗi test file, Vitest sẽ thực thi các `setupFiles` được chỉ định trong cấu hình.** Đây là vị trí phù hợp để đặt các khởi tạo và Hooks cần chạy ở mọi test files.

Ngoài ra, trong `testFn` ta có thể đặt các Hooks, khi đó, callbacks của các Hooks này có thể truy cập context thực thi test
- `onTestFinished`: gọi khi test hiện tại kết thúc, và gọi sau `afterEach`
- `onTestFailed`: gọi khi test hiện tại kết thúc nhưng failed, và gọi sau `afterEach`

Tất cả các hooks callback trên đều có thể nhận vào TestContext.

## Mocking

Tham khảo: https://vitest.dev/guide/mocking.html

Vitest hỗ trợ Mocking qua đối tượng `vi`

### Time

Vitest sử dụng package @sinonjs/fake-timers

```ts
const date = new Date(1998, 11, 19)

// affect all calls to timers (`setTimeout`, `setInterval`,..., `Date`)
vi.useFakeTimers()

vi.setSystemTime(date)

expect(Date.now()).toBe(date.valueOf())

vi.useRealTimers()
```

### Function

**Khi chỉ muốn đánh giá một hàm được gọi, với đối số xác định**, có thể sử dụng `spyOn`, nó sẽ tạo phiên bản **Mock cho một hàm gọi trên một object**, từ đó có thể mock giá trị trả về, mock implementation,... cho hàm đó.

```ts
// predefined
const cart = {
  getApples: () => 42,
}

let apples = 0
// fake that every time, a call `cart.getApples` will returns current value of `apples` variable
const spyGetApples = vi.spyOn(cart, 'getApples').mockImplementation(() => apples)
apples = 1

expect(cart.getApples()).toBe(1)

expect(spyGetApples).toHaveBeenCalled()
expect(spyGetApples).toHaveReturnedWith(1)
```

Phiên bản tổng quát hơn của `spyOn` là `fn`, nó cho phép Mock một hàm bất kỳ

```ts
function getApples(count: number){}
const mockFn = vi.fn().mockImplementation(getApples)
getApples(10)

// assert a call with arguments
mockFn.mock.calls[0] == 10
```

## Vitest + Prisma

Tham khảo: [Series: Testing with Prisma](https://www.prisma.io/blog/series/ultimate-guide-to-testing-eTzz0U4wwV)

```bash
npx try-prisma --template "orm/script" --name "vitest-prisma" --install npm
cd vitest-prisma

npx prisma migrate dev
npm i -D vitest

mkdir test
mkdir libs
touch libs/prisma.ts

npm i -D vitest-mock-extended
# update the `test/sample.test.ts` with `vi.mock('../libs/prisma')`

# create mocks for the whole module
mkdir libs/__mocks__
touch libs/__mocks__/prisma.ts
```
