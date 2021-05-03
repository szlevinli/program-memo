# Apollo Client Memo

## Local State

### Overview

Apollo Client 3 引入了一对具有互补机制的管理本地状态: *<u>field policies</u>* and *<u>reactive variables</u>*.

#### Field policies

通过为 **local-only fields** 定义 *<u>field policies</u>*, 我们可以使用存储在任何地方的数据来填充它(local-only field)的值, 比如存储在 **localStorage** 或者 *<u>reactive variables</u>* 中的数据.

#### Reactive variables

**Reactive variables** 可以读写应用程序中任意的本地数据, 而且不用使用 GraphQL 操作.

*<u>Reactive variables</u>* 不是存储在 Apollo Client cache 中, 因此可以不需要遵守严格的 cache type 约定.

当 <u>*reactive variables*</u> 发生改变, Apollo Client 会自动检测到它的改变. 即, 依赖于 <u>*reactive variables*</u> 的字段将会自动更新.

### Local-only fields

Apollo Client queries 可以包含 **local-only fields**, 也就是那些不在 GraphQL Server's schema 中定义的字段.

A single query 可以同时包含 local-only fields 和 GraphQL server's schema 中定义的字段.

#### Defining

1. 在 cache 对象中实现 <u>*field policies*</u>

   ```typescript
   // cache.ts
   import {InMemoryCache, makeVar} from '@apollo/client'
   
   const isAdmin = makeVar('OK!');
   const cache = new InMemoryCache({
     typePolicies: {
       User: {
         fields: {
           isAdmin: {
             // read 函数 API [详见官方文档](https://www.apollographql.com/docs/react/caching/cache-field-behavior/#fieldpolicy-api-reference)
             
             // read from reactive variable
             read: () => isAdmin()
             
             // read from other API. etc. `localStorage`
             // read: () => localStorage.getItem('isAdmin') || 'OK!'
             
          		// read from cache. 直接从 cache 中读取时, 可以省略这个定义, 但建议最好提供, 这样可以实现默认值的设定
             // read: (existsValue = 'OK!') => existsValue
           }
         }
       }
     }
   })
   ```

   

#### Querying

1. 在 GraphQL query 中使用 `@client` 指令指定 **Local-only fields**

   ```typescript
   const GET_REPOSITORIES = gql`
     query Repositories($count: Int!) {
       viewer {
         isAdmin @client
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

   

#### Storing

Apollo Client 自身提供两种方式存储 local state. (也可以使用第三方 API 存储, 比如: `localStorage`)

- Reactive variable
- The Apollo Client cache itself

##### Storing local state in reactive variables

1. 定义 <u>*reactive variable*</u>

   ```typescript
   // cache.ts
   export const isAdmin = makeVar('OK!');
   ```

   

2. 更新/修改 <u>*reactive variable*</u>

   ```tsx
   // SomeComponent.tsx
   import {isAdmin} from './cache'
   
   export const ToggleAdminButton = () => (
   	<button onclick={() => isAdmin() === 'OK!' ? 'NO!' : 'OK!'}>Toggle Admin</button>
   )
   ```

   

3. 响应 <u>*reactive variable*</u>

   1. 使用 GraphQL query + field policies

      ```tsx
      const GET_SOMETHING = gql`
      	query getSomething {
      		isAdmin @client
      	}
      `
      ```

      ```typescript
      // cache.ts
      const cache = new InMemoryCache({
        typePolicies: {
          User: {
            fields: {
              isAdmin: {
                // read from reactive variable
                read: () => isAdmin()
              }
            }
          }
        }
      })
      ```

      

   2. 使用 `useReactiveVar` hook. *必须使用这个 hook 否则组件不会接收到 reactive variable 更改的通知, 从而会导致页面不会进行响应的 render.*

   ```tsx
   // SomeComponent.tsx
   import { useReactiveVar } from '@apollo/client';
   import {isAdmin} from './cache'
   
   export default () => {
     const isAdmin_ = useReactiveVar(isAdmin)
     
     return (
     	<p>Admin: ${isAdmin_}</p>
     )
   }
   ```

   

##### Storing local state in the cache

这种方式比上面的采用 reactive variable 方式更具有一些优势, 但响应的代码编写也会更加复杂一些.

- 采用这种方式时可以不定义 field policies
- 当使用 `writeQuery` 和 `writeFragment` 修改一个被 cache 的字段时, 每一个包含该字段的 active query 将自动刷新 (automatically refreshes)

1. 调用 `cache API` 将 **Local-only fields** 写入 cache

   ```typescript
   const { data, error, refetch, networkStatus, client } = useQuery<
       TDataInResult,
       { count: number }
     >(GET_REPOSITORIES, {
       variables: { count: 3 },
       notifyOnNetworkStatusChange: true,
     });
   
     const cache = client.cache;
   
     if (data?.viewer) {
       cache.writeFragment({
         id: cache.identify(data.viewer),
         fragment: gql`
           fragment Frg on User {
             isAdmin
           }
         `,
         data: {
           // isAdmin() 是 `reactive variable`
           isAdmin: isAdmin(),
         },
       });
     }
   ```

上述完整的代码在 [codesandbox](https://codesandbox.io/s/apolloclientlocalstate-v8z73)

#### Modifying

修改 local state 的方法依赖于存储的方式:

- 使用 reactive variable 方法, 仅仅需要做的只是修改 reactive variable 的值.
- 使用 cache 方法, 调用 `cache.writeQuery`, `cache.writeFragment` 或者 `cache.modify`.
- 使用其他存储方式(比如: `localStorage`), 设置新值后, 需要调用 `cache.evict` 方法来触发刷新.

#### Using `@export`

具体详见 [code sandbox](https://codesandbox.io/s/apolloclientlocalstate-v8z73) 中的:

- cache.ts: cartItemsVar, currentItemIdVar, getItem, allItems, currentItemId
- App.tsx: GET_REPOSITORIES, ...

## Helper code

- **设置 field policies** 的帮助函数, 下面的帮助函数用于简化生成 `read` 策略.

```typescript
// forRead :: String -> String -> FieldReadFunction -> InMemoryCacheConfig
const forRead = R.curry((typeName: string, fieldName: string) =>
  R.compose<
    FieldReadFunction,
    FieldPolicy,
    Record<string, FieldPolicy>,
    TypePolicy,
    Record<string, TypePolicy>,
    InMemoryCacheConfig
  >(
    R.objOf('typePolicies'), // InMemoryCacheConfig
    R.objOf(typeName), // Record<string, TypePolicy>
    R.objOf('fields'), // TypePolicy
    R.objOf(fieldName), // Record<string, FieldPolicy>
    R.objOf('read') // FieldPolicy
  )
);
```

```typescript
const cacheConfigs: InMemoryCacheConfig[] = [];

//===== For Type's User, Field's isAdmin
const readForIsAdmin: FieldReadFunction = () => isAdmin();
cacheConfigs.push(forRead('User', 'isAdmin')(readForIsAdmin));
```

```typescript
const mergeConfigs = (
  acc: InMemoryCacheConfig,
  cur: InMemoryCacheConfig
): InMemoryCacheConfig => R.mergeDeepLeft(acc, cur) as InMemoryCacheConfig;

const config: InMemoryCacheConfig = R.reduce(mergeConfigs, {}, cacheConfigs);

const cache = new InMemoryCache(config);

export default cache;
```

