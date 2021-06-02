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

