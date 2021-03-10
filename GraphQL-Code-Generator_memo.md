# GraphQL Code Generator Memo

## Integration with React

- Config file: <u>codegen.yml</u>

```yaml
overwrite: true
schema: 'http://localhost:4000'
documents: 'src/**/!(*.d).{ts,tsx}'
generates:
  src/generated/graphql.tsx:
    schema: ./src/client-schema.graphql
    plugins:
      - 'typescript'
      - 'typescript-operations'
      - 'typescript-react-apollo'
```

## Integration with Apollo Server

- Config file: <u>codegen.yml</u>

```yaml
overwrite: true
schema: 'src/schema.ts'
documents: null
generates:
  src/generated/graphql.ts:
    plugins:
      - add:
          content: '/* eslint-disable */'
      - 'typescript'
      - 'typescript-resolvers'
    config:
      useIndexSignature: true
      defaultMapper: Partial<{T}>
      maybeValue: T
```

> `defaultMapper: Partial<{T}>` 是插件 `typescript-resolvers`  的配置项, 用于是保证 `resolver` 返回的结果可以是其中的一部分内容, 否则将返回整个类型定义的所有字段.

> `maybeValue: T` 是插件 `typescript` 的配置, 用于生成的类型中的可选字段去掉 `null` 值