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

