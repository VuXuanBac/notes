# Prisma

Prisma là một thư viện ORM mã nguồn mở cho NodeJS và TypeScript, thiết kế theo hướng **schema-first** (giống như GraphQL), tức là chỉ cần định nghĩa ORM schema, thư viện có thể sinh cả database migrations, models và queries API.

Prisma schema sử dụng một ngôn ngữ hướng khai báo riêng, được đặt tên là *Prisma Schema Language* (**PSL**), cho phép khai báo:
- Cấu hình để kết nối Database
- Cấu hình để sinh **Prisma Client** hoặc các tài nguyên khác từ Schema
- Cấu trúc Models

Schema là trung tâm của Prisma, kết nối giữa ứng dụng (các truy vấn ở mức cao) và databases (SQL). Nhìn từ Schema, có hai hướng tiếp cận khi sử dụng Prisma:
- Khai báo cấu trúc Models trong schema và tạo database migration sử dụng **Primsa Migrate**
- Sử dụng [**Introspection**](https://www.prisma.io/docs/orm/prisma-schema/introspection) để tạo cấu trúc Models từ database.

Sau khi có Schema thì có thể tạo **Prisma Client** để tạo các truy vấn và thay đổi database với cú pháp của JS (hoặc TS).

## CLI

Tải Prisma với lệnh sau:

```bash
# Prisma CLI
npm install prisma --save-dev

# Prisma Client
npm install @prisma/client
```

|Commands|Mô tả|
|--|--|
|`prisma init`|Tạo thư mục `prisma` và tệp `schema.prisma`|
|`prisma format`|Định dạng lại các tệp Schema|
|`prisma generate`|Sinh các tài nguyên khai báo trong các khối `generator`|
|`prisma db push`|Cập nhật trực tiếp (không tạo migrations) trạng thái hiện tại của Schema vào database|
|`prisma db seed`|Chạy lệnh `seed` định nghĩa trong `package.json`: `"prisma": {"seed": "command_to_seed"}`|
|`prisma migrate dev --name <migration_name>`|Sử dụng [**Shadow database**](https://www.prisma.io/docs/orm/prisma-migrate/understanding-prisma-migrate/shadow-database) để phát hiện các thay đổi và tạo migrations cần thiết, sau đó migrate các migrations này vào development database, cập nhật bảng `_prisma_migrations`|
|`prisma migrate reset`|Drop/Reset database và schema => Tạo mới => Chạy migate => Chạy seed scripts|
|`prisma migrate deploy`|Chỉ thực hiện chạy migrations đã có vào database (và tạo database nếu chưa có)|

[**Xem thêm**](https://www.prisma.io/docs/orm/reference/prisma-cli-reference)

## So sánh với các thư viện khác

Prisma là thư viện có mức abstraction cao, của một ORM. Trong khi [**knex.js**](https://knexjs.org/) lại hướng đến là một Query Builder (tránh phải viết SQL nhưng vẫn cần tư duy theo SQL)

Các ORMs truyền thống (như [**Sequelize**](https://sequelize.org/)) thiết kế theo hướng khai báo Models qua các Classes và ánh xạ chúng tới Database Tables. Việc ánh xạ này có thể dẫn tới [**object-relational impedance mismatch**](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch), mà một ví dụ của nó là **N+1 Query**.

Prisma, một ORM, nhưng được trang bị (và hướng tới cải thiện) khả năng tối ưu câu truy vấn và bộ công cụ sinh tự động, cho phép đạt được năng suất cao hơn so với các ORMs truyền thống.

Tất nhiên, hạn chế của nó là ít khả năng tùy chỉnh hóa và không đảm bảo câu truy vấn là tối ưu với mọi trường hợp.

## Hiểu về Prisma Client

Prisma Client là một thư viện (tự động sinh) giúp ứng dụng có thể tương tác với database với cú pháp của JS/TS.

Prisma Client bao gồm 3 phần chính là JS client library + TS type definitions + **Query engine**.

**Query engine** là một chương trình được triển khai bằng **Rust**, và kết quả compile của nó thì phụ thuộc vào platform. 

> Đó là lý do khi cấu hình cho Prisma Client cũng cần cân nhắc platform cho query engine
> Có thể chỉ định danh sách qua [`binaryTargets`](https://www.prisma.io/docs/orm/reference/prisma-schema-reference#binarytargets-options)
> Khi cấu hình sử dụng một query engine cho Prisma Client, chương trình của nó sẽ được tải về vào thư mục `node_modules/@prisma/engines` và được sao chép vào thư mục kết quả của Prisma Client (nơi chứa JS/TS APIs)

Query engine là cầu nối (API) giữa Prisma Client và databases. Có hai cách thức mà Prisma Client kết nối Query engine (`engineType`):
- `library` nạp engine như một Node-API
- `binary` chạy engine như một process phụ và sử dụng tương tác giữa hai processes

![Query engine bridge Prisma Client and Database](img/node_express/prisma_query_engine.png)

> Query engine thực hiện khởi tạo và quản lý connection pool
> Mọi yêu cầu từ ứng dụng tới database đều đi qua query engine.

## Cấu hình

### Data Source

Giúp cấu hình cho kết nối tới một database cho ứng dụng, bao gồm driver (`provider`) và connection string (`url`).

```prisma
datasource db {
  provider = "postgresql"
  url      = env("APP_DATABASE_URL")
}
```

Prisma hỗ trợ kết nối tới cả relational databases như **PostgreSQL**, **SQLite**, **MySQL**, **MS SQL Server** và cả NoSQL databases như **MongoDB**.

Ngoài ra còn hỗ trợ kết nối tới các nền tảng serverless databases như **Neon**, **Supabase** (PostgreSQL), **PlanetScale** (MySQL), **Turso** (SQLite),...

### Generators

Giúp cấu hình cho các bộ sinh tự động **Prisma Client** hay các tài nguyên khác từ Schema.

Có hai generators dùng để sinh Prisma Client là:
- `prisma-client-js`, sinh Prisma Client và lưu vào `node_modules/.prisma/client`
  - Yêu cầu `@prisma/client` package
- `prisma-client` là generator mới hơn:
  - API sinh ra là TS hoàn toàn
  - Có thể tùy chỉnh vị trí lưu (`output`)
  - Có thể sử dụng cú pháp ES hoặc CJS (`moduleFormat`)

Ngoài ra, có một số package giúp sinh một số tài nguyên từ Prisma Schema:
- [`prisma-erd-generator`](https://github.com/keonik/prisma-erd-generator) giúp sinh ERD
- [`prisma-generator-fake-data`](https://github.com/luisrudge/prisma-generator-fake-data) giúp sinh seed data
- [`zod-prisma`](https://github.com/CarterGrimmeisen/zod-prisma) giúp sinh Zod schemas
- [`prisma-joi-generator`](https://github.com/omar-dulaimi/prisma-joi-generator) giúp sinh Joi schemas
- [`prisma-class-generator`](https://github.com/kimjbstar/prisma-class-generator) giúp sinh Model Class
- [Xem thêm](https://www.prisma.io/docs/orm/prisma-schema/overview/generators#community-generators)

```prisma
// for generating Prisma Client
generator client {
  provider          = "prisma-client"
  output            = "../src/generated/prisma"
  moduleFormat      = "esm"
}

// for generating Zod schemas
generator zod {
  provider          = "zod-prisma"
  output            = "./zod" // (default) the directory where generated zod schemas will be saved
  modelCase         = "PascalCase"
}

// for generating ERD
generator erd {
  provider          = "prisma-erd-generator"
  output            = "../ERD.svg"
  theme             = "forest"
}
```

### Vị trí của Schema

Mặc định, tệp Schema mà Prisma tìm kiếm là `./prisma/schema.prisma` hoặc `./schema.prisma`. 

Có thể cấu hình lại qua:
- `prisma generate --schema=....`
- `"prisma": {"schema": "..."}` trong `package.json`

Ta có thể chia nhỏ Schema thành nhiều tệp và chỉ định sử dụng `previewFeatures` cho Generator. Khi đó, generator sẽ tìm kiếm các phần schema ở trong thư mục `prisma/schema/`

```prisma
// schema.prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["prismaSchemaFolder"]
}
```

## Model

**Model** được định nghĩa qua `model` block, bên trong đó định nghĩa các **fields** và **relations**.

Tên của Model theo convention: **Singular** + **PascalCase**. Có thể dùng `@@map("...")` để ánh xạ tên khác cho database tables.

### Fields

Fields trong Model được khai báo với: **Name** + **Type** + **Type Modifiers** + **Attributes**

Field Type có thể là:
- *Scalar*
  - `String` (TS `string`)
  - `Boolean` (TS `boolean`)
  - `Int` (TS `number`)
  - `Float` (TS `number`)
  - `DateTime` (TS `Date`)
  - `Json` (TS `object`)
  - `Bytes` (TS `Uint8Array`)
  - `Decimal` (TS `Decimal`)
  - `BigInt` (TS `BigInt`)
  - **Enum**
- *Relation*: Model
- *`Unsupported("...")`*: Dùng database type mà Prisma chưa hỗ trợ

Enum được khai báo qua block `enum`.
- PostgreSQL và MySQL hỗ trợ sẵn
- Prisma tạo interface để triển khai trên SQLite và MongoDB

Có thể sử dụng một trong các modifiers ngay sau Field Type:
- `[]`: Tạo kiểu mảng (hiện tại chỉ có PostgreSQL, MongoDB và CockroachDB là hỗ trợ)
- `?`: Tạo nullable field.

Có thể mô tả đặc trưng của Field qua các attributes:
- `@id`
- `@default` xác định giá trị mặc định: giá trị tĩnh (scalar, array, json), `autoincrement()`, `uuid(4|7)`, `now()` (thời điểm record được tạo) hoặc `dbgenerated(...)` để sử dụng một hàm của database.
- `@unique`
- `@updatedAt` (Prisma sinh giá trị, khác với )
- `@relation` mô tả cho kết nối giữa các bảng như reference name, reference from (`fields`), reference to (`references`), `onUpdate`, `onDelete`
- `@map` chỉ định tên của cột/thuộc tính tương ứng trong database
- `@ignore` không sinh Prisma Client API cho field này
- `@db` (với `db` là tên của `datasource` block) để ánh xạ kiểu sang một kiểu khác trong databases

Có thể khai báo các đặc trưng ở mức model: `@@id`, `@@unique`, `@@index`, `@@map`, `@@ignore`, `@@schema` (khi muốn [nhóm các bảng thành nhiều databases](https://www.prisma.io/docs/orm/prisma-schema/data-model/multi-schema))

### Relations

Sử dụng attribute `@relation` chỉ định fields được dùng làm relation, chúng giúp có thể tương tác (đọc/ghi) với relation như một thuộc tính khi sử dụng Prisma Client, mà không ánh xạ vào database.

**Với many-to-many relations**, có thể sử dụng chế độ ngầm định đối với bảng trung gian, Prisma sẽ quản lý bảng này và nó sẽ không thể truy cập qua Prisma Client. Điều đó giúp query đơn giản hơn khi không cần qua một bảng trung gian. *Prisma khuyến khích sử dụng chế độ này nếu bảng trung gian không có dữ liệu riêng*
- Tuy nhiên, các bảng chính phải đảm bảo chỉ sử dụng một trường làm ID, không được gán `@unique` cho ID
- Có thể đổi tên cho bảng trung gian qua `@relation`, tuy nhiên không thể xác định các options khác

```prisma
model User {
  id      Int      @id @default(autoincrement())
  posts   Post[]   // 1-n relation at Prisma level
}

model Post {
  id         Int        @id @default(autoincrement())
  authorId   Int
  author     User       @relation(fields: [authorId], references: [id]) // n-1 relation at Prisma level
  categories Category[] // implicit m-n relation at Prisma level
}

model Category {
  id    Int    @id @default(autoincrement())
  posts Post[] // implicit m-n relation at Prisma level
}
```

**Khi hai models có nhiều quan hệ với nhau** thì cần chỉ định `name` cho từng quan hệ khi khai báo `@relation`.

```prisma
model User {
  id           Int     @id @default(autoincrement())
  name         String?
  writtenPosts Post[]  @relation("WrittenPosts")
  pinnedPost   Post?   @relation("PinnedPost")
}

model Post {
  id         Int     @id @default(autoincrement())
  title      String?
  author     User    @relation("WrittenPosts", fields: [authorId], references: [id]) // this field is for `WrittenPosts`, not `PinnedPost`
  authorId   Int
  pinnedBy   User?   @relation("PinnedPost", fields: [pinnedById], references: [id])
  pinnedById Int?    @unique
}
```

**Với self-relations**, ta cần sử dụng `@relation` cho hai đầu của quan hệ, tức là cần truyền `name` để nhóm chúng lại.

```prisma
// 1-1 self-relation
model User {
  id          Int     @id @default(autoincrement())
  name        String?
  successorId Int?    @unique
  successor   User?   @relation("BlogOwnerHistory", fields: [successorId], references: [id])
  predecessor User?   @relation("BlogOwnerHistory")
}

// 1-n self-relation
model User {
  id        Int     @id @default(autoincrement())
  name      String?
  teacherId Int?
  teacher   User?   @relation("TeacherStudents", fields: [teacherId], references: [id])
  students  User[]  @relation("TeacherStudents")
}

// implicit m-n self-relation
model User {
  id         Int     @id @default(autoincrement())
  name       String?
  followedBy User[]  @relation("UserFollows")
  following  User[]  @relation("UserFollows")
}
```

**Về referential actions (`onUpdate`, `onDelete`)**, chúng xác định các hành động thực hiện trên `fields` khi có thao tác update/delete trên `references`. Các giá trị có thể là:
- `Cascade` update/delete `fields` tương ứng
- `SetNull` cập nhật `fields` thành NULL
- `SetDefault` cập nhật `fields` thành giá trị xác định trong `@default`
- `Restrict` ngăn xóa update/delete trên `references` nếu `fields` đang có giá trị
- `NoAction` tương tự như `Restrict` trong một số databases (MySQL, MS SQL Server) hoặc sẽ không thực hiện gì trong SQLite, MongoDB.

### Ví dụ

```prisma
model User {
  id      Int      @id @default(autoincrement())
  email   String   @unique
  name    String?
  role    Role     @default(USER)
  posts   Post[]
  profile Profile?

  @@map("account")
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String @map("info")
  userId Int    @unique
  user   User   @relation(fields: [userId], references: [id])
}

model Post {
  id         Int        @id @default(autoincrement())
  createdAt  DateTime   @default(now())
  updatedAt  DateTime   @updatedAt
  title      String
  data       Json       @default("{ \"hello\": \"world\" }")
  published  Boolean    @default(false)
  authorId   Int
  categories Category[]
  author     User       @relation(fields: [authorId], references: [id])

  @@unique([authorId, title])
  @@index([title])
}

model Category {
  id    Int    @id @default(autoincrement())
  name  String
  posts Post[]
}

enum Role {
  USER
  ADMIN
}
```

### STI và MTI

Table inheritance là một design pattern giúp mô hình hóa các models theo cấu trúc kế thừa, khi mà các models khác nhau có chung một số thuộc tính.

Có hai hướng triển khai kế thừa này:
- **STI** (single-table inheritance) chỉ dùng một bảng để chứa các thuộc tính chung của tất cả các entities.
- **MTI** (multi-table inheritance) dùng mỗi bảng cho mỗi entities, định nghĩa các thuộc tính đặc trưng của chúng, đồng thời tạo một bảng cơ sở, nơi chứa các *thuộc tính chung*.

> Có thể thấy STI phù hợp với các entities có ít sự khác biệt về thuộc tính
> Khi số lượng khác biệt càng lớn thì việc lưu trữ mỗi record càng kém hiệu quả, khi những thuộc tính đặc trưng của các entities khác sẽ không có giá trị.

Ví dụ: `Review` (đánh giá của người mua) và `Comment` (phản hồi của người bán) thực tế giống nhau về các thuộc tính như tài khoản người dùng, nội dung, thời điểm,... Trong trường hợp đơn giản thì không có sự khác biệt về dữ liệu, tuy nhiên, cách ứng dụng xử lý mỗi loại dữ liệu có thể khác nhau (tức là cần coi chúng là các entities độc lập). Trong trường hợp này, ta có thể sử dụng STI

```prisma
enum UserMessageType {
  Review
  Comment
}

model User {}

model UserMessage {
  id          Int       @id
  body        String
  timestamp   DateTime
  type        UserMessageType

  ownerId     Int
  owner       User      @relation(fields: [ownerId], references: [id])
}
```

> *Prisma chưa hỗ trợ STI*, vì mỗi `model` đều ánh xạ tới ít nhất một bảng trong database.
> Nên để thực sự triển khai STI, ta cần khai báo Model Class và TS interface riêng.

Ví dụ: Ứng dụng hỗ trợ nhiều dạng `Post` như `Blog` (text-based), `Short` (video-based),... Rõ ràng các thuộc tính giữa các dạng `Post` này khác biệt rất nhiều, khi đó ta có thể sử dụng MTI

```prisma
enum PostType {
  Blog
  Short
}

model Post {
  id          Int       @id
  timestamp   DateTime
  type        PostType

  // no totally neccessary, because most of the time, we reference from the child model
  blog        Blog?
  short       Short?
}

model Blog {
  id          Int       @id
  content     String

  post        Post      @relation(fields: [id], references: [id]) // reuse ID from Post
}

model Short {
  id          Int       @id
  url         String
  resolution  String

  post        Post      @relation(fields: [id], references: [id]) // reuse ID from Post
}
```

### Polymorphic

Polymorphic chỉ một entity có thể có quan hệ với một hoặc nhiều dạng entities khác nhau. Điều này khiến Polymorphic giống với Table Inheritance ở việc nhiều entities có chung thuộc tính, đặc biệt giống với MTI khi tách biệt entity chứa thuộc tính chung.

||MTI|Polymorphic|
|--|--|--|
|Ví dụ|`Post` có thể là `Blog` hoặc `Short`|`Comment` có thể thuộc vào `Blog` hoặc `Short`|
|Bản chất|Kế thừa|Composition|
|Đặc điểm|`Post` là trừu tượng hóa của `Blog` và `Short`|`Comment` là một thành phần trong `Blog` và `Short`|
|Khi bỏ qua relation|`Post` chưa chứa đầy đủ dữ liệu|`Comment` có thể coi là có đủ dữ liệu|
|Tham chiếu|Thường đặt tham chiếu ở `Blog` hoặc `Short`|Thường đặt tham chiếu vào `Comment`|

Hiện tại, ***Prisma cũng không hỗ trợ triển khai Polymorphic đầy đủ***, ta vẫn cần xử lý thủ công, hoặc sử dụng [ZenStack](https://zenstack.dev/)

```prisma
enum PostType {
  Blog
  Short
}

model Comment {
  id          Int       @id

  postType    PostType
  postId      Int
}

model Blog {
  id          Int       @id
  // comments    Comment[]   // ❌ Prisma doesn't know how to resolve this. Can use Prisma Extension to implement
}

model Short {
  id          Int       @id
  // comments    Comment[]   // ❌ Prisma doesn't know how to resolve this. Can use Prisma Extension to implement
}
```

## Prisma Client

Sau khi có Schema, ta chạy lệnh

```bash
prisma generate
```

để tạo Prisma Client. Mỗi khi Schema có sự thay đổi, cần chạy lệnh này để cập nhật lại Prisma Client. Cấu hình để sinh Prisma Client được xác định trong block `generator` của Schema.

Sau đó, ta có thể tạo instance cho Prisma Client với:

```ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient();

export default prisma;
```

Mỗi Prisma Client quản lý một connection pool (số lượng khoảng `num_physical_cpus * 2 + 1` cho relational databases, và khoảng 100 đối với MongoDB), nên thông thường ứng dụng chỉ cần tạo một instance cho Prisma Client. Xem thêm [Quản lý databases connections](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/databases-connections)

Sau khi khởi tạo, chỉ đến khi thực thi query đầu tiên, Prisma tự động thực hiện kết nối tới database và khởi tạo connection pool. Và sẽ chỉ ngắt kết nối khi process kết thúc.

Trong một số trường hợp, như chạy background jobs, ta có thể khởi tạo thêm Prisma Client và chỉ định `$disconnect()` để chủ động ngắt kết nối tới databases khi kết thúc job.

### Query

Mỗi Model trong Schema sẽ tự động được ánh xạ thành một property trên Prisma Client instance với tên ở dạng snake_case. 

Trên Model object, ta có thể thực hiện những query methods sau:
- R: `findMany()`, `findFirst()`, `findFirstOrThrow()`, `findUnique()`, `findUniqueOrThrow()`
- C: `create()`, `createMany()`, `createManyAndReturn()`
- U: `update()`, `updateMany()`, `updateManyAndReturn()`, `upsert()`
- D: `delete()`, `deleteMany()`

Các methods này đều nhận vào một JS object chứa các options như: `where`, `data`, `select`, `include`,...

#### Select

Khi thực hiện lệnh **R**, Prisma chỉ trả về objects cho các Scalar Fields (không trả về relations), ta có thể xác định giá trị cho các options sau:
- `select`/`omit` lọc các scalar fields hoặc relation fields
- `include` nạp cả các relation fields
- Có thể nested các options này để áp dụng trên các relation fields. Tuy nhiên, `select` và `include` không thể ở cùng mức
- `relationLoadStragy` [`previewFeatures = ["relationJoins"]`]
  - `join` join (subquery) các bảng ở mức databases, chỉ dùng một query
  - `query` mỗi query cho một bảng và join ở mức ứng dụng

```ts
const user = await prisma.user.findFirst({
  select: {
    email: true,
    posts: { // relation
      select: {
        title: true,
        categories: true  // relation
      },
    },
  },
})
```

#### Lọc, Sắp xếp và Pagination

Để lọc matched records, sử dụng **`where`**, có thể sử dụng các operators sau:
- `equals` (có thể bỏ qua, dùng trực tiếp giá trị), `lte`,..., `in`, `not` (NULL không match), `notIn` (NULL không match), `contains` (string), `startsWith`, `endsWith`
- Kết hợp: `NOT`, `AND`, `OR`
- Relation: 
  - `some` true nếu có ít nhất một related records thỏa mãn điều kiện
  - `every`
  - `none`
  - `is`, `isNot` dùng đối với một related record
- Đối với Array Field (PostgreSQL, MongoDB)
  - `has`, `hasEvery`, `hasSome`, `isEmpty`, `isSet`, `equals`
- [Xem thêm](https://www.prisma.io/docs/orm/reference/prisma-client-reference#filter-conditions-and-operators)

Để sắp xếp các matched records, sử dụng **`orderBy`**. Ta có thể sử dụng option `nulls: first | last` để chỉ định NULL được trả về trước hay sau các giá trị khác

```ts
const users = await prisma.user.findMany({
  orderBy: {
    updatedAt: { sort: 'asc', nulls: 'last' },
  },
})
```

Sử dụng `skip` (OFFSET), `take` (LIMIT) để phân trang

### Create, Update và Delete

Đối với việc cập nhật trên một model, ta có thể truyền option `data` cho các method dạng `create__`, `update__`, cùng với các options để lọc và `include`.

```ts
const createMany = await prisma.user.createMany({
  data: [
    { name: 'Bob', email: 'bob@prisma.io' },
    { name: 'Bobo', email: 'bob@prisma.io' }, // Duplicate unique key!
    { name: 'Yewande', email: 'yewande@prisma.io' },
    { name: 'Angelique', email: 'angelique@prisma.io' },
  ],
  skipDuplicates: true,
})
```

Prisma cũng hỗ trợ **Nested Writes**, cho phép cập nhật cả related model đồng thời với model hiện tại

```ts
const result = await prisma.user.create({
  data: {
    email: 'elsa@prisma.io',
    name: 'Elsa Prisma',
    posts: {
      // nested write: create on the way
      create: [
        { title: 'How to make an omelette' },
        { title: 'How to eat an omelette' },
      ],
    },
  },
  include: {
    posts: true, // Include all posts in the returned object
  },
})
```

Bên cạnh `create`, ta có thể sử dụng `createMany`, `connect` (gán bằng một related record), `connectOrCreate`, `disconnect`, `deleteMany`, `updateMany`, `update`, `upsert`,...

Xem thêm: [Nested writes](https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries#nested-writes)

```ts
const u = await prisma.user.create({
  include: {
    posts: {
      include: {
        categories: true,
      },
    },
  },
  data: {
    email: 'emma@prisma.io',
    posts: {
      create: [
        {
          title: 'My first post',
          categories: {
            connectOrCreate: [
              {
                create: { name: 'Introductions' },
                where: {
                  name: 'Introductions',
                },
              },
              {
                create: { name: 'Social' },
                where: {
                  name: 'Social',
                },
              },
            ],
          },
        },
        {
          title: 'How to make cookies',
          categories: {
            connectOrCreate: [
              {
                create: { name: 'Social' },
                where: {
                  name: 'Social',
                },
              },
              {
                create: { name: 'Cooking' },
                where: {
                  name: 'Cooking',
                },
              },
            ],
          },
        },
      ],
    },
  },
})
```

### Aggregation và Group

Để thực hiện các Aggregation, ta gọi method `aggregate` trên model object, với các options sau: 
- Aggregation: `_avg`, `_sum`, `_min`, `_max`, `_count`
- `where`, `orderBy`, `skip`, `take`

Để thực hiện các Group, ta gọi method `groupBy` trên model object, với các options sau: 
- Aggregation: `_avg`, `_sum`, `_min`, `_max`, `_count`
- `where`, `orderBy`, `skip`, `take`, `by`, `having`

```ts
const groupUsers = await prisma.user.groupBy({
  by: ['country', 'city'],
  _count: {
    _all: true,
    city: true,
  },
  _sum: {
    profileViews: true,
  },
  orderBy: {
    country: 'desc',
  },
  having: {
    profileViews: {
      _avg: {
        gt: 200,
      },
    },
  },
});

// [
//   {
//     country: 'Denmark',
//     city: 'Copenhagen',
//     _sum: { profileViews: 490 },
//     _count: {
//       _all: 70,
//       city: 8,
//     },
//   },
//   {
//     country: 'Sweden',
//     city: 'Stockholm',
//     _sum: { profileViews: 500 },
//     _count: {
//       _all: 50,
//       city: 3,
//     },
//   },
// ];
```


### Raw SQL

Prisma hỗ trợ 4 methods sau để thực hiện Raw SQL:
- `$queryRaw` và `$queryRawUnsafe`: Trả về records (như trong `SELECT`)
- `$executeRaw` và `$executeRawUnsafe`: Trả về số lượng records tác động (như trong `CREATE`,`UPDATE`,`DELETE`)

Trong đó phiên bản "safe" nhận vào một tagged template (`\`...\``) cho phép truyền giá trị trực tiếp. Prisma sẽ tiền xử lý chuỗi đầu vào này để tránh SQL injection. Tuy nhiên, nếu không sử dụng đúng có thể vẫn có nguy cơ.

Xem thêm: [SQL injection prevention](https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/raw-queries#sql-injection-prevention)

#### TypedSQL

Prisma giới thiệu TypedSQL để có thể thực hiện Raw SQL nhưng vẫn có được những sự hỗ trợ của TS như sử dụng query APIs.

Với hướng tiếp cận này, ta khai báo các lệnh SQL vào một tệp riêng `.sql` bên trong thư mục `prisma/sql`. Lệnh SQL này sẽ được import vào chương trình như một hàm và được thực thi với `$queryRawTyped`.

Tuy nhiên, đây vẫn là tính năng thử nghiệm của Prisma và nó chỉ phù hợp với query cố định.

```sql
-- ------- prisma/sql/getUserByAges.sql -------

-- @param {Int} $1:minAge
-- @param {Int} $2:maxAge
SELECT id, name, age
FROM users
WHERE age > $1 AND age < $2
```

```ts
import { PrismaClient } from '@prisma/client'
import { getUserByAges } from '@prisma/client/sql'

const prisma = new PrismaClient()

const users = await prisma.$queryRawTyped(getUserByAges(18, 60))
console.log(users)
```

Xem thêm: [TypedSQL](https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/typedsql)

### Transaction

Prisma Client tự động tạo transactions cho các lệnh sau:
- *Nested Write*: Tạo/cập nhật/xóa qua relation
- *`$transaction([])*
- *Batching* (`createMany`, `updateMany`, `deleteMany`,...)
- *Idempotent*
- *Optimistic concurrency control*
- *Interactive transactions*

```ts
// Sequential operations
const [posts, totalPosts] = await prisma.$transaction([
  prisma.post.findMany({ where: { title: { contains: 'prisma' } } }),
  prisma.post.count(),
  prisma.$queryRawTyped(updateUserName(2)),
])

// Interactive operations
function transfer(from: string, to: string, amount: number) {
  try {
    return await prisma.$transaction(async (tx) => {
      // Code running in a transaction...
      // throw Error to rollback
    },
    {
      maxWait: 5000, // default: 2000 (2s)
      timeout: 10000, // rollback when timeout. default: 5000 (5s)
    })
  } catch (err) {
    // Handle the rollback...
  }
}
```

### Extensions

Prisma Client cho phép định nghĩa thêm tính năng cho nó thông qua các Extensions
- [`model`](#extended-model) thêm methods hoặc fields cho model object
- [`client`](#extended-client) thêm methods cho Prisma Client
- [`query`](#extended-query) thay đổi cách thức thực thi query cho Prisma Client
- [`result`](#extended-result) thêm fields vào kết quả thực hiện truy vấn

Sau khi thêm extensions cho Prisma Client, ta thu được một phiên bản mới bao đóng (mà không thay đổi trực tiếp) phiên bản gốc. *Tất cả các phiên bản của Prisma Client đều cùng tương tác trên một query engine*.

Để thêm Extension cho Prisma Client, ta sử dụng `$extends` trên client instance. Có thể truyền cho nó một JS object hoặc một biến tạo bởi `Prisma.defineExtension` với cú pháp tương tự.

Extension vẫn là một Preview Features: `previewFeature = ["clientExtensions"]`

Xem thêm: [Một số Extensions và ví dụ triển khai](https://www.prisma.io/docs/orm/prisma-client/client-extensions/extension-examples)

#### Extended Model

```ts
// extend `user` model with `signUp` method
const prisma = new PrismaClient().$extends({
  model: {
    user: {
      async signUp(email: string) {
        await prisma.user.create({ data: { email } })
      },
    },
  },
})

// call directly
const user = await prisma.user.signUp('john@prisma.io')
```

Có thể khai báo methods cho tất cả models với option `$allModels`
- Lúc này, để lấy model object đang thực thi method, có thể gọi `Prisma.getExtensionContext(this)`, với `this` là đối số đầu tiên của method

```ts
const prisma = new PrismaClient().$extends({
  model: {
    $allModels: {
      async exists<T>(
        this: T,
        where: Prisma.Args<T, 'findFirst'>['where']
      ): Promise<boolean> {
        // Get the current model at runtime
        const context = Prisma.getExtensionContext(this)

        const result = await (context as any).findFirst({ where })
        return result !== null
      },
    },
  },
})

// call
// `exists` method available on all models
await prisma.user.exists({ name: 'Alice' })
await prisma.post.exists({
  OR: [{ title: { contains: 'Prisma' } }, { content: { contains: 'Prisma' } }],
})
```

#### Extended Client

Thêm methods để có thể gọi trực tiếp trên Prisma Client instance

```ts
const prisma = new PrismaClient().$extends({
  client: {
    $log: (s: string) => console.log(s),
    async $totalQueries() {
      const index_prisma_client_queries_total = 0
      // Prisma.getExtensionContext(this) in the following block
      // returns the current client instance
      const metricsCounters = await (
        await Prisma.getExtensionContext(this).$metrics.json()
      ).counters

      return metricsCounters[index_prisma_client_queries_total].value
    },
  },
})

async function main() {
  prisma.$log('Hello world')
  const totalQueries = await prisma.$totalQueries()
  console.log(totalQueries)
}
```

#### Extended Query

Thay đổi cách thức thực thi của các query có sẵn (như `findMany`, `aggregation`, `$queryRaw`...) trên một hay toàn bộ model object

```ts
const prisma1 = new PrismaClient().$extends({
  query: {
    user: {
      async findMany({ model, operation, args, query }) {
        // take incoming `where` and set `age`
        args.where = { ...args.where, age: { gt: 18 } }

        return query(args)
      },
    },
  },
})

const prisma2 = new PrismaClient().$extends({
  query: {
    $allModels: {
      async findMany({ model, operation, args, query }) {
        // set `take` and fill with the rest of `args`
        args = { ...args, take: 100 }

        return query(args)
      },
    },
  },
})

const prisma3 = new PrismaClient().$extends({
  query: {
    user: {
      $allOperations({ model, operation, args, query }) {
        /* your custom logic here */
        return query(args)
      },
    },
  },
})

await prisma1.user.findMany() // returns users whose age is greater than 18
```

#### Extended Result

Thêm fields và methods vào kết quả của truy vấn. 

> Giá trị của các fields và methods này được xác định khi đọc (không phải khi thực hiện query)
> Hiện tại không hỗ trợ relation trong computed fields.

```ts
// add custom field to query result object
const prisma1 = new PrismaClient().$extends({
  result: {
    user: {
      fullName: {
        // the dependencies
        needs: { firstName: true, lastName: true },
        compute(user) {
          // the computation logic
          return `${user.firstName} ${user.lastName}`
        },
      },
    },
  },
})

// add custom method to query result object
const prisma2 = new PrismaClient().$extends({
  result: {
    user: {
      save: {
        needs: { id: true },
        compute(user) {
          return () =>
            prisma.user.update({ where: { id: user.id }, data: user })
        },
      },
    },
  },
})

console.log((await prisma1.user.findFirst()).fullName)

const user = await prisma2.user.findUniqueOrThrow({ where: { id: someId } })
user.email = 'mynewmail@mailservice.com'
await user.save()
```

### Type

Prisma Client hỗ trợ sẵn các kiểu dữ liệu cho các đối tượng truyền vào query:

```ts
// can pass into `select` option
const userEmail: Prisma.UserSelect = {email: true}

// can pass into `include` option
const userPosts: Prisma.UserInclude = {posts: true}
```

Ta có thể sử dụng các tiện ích kiểu dữ liệu sau khi muốn xác định một kiểu custom dựa trên các kiểu mà Prisma Client đã sinh.
- `Args<Type, Operation>['ArgName']` lấy Type của `ArgName` bên trong `Operation` thực hiện trên model `Type`
- `Result<Type, Arguments, Operation>` lấy Type của kết quả trả về khi thực hiện `Operation` trên model `Type` với đối số `Arguments`
- `Payload<Type, Operation>` lấy cấu trúc của kết quả trả về khi thực hiện `Operation` trên model `Type`
- `Exact<Input, Shape>` thu hẹp `Input` về dạng `Shape`

```ts
type ExtendedUser = Prisma.Result<typeof prisma.user, { select: { id: true } }, 'findFirstOrThrow'>

type FindFirstUserWhere = Prisma.Args<typeof prisma.user, 'findFirst'>['where']
```
