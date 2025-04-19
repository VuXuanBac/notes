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

### Route path

Pattern cho `PATH` có thể là:
- Một string pattern
  - `(...)?` optional pattern. Từ v5 chuyển sang dùng cú pháp `{...}`
  - `+` tương tự Regex. Từ v5 không hỗ trợ
  - `*` khớp với một hoặc nhiều ký tự bất kỳ. Từ v5 yêu cầu phải xác định tên cho nó
- Một Regex pattern. Từ v5 không hỗ trợ cú pháp Regex bên trong string pattern
- Một mảng của string/regex pattern

|v4|v5|Ý nghĩa|
|--|--|--|
|`/abcd`|`/abcd`|Khớp với path bắt đầu bằng `abcd`|
|`/a(bc)?/*`|`/a{bc}/{*splat}`|Khớp với các path bắt đầu bằng `abc/` hoặc `a/ddddef` hoặc `a/ANY123/ddddef`|
|`/\/abc\|\/xyz/`|`/\/abc\|\/xyz/`|Khớp với các path bắt đầu bằng `abc` hoặc `xyz`|
|`/user-(\d+)`|Không hỗ trợ|Khớp với các path bắt đầu bằng `user-1` hoặc `user-123`|

ExpressJS cũng hỗ trợ định nghĩa parameter trên route path, với các path segment bắt đầu bằng `:`, và tên chỉ có thể là `[a-zA-Z0-9_]` hoặc kết hợp với `-` và `.` để tạo pattern phức tạp hơn. 
- *Giá trị thực tế của các parameters này sẽ được gán cho `req.params`*

```txt
- Route path: /exports/:from-:to.:format
- Request URL: http://localhost:3000/exports/2023-2025.csv?timestamp=false&header=true
- req.params: {"from": "2023", "to": "2025", "format": "csv"}
- req.query: {"timestamp": "false", "header": "true"}
```
