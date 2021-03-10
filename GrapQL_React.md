@[TOC](GraphQL React Apollo Client)

## 概述

学习  **GraphQL React Apollo Client** 通过下面一系列的 Blog:

- [Getting Started with GitHub's GraphQL API](https://www.robinwieruch.de/getting-started-github-graphql-apin)
- [GraphQL Tutorial for Beginners](https://www.robinwieruch.de/graphql-tutorial)
- [A complete React with GraphQL Tutorial](https://www.robinwieruch.de/react-with-graphql-tutorial)
- [Apollo Client Tutorial for Beginners](https://www.robinwieruch.de/graphql-apollo-client-tutorial)
- [React with Apollo and GraphQL Tutorial](https://www.robinwieruch.de/react-graphql-apollo-tutorial)
- [A minimal Apollo Client in React Example](https://www.robinwieruch.de/react-apollo-client-example)
- [A apollo-link-state Tutorial for Local State in React](https://www.robinwieruch.de/react-apollo-link-state-tutorial)
- [How to use Redux with Apollo Client and GraphQL in React](https://www.robinwieruch.de/react-redux-apollo-client-state-management-tutorial)

这份 Blog 关注点在前端, 因此后端采用 Github 提供的 [GraphQL API](https://docs.github.com/en/free-pro-team@latest/graphql) 接口作为服务端.

## Getting Started with GitHub's GraphQL API

介绍 GraphQL 的基本用法以及GraphQL IDE 客户端去查询 GitHub API, 无需赘述.

这里推荐的 GraphQL IDE 如下:

- [GraphiQL](https://github.com/graphql/graphiql)
- [Altair](https://github.com/imolorhe/altair)

## GraphQL Tutorial for Beginners

还是介绍 GraphQL 的基本操作, 不再记录.

## A complete React with GraphQL Tutorial

### 关于配置文件

读取配置 <u>.env</u> 的文件时, 需要在源代码中使用 `process.env.REACT_APP_<variables name>` 此时 Typescript 会报错, 显示 `process` 不存在, 解决该问题需要在 <u>tsconfig.json</u>   文件中加入如下内容, 并安装 @types/node 包

```json
{
  ...
  "types": ["node"]
  ...
}
```

```bash
npm i @types/node -D
```

### GraphQL HTTP

在不使用任何 GraphQL 包的前提下如何通过 HTTP 协议来发起基于 GraphQL 的请求, 以下描述了这种"原生"的 GraphQL 访问

1. **Request Method:** POST
2. **Content-Type:** application/json;charset=UTF-8

实际上 GraphQL 的请求是使用 HTTP 的 POST 方法, 采用 JSON 格式将请求数据发到 GraphQL Server 的, POST 方法的 Body 的内容类似下面的内容:

```json
{
  "query": {
    organization(login: "the-road-to-learn-react") {
      name
      url
    }
  }
}
```

完整的 GraphQL 请求, 包括使用 GraphQL variables

- **定义 SDL**

```js
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        name
        url
        issues(last: 5) {
          edges {
            node {
              id
              title
              url
            }
          }
        }
      }
    }
  }
`;
```



- **发起 POST 请求**

```typescript
const getIssuesOfRepository = (path: string) => {
  const [organization, repository] = path.split('/');
  return axiosGitHubGrapQL.post<TGetIssuesOfRepositoryRespond>('', {
    query: GET_ISSUES_OF_REPOSITORY,
    variables: { organization, repository },
  });
};
```

### 防抖 - Debounce

```jsx
export default function App() {
  const [path, setPath] = React.useState(
    'the-road-to-learn-react/the-road-to-learn-react'
  );
  
  React.useEffect(() => {
    onFetchFromGitHub(path);
  }, [path]);
  
  const handleChangeInput = (event: React.ChangeEvent<HTMLInputElement>) => {
    setPath(event.target.value);
  };
  
  return (
    <div>
      <form>
        <label htmlFor="url">Show open issues for https://github.com/</label>
        <input
          type="text"
          id="url"
          onChange={handleChangeInput}
          style={{ width: '300px' }}
          value={path}
        />
      </form>
    </div>
  );
}
```

上面代码中 `onFetchFromGitHub` 是一个调用 HTTP API 的函数, 当组件完成 mount 以及 `path` 发生变化后会自动调用, 现在的问题是在 `input` 控件上, 当用户修改 `input` 的内容时, 会不停的调用 `onFetchFromGitHub` , 因此我们需要使用 "防抖" 功能, 实现用户完成键入后再触发函数(即防抖 1 秒). 具体的 "防抖" 说明详见 [在 WEB 开发中的“节流”和“防抖”的不同](https://blog.csdn.net/levin_li/article/details/107491238).

这里采用 [loadsh](https://lodash.com/docs/4.17.15#debounce) 包的 `debounce` 函数来实现防抖功能. 在具体应用到 React 中的时候并不容易, 特别是采用 "函数式组件" 时. (因为函数式组件在每次 **再渲染 (re-render)** 时都会重新创建组件内部的函数/变量等内容).

- 第一感觉就是在 `useEffect` 中对 `onFetchFromGitHub` 函数进行防抖操作

```jsx
React.useEffect(() => {
  debounce(() => onFetchFromGitHub(path), 1000);
}, [path])
```

但是, 上面的代码并不能按预期执行, 实际上 `onFetchFromGitHub` 从来无法执行. 这是因为 `debounce` 函数在每次 **再渲染 (re-render)** 时都会被创建一个新的, 因此根本无法执行其内部的回调函数 `onFetchFromGitHub`.

为了解决这个问题, 需要引入 React 的 `useCallback` Hook.

- 使用 `useCallback` Hook

```jsx
const debounceFetchFromGitHub = React.useCallback(
  debounce(onFetchFromGitHub, 1000),
  []
);
React.useEffect(() => {
  debounceFetchFromGitHub(path);
}, [path, debounceFetchFromGitHub]);
```

上面的代码可以按预期执行, 但 lint 报错如下:

> React Hook useCallback received a function whose dependencies are unknown. Pass an inline function instead. (react-hooks/exhaustive-deps)

既然 lint 报错了就说嘛上面这种方法还是有隐患的, 因此改用 `useRef` Hook 来改写

- 使用 `useRef` Hook

```jsx
const debounceFetchFromGitHubRef = React.useRef(
  debounce(onFetchFromGitHub, 1000)
);
React.useEffect(() => {
  debounceFetchFromGitHubRef.current(path);
}, [path]);
```

上面的代码运行的很好, 且没有任何警告信息.

## React with Apollo and GraphQL Tutorial

使用 Apollo Client 进行 GraphQL 的操作.

### Configuration

- `HttpLink`: HTTP 通信.
- `InMemeoryCache`: 缓存管理.
- `ApolloClient`: 集成 `HttpLink` 和 `InMemeoryCache` 的实例.
- `ApolloProvider`: React Content 使用 `ApolloClient` 的实例.

<u>index.tsx</u>

```jsx
// index.tsx

import * as React from 'react';
import { render } from 'react-dom';
import {
  ApolloClient,
  HttpLink,
  InMemoryCache,
  ApolloProvider,
} from '@apollo/client';

import App from './App';

const GITHUB_BASE_URL = 'https://api.github.com/graphql';

const httpLink = new HttpLink({
  uri: GITHUB_BASE_URL,
  headers: {
    Authorization: `Bearer ${process.env.REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN}`,
  },
});

const cache = new InMemoryCache();

const client = new ApolloClient({
  link: httpLink,
  cache,
});

const rootElement = document.getElementById('root');
render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  rootElement
);
```

### 执行 GraphQL Query

需要使用的核心功能:

- `import { useQuery, gql } from '@apollo/client';`

步骤:

1. 定义 GraphQL 的查询语句.
2. 使用 `useQuery` Hook 发起查询.
3. 上一步的操作将返回的重要内容如下:
   1. loading: 服务器的是否返回结果.
   2. error: 错误信息对象.
   3. data: 服务器返回的查询结果数据.

完成的代码如下:

```jsx
import React from 'react';
import { useQuery, gql } from '@apollo/client';
import RepositoryList from '../Repository';
import Loading from '../Loading';
import ErrorMessage from '../Error';
import { TRepositoryEdgeNode } from '../types';

const GET_REPOSITORIES_OF_CURRENT_USER = gql`
  {
    viewer {
      repositories(first: 5, orderBy: { direction: DESC, field: STARGAZERS }) {
        edges {
          node {
            id
            name
            url
            descriptionHTML
            primaryLanguage {
              name
            }
            owner {
              login
              url
            }
            stargazers {
              totalCount
            }
            viewerHasStarred
            watchers {
              totalCount
            }
            viewerSubscription
          }
        }
      }
    }
  }
`;

type TData = {
  viewer: {
    repositories: {
      edges: Array<TRepositoryEdgeNode>;
    };
  };
};

const Profile = () => {
  const { loading, error, data } = useQuery<TData>(
    GET_REPOSITORIES_OF_CURRENT_USER
  );

  if (loading) return <Loading />;
  if (error) return <ErrorMessage error={error} />;

  return <RepositoryList repositories={data?.viewer.repositories!} />;
};

export default Profile;
```



### 在应用程序级别控制 GraphQL 异常

在 <u>index.tsx</u> 应用程序入口文件中定义对 GraphQL 的应用程序级别的异常处理

需要使用如下功能:

- `import {ApolloLink} from '@apollo/client';`
- `import {onError} from '@apollo/client/link/error';`

使用方式:

```jsx
const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    // do something with graphql error
    graphQLErrors.forEach(({ message, locations, path }) =>
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${JSON.stringify(
          locations
        )}, Path: ${path}`
      )
    );
  }

  if (networkError) {
    // do something with network error
    console.log(`[Network error]: ${networkError}`);
  }
});

const link = ApolloLink.from([errorLink, httpLink]);
```

完整的 <u>index.tsx</u> 代码如下:

```jsx
import * as React from 'react';
import { render } from 'react-dom';
import {
  ApolloClient,
  HttpLink,
  InMemoryCache,
  ApolloProvider,
  ApolloLink,
} from '@apollo/client';
import { onError } from '@apollo/client/link/error';

import App from './App';

const GITHUB_BASE_URL = 'https://api.github.com/graphql';

const httpLink = new HttpLink({
  uri: GITHUB_BASE_URL,
  headers: {
    Authorization: `Bearer ${process.env.REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN}`,
  },
});

const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    // do something with graphql error
    graphQLErrors.forEach(({ message, locations, path }) =>
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${JSON.stringify(
          locations
        )}, Path: ${path}`
      )
    );
  }

  if (networkError) {
    // do something with network error
    console.log(`[Network error]: ${networkError}`);
  }
});

const link = ApolloLink.from([errorLink, httpLink]);

const cache = new InMemoryCache();

const client = new ApolloClient({
  link,
  cache,
});

const rootElement = document.getElementById('root');
render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  rootElement
);
```

> 这里需要注意在 `const link = ApolloLink.from([errorLink, httpLink]);` 中数组中的链接是有顺序要求的, `HttpLink` 必须放在最后, 因为它是 **terminating link**.

### 执行 GraphQL Mutation

需要使用的核心功能:

- `import { useMutation, gql } from '@apollo/client';`

步骤:

1. 定义 GraphQL 的查询语句.
2. 使用 `useMutation Hook 定义 Mutation 操作 (注意: *这里是定义不是执行*).
3. 上一步的操作将返回一个 Tuple 结构体:
   1. [0]: 可执行的函数, 用于发起 GraphQL Mutation, 其接受的参数是 MUtation 所需要的参数, 其type 类型为 `{ variables: Record<string, any> }`.
   2. [1]: 一个对象, 包含第一步中 GraphQL 定义的返回数据.

部分代码:

```jsx
// ...
const STAR_REPOSITORY = gql`
  mutation($id: ID!) {
    addStar(input: { starrableId: $id }) {
      starrable {
        id
        viewerHasStarred
      }
    }
  }
`;
// ...
const [addStar] = useMutation<TAddStarResultData, { id: string }>(
  STAR_REPOSITORY
);
// ...
<Button
  className="RepositoryItem-title-action"
  onClick={() => addStar({ variables: { id } })}>
    {stargazers.totalCount} Stars
</Button>

```

**这里最重要的是不要忘记 Apollo Client 在 Mutation 成功执行后自动管理了 Local State, 也就是说无需我们手动使用诸如 `useState` 来管理本地状态, Apollo Client 自动管理端本地状态必须在 Mutation 的 SDL 中定义.**

### Local State Management with Apollo Client in React

## Apollo GraphQL 官方文档

### Get Started

#### Installation

- `@types/node`: 安装该包的目的是为了在源代码中使用 `process.env` 读取环境变量.
- `@apollo/client`:
- `graphql`:

在 <u>tsconfig.ts</u> 中添加如下内容:

```json
{
  ...
  "compilerOptions": {
    ...
    "types": [
      "node"
    ]
    ...
  }
  ...
}
```



#### Create a Client

详见: [codesandbox](https://codesandbox.io/s/react-with-apollo-graphql-templat-cw52h?file=/src/index.tsx)

### Fetching

#### Queries

核心步骤:

1. 定义 GraphQL query strings, 使用 `gql`.
2. 在组件中使用 `useQuery` hook.
3. 当组件渲染时,  `useQuery` hook 会执行, 并返回远程服务器响应的结果.

- 当发生 **Polling** 和 **Refeching** 时, 如何控制 UI?

> 通过判断网络状态来处理这种情况, 核心代码如下:
>
> ```jsx
> import { NetworkStatus } from '@apollo/client';
> 
> const {loading, error, data, networkStatus, refetch} = useQuery(
>   GRAPH_QUERY_SDL,
>   {
>     variables: {/*...*/},
>     notifyOnNetworkStatusChange: true,
>   });
> if (networkStatus === NetworkStatus.refetch) return 'Refetching!';
> ```
>
> 上面的代码不要忘记设置 `notifyOnNetworkStatusChange: true` 否则将无法接收到 `networkStatus`.
>
> 需要注意, 如果在调用 `refetch` 方法时传入了新的 `variables` 则将会触发 `loading` 而不是 `refetching`.
>
> 完整的代码详见 [codesandbox polling and refetching](https://codesandbox.io/s/polling-and-refetching-440nu?file=/src/App.tsx)

- **手工执行查询 --- Executing Queries Manually**

> 在之前我们使用 `useQuery` 时, 它是在组件渲染时向服务器发起请求, 如果我们需要响应某些事件, 比如说按钮点击事件来执行查询该如何处理呢? 这种情况下我们需要使用 `useLazyQuery`.
>
> `useLazyQuery` 应用于响应事件而不是组件渲染.
>
> ```jsx
> const [
>   getRepositories,
>   { loading, data, error, networkStatus },
> ] = useLazyQuery<TDataInResult, { count: number }>(GET_REPOSITORIES, {
>   variables: { count: 3 },
>   notifyOnNetworkStatusChange: true,
> });
> ```
>
> 完整代码详见: [codesandbox Lazy Query](https://codesandbox.io/s/lazy-query-zhj6p?file=/src/App.tsx)

- Configuring fetch logic

> 默认情况下, `useQuery` 首先查询 `cache`, 如果不满足查询要求 (比如: 某些请求返回的字段不存在于 `cache`) 才会向远程服务器发起请求, 这个行为称之为 **fetch policy**. 
>
> ```jsx
> const { loading, error, data } = useQuery(GET_DOGS, {
>   fetchPolicy: "network-only"
> });
> ```
>
> - `cache-first`: default
> - `cache-only`
> - ...
>
> 具体详见 [Apollo Docs](https://www.apollographql.com/docs/react/data/queries/)

#### Mutation

核心步骤:

1. 定义 GraphQL query strings, 使用 `gql`.
2. 在组件中使用 `useMutation` hook.
3. 当组件渲染时,  `useMutation` hook 会执行, 并返回如下内容:
   1. **mutate function**: 用于执行 mutation 操作.
   2. **an object**: 表示当前 mutation 执行状态.

Mutation 需要特别关注的是, 后端数据更新后, 前端的数据该如何处理的问题. 这里分两种情况, 一种是此次操作更新的数据是 **现有的单一实体 (a single existing entity)**, 另一种是其他的情况, 包括: 更新了多个现有实体/新增实体/删除实体等.

- **Updating a single existing entity**

> 这种情况下, Apollo Client 会自动更新 cache 中的值, 同时会通知 UI 层进行再渲染.

- **Making all other cache updates**

> 这种情况下, Apollo Client 不会自动更新 cache, 而是需要在调用 `useMutation` 时, 提供一个 **update function**.

这个 **update function** 接收两个参数:

1. `cache` 对象: Apollo Client Cache.
2. 包含 Mutation 操作返回的响应结果, 该结果是一个对象, 里面有一个 `data` 字段, 代表的是更新后的新数据.

### Caching

#### Configuration

无论何时 Apollo Client 从服务器获取查询结果, 它都会自动 **cache** 这些结果到本地. 这样后续执行同样的查询时将从 **cache** 中获取, 从而极大的提高查询速度.

Apollo Client 支持两种策略去保证 **cache** 中的数据是最新的, 这两种策略是: **polling** 和 **refetching**.

- **Polling**

> 即 **轮询**, 其思路是指定一个间隔时间(毫秒), 然后不停的去执行查询操作来更新 **cache** 中的数据.
>
> ```jsx
> const {loading, error, data} = useQuery(GRAPH_QUERY_SDL, {
>   variables: {/*...*/},
>   pollInterval: 500,
> });
> ```
>
> 上面的代码会每隔 500 毫秒发起一次查询请求.

- **Refecting**

> 即 **重读**, 其思路是提供一种重新查询的方式, 由客户自行调用.
>
> ```jsx
> const {loading, error, data, refetch} = useQuery(GRAPH_QUERY_SDL, {
>   variables: {/*...*/},
> });
> //...
> return (
> //...
>   <div>
>     <button onClick={() => refetch()}>Refetch!</btton>
>   </div>
> //...
> );
> ```
>
> 上面的代码当点击 Refetch 按钮后, 会再次发起查询操作, 如果不指定 variables 则使用之前的.

##### Data normalization

将 **查询响应对象 (query reponse object)** 写入内部数据结构前, `InMemoryCache` 会将其 **normalizes**. 所谓 **normalization** 包含以下步骤:

1. 对包含在响应中的可识别对讲创建一个唯一的 ID.
2. 按 ID 将对象存入一个平面查询表中 (flat lookup table). *<u>lookup table 在 [Wikipedia](https://en.wikipedia.org/wiki/Lookup_table#:~:text=In%20computer%20science%2C%20a%20lookup,computation%20or%20input%2Foutput%20operation.) 的解释为: 在计算机科学中, 查找表是用更简单的数组索引操作代替运行时计算的数组.</u>*
3. 任何时候, 接收到具有和现在 cache 中存储对象 ID 一致的对象时, 对象的字段会进行合并.

- **Default identifier generation**

> 默认情况下, `InMemoryCache` 为任意的包括 `__typename` 字段的对象生成唯一的 ID, 生成格式为 `__typename:id` 或 `__typename:_id`.
>
> 比如某个对象 {__typename: "Task", id: 14}, 则生成的唯一 ID 为 `Task:14`

- **Customizing identifier generation by type**

> 如果对象中不包含 `id` 或 `_id`, 则可以通过 `TypePolicy` 来提供自定义的 ID 生成器
>
> ```js
> const cache = new InMemoryCache({
>   typePolicies: {
>     AllProducts: {
>       // Singleton types that have no identifying field can use an empty
>       // array for their keyFields.
>       keyFields: [],
>     },
>     Product: {
>       // In most inventory management systems, a single UPC code uniquely
>       // identifies any product.
>       keyFields: ["upc"],
>     },
>     Person: {
>       // In some user account systems, names or emails alone do not have to
>       // be unique, but the combination of a person's name and email is
>       // uniquely identifying.
>       keyFields: ["name", "email"],
>     },
>     Book: {
>       // If one of the keyFields is an object with fields of its own, you can
>       // include those nested keyFields by using a nested array of strings:
>       keyFields: ["title", "author", ["name"]],
>     },
>   },
> });
> ```

- **Disabling normalization**

> 通过设置 `keyFields` 为 `false` 可以禁止对象的 normalization.
>
> 没有 normalization 的对象不能直接访问, 但可以通过其 parent 对象进行访问.

##### TypePolicy fields

映射 (map) SDL 中定义的 type 到 `TypePolicy` 对象.

#### Reading and Writing

Apollo Client 提供如下的方法读写 cache 中的数据:

- `readQuery` and `readFragment`
- `writeQuery` and `writeFragment`
- `cache.modify` (`InMemoryCache` 提供)

##### readQuery

直接在 cache 中执行 GraphQL query.

- 如果 cache 中满足 query 的请求, 则从 cache 中返回数据, 就像 GraphQL server 做的一样.
- 否则, 抛出异常.

##### readFragment

它与 `readQuery` 的不同点在于, 可以直接从 cache 中读取 normalized 的数据. 如读取 `id=5` 的 Todo 对象, 用 `readQuery` 和 `readFragment` 写法分别如下:

- 使用 `readQuery`

```js
const { todo } = client.readQuery({
  query: gql`
    query ReadTodo($id: Int!) {
      todo(id: $id) {
        id
        text
        completed
      }
    }
  `,
  variables: {
    id: 5,
  },
});
```

- 使用 `readFragment`

```js
const todo = client.readFragment({
  id: 'Todo:5', // The value of the to-do item's unique identifier
  fragment: gql`
    fragment MyTodo on Todo {
      id
      text
      completed
    }
  `,
});
```

请注意上面代码中:

- 如果没有 `id=5` 的 `Todo` 对象, 则 `readFragment` 返回 `null`
- 如果缺失字段, 比如没有 `text` 字段, 则 `readFragment` 抛出异常.

##### writeQuery and writeFragment

使用 `writeQuery` 和 `writeFragment` 写入 cache 中的数据不会发送到 GraphQL server 端, 当 reload 发生, 这些写入的数据将丢失.

##### Combining reads and writes

从 cache 中读取数据, 然后可以选择性的进行修改.

##### cache.modify

`InMemoryCache` 的 `modify` 方法, 可以直接修改各个字段的缓存值, 甚至还可以删除掉整个字段.

> 关于上面一句话所描述的 **字段** 困扰了我好几个小时, 我一直弄不清楚这个所谓的 **字段** 到底指的是什么, 从而导致后面写了很多代码都无法按预期执行. 如下面的 SDL
>
> ```js
> const GET_REPOSITORIES = gql`
>   query Repositories($count: Int!) {
>     viewer {
>       id
>       name
>       repositories(
>         first: $count
>         orderBy: { direction: DESC, field: STARGAZERS }
>       ) {
>         edges {
>           cursor
>           node {
>             id
>             name
>           }
>         }
>       }
>     }
>   }
> `;
> ```
>
> 上面的 SDL 中能称为 **字段** 的只有一个, 那就是 `viewer`

在使用 `modify` 函数提供的帮助函数 `readField` 时, 如果提取的字段是个具有参数的字段 (比如上面代码中的 `repositories`) 那么必须使用如下方法提取

```js
const existingRepositories = readField({
  from: existingViewerRef,
  fieldName: 'repositories',
  args: {
    first: 3,
    orderBy: {
      direction: 'DESC',
      field: 'STARGAZERS',
    },
  },
});
```

上面的方法可以按预期工作, 但是我一直觉得非常别扭, 查了相关文档也没有找到如何提取此类字段的建议, 我认为这种直接改写 cache 数据的做法应该很少见.

完整的代码详见: [codesandbox Apollo Client Cache.modify()](https://codesandbox.io/s/apollo-client-cachemodify-p1el8?file=/src/App.tsx:2080-2393)

#### Garbage collection and cache eviction

`gc` 方法用于移除 cache 中不再被引用的数据, 或称之为 "无法抵达" (**not reachable**), `evict` 方法用于强制移除 cache 中的数据.

`retain` 用于防止 `gc` 的移动动作, `release` 用于恢复被 `retain` 的数据, 从而可以使得 `gc` 可以正常移除数据.

**Dangling references** 指的是数据已被删除, 但是还有其他对象使用这些 *引用*. 帮助函数 `canRead` 可以用于判断当前对象是否包含 **dangling references**

#### Customizing the behavior of cached fields

Apollo Client 可以让你自定义被缓存的字段如何 **读 (read)** 和 **写 (write)**. 这个行为称之为 **字段策略 (field policy)**. 一个 **字段策略** 包括:

- `read` 函数: 当读取缓存的字段时触发.
- `merge` 函数: 当写入数据到缓存的字段时触发.
- `key arguments` 数组: 帮助缓存避免存入不必要的重复数据.

##### The `read` function

当为某个字段设置了 `read` 函数, 那么无论什么时候查询这个字段, 该 `read` 函数将被调用. 在响应中, 字段的值由 `read` 函数的返回值替代了 cache 中该字段的值.

`read` 使用场景:

- 为字段返回默认值. (服务端返回的值为 null 的情况)
- 返回 local-only 字段
- 数值型数据的处理. (如四舍五入)
- 从一个或多个 schema field 中衍生出 local-only 字段, 比如从 `birthday` 字段中衍生出 `age` 字段

`read` 使用步骤

1. 确定类型, 这个类型指的是在 GraphQL 中定义的类型 `type TypeName`
2. 从上面选择的类型中选择需要处理的字段名称
3. 编写 `read` 函数. 

具体的 `read` 代码详见 [codesandbox Apollo Client Field Policy](https://codesandbox.io/s/apollo-client-field-policy-3leon?file=/src/index.tsx)

##### The `merge` function

当为某个字段设置了 `merge` 函数, 那么无论什么时候该字段被一个如传入值写入时, 该 `merge` 函数将被调用. 当写入操作发生时, 真正写入 cache 中的该字段的值为 `merge` 函数返回的值, 而不是原始传入的值.

`merge` 函数的一个常用场景是处理字段类型是 `array` 的字段.

`merge` 使用步骤

1. 确定类型, 这个类型指的是在 GraphQL 中定义的类型 `type TypeName`
2. 从上面选择的类型中选择需要处理的字段名称
3. 编写 `merge` 函数. 

> 需要注意的是, `merge` 函数的第二个参数, 即准备传入 cache 中的数据已经被 ApolloClient 处理过了, 因此那些能被 normalization 的字段已经被处理了.

> 需要注意 `keyArgs` 字段的使用, 当需要 `merge` 的字段带有参数, 比如下面代码中 SDL 中定义的 `repositories` 字段, 如果不使用 `keyArgs` 参数, 则 cache 中 `repositories` 字段会出现多个, 导致 `merge` 操作失败. 具体可参见下面的章节 [Specifying key arguments]

具体的 `merge` 代码详见 [codesandbox Apollo Client Field Policy](https://codesandbox.io/s/apollo-client-field-policy-3leon?file=/src/index.tsx)

`merge` 函数的使用场景

- Merging array
- Merging non-normalized objects: 甚至可以合并对象字段, 比如目前 cache 中的对象 `{name: 'levin', info: {age: 10}}`, 传入对象 `{name: 'levin', info: {sex: 1}}`, merge 后存入 cache 的对象 `{name: 'levin', info: {age: 10, sex: 1}}`. 注意: 在 merge 对象时需要使用帮助函数 `mergeObject`, 以便于可以嵌套触发字段的其他 `merge` 函数.
- Merging array of non-normalized objects
- Handing pagination

##### Specifying key arguments

### Pagination

Apollo Client 提供两种分页方案, 一种是按 **页码 (numbered pages)**, 一种是按 **游标 (cursor)**.

在数据显示上也分为两种, 一种是 **离散页面 (discrete pages)**, 一种是 **无限滚动 (infinite scrolling)**.

使用 `fetchMore` 函数实现分页的核心操作, 该函数是 `useQuery` hook 返回的一个函数.

### Local State



## 其他内容

### 生成 TypeScript Types from GraphQL Schema

- [Generating TypeScript Types from GraphQL Schema in Apollo](https://www.leighhalliday.com/generating-types-apollo)
- [Generate JavaScript Static Types From GraphQL: TypeScript and Flow](https://medium.com/atheros/generate-javascript-static-types-from-graphql-typescript-and-flow-4d28b46b8d13)
- [GraphQL Code Generator](https://graphql-code-generator.com/docs/getting-started/index)

### Flat Lookup Table

关于这个概念没有找到很合适的说明, 通过观察 Apollo Client Cache 的内容, 粗浅的理解是, 所谓 flat 即将服务器响应的对象中包含数组的元素提升到对象 top 级别, 我们来看下下面的例子.

- 定义的 GraphQL Query SDL

```js
// GraphQL Query SDL
const GET_REPOSITORIES = gql`
  query Repositories($count: Int!) {
    viewer {
      id
      repositories(
        first: $count
        orderBy: { direction: DESC, field: STARGAZERS }
      ) {
        edges {
          cursor
          node {
            id
            name
          }
        }
      }
    }
  }
`;
```

- 服务器返回的数据

```json
{
  "viewer": {
    "__typename": "User",
    "id": "MDQ6VXNlcjEyNzgzNDM4",
    "repositories": {
      "__typename": "RepositoryConnection",
      "edges": [
        {
          "__typename": "RepositoryEdge",
          "cursor": "Y3Vyc29yOnYyOpIBzhAv580=",
          "node": {
            "__typename": "Repository",
            "id": "MDEwOlJlcG9zaXRvcnkyNzE1NzQ5ODk=",
            "name": "handle-excel-with-node"
          }
        },
        {
          "__typename": "RepositoryEdge",
          "cursor": "Y3Vyc29yOnYyOpIBzhAXvRY=",
          "node": {
            "__typename": "Repository",
            "id": "MDEwOlJlcG9zaXRvcnkyNjk5OTExOTA=",
            "name": "study-mongodb-node-typescript"
          }
        },
        {
          "__typename": "RepositoryEdge",
          "cursor": "Y3Vyc29yOnYyOpIAzg-pwHA=",
          "node": {
            "__typename": "Repository",
            "id": "MDEwOlJlcG9zaXRvcnkyNjI3ODMwODg=",
            "name": "example-login-with-auth0-and-typescript"
          }
        }
      ]
    }
  }
}
```

上面服务器返回的数据实际并没有 `__typename` 字段, 而是 Apollo Client 在接收到服务器的返回数据后, 在存入 cache 前进行的 **normalization** 操作时增加的. 同样的 SDL 在 GraphiQL 工具中服务器返回的数据如下

```json
{
  "data": {
    "viewer": {
      "id": "MDQ6VXNlcjEyNzgzNDM4",
      "repositories": {
        "edges": [
          {
            "cursor": "Y3Vyc29yOnYyOpIBzhAv580=",
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkyNzE1NzQ5ODk=",
              "name": "handle-excel-with-node"
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpIBzhAXvRY=",
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkyNjk5OTExOTA=",
              "name": "study-mongodb-node-typescript"
            }
          },
          {
            "cursor": "Y3Vyc29yOnYyOpIAzg-pwHA=",
            "node": {
              "id": "MDEwOlJlcG9zaXRvcnkyNjI3ODMwODg=",
              "name": "example-login-with-auth0-and-typescript"
            }
          }
        ]
      }
    }
  }
}
```



- Cache 中的数据

```json
{
  "Repository:MDEwOlJlcG9zaXRvcnkyNzE1NzQ5ODk=": {
    "id": "MDEwOlJlcG9zaXRvcnkyNzE1NzQ5ODk=",
    "__typename": "Repository",
    "name": "handle-excel-with-node"
  },
  "Repository:MDEwOlJlcG9zaXRvcnkyNjk5OTExOTA=": {
    "id": "MDEwOlJlcG9zaXRvcnkyNjk5OTExOTA=",
    "__typename": "Repository",
    "name": "study-mongodb-node-typescript"
  },
  "Repository:MDEwOlJlcG9zaXRvcnkyNjI3ODMwODg=": {
    "id": "MDEwOlJlcG9zaXRvcnkyNjI3ODMwODg=",
    "__typename": "Repository",
    "name": "example-login-with-auth0-and-typescript"
  },
  "User:MDQ6VXNlcjEyNzgzNDM4": {
    "id": "MDQ6VXNlcjEyNzgzNDM4",
    "__typename": "User",
    "repositories({\"first\":3,\"orderBy\":{\"direction\":\"DESC\",\"field\":\"STARGAZERS\"}})": {
      "__typename": "RepositoryConnection",
      "edges": [
        {
          "__typename": "RepositoryEdge",
          "cursor": "Y3Vyc29yOnYyOpIBzhAv580=",
          "node": {
            "__ref": "Repository:MDEwOlJlcG9zaXRvcnkyNzE1NzQ5ODk="
          }
        },
        {
          "__typename": "RepositoryEdge",
          "cursor": "Y3Vyc29yOnYyOpIBzhAXvRY=",
          "node": {
            "__ref": "Repository:MDEwOlJlcG9zaXRvcnkyNjk5OTExOTA="
          }
        },
        {
          "__typename": "RepositoryEdge",
          "cursor": "Y3Vyc29yOnYyOpIAzg-pwHA=",
          "node": {
            "__ref": "Repository:MDEwOlJlcG9zaXRvcnkyNjI3ODMwODg="
          }
        }
      ]
    }
  },
  "ROOT_QUERY": {
    "__typename": "Query",
    "viewer": {
      "__ref": "User:MDQ6VXNlcjEyNzgzNDM4"
    }
  }
}
```

从上面的结果可以发现, 服务器返回的结果对象只包含一个 item, 而写入 cache 后变为了五个 item, 其中主要是将具有 `id` 和 `__typename` 属性的对象提到 top 级别, 这个做法应该就是 flat.

上面的代码具体可参见 [codesandbox Apollo Client Cache](https://codesandbox.io/s/apollo-client-pagination-f9w7g?file=/src/App.tsx)



