# TypeScript Memo

## TSconfig

### Module name map

启用 module name map 功能需要在 <u>tsconfig.json</u> 文件中增加如下选项

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@app/*": ["src/*"],
      "@utils/*": ["src/utils/*"],
      "@auth/*": ["src/auth/*"],
      "@datasources/*": ["src/datasources/*"],
      "@generated/*": ["src/generated/*"],
      "@graphql/*": ["src/graphql/*"]
    }
  }
}
```

在 ts 文件中可以如下引用模块

```typescript
import { genQueryDocsParaById } from '@utils/tools';
```

添加 `tsconfig-paths` 包

```bash
yarn add -D tsconfig-paths
```

增加参数到 `ts-node` 命令行

```bash
ts-node -r tsconfig-paths/register
```

添加 `tsconfig-paths-jest` 包

```bash
yarn add -D tsconfig-paths-jest
```

修改 <u>jest.config.js</u> 文件

```js
/* eslint-disable */
const tsconfig = require('./tsconfig.json');
const moduleNameMapper = require('tsconfig-paths-jest')(tsconfig);

module.exports = {
  moduleNameMapper
}
```

> 如果 `jest` 报错, 则需要将 `tsconfig.json` 中的所有注释去掉

## JavaScript

### Null vs. Undefined

```typescript
// Both null and undefined are only `==` to themselves and each other:
console.log(null == null); // true (of course)
console.log(undefined == undefined); // true (of course)
console.log(null == undefined); // true

// You don't have to worry about falsy values making through this check
console.log(0 == undefined); // false
console.log('' == undefined); // false
console.log(false == undefined); // false
```

使用 `== null` 检测 `null` 和 `undefined`

### Checking for root level undefined

在 root level 中如果变量 `foo` 是 `undefined` 那么在使用时会报 `ReferenceError` **exception**.

```typescript
// bar.ts
import something from 'lib';

if (foo != undefined) {
  // ...
}

// or
if (foo != null) {
  // ...
}
```

上面的代码将报错, 需要修改为

```typescript
// bar.ts
import something from 'lib';

if (typeof foo !== 'undefined') {
  // ...
}
```

### Number

`Number` 对象的静态值

- `Number.MAX_SAFE_INTEGER`: 9007199254740991
- `Number.MIN_SAFE_INTEGER`: -9007199254740991
- `Number.MAX_VALUE`: 1.7976931348623157e+308 (无穷大)
- `-Number.MAX_VALUE`: -1.7976931348623157e+308 (负的无穷大)
- `Number.POSITIVE_INFINITY`: 等同于 `Infinity`
- `Number.NEGATIVE_INFINITY`: 等用于 `-Infinity`
- `Number.MIN_VALUE`: 5e-324 (无穷小). 比无穷小还小的数是 `0`

一些特殊的值

- `NaN`
- `Infinity`
- `-Infinity`

---

- Interger

  奇怪的 `Number.MAX_SAFE_INTEGER` 和 `Number.MIN_SAFE_INTEGER`

  下面这段看不懂

  > **Safe** in this context refers to the fact that the value cannot be the result of a rounding error.
  >
  > The unsafe values are `+1 / -1` away from these safe values and any amount of addition / subtraction will round the result.

```typescript
console.log(Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2); // true!
console.log(Number.MIN_SAFE_INTEGER - 1 === Number.MIN_SAFE_INTEGER - 2); // true!

console.log(Number.MAX_SAFE_INTEGER);      // 9007199254740991
console.log(Number.MAX_SAFE_INTEGER + 1);  // 9007199254740992 - Correct
console.log(Number.MAX_SAFE_INTEGER + 2);  // 9007199254740992 - Rounded!
console.log(Number.MAX_SAFE_INTEGER + 3);  // 9007199254740994 - Rounded - correct by luck
console.log(Number.MAX_SAFE_INTEGER + 4);  // 9007199254740996 - Rounded!
```

To check safety you can use ES6 `Number.isSafeInteger`

```typescript
// Safe value
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER)); // true

// Unsafe value
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1)); // false

// Because it might have been rounded to it due to overflow
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 10)); // false
```

- Infinity

  无穷大. `Number.MAX_VALUE`

  ```typescript
  console.log(Number.MAX_VALUE);  // 1.7976931348623157e+308
  console.log(-Number.MAX_VALUE); // -1.7976931348623157e+308
  ```

  精确度没有改变的范围之外的值被限定在这些限制内:

  ```typescript
  console.log(Number.MAX_VALUE + 1 == Number.MAX_VALUE);   // true!
  console.log(-Number.MAX_VALUE - 1 == -Number.MAX_VALUE); // true!
  ```

  精确度更改范围之外的值解析为特殊值 `Infinity` / `-Infinity`

  ```typescript
  console.log(Number.MAX_VALUE + 10**1000);  // Infinity
  console.log(-Number.MAX_VALUE - 10**1000); // -Infinity
  ```

  除零操作也会被解析为 `Infinity` / `-Infinity`

  ```typescript
  console.log( 1 / 0); // Infinity
  console.log(-1 / 0); // -Infinity
  ```

  还有

  ```typescript
  console.log(Number.POSITIVE_INFINITY === Infinity);  // true
  console.log(Number.NEGATIVE_INFINITY === -Infinity); // true
  ```

- Infinitesimal

  无穷小. `Number.MIN_VALUE`

  ```typescript
  console.log(Number.MIN_VALUE);  // 5e-324
  ```

  比无穷小还小的值会被转换为 `0`

  ```typescript
  console.log(Number.MIN_VALUE / 10);  // 0
  ```

### Truthy

|                  **Variable Type**                   | **When it is falsy** | **When it is truthy** |
| :--------------------------------------------------: | :------------------: | :-------------------: |
|                      `boolean`                       |       `false`        |        `true`         |
|                       `string`                       | `''` (empty string)  |   any other string    |
|                       `number`                       |      `0` `Nan`       |   any other number    |
|                        `null`                        |        always        |         never         |
|                     `undefined`                      |        always        |         never         |
| Any other object including empty ones like `{}` `[]` |        never         |        always         |

## Future JavaScript Now

### Arrow Function

如何在子类的 override 中调用父类的方法

```typescript
class Adder {
  constractor(public a: number) {}
  // This function is now safe to pass around
  add = (b: number) => this.a + b;
}

class ExtendedAdder extends Adder {
  // Create a copy of parent before creating our own
  private superAdd = this.add
  // Now create our override
  add = (b: number) => this.supperAdd(b);
}
```

### Iterator

Iterator itself is not a TypeScript or ES6 feature, Iterator is a Behavioral Design Pattern common for Object oriented programming languages. It 's, generally, an object which implements the following iterface:

```typescript
interface Iterator<T> {
  next(value?: any): IteratorResult<T>;
  return?(value?: any): IteratorResult<T>;
  throw?(e?: any): IteratorResult<T>;
}
```

```typescript
interface IteratorResult<T> {
  donw: boolean;
  value: T;
}
```

### Generator

#### Externally Controlled Execution

**双向通信:** One extremely powerful feature of generators in JavaScript is that they allow two way communications

- You can control the resulting value of the `yield` expression using `iterator.next(valueToInject)`
- You can throw an exception at the point of the `yield` expression using `iterator.throw(error)`

```typescript
function* generator() {
    const bar = yield 'foo'; // bar may be *any* type
    console.log(bar); // bar!
}

const iterator = generator();
// Start execution till we get first yield value
const foo = iterator.next();
console.log(foo.value); // foo
// Resume execution injecting bar
const nextThing = iterator.next('bar');
```

## 资源

- [一篇 Stack Overflow 上关于泛化 Enum 的优秀帖子](https://stackoverflow.com/questions/50376977/generic-type-to-get-enum-keys-as-union-string-in-typescript)
- [一篇相当优秀的高级 TypeScript 模式 (How to master advanced TypeScript patterns)](https://www.freecodecamp.org/news/typescript-curry-ramda-types-f747e99744ab/)
- [对 tsconfig 解释的比较透彻的文章](https://www.stackchief.com/blog/tsconfig%20%7C%20the%20missing%20docs)
- [一篇对 unknown 解释的非常好的文章](https://mariusschulz.com/blog/the-unknown-type-in-typescript)
