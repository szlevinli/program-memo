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

#### Portable / Self-documenting

