# This contains use cases

## Typescript

### UC1

**Vấn đề**: Tôi muốn biết chính xác dữ liệu đi kèm đối với một loại.
**Ví dụ**:
- Tôi có một ánh xạ tên và template (mail), tôi muốn biết với một tên của template, tôi sẽ cần cung cấp giá trị nào cho các placeholders của template đó.
- Tôi có một const ánh xạ tên chức năng và hàm cần gọi, tôi muốn biết đối số cung cấp cho hàm khi sử dụng chức năng đó.

**Ý tưởng**:

- Đây là use cases điển hình cho **discriminated union**
- Ví dụ:

```ts
type TMessage = { type: 'error', data: any } | { type: 'success', message: string }
```

- Tuy nhiên, cách viết này hơi phức tạp khi số lượng items nhiều

**Giải pháp**
- Ví dụ 1:

```ts
// A store specifies the data to be populated when applying a template.
export type TemplateStore = {
  THANKS: { user_id: string };
  INTERVIEW_RESERVED: { user_id: string, address: string, time: Date };
  REGISTRANT_SUCCESS: { email: string };
};

export type Template = {
  [K in keyof TemplateStore]: {
    template: K;
    data: TemplateStore[K];
  };
}[keyof TemplateStore];

// === Same as
// export type Template =
//   | { template: 'THANKS'; data: { user_id: string } }
//   | { template: 'INTERVIEW_RESERVED'; data: { user_id: string; address: string; time: Date } }
//   | { template: 'REGISTRANT_SUCCESS'; data: { email: string } };
```

- Ví dụ 2:

```ts
function login(req, res, strategy: string) {}
function logout(req, res) {}

export const ACTIONS_STORE = {
  [Action.LOGIN]: login,
  [Action.LOGOUT]: logout,
};

type ActionStore = typeof ACTIONS_STORE;

export type ActionEntry =
  | {
      [A in keyof ActionStore]: Parameters<ActionStore[A]> extends [any, any, infer C]
        ? { name: A; args: C }
        : { name: A; args?: any };
    }[keyof ActionStore]
  | { name: string; action: (...args: any[]) => void };

// === Same as
// export type ActionEntry =
//   | { name: Action.LOGIN; args: string }
//   | { name: Action.LOGOUT; args?: any }
//   | { name: string; action: (...args: any[]) => void };
```
