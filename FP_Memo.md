# Functional Programming Memo

## Pure Happiness with pure functions

### Side Effects

Side effects may include, but are not limit to

- Changing the file system
- Inserting a record into a database
- making an http call
- mutations
- Printing to the screen / logging
- obtaining user input
- Querying the DOM
- accessing system state

FP 不是禁止上面的操作,而是 *we want to contain them and run them in a controlled way.*, 主要通过 `functors` 和 `monads` ('函子' 和 '单子')

### The Case for Purity

#### Cacheable

```js
const memoize = (f) => {
  const cache = {};

  return (...args) => {
    const argStr = JSON.stringify(args);
    cache[argStr] = cache[argStr] || f(...args);
    return cache[argStr];
  };
};
```

```js
const squareNumber = memoize(x => x * x);

squareNumber(4); // 16

squareNumber(4); // 16, returns cache for input 4

squareNumber(5); // 25

squareNumber(5); // 25, returns cache for input 5
```

Something to note is that you can transform some impure function into pure ones by delaying evaluation:

*使用'延迟计算'方法可以将一个'不纯'的函数变成一个'纯'函数.*

```js
const pureHttpCall = memoize((url, params) => () => $.getJSON(url, params));
```

上面的代码将一个本来不纯的函数 `$.getJSON(url, params)` 转换为一个纯函数 `pureHttpCall` 

#### Referential Transparency

From Wikipedia:

> **Referential transparency** and **referential opacity** are properties of parts of computer programs. An expression is called referential transparency if it can be replaced with its corresponding value (and vice-versa) without changing the program's behavior.

引用透明性的重要性是: 当重写系统时, 允许程序员和编译器去推理程序的行为. (*The importance of referential transparency is that it allows the programmer and the compiler to reason about program behavior as a rewrite system*)

纯函数具有引用透明性. (*Since pure function don't have side effects, they can only influence the behavior of a program through their output value. Furthermore, since their output values can reliably be calculated using only their input values, pure function will always preserve referential transparency.*)

```js
const { Map } = require('immutable');

// Aliases: p = player, a = attacker, t = target
const jobe = Map({ name: 'Jobe', hp: 20, team: 'red' });
const michael = Map({ name: 'Michael', hp: 20, team: 'green' });
const decrementHP = p => p.set('hp', p.get('hp') - 1);
const isSameTeam = (p1, p2) => p1.get('team') === p2.get('team');
const punch = (a, t) => (isSameTeam(a, t) ? t : decrementHP(t));

punch(jobe, michael); // Map({name:'Michael', hp:19, team: 'green'})
```

`decrementHP`, `isSameTeam` and `punch` are all pure and therefore referential transparency.

First we'll inline the function `isSameTeam`

```js
const punch = (a, t) => (a.get('team') === t.get('team') ? t : decrementHP(t));
```

then

```js
const punch = (a, t) => ('red' === 'green' ? t : decrementHP(t));
```

then

```js
const punch = (a, t) => decrementHP(t);
```

And if we inline `decrementHP`, we see that, in this case, punch becomes a call to decrement the `hp` by 1.

```js
const punch = (a, t) => t.set('hp', t.get('hp') - 1);
```

This ability to reason about code is terrific for refactoring and understanding code in general.

## Currying

### Partial Application

理解'柯里化'前, 我们有必要先弄清楚什么是 `partial application`

> **From Wikipedia:**
>
> In computer science, **partial application** (or **partial function application**) refers to the process of fixing a number of arguments to a function, producing another function of smaller arity.

根据维基百科的解释, `partial application` 是一种 **过程 (proccess)**, 一种将函数参数固定(或称之为'预填充'), 从而产生另一个函数, 该函数(就是上面产生的另一个函数)将剩余参数(没有被固定的参数)作为其参数.

下面用一个例子解释一下为什么我们需要 **partial application**

1. 创建一个函数用于给指定的 DOM 元素添加 className 属性

   ```js
   const addClass = (className, element) => {
   	element.className = `${element.className} ${className}`;
   	return element;
   }
   ```

2. 现在我们需要使用 `map` 给一组 DOM element 增加同一个 className, 因为 `map` 的回调函数将数组中的元素放置在回调函数的第一位, 而 `addClass` 的第一个参数是 className, 这不符合 `map` 的规范, 因此我们增加一个函数

   ```js
   const addTweedleClass = (element) => addClass('tweedle', element);
   ```

3. 现在满足了要求, 我们可以继续操作了

   ```js
   const ids = ['DEE', 'DUM'];
   let elements = ids.map(document.getElementById);
   elements = elements.map(addTweedleClass);
   ```

4. 现在我们需要添加另外一个 className, 我们不得不再创建一个函数

   ```js
   const addBoyClass = (element) => addClass('boy', element);
   ```

5. 这已经明显的违反了 DRY 原则. 此时我们就可以使用 **partial application** 了, 首先我们创建一个 `partial` 函数, 这是一个高阶函数(hight-order function), 也就是创建函数的函数

   ```js
   const partial = (...args) {
   	// Grab the function (the first argument). args now contains the remaining
     const fn = args.shift();
     // Return a function that calls fn
     return (...argsOthers) => fn.apply(null, args.concat(argsOthers))
   }
   ```

6. 使用 `partial` 高阶函数改写上面的代码

   ```js
   const ids = ['DEE', 'DUM'];
   let elements = ids.map(document.getElementById);
   elements = elements.map(partial(addClass, 'tweedle'));
   elements = elements.map(partial(addClass, 'boy'));
   ```

7. 其实 JS 提供了内建的方法 `bind` 实现了 `partial` 的功能, 上面的代码使用 `bind` 改写

   ```js
   const ids = ['DEE', 'DUM'];
   let elements = ids.map(document.getElementById);
   elements = elements.map(addClass.bind(null, 'tweedle'));
   elements = elements.map(addClass.bind(null, 'boy'));
   ```

'柯里化' 实际上 `partial application` 的一种应用.

> **From Wikipeida**:
>
> In mathematics and computer science, **currying** is the technique of converting a function that takes multiple arguments into a sequence of functions that each take a single argument.

## Declarative Programming

### Arithmetic

- `+` -> `add`
- `-` -> `subtract`
- `*` -> `multiply`
- `/` -> `divide`
- `++` -> `inc`
- `--` -> `dec`
- `-(负)` -> `negate`

### Comparison

- `===` -> `equals` `identical`
- `>=` -> `gte`
- `<=` -> `lte`
- `>` -> `gt`
- `<` -> `lt`
- `isEmpty` -> checking if a string or array is empty (`str === ''` or  `arr.length === 0`)
- `isNul` -> checking if a variable is `null` or `undefined`

### Logic

- `!` -> `complement` for function. `not` for value.
- `&&` -> `both `for function. `and` for value.
- `||` -> `either `for function. `or` for value.
- `defaultTo` -> `const x = setting.length || 80` 等于 `const x = defaultTo(80, setting.length)`

### Conditionals

- `ifElse` -> 返回一个函数, 且所有接收的参数均为函数 `if...then...else` or `?...:...`
- `when` -> 返回一个函数. `if...return`
- `unless` -> 返回一个函数. `if...else...return`
- `cond` -> `switch`

### Constants

- `always` -> return a function `const t = always('Tee'); t(); //=> 'Tee'`
- `T` -> `T(); //=> true`
- `F` -> `F(); //=> false`

### Identity

- `identity` ->  return a value. just return first argument
- `nthArg` -> return a function. `nthArg(1)('a', 'b', 'c'); //=> 'b'` `nthArg(-1)('a', 'b', 'c'); //=> 'c'`
- 

## Coding by Composing

关于 `composing` 有一点是值得注意的, 就是被组合的函数满足结合律(associativity)

```js
// associativity
compose(f, compose(g, h)) === compose(compose(f, g) h)
```

### Pointfree

Pointfree 指的是一种基于函数式编程下的编程风格, 其特征为, 函数的入参不包含数据. (*Pointfree style means functions that never mention the data upon which they operate.*)

```js
// not poinfree because we mention the data: word
const snakeCase = (word) => word.toLowerCase().repleace(/\s+/ig, '_');

// pointfree
const snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase)

// not pointfree because we mention the data: name
const initials = name => name.split(' ').map(compose(toUpperCase, head)).join('. ');

// pointfree
// NOTE: we use 'intercalate' from the appendix instead of 'join' introduced in Chapter 09!
const initials = compose(intercalate('. '), map(compose(toUpperCase, head)), split(' '));

initials('hunter stockton thompson'); // 'H. S. T'
```

### Category Theory

范畴论. 函数式编程的很多理论依据是来自范畴论.

范畴, 被定义为一个集合 (a collection), 该集合中有如下属性的组件(component):

- A collection of objects (*一个对象集合*)
- A collection of morphisms (*一个态射集合*)
- A notion of composition on the morphisms (*一个关于态射的 composition 概念*)
- A distinguished morphism called identity (*一个称为同一性的特殊形态*)

将上面这些范畴论里的概念对应到函数式编程中:

**A collection of objects** 指的就是数据类型(data type), `String`, `Boolean` ...

**A collection of morphisms** 指的就是纯函数(pure function)

**A notion of composition on the morphisms** 指的是 `compose` 函数

**A distinguished morphism called identify** 指的是 `id` 函数

### `id` Function

`id` 函数看起来是这样的

```js
const id = (x) => x;
```

`id` 函数最显著特色是: 将一个函数伪装成日常使用的数据(*a function masquerading as every day data*). 

`id` 扮演==数据==角色的函数.

`id` 对编写 pointfree style code 非常有用.

`id` 和 `compose` 

```js
// identity
compose(id, f) === compose(f, id) === f;
// true
```

## Example Application

### Declarative Coding

Declarative Coding (声明式编码) 与之相对的是 Imperative Coding (命令式编码).

**Imperative Coding** 告诉计算机应该干什么. (*telling the computer how to do its job*)

**Declarative Coding** 告诉计算机我想要什么. (*write a specification of what we'd like as a result*)

```js
// imperative
const makes = [];
for (let i = 0; i < cars.length; i +=1) {
  makes.push(cars[i].make);
}

// declarative
const makes = cars.map((car) => car.make);
```

```js
// imperative
const authenticate = (form) => {
  const user = toUser(form);
  return logIn(user);
}

// declarative
const authenticate = compose(login, toUser);
```

==**Declarative Coding** leaves wiggle room for support code changes and results in our application code being a high level specification. (声明式代码为支持代码更改留下了回旋余地, 并使得我们的程序代码成为一个高层次的规范)==

## Hindley-Milner and Me

欣德利-米尔纳类型签名. 这是一种专用于函数式编程的签名语言.

```js
// capitalize :: String -> String
const capitalize = s => toUpperCase(head(s)) + toLowerCase(tail(s));

capitalize('smurf'); // 'Smurf'
```

这里 `capitalize` 接收一个 `String` 作为入参, 返回一个 `String` 结果.

```js
// match :: Regex -> (String -> [String])
const match = curry((reg, s) => s.match(reg));
```

这里 `match` 接收一个 `Regex` 返回一个函数, 该函数接收一个 `String` 并返回一个 `[String]`.

```js
// replace :: Regex -> (String -> (String -> String))
const replace = curry((reg, sub, s) => s.replace(reg, sub));
```

这里 `replace` 接收一个 `Regex` 返回一个函数, 该函数接收一个 `String` 返回函数, 这个函数也接收一个 `String` 并最终返回一个 `String` 作为结果.

```js
// id :: a -> a
const id = x => x;

// map :: (a -> b) -> [a] -> [b]
const map = curry((f, xs) => xs.map(f));
```

`id` 接收一个类型为 `a` 的参数, 并返回同样是类型 `a` 的结果.

==`map` 接收一个函数, 该函数接收一个类型为 `a` 的参数, 并返回一个类型为 `b` 的结果, 接着将该函数应用到类型为 `a` 的数组中, 最终得到一个类型为 `b` 的数组.==

```js
// reduce :: ((b, a) -> b) -> b -> [a] -> b
const reduce = curry((f, x, xs) => xs.reduce(f, x));
```

==`reduce` 接收一个有两个参数分别是类型 `a` 和类型 `b` 的函数, 该函数返回类型 `b`, 将该函数应用到类型 `b` 和 类型 `a` 的数组中, 并最终返回类型 `b` 作为结果.==

## Functor

函数式编程引入 functor 的目的是解决 `control flow`, `error handing`, `asynchronous actions`, `state` and `effects`.

### Definition

函子. 定义如下:

> A Functor is a type that implements `map` and obeys some laws

根据这篇帖子 [Blog-What is a functor](https://medium.com/@dtinth/what-is-a-functor-dcf510b098b6#:~:text=A%20functor%20is%20a%20mapping,its%20structure%20after%20the%20mapping.) 的描述:

> A functor is simply something that can be mapped over.

Wikipedia 的定义如下:

> In functional programming, a **functor** is a design pattern inspired by the definition from category theory, that allows for a generic type to apply a function inside without changing the structure of the generic type.

JS 中的 Array 就是 Functor.

我们来看下代码

```js
class Container {
  constructor(x) {
    this.$value = x;
  }
  
  static of(x) {
    return new Container(x);
  }
}

// (a -> b) -> Container a -> Container b
Container.prototype.map = (f) => Container.of(f(this.$value));
```

`Container` 就是 **Functor**, 因为它实现了 `map` 接口, 并遵循了函子设计模式的要求, 即==在泛型类型不改变泛型类型结构的前提下应用函数==.

提取 Functor 中的值:

```js
// maybe :: b -> (a -> b) -> Maybe a -> b
const maybe = curry((v, f, m) => {
  if (m.isNothing) {
    return v;
  }

  return f(m.$value);
});
```

### The functor law

1. **The identity law**: `functor` 的 `map` 函数必须返回该 `functor`. 也就是==在不改变类型结构的前提下应用 `map` 函数.==

   ```js
   functor.map(f) => functor
   ```

2. **The composition law**

   ```js
   functor.map((x) => f(g(x))) === functor.map(g).map(f)
   ```

### Summary

整体上可以这么理解, `functor` 是一个实现了 `map` 函数, 并将 `something value` 进行封装的对象, 这个 `map` 函数满足 `functor law`, 即 `1. The indentity law` 和 `2. The composition law`.

### Pure Error Handing



## Monad

单子

### Pointed Functor

定义:

> A pointed functor is a functor with an `of` method.

在 **Algebraic Structures** 中

## Applicative Functors



## Why!

### Q1

他们说, 下面的等式成立.

```js
// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) === compose(filter(p), map(f));
```

怎么可能呢? `f` 函数在左边调用了 2 次, 而在右边只调用了 1 次, 结果怎么可能相等? 做个测试试一试.

```js
const f = (a) => a + 1;
const p = (a) => a > 5;
const data = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

const fn1 = compose(map(f), filter(compose(p, f)));
const fn2 = compose(filter(p), map(f));

console.log(`
fn1: ${fn1(data)}
fn2: ${fn2(data)}
`);
// output:
// fn1: 6,7,8,9,10
// fn2: 6,7,8,9,10
```

真的是相等的...为什么呢? 直观上理解 `fn1` 应该返回 `7,8,9,10,11`. 这里的陷阱在于 `filter(compose(p, f)))` 这部分, 直观上觉得塌应该返回 `[6,7,8,9,10]`, 但实际它返回的是 `[5,6,7,8,9]`, 这是因为 `filter(compose(p, f)))` 实际等于

```js
filter((data) => (data + 1) > 5)
```

开始的理解错误主要在于理解 `compose(p, f)` 时, 总觉得 `f` 返回了一个新数组, 实际上它返回的是个数值, 而不是数组, 这是错误理解的根本.

## Resources

- [使用递归的方式遍历树模型](https://jrsinclair.com/articles/2019/functional-js-traversing-trees-with-recursive-reduce/)
- [一个 Blog 介绍 `partial`, `currying`, `composition`](https://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-functions/)
- [介绍 `purity` 和 `pointfree style`](https://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-style/)
- [Think in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/)

