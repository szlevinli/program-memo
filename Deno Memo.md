# Deno Memo

## The Runtime

### Program lifecycle

在 deno 中有两种事件注册方式

- `window.addEventListener(eventName, eventHandler)`
- `window.event = eventHandler`

```typescript
const handler = (e: Event): void => {
	// ...
}

// 方法 1
window.addEventListener('load', handler);

// 方法 2
window.onload = (e: Event): void => {
  // ...
}

```

以上两种方式的区别在于 `window.onload` 方式会出现被重载(override)的情况, 导致事件控制器无法执行, 而 `window.addEventListner` 方式则不会, 因此官方建议优先使用 `window.addEventListener` 的方式. 具体细节详见 [官方文档说明](https://deno.land/manual/runtime/program_lifecycle)

### Permission API

除了通过 `deno` CLI 运行命令时赋权, 还可以通过 API 进行赋权.

- Permission descriptors - 使用权限描述器来描述权限

```typescript
const descriptor = {name: 'read', path: '/foo'} as const;
```

- Query permissions - 查询权限的当前状态状态

```typescript
// deno run --allow-read=/foo main.ts

const descriptor = {name: 'read', path: '/foo'} as const;
await Deno.permissions.query(descriptor);
// output:
// PermissionStatus {state: 'granted'}
```

- Permission states

1. granted
2. prompt
3. denied

- Request permissions - 在代码中请求用户赋权

```typescript
// deno run main.ts


const desc1 = { name: "read", path: "/foo" } as const;
const status1 = await Deno.permissions.request(desc1);
// ⚠️ Deno requests read access to "/foo". Grant? [g/d (g = grant, d = deny)] g
console.log(status1);
// PermissionStatus { state: "granted" }

const desc2 = { name: "read", path: "/bar" } as const;
const status2 = await Deno.permissions.request(desc2);
// ⚠️ Deno requests read access to "/bar". Grant? [g/d (g = grant, d = deny)] d
console.log(status2);
// PermissionStatus { state: "denied" }
```

- Revoke permissions

```typescript
// deno run --allow-read=/foo main.ts

const desc = { name: "read", path: "/foo" } as const;
console.log(await Deno.permissions.revoke(desc));
// PermissionStatus { state: "prompt" }
```

