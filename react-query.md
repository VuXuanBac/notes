# React Query

## Reason for using React Query

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

### Prepopulate and Placeholder

Để tăng trải nghiệm người dùng, ta nên hạn chế hiển thị giao diện Loading, và để làm được điều đó thì cần phải làm cho dữ liệu sẵn sàng ngay khi cần - Query có thể ở trạng thái `success` ngay lập tức.

Ta có thể triển khai chức năng đó bằng: **Initial Data** và **Placeholder Data**.
- **Initial Data** cho phép lưu tạm dữ liệu vào cache entry => **Prepopulate**
- **Placeholder Data** cho phép lưu dữ liệu ở mức **observer** (một đối tượng phát hiện thay đổi dữ liệu trong một cache entry và thông báo cho đối tượng đăng ký nó, ví dụ đăng ký qua `useQuery`, tức là một cache entry có thể có nhiều observers)

Điều đó khiến cho hai chức năng này khác nhau:
- **Initial Data** ra lệnh ghi dữ liệu vào cache entry ngay khi tạo nó. Khi có một observer khác cũng dùng Initial Data để ghi (vào cùng cache entry) thì sẽ **không làm thay đổi dữ liệu đã có**.
  - Do trong cache có dữ liệu nên React Query chỉ refetch khi cache entry hết hạn (`staleTime`)
  - Khi refetch mà lỗi thì `status = error` nhưng `data` thì không đổi
- **Placeholder Data** thì không ghi trực tiếp vào một cache entry mà dùng lại dữ liệu đã có (ví dụ dùng lại dữ liệu ở cache entry của Query cũ)
  - Do trong cache chưa có dữ liệu nên React Query sẽ cần fetch
  - Nếu fetch lỗi thì `status = error` và `data = undefined`

Sự khác nhau này do mục đích sử dụng khác nhau:
- Sử dụng **Initial Data** khi ta đã có dữ liệu đầy đủ cho Query, hoàn toàn có thể đại diện cho dữ liệu thực sự trong một khoảng thời gian.
- Sử dụng **Placeholder Data** khi ta chỉ có một phần dữ liệu hoặc một dữ liệu tương tự, lưu vào cache entry là không hợp lý, việc fetching là cần thiết. Tuy nhiên, một phần dữ liệu này có thể tận dụng để hiển thị ngay lập tức trong layout. 
  - VD: Khi đã lấy danh sách `Post`, ta đã có `title` và `author` cho một item cụ thể. => Chỉ có một phần dữ liệu
  - VD: Trong màn phân trang, trang mới có thể hiển thị tạm thời dữ liệu của trang trước đó => Dữ liệu tương tự

#### Initial Data

**Initial Data** được cung cấp cho Query qua option: `initialData`.
- Có thể xác định một hàm để xác định giá trị cho `initialData`. Hàm này chỉ được thực thi một lần ngay lần đầu thực thi Query (trước khi tạo cache entry).
- Nếu đã xác định trước thời gian cập nhật cuối cùng của **Initial Data**, có thể xác định trong option `initialDataUpdatedAt` (một timestamp, theo ms). Giá trị này kết hợp với `staleTime` sẽ xác định thời điểm cần refetch.

```tsx
const result = useQuery({
  queryKey: ['todo', todoId],
  queryFn: () => fetch('/todos'),
  initialData: () => 
    // Use a todo from the 'todos' query as the initial data for this todo query
    queryClient.getQueryData(['todos'])?.find((d) => d.id === todoId),
  initialDataUpdatedAt: () =>
    queryClient.getQueryState(['todos'])?.dataUpdatedAt,
})
```

#### Placeholder Data

**Placeholder Data** được cung cấp cho Query qua option: `placeholderData`.
- Có thể xác định một hàm để xác định giá trị cho `placeholderData`. React Query sẽ cung cấp dữ liệu của lần thực thi Query trước vào đối số của hàm này. Tức là,ta có thể dùng lại dữ liệu của lần thực thi trước để tính toán dữ liệu cho lần thực thi hiện tại. 
- Xem thêm [Paginated Query](#paginated-queries)

```tsx
function Todo({ blogPostId }) {
  const queryClient = useQueryClient()
  const result = useQuery({
    queryKey: ['blogPost', blogPostId],
    queryFn: () => fetch(`/blogPosts/${blogPostId}`),
    placeholderData: () => {
      // Use the smaller/preview version of the blogPost from the 'blogPosts'
      // query as the placeholder data for this blogPost query
      return queryClient
        .getQueryData(['blogPosts'])
        ?.find((d) => d.id === blogPostId)
    },
  })
}
```

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

## Mutation

Nếu như Query thường dùng để fetch, tức là các HTTP Request dạng GET thì Mutation được thiết kế để tương tác với Server với các Request dạng POST, PATCH, PUT, DELETE.

Do đó, Mutation được thiết kế để thực thi theo hướng mệnh lệnh (còn Query sẽ thực thi tự động, theo các khai báo). Mặc định, các Mutations là độc lập với nhau, chúng không được cache dữ liệu như Queries.

Mỗi Mutation được tạo với một `mutationFn` (một async/Promise function) thực thi request và truyền cho `useMutation` hook.

Tương tự Query, `useMutation` cũng trả về các dữ liệu sau: 
- `status` (`isPending`, `isError`, `isSuccess`, `isIdle`)
- `isPaused`
- `error`, `data`
- `reset`: Một hàm để xóa dữ liệu trong `error` và `data`, đưa Mutation về trạng thái khởi tạo.
- `mutate`: Một hàm (async) trong đó sẽ gọi `mutationFn` và các **Callbacks**
  - Đối số đầu tiên sẽ được truyền cho `mutationFn`
  - Các đối số tiếp theo là các custom callbacks
- `mutateAsync`: Dạng Asynchronous của `mutate`
  - `mutate` không trả về giá trị, nhưng `mutateAsync` trả về Promise.
  - Có thể sử dụng khi muốn đồng bộ các Promises


Khác với Query, mặc định Mutation sẽ không retry. Tuy nhiên, ta có thể chỉ định số lần retry cho Mutation qua option `retry`

### Callbacks

Ta có thể thực thi một số logic trước/sau quá trình React Query thực hiện một `mutationFn` thông qua các Callbacks (Các Callbacks và `mutationFn` sẽ được kích hoạt trong `mutate` hoặc `mutateAsync`):
- `onMutate(variables)`: Gọi trước khi `mutationFn` thực thi
  - Nhận vào đối số đầu tiên của `mutate` (hay `mutateAsync`) - `variables`
  - Kết quả trả về của nó sẽ được truyền cho `onError` và `onSettled` (thông qua đối số `context`)
  - Phù hợp để thực thi [**Optimistic Updates**](#optimistic-updates)
- `onSuccess(data, variables, context)`: Gọi khi `mutationFn` kết thúc thành công 
  - Đối số là `data` trả về từ `mutationFn`
  - Phù hợp để thực thi [**Invalidation**](#invalidation)
  - Phù hợp để thực thi [**Direct Updates**](#direct-updates)
- `onError(error, variables, context)`: Gọi khi có lỗi khi thực thi `mutationFn` 
  - Đối số là `error` trả về từ `mutationFn`
  - Phù hợp để thực thi [**Rollback trong Optimistic Updates**](#optimistic-updates)
- `onSettled(data, error, variables, context)`: Gọi khi `mutationFn` kết thúc với bất kể trạng thái nào (thành công hay có lỗi)
  - Đối số là `data` hoặc `error` trả về từ `mutationFn`
  - Phù hợp để thực thi [**Invalidation**](#invalidation)


**Chú ý**: *3 Callbacks `onSuccess`, `onError` và `onSettled` có thể trả về Promise. Khi đó React Query sẽ tự động await nó trước khi thực hiện tiếp phần còn lại của `mutate` (hay `mutateAsync`). Điều đó có nghĩa là thao tác `mutate` sẽ chưa thể kết thúc nếu chưa hoàn thành các Callbacks, tức là Mutations sẽ vẫn ở trạng thái `pending`*.

4 Callbacks này có thể truyền vào `useMutation` và sẽ được thực thi với mọi lời gọi `mutate` và `mutateAsync`

Ngoài ra, trong lời gọi `mutate` và `mutateAsync`, ta có thể truyền `onSuccess`, `onError` và `onSettled` để **thêm** logic xử lý cho các Callbacks định nghĩa trong `useMutation`. Các Callbacks trong lời gọi `mutate` và `mutateAsync` này sẽ được thực thi sau các Callbacks tương ứng truyền cho `useMutation`.

**Chú ý**: *Khác với các phiên bản Callbacks truyền cho `useMutation` thì các Callbacks trong `mutate` không trả về giá trị.*

**Chú ý**: *Nếu có nhiều lời gọi `mutate` thì `onSuccess` truyền vào hàm này sẽ chỉ được gọi một lần, ứng với lần gọi `mutate` cuối cùng.*

```tsx
useMutation({
  mutationFn: addTodo,
  onSuccess: (data, variables, context) => {
    // Will be called 3 times
  },
})

const todos = ['Todo 1', 'Todo 2', 'Todo 3']
todos.forEach((todo) => {
  mutate(todo, {
    onSuccess: (data, variables, context) => {
      // Will execute only once, for the last mutation (Todo 3),
      // regardless which mutation resolves first
    },
  })
})
```

**Chú ý**: *Các Callbacks trong `mutate` có thể không được thực thi nếu như component bị xóa trước khi Mutation kết thúc*
- Điều đó nghĩa là nên đưa các logic quan trọng lên Callbacks trong `useMutation` (như **Invalidation**), và chỉ để các xử lý ít quan trọng (thiên về UI) ở `mutate` (như hiển thị thông báo, redirect,...)

### Serial Mutations

Mặc định các lời gọi `mutate` sẽ được thực thi song song. Để có thể thực hiện tuần tự chúng, ta cần chỉ định option `scope` là một object với `id` giống nhau.

Khi một lời gọi `mutate` với `scope.id = xxx` được thực thi, React Query sẽ kiểm tra có bất kỳ lời gọi nào khác cùng `scope.id = xxx` hay không. Nếu không thì thực thi `mutate` hiện tại. Nếu có thì `mutate` hiện tại sẽ ở trạng thái `paused` và được đẩy vào hàng đợi, chờ đến khi các `mutate` khác (có cùng `scope`) hoàn thành.

## Tie Mutations to Queries

Mutations và Queries là độc lập với nhau, chúng không cùng chia sẻ bộ nhớ cache. Điều đó đòi hỏi cơ chế đồng bộ giữa Mutations và Queries, ví dụ như việc thêm một item cần phải đồng bộ vào danh sách các items.

React Query cung cấp hai cách để thực hiện điều đó: **Invalidation** và **Direct Updates**

**Invalidation** là việc chủ động thông báo cho React Query biết rằng dữ liệu trong cache của một Query (hay nhiều Queries) đã hết thời hạn. Từ đó, **chỉ khi mà dữ liệu Query đó được sử dụng** (**active query**), React Query sẽ thực hiện refetch.

**Direct Updates** được sử dụng khi mà bản thân kết quả của Mutation có thể giúp xác định dữ liệu mới cho Query (ví dụ, Mutation để xóa có thể xác định ngay phần tử nào nên loại bỏ khỏi danh sách). Khi đó, việc refetch là không cần thiết.

### Invalidation

Để thực hiện **Invalidation**, đánh dấu một hay nhiều cache entry là hết hạn, ta sử dụng `queryClient.invalidateQueries(filters, options)`

[Filters](https://tanstack.com/query/latest/docs/framework/react/guides/filters#query-filters) cho `invalidateQueries` là một object chứa các điều kiện để lọc các queries trong cache.
- Nếu object rỗng thì sẽ tác động đến mọi Queries
- Có thể truyền object với `queryKey` để lọc những Queries có trùng (`exact: true`) hoặc có prefix xác định
- Có thể truyền cho `predicate` một hàm nhận vào Query object (từng phần tử cache entry) và trả về `true/false` xác định xem cache entry đó có được đánh dấu không. 

Mặc định, sau khi đã đánh dấu hết hạn **invalid**, chỉ những Queries được sử dụng - **active queries** (queries được dùng trong khi render). Ta có thể cấu hình lại cơ chế này bằng option cho `Filters`
- `refetchType: 'active'`: (Mặc định) Chỉ refetch active queries
- `refetchType: 'inactive'`: Chỉ refetch inactive queries
- `refetchType: 'none'`: Không refetch bất kể queries nào
- `refetchType: 'all'`: Refetch tất cả matched queries

`Options` cho `invalidateQueries` có hai thuộc tính:
- `throwOnError`: Thiết lập `true` nếu muốn hàm này trả về lỗi nếu quá trình refetch không thành công (với bất kỳ query item nào)
- `cancelRefetch`: Thiết lập `false` để chỉ định việc refetch sẽ không thực hiện nếu như query item đó đang được refetch. Nếu không, sẽ hủy refetch trước để thực hiện refetch hiện tại.

### Direct Updates

Khi kết quả trả về của một Mutation có thể xác định ngay dữ liệu (đầy đủ) cho một Query, ta có thể thực hiện tạo/cập nhật cache entry cho Query đó thông qua `queryClient.setQueryData(queryKey, updater)`.

Ở đây `updater` là một hàm nhận vào dữ liệu cũ của cache entry, và cần trả về dữ liệu mới cho nó. *Không nên thiết kế để `updater` cập nhật trực tiếp lên giá trị cũ, mà nên tạo object mới*.

Có thể cung cấp giá trị mới trực tiếp cho `updater`.

Nếu giá trị mới là `undefined` thì dữ liệu cache entry sẽ không thay đổi.

## Optimistic Updates

Đây là một kỹ thuật để hạn chế ảnh hưởng của độ trễ phản hồi từ Server đến trải nghiệm người dùng. Cụ thể, với một thao tác cập nhật đơn giản, ví dụ như reaction hay follow, ta có thể thực hiện cập nhật UI ngay lập tức (coi như ngay lập tức nhận được phản hồi thành công) ngay cả khi Mutation vẫn đang được gửi đi. Đến khi nhận được phản hồi từ Server, *nếu có sự khác biệt*, ta có thể thực hiện **rollback** về trạng thái chưa thực hiện Mutation.

Kỹ thuật này có thể được triển khai theo một số cách sau:

#### Via the UI

Cách này sử dụng trạng thái `pending` của Mutation (khi đang **Invalidation**) để tùy chỉnh UI trong 3 cases:
- Optimistic Updates (`isPending = true`)
- Refetch success (`isPending = false`)
- Mutation error => Refetch (`isPending = false, isError = true`)

```tsx
const addTodoMutation = useMutation({
  mutationFn: (newTodo: string) => axios.post('/api/data', { text: newTodo }),
  // make sure to _return_ the Promise from the query invalidation
  // so that the mutation stays in `pending` state until the refetch is finished
  onSettled: async () => {
    return await queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})

const { isPending, submittedAt, variables, mutate, isError } = addTodoMutation

render(
  <ul>
    {todoQuery.items.map((todo) => (
      <li key={todo.id}>{todo.text}</li>
    ))}
    {
      /** `variables` is the added todo 
       * while refetch not finishing, the `isPending` still be `true`
      */ 
    }
    {isPending && <li style={{ opacity: 0.5 }}>{variables}</li>}
    {
      isError && (
        <li style={{ color: 'red' }}>
          {variables}
          <button onClick={() => mutate(variables)}>Retry</button>
        </li>
      )
    }
  </ul>
)
```

Ưu điểm của cách này là đơn giản do chỉ tùy chỉnh UI theo state (và vì thế không phải thực hiện **rollback**). Tuy nhiên, hạn chế của nó là phạm vi ảnh hưởng giới hạn.

#### Via the cache

Cách này thực hiện Update dữ liệu Optimistic trực tiếp vào cache, và sau đó sẽ **Rollback** cache entry về giá trị cũ nếu có lỗi.
- Lấy trạng thái của cache trước khi `mutate` (trong `onMutate`)
- Optimistic Updates cho cache trước khi `mutate` (trong `onMutate`)
- Trả về trạng thái cũ của cache cho các Callbacks phía sau
- Khi có lỗi, khôi phục trạng thái của cache (trong `onError`) với dữ liệu cung cấp từ `onMutate`
- Thực hiện **Invalidation** sau Mutation (trong `onSettled`)

```tsx
const queryClient = useQueryClient()

useMutation({
  mutationFn: updateTodo,
  // When mutate is called:
  onMutate: async (newTodo) => {
    // Cancel any outgoing refetches
    // (so they don't overwrite our optimistic update)
    await queryClient.cancelQueries({ queryKey: ['todos'] })

    // Snapshot the previous value
    const previousTodos = queryClient.getQueryData(['todos'])

    // Optimistically update to the new value
    queryClient.setQueryData(['todos'], (old) => [...old, newTodo])

    // Return a context object with the snapshotted value
    return { previousTodos }
  },
  // If the mutation fails,
  // use the context returned from onMutate to roll back
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(['todos'], context.previousTodos)
  },
  // Always refetch after error or success:
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})
```

Cách này, do cập nhật trực tiếp vào cache nên phạm vi tác động của nó rộng hơn. Tuy nhiên, thao tác rollback là quan trọng.

## Prefetching

Nếu có thể dự đoán trước dữ liệu sẽ được dùng tiếp theo, ta có thể thực hiện prefetching để làm đầy cache cho Query đó. Ta có thể sử dụng `queryClient.prefetchQuery` và `queryClient.prefetchInfiniteQuery` với cú pháp tương tự `useQuery` và `useInfiniteQuery`.

**Chú ý**: *Do chỉ là Prefetching (với mục đích ghi vào cache) nên các hàm này sẽ luôn trả về `Promise<void>` (không trả về dữ liệu) và cũng không trả về lỗi.*
- Lỗi nên được xử lý ở Query thực sự.

```tsx
const prefetchTodos = async () => {
  // The results of this query will be cached like a normal query
  await queryClient.prefetchQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })
}
```

```tsx
const prefetchProjects = async () => {
  // The results of this query will be cached like a normal query
  await queryClient.prefetchInfiniteQuery({
    queryKey: ['projects'],
    queryFn: fetchProjects,
    initialPageParam: 0,
    getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
    pages: 3, // prefetch the first 3 pages
  })
}
```

### Event Handlers

Ta có thể Prefetching khi người dùng tương tác với UI. VD: Ngay khi người dùng hover vào button xem chi tiết, ta có thể prefetch dữ liệu của trang xem chi tiết đó.

```tsx
function ShowDetailsButton() {
  const queryClient = useQueryClient()

  const prefetch = () => {
    queryClient.prefetchQuery({
      queryKey: ['details'],
      queryFn: getDetailsData,
      // Prefetch only fires when data is older than the staleTime,
      // so in a case like this you definitely want to set one
      staleTime: 60000,
    })
  }

  return (
    <button onMouseEnter={prefetch} onFocus={prefetch} onClick={...}>
      Show Details
    </button>
  )
}
```

# References

- [Why React Query](https://ui.dev/why-react-query)
- [Documentations](https://tanstack.com/query/latest)
