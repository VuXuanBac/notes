# React Query

## Tại sao lại cần React Query

- [Why React Query](https://ui.dev/why-react-query)

React vốn là một thư viện để tạo UI dưới cấu trúc cây của các Components, cái mà đóng gói cả biểu diễn hiển thị của một phần trong giao diện và logic biến đổi giao diện dựa trên `state` và `event`. Nếu như Components giúp ta có thể dùng lại để xây dựng giao diện thì React giới thiệu Hooks như một cách thức để dùng lại các logic xử lý trong Component.

Một trong những bài toán thường xuyên xuất hiện trong khi phát triển ứng dụng với React là việc tương tác với Server (để lấy dữ liệu - **fetching**, hay cập nhật dữ liệu - **mutating**). Thông thường, ta sẽ dùng `useEffect`:

```jsx
export default function App () {
  const [id, setId] = React.useState(1)
  const [data, setData] = React.useState(null)

  React.useEffect(() => {
    const handleFetch = async () => {
      setData(null)

      const res = await fetch(`${API}/${id}`)
      const json = await res.json()
      setData(json)
    }

    handleFetch()
  }, [id])

  return (
    <>
      <Card data={data}/>
      <Button onClick={() => setId(id => id + 1)} />
    </>
  )
}
```

Dưới đây là một số vấn đề của cách dùng đơn thuần trên:
- **Chưa xử lý lỗi**: Tương tác với Server (hay các services bên ngoài nói chung) luôn có rủi ro có lỗi (dữ liệu gửi lên không hợp lệ, mất kết nối, 404, 500,...)
- **Chưa xử lý trường hợp bất đồng bộ**
  - Luôn có độ trễ khi gửi và nhận dữ liệu từ Server (**loading**)
  - Vì luôn có độ trễ nên sẽ xảy ra trường hợp có nhiều requests được gửi lên Server trong khoảng thời gian ngắn (VD: Người dùng thay đổi ID liên tục). Khi đó, thứ tự response gửi về là không xác định trước (**race condition**)
- **Vấn đề chia sẻ dữ liệu giữa các Components**
  - Dữ liệu từ Server có thể thay đổi liên tục, nếu với mỗi Component ta thực hiện fetching độc lập thì dữ liệu của chúng có thể khác nhau (VD: Thay đổi `title` trong `PostDetail` nhưng không cập nhật vào danh sách `Post` ở sidebar) (**data duplication**)

Ta có thể xử lý chúng với các kỹ thuật trong React như sau:
- **Error Handling**: Thêm biến state cho lỗi
- **Loading**: Để xử lý trạng thái chờ nhận kết quả, sử dụng biến state `loading` và cung cấp một giao diện `<Loading>` thay thế
- **Race condition**: Sử dụng cleanup function trong `useEffect` để hủy kết quả của một request đã cũ
- **Data duplication**: Sử dụng `Context` để chia sẻ dữ liệu global

```jsx
export default function App () {
  const [id, setId] = React.useState(1)
  const [data, setData] = React.useState(null)
  const [loading, setIsLoading] = React.useState(true)
  const [error, setError] = React.useState(null)

  React.useEffect(() => {
    let invalidResponse = false

    const handleFetch = async () => {
      setData(null)
      setIsLoading(true)
      setError(null)
    
      try {
        const res = await fetch(`${API}/${id}`)

        if (invalidResponse) {
            return
        }
        if (res.ok === false) {
          throw new Error(`Error on fetching data #${id}`)
        }
        const json = await res.json()
        setData(json)
      }
      catch(e) {
        setError(e)
      }
      finally {
        setIsLoading(false)
      }

    }

    handleFetch()

    return () => {
        invalidReponse = true
    }
  }, [id])

  return (
    <>
      <Card data={data} error={error} loading={loading} />
      <Button onClick={() => setId(id => id + 1)} />
    </>
  )
}
```

Đoạn code trên có vẻ đã xử lý tốt ba vấn đề bên trên, và vì nó có vẻ có thể dùng lại nên ta có thể gộp lại thành một Hook.

Với vấn đề **Data duplication** giải quyết nó khiến ứng dụng dần có khuynh hướng:
- Dữ liệu được chia sẻ trong ứng dụng ngày càng rộng rãi, tức là các Contexts (hay rộng hơn là các kho dữ liệu) ngày càng bị đẩy lên Component Root. 
- Khi đó cần sử dụng thêm một State Manager (như Redux)
- `Context` cũng là một cách xử lý, tuy nhiên nhược điểm của `Context` là không cô lập dữ liệu cho từng Component, việc thay đổi một phần nhỏ trong `Context` cũng khiến cho mọi Components sử dụng nó cũng phải render lại.

React Query ra đời như một giải pháp cho bài toán tương tác với Server này, hay nói rộng hơn là một thư viện quản lý **Server State**.
- Server State là các state mà ứng dụng không hoàn toàn kiểm soát
- Giá trị của nó chỉ là một snapshot tại thời điểm mà ứng dụng đọc nó
- Dữ liệu của nó có thể tồn tại lâu dài
- Dữ liệu của nó có thể bị thay đổi bởi những tác nhân khác
- Luôn có độ trễ (đáng kể) khi trao đổi (đọc/ghi) dữ liệu.

React Query (hay TanStack Query) là một thư viện hỗ trợ **fetching**, **caching**, **synchronizing** và **updating** các Server States, bao gồm:
- Đóng gói thao tác query và mutation với các states như error, loading,...
- Caching: có thể chia sẻ dữ liệu cho nhiều Components, giảm số lượng requests lên server, tự động cập nhật dữ liệu cache cũ

Trong hầu hết ứng dụng, nếu như đã chuyển các Server States cho React Query xử lý thì sẽ chỉ còn rất ít Client States, ta hoàn toàn có thể sử dụng Context API để xử lý chúng. Với một vài ứng dụng đặc thù, khi số lượng Client State rất lớn (ví dụ ứng dụng đồ họa), ta có thể kết hợp sử dụng một thư viện quản lý Client State như Redux hay Zustand,...

Tải React Query cho ứng dụng

```bash
npm i @tanstack/react-query

# for lint (detect suspicious errors, bugs, stylistic errors)
npm i -D @tanstack/eslint-plugin-query
```

Một số cấu hình mặc định của React Query và cách cấu hình lại
- Queries tạo từ `useQuery` và `useInfiniteQuery` mặc định coi dữ liệu cache là cũ (tức là liên tục refetch) => `staleTime`
- Queries tự động được làm mới lại (refetch) mỗi khi:
  - Một instance mới của nó được mount => `refetchOnMount`
  - Cửa sổ hiện tại nhận lại focus => `refetchOnWindowFocus`
  - Kết nối mạng được khôi phục => `refetchOnReconnect`
  - Sau khi hết một khoảng thời gian chờ => `refetchInterval`
- **Inactive Queries** sẽ được dọn dẹp trong 5 phút => `gcTime` (milliseconds)
- Query không thành công sẽ được thử lại 3 lần (với độ trễ tính theo **exponential backoff**) trước khi trả về error => `retry` và `retryDelay`

## Query

Mỗi Query được tạo với key (`queryKey`) và một method trả về Promise (`queryFn`) dùng để lấy (đọc) dữ liệu từ Server, truyền cho `useQuery` hook.

Key của Query được dùng nội bộ phục vụ cho các chức năng của React Query như caching, refetching và sharing.

`useQuery` sẽ trả về một Object chứa các dữ liệu liên quan mô tả cho quá trình thực thi Query: 
- `status` - mô tả trạng thái sẵn sàng của dữ liệu (`pending`, `error`, `success`)
- `isPending`, `isError`, `isSuccess`
- `error`, `data`
- `fetchingStatus` - mô tả trạng thái của việc thực thi (`fetching`, `paused` - đã đến thời điểm để fetch nhưng điều kiện chưa cho phép, `idle`) - mô tả trạng thái của việc thực thi `queryFn`
- `isFetching` - có thể sử dụng giá trị này để biết khi nào React Query tự động refetch 

Chú ý: [Tham khảo](https://tkdodo.eu/blog/status-checks-in-react-query) React Query sử dụng cơ chế **stale-while-revalidate**, tức là khi đang refetch (để lấy dữ liệu mới) thì dữ liệu cũ (trong `data`) được giữ nguyên.
- Do React Query có cơ chế tự động refetch nên trạng thái của dữ liệu `status` có thể thay đổi liên tục.
- Tức là hoàn toàn có thể xảy ra trường hợp refetch bị lỗi và `data` vẫn có dữ liệu (dữ liệu cũ)
- Như vậy, thứ tự đọc trạng thái của dữ liệu để hiển thị UI cũng quan trọng.

### Query Keys

Mỗi Query instance có thể được tạo từ một hoặc một mảng các keys, miễn là nó serializable (để có thể hash).

VD: Query cho Detail API có thể sử dụng biến là ID của object.

Thông thường, danh sách các biến mà `queryFn` phụ thuộc vào nên xuất hiện trong `queryKey`.

Về caching, keys được dùng để truy xuất dữ liệu cache bằng việc hashing giá trị keys và tìm kiếm. `queryFn` sẽ được gọi tự động (refetch) khi `queryKey` thay đổi (cache miss).

`queryFn` không nên được thiết kế để được gọi thủ công (imperative, như trong một event handler), thay vào đó cần thiết kế lại để event handler thực hiện thay đổi `queryKey` (từ đó chạy lại `queryFn`).

Ta có thể định nghĩa Query Keys cho toàn bộ ứng dụng trong một file, từ đó dễ dàng bảo trì và phát triển. Kỹ thuật này gọi là **Query Key Factory**. Query Key Factory đơn giản chỉ là một object với giá trị của mỗi entry là một hàm trả về Query Keys cho tính năng tương ứng.

VD:

```js
export const queries = createQueryKeyStore({
  users: {
    all: null,
    detail: (userId: string) => ({
      queryKey: [userId],
      queryFn: () => api.getUser(userId),
    }),
  },
  todos: {
    detail: (todoId: string) => [todoId],
    list: (filters: TodoFilters) => ({
      queryKey: [{ filters }],
      queryFn: (ctx) => api.getTodos({ filters, page: ctx.pageParam }),
      contextQueries: {
        search: (query: string, limit = 15) => ({
          queryKey: [query, limit],
          queryFn: (ctx) => api.getSearchTodos({
            page: ctx.pageParam,
            filters,
            limit,
            query,
          }),
        }),
      },
    }),
  },
});
```

### Query Functions

`queryFn` có thể là bất kỳ function nào trả về `Promise`. Khi `queryFn` trả về rejected Promise (hoặc throw Error), error object sẽ được lưu trong `error` của Query instance.
- `axios` và `graphql-request` sẽ throw Error cho các HTTP requests lỗi
- `fetch` API lại không throw Error mà kiểm tra trạng thái dựa trên `response.ok`

`queryKey` bên cạnh làm key cho Query instance, nó cũng được truyền làm đối số cho `queryFn`

```tsx
function Todos({ status, page }) {
  const result = useQuery({
    queryKey: ['todos', { status, page }],
    queryFn: fetchTodoList,
  })
}

// Access the key, status and page variables in your query function!
function fetchTodoList({ queryKey }) {
  const [_key, { status, page }] = queryKey
  return new Promise()
}
```

Tập các đối số truyền cho `queryFn` được gọi là `QueryFunctionContext`, một object với các key sau:
- `queryKey`
- `client`: **QueryClient**
- `signal`: \[Optional] Một `AbortSignal` dùng để hủy Query
- `meta`: \[Optional] Metadata của Query

### Retries

Khi thực hiện Query thất bại (`queryFn` trả về một lỗi) thì nó sẽ tự động được thử lại. Sau khi thử lại một số lần (mặc định 3) đều thất bại thì khi đó trạng thái mới là `error`.

Ta có thể cấu hình cách thức retries cho Query (global hoặc local) như sau:
- `retry = false`: Không bao giờ retry
- `retry = true`: Retry không ngừng
- `retry = 6`: Thay đổi số lần retry
- `retry = (failureCount, error) => bool`: Tùy chỉnh cách thức xử lý khi Query có lỗi.

Thực tế, giữa các lần gọi `queryFn` sẽ luôn có khoảng thời gian chờ, và tăng dần sau mỗi lần retry (tối đa 30s). Có thể cấu hình lại thời gian chờ này như sau:
- `retryDelay = 1000`: Luôn chờ 1000ms
- `retryDelay = (attemptIndex) => number`: Thời gian chờ tùy vào thứ tự gọi `queryFn` (1,2,...)

### Network Modes

React Query cung cấp 3 modes, ở đó cách thức xử lý (cho Query và Mutation) khác nhau khi trạng thái của kết nối (tới Server) khác nhau. Ta có thể cấu hình mode ở mức Query/Mutation instance, hoặc ở mức toàn cục.

#### Always

Đây là chế độ mà React Query luôn luôn thực hiện fetching bất chấp trạng thái kết nối.
- Query không có trạng thái `paused`, luôn luôn thực hiện Retries
- Retries thất bại sẽ chuyển thành `error`

Mode này chỉ phù hợp nếu như việc kết nối tới Server có thể diễn ra ngay lập tức và có thể đảm bảo kết quả nhận được, ví dụ như truy cập vào bộ nhớ cục bộ `AsyncStorage`.

#### offlineFirst

Đây là chế độ mà React Query chỉ thực hiện `queryFn` một lần duy nhất (bất kể có kết nối mạng hay không), và không thực hiện Retries nếu lần fetch đó không thành công.

Điều này là hợp lý vì: *Tại sao phải thử lại khi tôi biết là đã không thể kết nối mạng?*

Mode này phù hợp khi ứng dụng triển khai một lớp cache nữa, ví dụ Service Worker, một bộ chặn request và trả dữ liệu cache nếu có.

#### Online

Đây là chế độ mặc định của React Query, chỉ thực hiện fetch (và retries) nếu có kết nối. Nếu không có kết nối, fetch (và retries) sẽ ở trạng thái `paused`. Sau đó, nếu kết nối được khôi phục, lần fetch (và retries) đó sẽ tự động được tiếp tục (nếu Query chưa bị cancel)

### Parallel Queries

Khi Components định nghĩa nhiều `useQuery` thì các Queries đó sẽ được thực thi song song.

Khi số lượng Queries là không xác định trước (thay đổi ở mỗi lần render Component) thì ta có thể sử dụng `useQueries`

```tsx
function App({ users }) {
  const userQueries = useQueries({
    queries: users.map((user) => {
      return {
        queryKey: ['user', user.id],
        queryFn: () => fetchUserById(user.id),
      }
    }),
  })
}
```

### Serial Queries

Khi các Queries phụ thuộc vào nhau (Query này kết thúc mới thực hiện Queries tiếp theo), ta có thể sử dụng option `enabled` để xác định điều kiện thực thi cho một Query. VD: Khi fetch projects ứng với một user, ta cần một Query để lấy user và chỉ `enabled` Query lấy các projects chỉ khi đã có kết quả cho user.

```tsx
// Get the user
const { data: user } = useQuery({
  queryKey: ['user', email],
  queryFn: getUserByEmail,
})

// if user is available, React Query changes the state then the component rerender with populated user.
const userId = user?.id

// Then get the user's projects
const {
  status,
  fetchStatus,
  data: projects,
} = useQuery({
  queryKey: ['projects', userId],
  queryFn: getProjectsByUser,
  // The query will not execute until the userId exists
  enabled: !!userId,
})
```

Trạng thái của Projects Query như sau:
- Khởi tạo: `status = "pending"; fetchStatus = "idle"`
- Khi User Query thành công: `status = "pending"; fetchStatus = "fetching"`
- Khi Projects Query thành công: `status = "success"; fetchStatus = "idle"`

Khi Query phụ thuộc là Parallel Queries, ta có thể disable nó bằng việc truyền một mảng rỗng cho `useQueries`

```tsx
// Get the users ids
const { data: userIds } = useQuery({
  queryKey: ['users'],
  queryFn: getUsersData,
  select: (users) => users.map((user) => user.id),
})

// Then get the users messages
const usersMessages = useQueries({
  queries: userIds
    ? userIds.map((id) => {
        return {
          queryKey: ['messages', id],
          queryFn: () => getMessagesByUsers(id),
        }
      })
    : [], // if users is undefined, an empty array will be returned
})
```

### Paginated Queries

Ta có thể triển khai chức năng phân trang đơn giản như Query sau:

```tsx
const result = useQuery({
  queryKey: ['projects', page],
  queryFn: fetchProjects,
})
```

Tuy nhiên, UI của các pages sẽ liên tục thay đổi ở trạng thái có dữ liệu và loading khi di chuyển giữa các trang. Ta có thể khắc phục bằng option `placeholderData` để sử dụng lại dữ liệu trang cũ khi đang fetching trang mới (kể cả khi `queryKey` đã thay đổi).

```tsx
const { isPending, isError, error, data, isFetching, isPlaceholderData } =
    useQuery({
      queryKey: ['projects', page],
      queryFn: () => fetchProjects(page),
      placeholderData: keepPreviousData,
    })
```

Giá trị của `placeholderData` có thể là:
- `(previousData) => ...`
- `keepPreviousData`: Tương đương `(previousData) => previousData`

`isPlaceholderData` của Query instance có thể được dùng để kiểm tra dữ liệu mới đã có sẵn hay chưa.

### Infinite Queries

Với tính năng Infinite Scroll, ta có thể sử dụng `useInfiniteQuery`. Tính năng này được thiết kế để tự động gọi `queryFn` mỗi khi có sự kiện lấy dữ liệu page tiếp theo được kích hoạt. React Query hỗ trợ lấy page ở hai hướng: page phía sau (fetch **Next**) và page phía trước (fetch **Previous**).


Hook này tương tự `useQuery` với một số khác biệt về đối số:
- `initialPageParam`: Đối số sẽ dùng trong `queryFn` khi fetch page đầu tiên.
- `getNextPageParam: (lastPage, allPages, lastPageParam, allPageParams)`: Hàm được gọi để lấy đối số truyền cho `queryFn` trong lần fetch **Next** tiếp theo. Trả về `null | undefined` tức là không còn page phía sau.
- `getPreviousPageParam: (firstPage, allPages, firstPageParam, allPageParams)`: Hàm được gọi để lấy đối số truyền cho `queryFn` trong lần fetch **Previous** tiếp theo. Trả về `null | undefined` tức là không còn page phía trước.
- `maxPages`: Số lượng pages tối đa. Khi các pages đã được fetch đầy đủ thì mảng dữ liệu các pages sẽ bị loại phần tử ở đầu (nếu fetch **Next**) hoặc ở cuối (nếu fetch **Previous**). Xác định `0 | undefined` tức là không giới hạn (không biết trước) số lượng pages.

Và nó trả về các dữ liệu sau:
- `data.pages`: Mảng chứa dữ liệu của tất cả các pages.
- `data.pageParams`: Mảng chứa param (đối số cho `queryFn`) của tất cả các pages.
- `isFetchingNextPage`, `isFetchingPreviousPage`: Khi fetch **Next**/**Previous** đang được thực thi.
- `hasNextPage`, `hasPreviousPage`: Dựa trên page Param để biết còn page phía sau/phía trước hay không.
- `fetchNextPage: (options) => Promise`: Một hàm có thể gọi để kích hoạt sự kiện fetch **Next**. Có thể cấu hình cách thức fetch **Next** bằng việc truyền vào đối số `options` khi gọi hàm này
  - `options.cancelRefetch`: Cách xử lý khi sự kiện fetch **Next** kích hoạt liên tục. Truyền `false` nếu muốn việc kích hoạt sẽ bị bỏ qua nếu như lần kích hoạt đầu tiên chưa hoàn thành (`queryFn` chưa hoàn thành). Khi truyền `true` (mặc định) thì các lần kích hoạt sau sẽ hủy việc thực thi của các lần kích hoạt trước.
- `fetchPreviousPage: (options) => Promise`: Tương tự `fetchNextPage` nhưng dành cho sự kiện fetch **Previous**.

## References

- [Why React Query](https://ui.dev/why-react-query)
