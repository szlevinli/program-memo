# Ramda Memo

## addIndex

在 ramda 中 map 函数是不提供索引的, 如果需要使用索引则需要借助 addIndex 来实现.

然而结合 typescript 后, 类型的问题层出不穷, 在这里记录一下花了一下午时间整清楚的 addIndex 的类型应用.

假设希望将一个 `string[]` 类型转换成某种对象数组, 那么可以按下面的列子实现.

```typescript
import {addIndex, map} from 'ramda';

type Person = {
  id: number;
  name: string;
};

const transformStringToPerson = (name: string, idx: number) => ({
  id: idx,
  name,
});

const mapStringsToPersons = addIndex<string, Person, string[], Person[]>(map);

const transformStringsToPersons = mapStringsToPersons(transformStringToPerson);
```

上面代码中指的深入分析的是 addIndex 的类型设定, 按顺序数码如下:

- `string`: 对应 `transformStringToPerson` 的第一个参数类型, 即 `name: string`
- `Person`: 对应 `transformStringToPerson` 的返回值类型, 即 `Person`
- `string[]`: 对应需要进行 map 转换的数组类型
- `Person[]`: 转换后的最终类型
