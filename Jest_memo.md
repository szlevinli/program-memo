# Jest Memo

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

