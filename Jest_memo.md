# Jest Memo

## Install Jest for NextJs with Typescript Project

### Add Jest

```bash
yarn add -D jest @types/jest
```

### Add Babel

```bash
yarn add -D babel-jest
```

### Add testing-library library

```bash
yarn add -D @testing-library/react @testing-library/jest-dom @testing-library/user-event @testing-library/dom
```

### Config Babel

```bash
touch .babelrc
```

add to the `.babelrc` file

```json
// .babelrc
{
  "presets": ["next/babel"]
}
```

### Config Jest (Options 1)

```bash
yarn jest --init
```

### Config Jest (Option 2)

```bash
touch jest.config.js
touch jest.setup.js
```

add to the `jest.config.js` file

```json
// jest.config.js
module.exports = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/'],
};
```

add to the `jest.setup.js` file

```json
// jest.setup.ts
import '@testing-library/jest-dom';
```

### Add ts-node

For vscode extension jest can rull

```bash
yarn add -D ts-node
```

### Restar VsCode

Close Folder

Open Folder

## Mock localStorage

```typescript
jest.spyOn(Storage.prototype, 'getItem')
jest.spyOn(Storage.prototype, 'setItem')
```

## Mock default value

```typescript
import defaultFunction from './some_path';

jest.mock('./some_path');
const mockedDefaultFunction = <jest.MockedFunction<typeof defaultFunction>>defaultFunction;

it('...', () => {
  mockedDefaultFunction.mock...
});
```

使用 `ts-jest/utils` 帮助函数

```typescript
import { mocked } from 'ts-jest/utils';
import getWXAccessToken from '../utils/getWXAccessToken';

jest.mock('../utils/getWXAccessToken');
const mockedGetWXAccessToken = mocked(getWXAccessToken);
```

## Mock module With TypeScript

```typescript
import axios from 'axios';

jest.mock('axios');
const mockedAxios = axios as jest.Mocked<typeof axios>;
it('...', () => {
  mockedAxios.get.mock...
});
```

使用更好的方法.

```typescript
// d.ts
export const fn1 = () => console.log('fn1');
export const fn2 = () => console.log('fn2');
export const fn3 = () => console.log('fn3');
const fn4 = () => console.log('fn4');
export default fn4;

// a.ts
import fnd, { fn1, fn2, fn3 } from './d';

export const a = () => {
  fnd();
  fn1();
  fn2();
  fn3();
};
```

使用 `jest.mock()` 自动 mock module 中的所有内容

```typescript
// a.fullMock.spec.ts
import { a } from './a';
import fnd, { fn1 } from './d';

jest.mock('./d');

const mockedFn1 = fn1 as jest.MockedFunction<typeof fn1>;
const mockedFnd = fnd as jest.MockedFunction<typeof fnd>;

it('should ...', () => {
  a();
  expect(mockedFn1).toBeCalledTimes(1);
  expect(mockedFnd).toBeCalledTimes(1);
});
```

使用 `jest.requireActual` mock module 中需要 mock 的函数

```typescript
// a.requireActualMock.spec.ts
import { a } from './a';
import fnd, { fn1 } from './d';

jest.mock('./d', () => {
  const originalModule = jest.requireActual('./d');

  return {
    __esModule: true,
    ...originalModule,
    fn1: jest.fn().mockReturnValue(1),
    default: jest.fn(),
  };
});

it('should ...', () => {
  a();
  expect(fn1).toBeCalledTimes(1);
  expect(fnd).toBeCalledTimes(1);
});
```

## Mock abstract class with protected fields

```typescript
const mockPost = jest.fn(
  async (path: string, body: { env: string; query: string }) => {
    // 以下 if 语句用于 queryDocuments 函数抛出异常
    // 当调用 queryDocuments 函数时, 第一个参数 collectionName 设置为 'error'
    // 可以使得 queryDocuments 函数抛出异常
    if (body.query.indexOf('error') > 0)
      return {
        errcode: 1,
        errmsg: 'Some error',
      };

    return {
      errcode: 0,
      errmsg: '',
      page: {
        Offset: 0,
        Limit: 10,
        Total: 37,
      },
      data: ['123'],
    };
  }
);

jest.mock('apollo-datasource-rest', () => {
  class MockRESTDataSource {
    baseUrl = '';
    post = mockPost;
  }
  return {
    RESTDataSource: MockRESTDataSource,
  };
});
```
