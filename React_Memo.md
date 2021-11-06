# React Memo

## React With Typescript

### What is the difference between JSX.Element, ReactNode and ReactElement?

A **ReactElement** is an object with a type and props.

```typescript
interface ReactElement<P = any, T extends string | JSXElementConstructor<any> = string
  | JSXElementConstructor<any>> {
    type: T;
    props: P;
    key: Key | null;
}
```

A **ReactNode** is a ReactElement, a ReactFragment, a string, a number or an array of ReactNodes, or null, or undefined, or a boolean:

```typescript
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;

interface ReactNodeArray extends Array<ReactNode> {}
type ReactFragment = {} | ReactNodeArray;

type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;
```

A **JSX.Element** is ReactElement, with the generic type for props and type being any. It exists, as various libraries can implement JSX in their own way, therefore JSX is a global namespace that then get set by the library, React sets it like this:

```typescript
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> { }
  }
}
```

**By Example:**

```react
 <p> // <- ReactElement = JSX.Element
   <Custom> // <- ReactElement = JSX.Element
     {true && "test"} // <- ReactNode
  </Custom>
 </p>
```

### Import png file

Add floder and files <u>src/typings/png.d.ts</u>

```typescript
declare module '*.png';
```

## React Hooks - Understanding Component Re-renders

> [Source: Medium](https://medium.com/@guptagaruda/react-hooks-understanding-component-re-renders-9708ddee9928)

### useState

1. Mounting Component
2. Begin Render
3. Call useState (set state value to default)
4. [user action] - trigger event
5. setState
6. Begin Render
7. Call useState (set state value to new)

> When the parent component re-renders, its children would re-render irrespective of whether they consume the parent state (props). This is React's default behavior and it can be altered by wrapping the child components with the <u>useMemo</u> hook.

**Additional notes:**

- When a state variable is initialized using some prop value as default (snippet below), the prop value is used *only* the first time when the state is created.

```js
const [localState, setLocalState] = useState(props.theme)
```

- When setState handler is called multiple times, React batches these calls and triggers re-render only once when the calling code is inside React based event handlers. If these calls are made from non-React based handlers like setTimeout, each call will trigger a re-render.

### useEffect

- React runs useEffect handler after it fully synchronizes the current component state to DOM.

### useLayoutEffect

当组件的 props 或 state 发生改变时, 同步的需要对界面布局进行调整(通常采用对组件的 className 或 HTML 元素的 classList 进行调整), 使用 useLayoutEffect, 而不要使用 useEffect.

### useMemo & useCallback

- **useMemo** 返回的是值
- **useCallback** 返回的是函数

应用场景: 列表显示的场景, 如果提供删除操作, 那么必须使用 **useMemo** 去 wrapping 列表中的 item, 用 **useCallback** 去 wrapping *remove* 函数(这里需要注意 dependencies 以及 setState 的用法).

## Testing

### Testing-Library

#### Search Variants

- **getBy*:** 如果找不到元素会抛出异常
- **queryBy*:** 如果找不到元素不会抛出异常
- **findBy*:** 用于异步情况

#### Fire Event

`fireEvent` vs. `userEvent`, `userEvent` 包更贴近用户在浏览器中操作, 比如 `userEvent.type` 方法不但可以触发 `change` 事件, 还可以触发 `keyDown`, `keyPress`, `keyUp` 等的事件, 而 `fireEvent.change` 方法仅能触发 `change` 事件.

#### waitFor

waitFor 的内部实现逻辑类似下面的代码.

实现思路是: 每 x 毫秒 (step) 执行一次回调函数 `cb`, 如果该回调函数执行成功则结束, 否则继续执行直至超时.

`cb` 函数执行不成功会抛出异常, 由 catch 捕获, 但不处理, 继续循环.

```js
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function waitFor(cb, timeout = 500) {
  const step = 10;
  let timeSpent = 0;
  let timeOut = false;

  while (true) {
    try {
      await sleep(step);
      timeSpent += step;
      cb();
      break;
    } catch {}
    if (timeSpent >= timeout) {
      timeOut = true;
      break;
    }
  }

  if (timeOut) {
    throw new Error('timeout');
  }
}
```

### Test react hooks

[一个相当优秀且完整的关于如何测试 react hook 的文章.](https://www.toptal.com/react/testing-react-hooks-tutorial)

待测试的 hook:

```typescript
import { useEffect, useState } from 'react';

const CACHE: Record<string, any> = {};

export default function useStaleRefresh(url: string, defaultValue = []) {
  const [data, setData] = useState<any>(defaultValue);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const cacheID = url;
    if (CACHE[cacheID] !== undefined) {
      setData(CACHE[cacheID]);
      setIsLoading(false);
    } else {
      setIsLoading(true);
      setData(defaultValue);
    }

    fetch(url)
      .then((res) => res.json)
      .then((newData) => {
        CACHE[cacheID] = newData;
        setData(newData);
        setIsLoading(false);
      });
  }, [url, defaultValue]);

  return [data, isLoading];
}
```

测试 hook 的测试代码

```js
import { render, unmountComponentAtNode } from 'react-dom';
import useStaleRefresh from './useStaleRefresh';
import { act, waitFor } from '@testing-library/react';

function fetchMock(url, suffix = '') {
  return new Promise((resolve) =>
    setTimeout(() => {
      resolve({
        json: {
          data: url + suffix,
        },
      });
    }, 200 + Math.random() * 300)
  );
}

const defaultValue = { data: '' };

function TestComponent({ url }) {
  const [data, isLoading] = useStaleRefresh(url, defaultValue);

  if (isLoading) {
    return <div>loading</div>;
  }

  return <div>{data.data}</div>;
}

beforeAll(() => {
  global.fetch = jest.fn().mockImplementation(fetchMock);
});

afterAll(() => {
  global.fetch.mockClear();
  delete global.fetch;
});

let container = null;

beforeEach(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
});

afterEach(() => {
  unmountComponentAtNode(container);
  container.remove();
  container = null;
});

it('useStaleRefresh hook runs correctly', async () => {
  act(() => {
    render(<TestComponent url="url1" />, container);
  });
  expect(container.textContent).toBe('loading');
  await waitFor(() => {
    expect(container.textContent).toBe('url1');
  });

  act(() => {
    render(<TestComponent url="url2" />, container);
  });
  expect(container.textContent).toBe('loading');
  await waitFor(() => {
    expect(container.textContent).toBe('url2');
  });

  global.fetch.mockImplementation((url) => fetchMock(url, '__'));

  act(() => {
    render(<TestComponent url="url1" />, container);
  });
  expect(container.textContent).toBe('url1');
  await waitFor(() => {
    expect(container.textContent).toBe('url1__');
  });

  act(() => {
    render(<TestComponent url="url2" />, container);
  });
  expect(container.textContent).toBe('url2');
  await waitFor(() => {
    expect(container.textContent).toBe('url2__');
  });
});
```

使用 @testing-library/react-hooks 测试包测试

```js
it('useStaleRefresh hook runs correctly with test library', async () => {
  const { result, rerender } = renderHook(
    ({ url }) => useStaleRefresh(url, defaultValue),
    {
      initialProps: {
        url: 'url1',
      },
    }
  );

  expect(result.current[0]).toEqual(defaultValue);
  expect(result.current[1]).toBeTruthy();
  await waitFor(() => {
    expect(result.current[0].data).toEqual('url1');
    expect(result.current[1]).toBeFalsy();
  });

  rerender({ url: 'url2' });
  expect(result.current[0]).toEqual(defaultValue);
  expect(result.current[1]).toBeTruthy();
  await waitFor(() => {
    expect(result.current[0].data).toEqual('url2');
    expect(result.current[1]).toBeFalsy();
  });

  global.fetch.mockImplementation((url) => fetchMock(url, '__'));

  rerender({ url: 'url1' });
  expect(result.current[0].data).toEqual('url1');
  expect(result.current[1]).toBeFalsy();
  await waitFor(() => {
    expect(result.current[0].data).toEqual('url1__');
    expect(result.current[1]).toBeFalsy();
  });

  rerender({ url: 'url2' });
  expect(result.current[0].data).toEqual('url2');
  expect(result.current[1]).toBeFalsy();
  await waitFor(() => {
    expect(result.current[0].data).toEqual('url2__');
    expect(result.current[1]).toBeFalsy();
  });
});
```

## Reference

- [Writing Resilient Components](https://overreacted.io/writing-resilient-components)
- [React Hook - Understanding Component Re-renders](https://medium.com/@guptagaruda/react-hooks-understanding-component-re-renders-9708ddee9928)
