# Monorepo

Tham khảo: https://monorepo.tools/

Monorepo đề cập đến việc tổ chức mã nguồn ở đó ứng dụng chia thành nhiều projects độc lập (nhưng chúng có mối liên hệ với nhau) và chứa chung trong một repository.
- **Khác với monolithic**: Ứng dụng chứa trong một repository nhưng các logic không có sự tách biệt, ngoại trừ việc tổ chức thành các layers và libraries
  - Thay đổi một layer yêu cầu phải built lại toàn bộ ứng dụng và test degrade
- **Khác với polyrepo**: Một hướng cải tiến là chia ứng dụng thành các microservices, chúng giao tiếp với nhau thông qua gRPC hoặc HTTP (thay vì đơn thuần là gọi hàm). Mỗi microservices (projects) được chứa trong repository riêng.
  - Khi các projects có chung một số logic, hoặc sẽ phải duplicates ở các repo, hoặc sẽ phải tạo repo mới cho logic chung đó (nhưng như thế cũng sẽ luôn phải cập nhật version của repo chung ở các repos dùng nó và theo dõi sự tích hợp này có ảnh hưởng hay không).
  - Khác biệt về phiên bản các dependencies giữa các projects
  - Luôn phải đồng bộ code ở nhiều repositories khi có break changes hoặc bug.

Lợi ích của monorepo
- Triển khai một tính năng hay fix bug có thể thực hiện trong một PR. Từ đó có thể chạy CI/Integration Test trực tiếp

## Monorepo với Workspace

Các phiên bản package managers mới nhất của NPM, PNPM, Yarn, Bun,... đều triển khai tính năng **workspaces** cho phép quản lý tập trung nhiều projects, rất phù hợp với đặc điểm của monorepo.

Quản lý tập trung được thể hiện qua các đặc điểm sau:
- Tải các dependencies ở tất cả các projects (workspaces) vào cùng một `node_modules` ở root
- Bản thân các projects cũng được **link** như một dependency ở root `node_modules`
  - **Do đó mà cũng không cần **path aliases** khi imports dependencies projects**
- Các scripts có thể chạy ở root và tham chiếu tới script ở từng workspace cụ thể, hoặc chạy đồng thời cho toàn bộ workspaces.

### Dependencies

Dưới đây mô tả một số commands thường dùng khi quản lý dependencies ở monorepo

```bash
# add `express` as a dependency of just `apps/api` workspace
npm install -w apps/api express

# add `typescript` as a dependency of all single workspaces
npm install -ws typescript

# add `prettier` as a dependency of root package
npm install -D prettier

# add `logger` internal package as a dependency of `apps/api` and `apps/web` workspaces
npm install @sample/logger@* -w apps/api -w apps/web

# add `eslint` as a dependency of `apps/api` workspace and root package as well
npm install eslint -w apps/api --include-workspace-root

#############################################

# install all dependencies from all workspaces and root package
npm install

# install all dependencies from just `apps/api` workspace and root package
npm install -w apps/api --include-workspace-root
```

### Scripts

Dưới đây mô tả một số commands thường dùng khi chạy scripts

```bash
# run dev scripts in `apps/api` and `apps/web` workspaces
npm run dev -w apps/api -w apps/web

# run dev scripts in all `apps/*` workspaces
npm run dev -w apps

# run dev scripts in all workspaces, if they defines the `dev` scripts
npm run dev -ws --if-present
```

## Monorepo với Typescript

Tham khảo:
- https://nx.dev/blog/managing-ts-packages-in-monorepos
- https://moonrepo.dev/docs/guides/javascript/typescript-project-refs

Khi logic của ứng dụng được tách nhỏ thành nhiều projects, điều tất yếu cần làm là liên kết chúng lại, cụ thể chính là cách thức import.

### Relative imports

Cách đơn giản nhất là sử dụng **relative imports**:
- Cách này đơn giản, không phải cấu hình
- Nhưng sẽ trở nên khó kiểm soát khi ứng dụng lớn dần, và các đường dẫn cũng dần phức tạp theo
- Nó cũng đánh mất tính module hóa (của việc tách nhỏ ứng dụng)
- Dưới góc nhìn của TS thì repo cũng chỉ là một project. Điều này dẫn đến vấn đề hiệu năng của quá trình type checking và building

### Path aliases

Có thể cải thiện bằng **path aliases**:
- Đường dẫn import lúc này ngắn hơn và có tính module hơn. Khi có sự thay đổi đường dẫn, chỉ cần cập nhật lại tệp cấu hình.
- Tuy nhiên, toàn bộ ứng dụng vẫn chỉ là một project đối với TS, và vẫn đề hiệu năng cũng không khác biệt với việc sử dụng **relative import**

### References và Composite

Vấn đề trên được giải quyết như sau:
- Do vấn đề là TS đang coi toàn bộ ứng dụng như một project, nên trước hết, cần setup project để chúng như một đơn vị độc lập, có biên giới rõ ràng.
- Tuy nhiên, như vậy việc biên dịch mỗi project lại độc lập với nhau, ý nghĩa chia sẻ mã nguồn khó triển khai.
- Do đó, TS giới thiệu `references` để chỉ định project A phụ thuộc vào project B (hoặc nhiều projects khác) và
  - Khi biên dịch, đảm bảo project B được biên dịch trước khi biên dịch project A.
  - Khi typechecking, TS chỉ cần sử dụng declaration file (`.d.ts`) của project B để kiểm tra.
- Như vậy, ở project B cần
  - Thiết lập [`declaration: true`](https://www.typescriptlang.org/tsconfig/#declaration), để sinh `.d.ts` cho nó.
  - Cần kiểm tra xem kết quả biên dịch có up-to-date với mã nguồn hay không. Đây là chức năng của [`incremental: true`](https://www.typescriptlang.org/tsconfig/#incremental), cho phép lưu project graph mô tả kết quả biên dịch ra file dạng `.tsbuildinfo` (setting đường dẫn lưu các tệp này qua [`tsBuildInfoFile`](https://www.typescriptlang.org/tsconfig/#tsBuildInfoFile))
  - Và để đơn giản, ta có thể [`composite: true`](https://www.typescriptlang.org/tsconfig/#composite), với mục đích để xác định xem kết quả biên dịch có up-to-date với mã nguồn hay không.
- Cùng với đó, TS giới thiệu Build Tool (`tsc --build`) với nhiều tính năng thông minh bên cạnh tính năng biên dịch:
  - Xác định các referenced projects có up-to-date hay không
  - Build chúng theo thứ tự dependenencies

## Ví dụ

Cấu trúc repo như sau

```txt
example-monorepo
   ├─ apps
   │  └─ myapp
   │     ├─ src
   │     │  └─ index.ts
   │     └─ tsconfig.json
   ├─ packages
   │  └─ lib-a
   │     ├─ src
   │     │  └─ index.ts
   │     └─ tsconfig.json
   ├─ tsconfig.base.json
   └─ tsconfig.json
```

Trong đó, `apps/myapp` gọi đến một hàm trong `packages/lib-a`, dưới đây là nội dung các tệp `tsconfig`:

- Về config ở mức ứng dụng, ta tách làm hai tệp, một để liệt kê các projects (đường dẫn tới tsconfig file) và một để chứa các configs chung cho toàn bộ ứng dụng.

- `tsconfig.json` ở root xác định các projects của ứng dụng, từ đó, ở root folder, có thể chạy `tsc --build` để build tất cả chúng

```json
// tsconfig.json
{
  "extends": "./tsconfig.base.json",
  "files": [], // compile no files
  "references": [{ "path": "./packages/lib-a" }, { "path": "./apps/myapp" }]
}
```

- `tsconfig.base.json` chứa các common configs cho tất cả các config ở các projects (có thể ghi đè sau)
  - `"composite": true`: Áp dụng chung cho config ở các projects
  - `"paths": {}`: Các projects có thể dùng chung path aliases ở đây
    - Nếu kết hợp sử dụng tính năng **workspaces** của các package managers thì có thể không cần path aliases cho chúng nữa
  - [`"declarationMap": true`](https://www.typescriptlang.org/tsconfig/#declarationMap): Hỗ trợ IDE trong tính năng `Go to Definition`: Sinh map file để IDE có thể dẫn tới file `.ts` thay vì file `.d.ts`.

```json
// tsconfig.base.json
{
  "compilerOptions": {
    // ===== recommend for ECMAScript =====
    // "target": "ESNext",
    // "module": "ESNext",
    // "moduleResolution": "bundler",
    // "strict": true,
    // "allowSyntheticDefaultImports": true,
    // "esModuleInterop": true,
    // "isolatedModules": true,
    // ===== recommend for Monoscript =====
    "composite": true,
    // "declaration": true,
    // "incremental": true,
    "declarationMap": true,
    "sourceMap": true,
    "noEmitOnError": true,
    "skipLibCheck": true,
    "paths": {
      "@example-monorepo/lib-a": ["packages/lib-a/src/index.ts"]
    }
  }
}
```

- `apps/myapp/tsconfig.json` tham chiếu tới project `lib-a` nên sẽ cần khai báo `references` ở đây để thể hiện sự phụ thuộc
  - `rootDir` khai báo ở đây để `myapp` được coi như một project độc lập
  - `tsBuildInfoFile` chỉ định đường dẫn sinh `.tsbuildinfo` (chỉ là một TS checking helper), tránh ảnh hưởng đến các files mã nguồn
  - Do dùng `composite: true` từ `tsconfig.base.json` nên phải khai báo `include`

```json
// apps/myapp/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "../../dist/apps/myapp",
    "rootDir": "src",
    "tsBuildInfoFile": "../../dist/apps/myapp/tsconfig.tsbuildinfo"
  },
  "references": [{ "path": "../../packages/lib-a" }],
  "include": ["src/**/*"]
}
```

- `packages/lib-a/tsconfig.json` tương tự `myapp`

```json
// packages/lib-a/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "../../dist/packages/lib-a",
    "rootDir": "src",
    "tsBuildInfoFile": "../../dist/packages/lib-a/tsconfig.tsbuildinfo"
  },
  "include": ["src/**/*"]
}
```
