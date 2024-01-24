# React安装

> 使用create-react-app

```cmd
npm i -g create-react-app
#全局安装create-react-app

...
#进入创建项目的目录,略

create-react-app YOUR_PROJECT
#创建名为“YOUR_PROJECT”的项目
```

> 使用Vite创建react脚手架

```cmd
#Vite 需要 Node.js 版本 18+，20+。

npm create vite@latest
? Project name: » vite-project
#项目名称
? Select a framework: » - Use arrow-keys. Return to submit.
	...
    Vue
>   React
    Preact
    ...
#选择框架
? Select a variant: » - Use arrow-keys. Return to submit.
>   TypeScript
    TypeScript + SWC
    JavaScript
    JavaScript + SWC
#选择需要的配置，SWC是将ES6语法转换为ES5语法的工具
```

# React概览

## 组件的创建与嵌套

React 应用程序是由 **组件** 组成的。一个组件是 UI（用户界面）的一部分，它拥有自己的逻辑和外观。组件可以小到一个按钮，也可以大到整个页面。

相比于Vue，React的组件更偏向于纯粹的 `JS` ，React组件是返回标签的 `JavaScript` 函数

```jsx
import "./style.css"
//可以引入样式

function MyButton(){
    //一些其他逻辑
	return (
    	<button>I'm a button</button>
    )
}
```

像这样，一个函数组件内部包含自己的样式，逻辑，返回一个标签

React组件可以进行嵌套：

```jsx
import MyButton from "目录"

export default function MyApp(){
//注意：必须将组件暴露出去，才可以在其他文件中导入该组件
    return (
    	<div>
            <h1>My React App</h1>
            <MyButton />
        </div>
    )
}
```

React 组件必须以大写字母开头，而 HTML 标签则必须是小写字母。

## JSX

`React` 使用` JSX` 来替代常规的`JavaScript`

`JSX` 比 `HTML` 更加严格。你必须闭合标签，如 `<br />`。

组件不能返回多个 JSX 标签,必须将它们包裹到一个共享的父级中，比如 `<div>...</div>` 或使用空的 `<>...</>` 包裹：

```jsx
function ExampleComponent(){
    return (
        <>
        	<h1>Title</h1>
        	<p>Something</p>
        </>
    );
}
```

## 添加样式

在 `React` 中，为标签添加 `CSS` 的 `class` 需要使用 `className` ，其与 `HTML` 的 `class` 属性的工作方式相同

```jsx
<img className='avatar' />
```

## 显示数据

在 `JSX` 中，函数组件最终会 `return` 一个标签，而如果想在标签内部嵌入 `JS` 代码，需要使用到大括号：`{}`

```jsx
return (
	<h1>
    	{user.name}
    </h1>
);
```

在标签上同样可以使用大括号将 `JSX` 属性转移到 `JavaScript` 

```jsx
return (
	<img
    	className="avatar"
        src={user.imageUrl}
    />
)
```

只要在大括号之内，就可以转义为 `JavaScript` 代码，从而更加灵活地实现代码逻辑

**一个误区：**在某些时候 ，我们使用这种语法时会将某个对象转义为 `JavaScript` 代码

```jsx
<li
  key={product.id}
  style={{
    color: product.isFruit ? 'magenta' : 'darkgreen'
  }}
>
```

这样的语法并不是双大括号，而是在大括号内蕴含一个对象

## 条件渲染

`React` 没有特殊的语法来编写条件语句，因此使用的就是普通的 JavaScript 代码。

```jsx
let content;
if (isLoggedIn) {
  content = <AdminPanel />;
} else {
  content = <LoginForm />;
}
return (
  <div>
    {content}
  </div>
);
```

例如 `if` 语句，三目运算符，甚至利用逻辑短路

## 渲染列表

假设有一个产品数组：

```jsx
const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

可以借助 `map` 函数来渲染对应列表标签

```jsx
const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```

对于列表渲染，常常需要使用到 `key` 

对于列表中的每一个元素，都应该传递一个字符串或者数字给 `key`，用于在其兄弟节点中唯一标识该元素。

## 响应事件

可以在组件中声明**事件处理函数**来响应事件

```jsx
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

## 响应式

在 `React` 中，使用 `state` 来进行某些属性的记录与更新

```jsx
import { useState } from 'react';
//首先，从 React 引入 useState：

function MyButton() {
  const [count, setCount] = useState(0);
  //声明一个state变量
    
  function handleClick() {
    setCount(count + 1);
  }
  //事件处理函数

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}
```

我们将从 `useState` 中获得两样东西：当前的 state（`count`），以及用于更新它的函数（`setCount`）。

在这一步，给 `onClick` 传递的是一个回调函数，在合适时间会被调用

如果需要给函数传参，需要将其修改：

```jsx
<button onClick={()=>{handleClick(123)}}>
  Clicked {count} times
</button>
```

实际上，以 `use` 开头的函数被称为 **Hook**。`useState` 就是 React 提供的一个内置 Hook。