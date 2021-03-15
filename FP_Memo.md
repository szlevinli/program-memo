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



## 资源

- [使用递归的方式遍历树模型](https://jrsinclair.com/articles/2019/functional-js-traversing-trees-with-recursive-reduce/)
- [一个 Blog 介绍 `partial`, `currying`, `composition`](https://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-functions/)
- [介绍 `purity` 和 `pointfree style`](https://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-style/)
- 