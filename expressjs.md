# ExpressJS

[ExpressJS](https://expressjs.com/) là một Web Framework nhỏ gọn dựa trên module `http` của NodeJS.

```bash
npm install express --no-save [-g]

# create application skeleton
npx express-generator
```

Với `http` module, việc xây dựng ứng dụng web đòi hỏi phải thực hiện các thao tác xử lý thô: parse request, routing, session management, response....

```js
import http from 'http';

// Create the HTTP server
const server = http.createServer((req, res) => {
  // Set the response HTTP header
  res.writeHead(200, { 'Content-Type': 'text/html' });

  // Handle routing logic
  if (req.url === '/') {
    res.end('<h1>Welcome to the Home Page</h1>');
  } else if (req.url === '/about') {
    res.end('<h1>About Page</h1><p>This is a simple Node.js app.</p>');
  } else {
    res.writeHead(404, { 'Content-Type': 'text/html' });
    res.end('<h1>404 - Page Not Found</h1>');
  }
});

// Start the server
server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

ExpressJS giúp đơn giản hóa quá trình này:

```js
import express from 'express';
const app = express();

// Define routes
app.get('/', (req, res) => {
  res.send('<h1>Welcome to the Home Page</h1>');
});

app.get('/about', (req, res) => {
  res.send('<h1>About Page</h1><p>This is a simple Express app.</p>');
});

// 404 handler
app.use((req, res) => {
  res.status(404).send('<h1>404 - Page Not Found</h1>');
});

// Start the server
server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

## Routing

Routing đề cập đến cách ứng dụng phản hồi một request từ client truy cập tới một **entrypoint** (URI + HTTP method). Một route có thể gán nhiều hàm xử lý, chúng sẽ được thực thi khi route của chúng khớp với một client request (route matched).

Các routes được định nghĩa theo cú pháp:

```js
app.METHOD(PATH, HANDLERs)
```

trong đó:
- `app` là một instance của `express` (imported từ module `'express'`)
- `METHOD` là một trong các HTTP method: `get`, `post`, `delete`, `head`,...
  - Có thể sử dụng `all` để khớp với mọi HTTP method.
- `PATH` là một pattern để khớp một/nhiều URIs
- `HANDLER`s là một/nhiều callbacks function, được thực thi khi route matched.
  - Callback này có thể nhận vào `req` (đại diện cho request) và `res` (đại diện cho response) và `next` (đại diện cho callback tiếp theo)

*Các methods này được thiết kế như **fluent interface**, cho phép tạo **chaining calls***

Khi mà nhiều routes có `PATH` giống nhau (chỉ khác nhau về HTTP method), ta có thể sử dụng `app.route(PATH)` và lợi dụng đặc điểm có thể **chaining calls** mà gọi các methods `get`, `post`,... để định nghĩa `HANDLER`s riêng.

### Route path

Pattern cho `PATH` có thể là một string/regexp pattern hoặc một mảng các string/regexp pattern
- *Named Path Segment*: `:name`, path segment có thể nhận giá trị bất kỳ. 
  - Có thể kết hợp sử dụng với `-` và `.` để tạo các path segment phức tạp hơn
  - Có thể định nghĩa một [callback để tiền xử lý các named path](#param-callback)
- *Optional*: `{...}`, phần tương ứng có thể có hoặc không
- *Splat*: `*name` khớp với một hoặc nhiều ký tự bất kỳ
- Reserved Characters: `()[]?+!` (cần dùng `\` để escape chúng trong pattern)

*Giá trị thực tế của các parameters này sẽ được gán cho `req.params`*

| Pattern             | Ý nghĩa                                               |
| ------------------- | ----------------------------------------------------- |
| `/abcd`             | Khớp với path bắt đầu bằng `abcd`                     |
| `/a{/:bc}/{*splat}` | Khớp với các path bắt đầu bằng `abc` hoặc `a/ANY123/` |
| `/\/abc\|\/xyz/`    | Khớp với các path bắt đầu bằng `abc` hoặc `xyz`       |


```txt
- Route path: /exports/:from-:to.:format
- Request URL: http://localhost:3000/exports/2023-2025.csv?timestamp=false&header=true
- req.params: {"from": "2023", "to": "2025", "format": "csv"}
- req.query: {"timestamp": "false", "header": "true"}
```

### Route handler

[Xem thêm](#route-handling)

### Router

ExpressJS hỗ trợ khai báo Router để gom nhóm các routes có điểm chung. Có thể coi Router như một "mini-app" khi nó bao đóng một nhóm các routes (cùng với middlewares), sau đó có thể nạp (mount) chúng vào một route của `app` như một middleware (sử dụng `use()`)

Với sự hỗ trợ của Router, thông thường ứng dụng sẽ module hóa dựa trên routes, tạo Router cho từng module trong thư mục `routes` và mount chúng vào `app`.

Thậm chí có thể mount một Router vào một Router, thực tế `app` cũng là một Router và các Routers đều là các middlewares.

```js
// src/routes/userRoutes.js
import { Router } from 'express';
const router = Router();

router.get('/', (req, res) => {
  res.send('Get all users');
});

router.get('/:id', (req, res) => {
  const { id } = req.params;
  res.send(`Get user with ID ${id}`);
});

router.post('/', (req, res) => {
  const userData = req.body;
  res.status(201).send('User created');
});

export default router;

// src/app.js
import express from 'express';
import userRoutes from './routes/userRoutes.js';

const app = express();

app.use(express.json());

// Mount the router. So the matched routes becomes
//  GET /users         ==> 'Get all users'
//  GET /users/:id     ==> `Get user with ID ${id}`
// POST /users         ==> 'User created'
app.use('/users', userRoutes);

app.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

### Param Callback

Cả app và router đều cho phép định nghĩa một bộ tiền xử lý với named path segment với method `app.param(name, callback)` hoặc `router.param(name, callback)`.

Cụ thể, với mỗi request gửi đến server mà request URL của nó khớp với ít nhất một path pattern chứa named segment thì Express sẽ gọi tới callback tương ứng với named segment đó.
- *Callback này được gọi ngay trước các routes handlers và middlewares định nghĩa riêng cho route đó*
- *Khi có nhiều routes cùng khớp với request URL thì callback chỉ gọi một lần*

Param callback được định nghĩa với các tham số theo thứ tự: `req`, `res`, `next`, `value` và `name` (trong đó `value` là giá trị của path segment và `name` là tên của path segment đó)

Riêng app cho phép truyền vào một mảng các named segment, khi đó, gọi `next` sẽ thực hiện gọi param callback cho named segment phía sau, và gọi `next` ở named segment cuối cùng trong mảng sẽ gọi middleware/route handler của route hiện tại.

```ts
app.param('id', (req, res, next, id) => {
  console.log('CALLED ONLY ONCE')
  next()
})

app.get('/user/:id', (req, res, next) => {
  console.log('although this matches')
  next()
})

app.get('/user/:id', (req, res) => {
  console.log('and this matches too')
  res.end()
})

// GET /user/42
//
// CALLED ONLY ONCE
// although this matches
// and this matches too
```

## Middleware

ExpressJS cho phép triển khai logic của ứng dụng theo một chuỗi các lời gọi hàm, mỗi hàm có trách nhiệm xử lý trên request hoặc response từ kết quả xử lý của hàm phía trước nó. Mỗi hàm như vậy được gọi là **middleware**.

Việc triển khai logic ứng dụng thành các middlewares cho phép tuần tự hóa các logic xử lý, và cũng có thể kết thúc logic xử lý sớm hơn: *chỉ khi middleware phía trước cho phép tiếp tục xử lý request thì các middlewares phía sau mới được gọi*.

Middleware đơn giản chỉ là các hàm callbacks với các đối số là `req` (đại diện cho Request), `res` (đại diện cho Response), `next` (đại diện cho middleware ngay sau nó), nó có thể thực hiện các chức năng:
- Kiểm tra điều kiện: xác thực, phân quyền, tài nguyên,...
- Tạo thay đổi trên `req` và `res`
- Kết thúc xử lý đối với request hoặc gọi middleware tiếp theo để xử lý tiếp request.

Middleware có thể áp dụng ở mức ứng dụng (gán cho `app`), ở mức "mini-app" (gán cho router).

Để gán middleware, ta có thể sử dụng `use()` hoặc `METHOD()` trên `app` hoặc Router. Các methods này đều có thể nhận vào đối số tùy chọn (đầu tiên) là [`PATH`](#route-path), khi đó middleware chỉ được gọi khi route matched, còn nếu không chỉ định thì sẽ được gọi với mọi requests ở phạm vi `app` hoặc Router.

### Route handling

Route handlers (cái mà gán cho `METHOD()`) thực tế là các middlewares được áp dụng chỉ cho một route cụ thể. Điểm đặc biệt là có thể gọi `next('route')` (đối với `app`) hoặc `next('router')` (đối với Router) để kết thúc xử lý đối với matched route hiện tại (và chuyển sang matched route tiếp theo ở scope tương ứng).

```js
import express from 'express';
const app = express();

app.get('/users/:id', (req, res, next) => {
  // if the user ID is 0, skip to the next route
  if (req.params.id === '0') next('route');
  else next();
}, (req, res, next) => {
  res.send('regular');
})

app.get('/users/*splat', (req, res, next) => {
  res.send('special');
})

app.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});

// GET users/1  ==> 'regular'
// GET users/0  ==> 'special'
```

### Error handling

Mặc định các lỗi (kế thừa từ `Error`) xảy ra trong ứng dụng đều được Express tự động bắt và xử lý.

Ngoài ra, việc gọi `next()` với đối số bất kỳ khác `'route'` (và `'router'`, nếu scope là Router) đều được coi là báo lỗi.

**Đối với v4, các lỗi xảy ra *bất đồng bộ* cần thủ công gọi `next(err)`**.

```js
// auto caught
app.get('/', (req, res) => {
  throw new Error('BROKEN') // Express will catch this on its own.
})

// need to call `next(err)`
app.get('/', (req, res, next) => {
  Promise.resolve().then(() => {
    throw new Error('BROKEN')
  }).catch(next) // Errors will be passed to Express.
})
```

**Từ v5, nếu middleware trả về một Rejected Promise thì Express sẽ tự động gọi `next(value)`**

```js
// auto call `next(err)` if getUserById rejected
app.get('/user/:id', async (req, res, next) => {
  const user = await getUserById(req.params.id)
  res.send(user)
})
```

Tuy nhiên, nếu bên trong middleware có gọi một lời gọi bất đồng bộ thì vẫn cần xử lý thủ công lỗi đó

```js
// missing try...catch will hanging the request because of Express can not call the error handler to handle this asynchronous error
app.get('/user/:id', async (req, res, next) => {
  setTimeout(() => {
    try {
      throw new Error('BROKEN')
    } catch (err) {
      next(err)
    }
  }, 100)
})
```

Khi có lỗi xảy ra, Express sẽ hủy việc thực thi mọi middlewares thông thường phía sau, mà chuyển sang gọi các middlewares xử lý lỗi.

Các middlewares xử lý lỗi luôn phải định nghĩa đầy đủ 4 tham số theo thứ tự: `err` (đại diện cho lỗi), `req`, `res` và `next`.

Express có sẵn một error handling middleware mặc định, nó sẽ trả về stack trace (với môi trường development) hoặc `res.statusMessage` (với môi trường production). Cụ thể, middleware này thực hiện:
- Gán `res.statusCode = err.status || err.statusCode`
- Gán `res.statusMessage = nameOf(res.statusCode)`
- Gộp `res.headers |= err.headers`

### Một số middlewares

[Dưới đây là danh sách các middlewares dùng phổ biến trong ExpressJS](https://expressjs.com/en/resources/middleware.html)

| Middleware      | Chức năng                                                                                                                                                   |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| static          | Phản hồi lại các tài nguyên tĩnh: JS, CSS, images,..., thường ở thư mục `public`                                                                            |
| serve-static    | Tương tự `static` nhưng cần tải từ registry, có nhiều options hơn                                                                                           |
| body-parser     | Phân giải HTTP request body theo một số định dạng cơ bản (JSON, text, binary, URL Form)                                                                     |
| compression     | Nén HTTP responses                                                                                                                                          |
| cookie-parser   | Phân giải `Cookie` header và gán cho `req.cookies`                                                                                                          |
| session         | Thiết lập session ở phía server (sessionID) (development)                                                                                                   |
| cookie-session  | Thiết lập session, lưu vào cookie ở phía client                                                                                                             |
| cors            | Cho phép cross-origin resource sharing (CORS)                                                                                                               |
| method-override | Cho phép ghi đè `req.method` dựa trên giá trị của `X-HTTP-Method-Override` (hỗ trợ dùng `POST` thay thế cho `PUT` và `DELETE` nếu client code không hỗ trợ) |
| morgan          | HTTP request logger                                                                                                                                         |
| multer          | Xử lý multi-part form (upload files)                                                                                                                        |
| response-time   | Lưu lại thời gian phản hồi (tính từ lúc bắt đầu middleware đầu tiên đến khi response header được gửi) và ghi vào `X-Response-Time`                          |
| timeout         | Thiết lập timeout cho request                                                                                                                               |

## Request và Response

[Danh sách các properties và methods hỗ trợ trên `req` và `res`](https://expressjs.com/en/5x/api.html)
- https://expressjs.com/en/guide/overriding-express-api.html

Ngoài ra, ta cũng có thể ghi đè cách thức hoạt động của các methods hoặc các properties dạng getter

```js
// override the signature of `res.sendStatus`
app.response.sendStatus = function (statusCode, type, message) {
  // code is intentionally kept simple for demonstration purpose
  return this.contentType(type)
    .status(statusCode)
    .send(message)
}
```

## Cấu hình

Có thể lưu và cấu hình cho server thông qua method `get` và `set` trên `app`. Bên cạnh [các cấu hình được định nghĩa sẵn](https://expressjs.com/en/5x/api.html#app.settings.table), ta có thể định nghĩa các cấu hình tùy chỉnh.

Khi giá trị của biến cấu hình là true/false, có thể sử dụng các method và properties: `enable()`, `disable()`, `enabled`, `disabled`.

| Biến                     | Mô tả                                                                                                                                                                                                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `env`                    | Environment mode                                                                                                                                                                                                                                   |
| `view engine`            | Template engine được sử dụng                                                                                                                                                                                                                       |
| `views`                  | Thư mục chứa templates cần được xử lý bởi template engines, cũng như tìm kiếm template để phản hồi                                                                                                                                                 |
| `view cache`             | True để cache các template đã được xử lý                                                                                                                                                                                                           |
| `case sensitive routing` | True nếu muốn so khớp chính xác, xét cả hoa thường                                                                                                                                                                                                 |
| `strict routing`         | True nếu muốn so khớp chính xác, tính cả `/` ở cuối. Tức là `/users` và `/users/` là hai PATH khác nhau                                                                                                                                            |
| `query parser`           | False nếu không muốn sử dụng query parser, `'simple'` nếu muốn sử dụng [querystring](http://nodejs.org/api/querystring.html), `'extended'` nếu muốn sử dụng [qs](https://www.npmjs.org/package/qs) hoặc có thể truyền vào một hàm parser tùy chỉnh |

## Template Engine

Template Engine cho phép sinh giao diện động, kết hợp cấu trúc giao diện và biến chứa dữ liệu thành một View, tức là cho phép nhúng mã nguồn NodeJS vào trong giao diện (HTML, JS,...)

ExpressJS (generator) mặc định sử dụng **Pug** làm template engine, bên cạnh hỗ trợ Handlebars, Mustache, EJS.

Để sử dụng một template engine, cần thực hiện:

- `app.set('view engine', 'pug')`: Xác định template engine sử dụng và nạp module xử lý tương ứng
- `app.set('views', './views')`: Chỉ định thư mục chứa các templates là `views`
- `npm install pug`: Tải template engine
- Gọi `res.render(<template-file-name>, <data>)` để sinh giao diện trả về ứng với request hiện tại


```js
// my-express-pug-app/
// ├── package.json
// ├── app.js
// └── views/
//     └── index.pug

// ============ app.js ============
import express from 'express';

const app = express();

app.set('view engine', 'pug');
app.set('views', './views');

app.get('/', (req, res) => {
  res.render('index', { title: 'Hello, Pug!', message: 'Welcome to Express with Pug!' });
});

app.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});

// ============ views/index.pug ============
doctype html
html
  head
    title= title
  body
    h1= message
    p This is a page rendered with the Pug template engine.
```
