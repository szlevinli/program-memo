@[TOC](React 中的状态管理)

# 缘由

我在考虑在最近的 React 项目中应该使用哪种方式去管理应用程序状态, Redux 感觉太重了, 我想使用 React 原生提供的 useReducer 功能, 但对于这个功能我几乎一无所知, 不知道它是否能像 Redux 一样是一种单源状态管理理念, 鉴于此我谷歌了一下, 除了 React 官网对 useReducer 的文档外, 排在第二位置的是 [How to useReducer in React](https://www.robinwieruch.de/react-usereducer-hook), 打开该网站后, 发现它还有个姊妹篇, 且作者推荐优先阅读, 它就是 [What is a Reducer in JavaScript/React/Redux?](https://www.robinwieruch.de/javascript-reducer). 恩, 本着认真的态度, 我觉得现将这两篇文章看一遍, 再决定采用什么方式去管理 React 应用程序总的状态.

# What is a Reducer in JavaScript/React/Redux?

基本上 Reducer 是用于管理应用程序状态的. 本质上, 一个 Reducer 是一个函数, 该函数有两个参数 -- 当前状态和动作, 返回一个新状态.

Reducer 的函数签名看起来应该像下面展示的一样

```javascript
(state, action) => newState
```

同时 Reducer 还是纯函数.

`action` 通常被定义为一个拥有 `type` 字段的对象. 基于动作 `action` 的类型 (`type`) 来条件化的执行状态的变化

```javascript
const counterReducer = (count, action) => {
  switch (action.type) {
    case 'INCREASE':
      return count + 1;
    case 'DECREASE':
      return count - 1;
    default:
      return count;
  }
};
```

在我们实际的项目中状态通常是一个对象,因此 Reducer 看起来更像下面的格式:

```javascript
const counterReducer = (state, action) => {
  switch (action.type) {
    case 'INCREASE':
      return { ...state, count: state.count + 1 };
    case 'DECREASE':
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
};
```

总结:

- **语法:** `(state, action) => newState`
- **不变性/稳固性:** 永远不要直接修改状态, 而是创建一个新的状态.
- **状态转换:** Reducer 可以进行条件化的状态转换.
- **动作:** 一个通用的动作对象包括 `type` 类型和一个可选的信息载体 `payload`:
  - `type` 属性用于条件化状态转换.
  - `payload` 属性提供状态转换所需的信息.

# How to useReducer in React

这里作者用了典型的 Todo List 为案例说明了在 React 中如何使用 useReducer 去管理状态, 基本上比较简单也容易理解, 因此不再记录任何内容了, 反而是在最后他给出了一个使用 useReducer + useState + useContext 的方案去代替 Redux, 并且明确说明使用这种方式比较适合中型项目, 这正是我需要寻找的解决方案.

# React State Hooks: useReducer, useState, useContext

[Source](https://www.robinwieruch.de/react-state-usereducer-usestate-usecontext)

## React useState: Simple State



## React useReducer: Complex State

## React useContext: Global State

# How to create Redux with React Hooks?

[Source](https://www.robinwieruch.de/redux-with-react-hooks)

## Global Dispatch with React Hooks

## Global State with  React Hooks

## Use Combined Reducers Hooks