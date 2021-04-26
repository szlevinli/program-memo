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

## Reference

- [Writing Resilient Components](https://overreacted.io/writing-resilient-components)
- [React Hook - Understanding Component Re-renders](https://medium.com/@guptagaruda/react-hooks-understanding-component-re-renders-9708ddee9928)