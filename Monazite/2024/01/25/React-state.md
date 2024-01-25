# `State`的实现

我们知道，在 `React` 中，组件内部某些属性的值依靠对应的 `state` 进行储存记忆

`useState` 的唯一参数是 state 变量的**初始值**，每一次组件渲染时， `useState` 会返回一个数组，其中包含：

- `state` 变量，会保存上次渲染的值
- `state setter` 函数，用于更新 `state` 变量的值

在实际运用中，`state` 是通过这样的步骤运行的

1. **组件进行第一次渲染。**此时传入 `useState` 的初始值作为 `state` 变量的值。
2. **更新 `state` 。**当调用 `state setter` 函数时，会传入一个新的 `state` 值，`React` 现在记住新的值并进行下一步渲染。
3. **组件的第二次渲染。** 此时 `useState` 的参数依然是初始值，但 `state setter `使 `React` 记住了新值，所以会返回 `[newState, stateSetter] `

# 关于`UseState`的工作原理

- 在 React 内部，为每个组件保存了一个数组，其中每一项都是一个 state 对。
- 它维护当前 state 对的索引值，在渲染之前将其设置为 “0”。
- 每次调用 useState 时，React 都会为你提供一个 state 对并增加索引值

##  `useState` 调用，`state`初始化

```js
function Gallery() {
  // 每次调用 useState() 都会得到新的 pair
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);
  //...
    
}    
```

在 `useState` 内部

```js
let componentHooks = [];
let currentHookIndex = 0;

function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  //...
  pair = [initialState, setState]
  //...
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}
```

依据传入的`initialState`进行初始化，在`componentHooks` 中存储`pair`，并更新`currentHookIndex`

## 使用`setState`进行 `state` 的更新

```js
let componentHooks = [];
let currentHookIndex = 0;

function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // 非首次渲染，pair已经存在
    // 将其返回并为下一次 hook 的调用做准备
    currentHookIndex++;
    return pair;
  }

  pair = [initialState, setState];
  
  function setState(nextState){
    //将新的值放入pair
    pair[0] = nexxtState;
    //UpdateDom将currentHookIndex赋值为0，保证了每一次更新的state都在initialState序号为零的位置
    UpdateDom();
  }
  //存储pair用于之后的渲染
  //并为下一次的hook调用做准备
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}
```

## `DOM`的渲染

```js
function updateDOM() {
  // 在渲染组件之前
  // 重置当前 Hook 的下标
  currentHookIndex = 0;

  // 更新 DOM 的具体代码省略
  //...

```

**如果一个组件进行两次渲染，那么每个组件副本都会有独立的 `state`**

# 渲染与提交

## 渲染

组件被 `React` 渲染之后，显示到页面中。组件的渲染包括三个步骤：

1. **触发** 一次渲染
   - 组件的 **初次渲染。**
   - 组件（或者其祖先之一）的 **状态发生了改变。**
2. **渲染** 组件
   - **在进行初次渲染时,** React 会调用根组件。
   - **对于后续的渲染,** React 会调用内部状态更新触发了渲染的函数组件。
3. **提交** 到 DOM
   - **对于初次渲染，** React 会创建的所有 DOM 节点放在屏幕上。
   - **对于重渲染，** React 将应用最少的必要操作，以使得 DOM 与最新的渲染输出相互匹配。

## 渲染与 `state`

-  设置 `state` 会触发渲染，而渲染会产生一张快照。
- 在一次渲染中，`state` 不会发生改变，`React `会使 `state` 的值始终”固定“在一次渲染的各个事件处理函数内部。
- `state` 改变是通过重新渲染实现的

因此，对于重复的`setState`：

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}

```

此次渲染的`number`都是同一个值，多次调用`setNumber` 不会使`number+3`

`setNumber` 事实上是告知了`React`:我下一次渲染的时候，这个`number`要加一。

而多次的`setNumber` ，仅仅只是告知了三次`number`要加一，而非`number`要加三

## 将多次更新加入队列

`setState(x) `实际上会像 `setState(n => x)` 一样运行

`setState`的参数实际感觉起来更像是一个回调

因此如果使用这样的方法，我们可以实现将一系列更新通过一次渲染实现：

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

`React` 是这样处理的：

1. `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。
2. `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。
3. `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。

| 更新队列     | `n`  | 返回值      |
| ------------ | ---- | ----------- |
| `n => n + 1` | `0`  | `0 + 1 = 1` |
| `n => n + 1` | `1`  | `1 + 1 = 2` |
| `n => n + 1` | `2`  | `2 + 1 = 3` |

**总结：**

我们可以将两种内容传递给 `setNumber state` ：

- **一个更新函数**（例如：`n => n + 1`）会被添加到队列中。
- **任何其他的值**（例如：数字 `5`）会导致“替换为 `5`”被添加到队列中，已经在队列中的内容会被忽略。