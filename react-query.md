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
- `isFetching`
- `fetchingStatus` - mô tả trạng thái của việc thực thi (`fetching`, `paused` - đã đến thời điểm để fetch nhưng điều kiện chưa cho phép, `idle`) - mô tả trạng thái của việc thực thi `queryFn`

Chú ý: [Tham khảo](https://tkdodo.eu/blog/status-checks-in-react-query) React Query sử dụng cơ chế **stale-while-revalidate**, tức là khi đang refetch (để lấy dữ liệu mới) thì dữ liệu cũ (trong `data`) được giữ nguyên.
- Do React Query có cơ chế tự động refetch nên trạng thái của dữ liệu `status` có thể thay đổi liên tục.
- Tức là hoàn toàn có thể xảy ra trường hợp refetch bị lỗi và `data` vẫn có dữ liệu (dữ liệu cũ)
- Như vậy, thứ tự đọc trạng thái của dữ liệu để hiển thị UI cũng quan trọng.

### Query Keys

Mỗi Query instance có thể được tạo từ một hoặc một mảng các keys, miễn là nó serializable (để có thể hash).

VD: Query cho Detail API có thể sử dụng biến là ID của object.

Thông thường, danh sách các biến mà `queryFn` phụ thuộc vào nên xuất hiện trong `queryKey`.

Về caching, keys được dùng để truy xuất dữ liệu cache bằng việc hashing giá trị keys và tìm kiếm. `queryFn` sẽ được gọi tự động (refetch) khi `queryKey` thay đổi (cache miss).

`queryFn` không nên được thiết kế để được gọi thủ công (imperative, như trong một event handler), thay vào đó cần thiết kế lại để event handler thực hiện thay đổi `queryKey` (từ đó chạy lại `queryFn`)

## References

- [Why React Query](https://ui.dev/why-react-query)
