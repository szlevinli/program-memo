# Lodash Memo

## Object

### assignWith

`assignWith(object, sources, [customizer])`

其中第三个参数 customizer 函数入参的说明以及该该函数调用的顺序

`customizer(objValue, srcValue, key, object, source)`

1. 依次由左到右遍历 `sources`. 比如: `assignWith({}, obj1, obj2, obj3, customizer)` 中的 `obj1, obj2, obj3` 就指的 `sources`
   1. 依次遍历 `source` 中的 `property`

`assignWith({}, obj1, obj2, obj3, customizer)`

- `objValue`: {} 中的  `property value`
- `srcValue`: `obj1, obj2, obj3` 中的 `property value`
- `key`: `obj1, obj2, obj3` 中的 `property key`
- `obj`: 在不停的变化, 没执行一次 `customizer` 函数, 该对象的`property` 都会随之变化
- `source`: `obj1, obj2, obj3`

```js
const obj = {
  name: 'levin',
  age: 10,
  list: [
    { a: 'A', b: 1 },
    { a: 'A2', b: 2 },
  ],
};

const customizer = (objValue, srcValue, key, object, source) => {
  console.log(`
  key: ${key} srcValue: ${srcValue} objValue: ${objValue}
  object: ${JSON.stringify(object, null, 2)}
  source: ${JSON.stringify(source, null, 2)}
  `);
};

const objResult = _.assignWith(
  {},
  obj,
  { type: 'type1' },
  { type2: 'type2' },
  customizer
);

console.log(`objResult: ${JSON.stringify(objResult, null, 2)}`);
```

输出结果:

```
  key: name srcValue: levin objValue: undefined
  object: {}
  source: {
  "name": "levin",
  "age": 10,
  "list": [
    {
      "a": "A",
      "b": 1
    },
    {
      "a": "A2",
      "b": 2
    }
  ]
}
  

  key: age srcValue: 10 objValue: undefined
  object: {
  "name": "levin"
}
  source: {
  "name": "levin",
  "age": 10,
  "list": [
    {
      "a": "A",
      "b": 1
    },
    {
      "a": "A2",
      "b": 2
    }
  ]
}
  

  key: list srcValue: [object Object],[object Object] objValue: undefined
  object: {
  "name": "levin",
  "age": 10
}
  source: {
  "name": "levin",
  "age": 10,
  "list": [
    {
      "a": "A",
      "b": 1
    },
    {
      "a": "A2",
      "b": 2
    }
  ]
}
  

  key: type srcValue: type1 objValue: undefined
  object: {
  "name": "levin",
  "age": 10,
  "list": [
    {
      "a": "A",
      "b": 1
    },
    {
      "a": "A2",
      "b": 2
    }
  ]
}
  source: {
  "type": "type1"
}
  

  key: type2 srcValue: type2 objValue: undefined
  object: {
  "name": "levin",
  "age": 10,
  "list": [
    {
      "a": "A",
      "b": 1
    },
    {
      "a": "A2",
      "b": 2
    }
  ],
  "type": "type1"
}
  source: {
  "type2": "type2"
}
  
objResult: {
  "name": "levin",
  "age": 10,
  "list": [
    {
      "a": "A",
      "b": 1
    },
    {
      "a": "A2",
      "b": 2
    }
  ],
  "type": "type1",
  "type2": "type2"
}
```

